---
title: "Quadratic Approximation for a Proportion"
date: 2024-03-24T13:53:42+11:00
draft: false
math: true
tags: ['statistics', 'julia', 'bayesian-analysis']
---


This notebook demonstrates the advantage of working with a transformed parameter when making a quadratic approximation (i.e. an approximation to the probability density function by a Gaussian distribution). 
If, for example, the parameter to estimate is a proportion and therefore bound to the interval $[0,1]$, it might be difficult to fit a Gaussian, where a significant amount of probability mass might lie in the tails and outside of $[0,1]$. To overcome this we can consider a transformed parameter which is supported on the whole real line, make our approximation, find whatever summary statistic we are interested in, and then transform it back to the original scale.


```julia
begin
	using CairoMakie
	using Distributions
	using KernelDensity
	using Optim
	using Random
	using Zygote
end
```



```julia
"Visualise parameter values with confidence intervals."
function coef_table!(ax, intervals, names; grouping = nothing)
	mean(v) = sum(v) / length(v)
	function spacing(d)
		if length(d) == 1
			return [1]
		end
		steps = map(i -> .2 + (d[i] != d[i+1]), 1:(length(d) - 1))
		prepend!(steps, [1]) |> cumsum
	end
	ypos = grouping != nothing ? spacing(grouping) : 1:length(intervals) 
	rangebars!(ax, ypos, intervals, direction = :x, color = :black, 
	whiskerwidth = 10)
	scatter!(ax, map(mean, intervals), ypos, color = :black)
	vlines!(ax, [0], linestyle = :dash, color = :black, alpha = .5)
	ax.yticks = ypos
 	ax.ytickformat = _ -> names
	ax.yticklabelsize = 15
	ax.xgridvisible = false
	ax
end;
```


#### Model
We assume that the data follow a $Bernoulli(γ)$ distribution.


```julia
begin
	Random.seed!(402)
	p = 0.95                                      # True proportion.
	sample_data() = rand(Bernoulli(p), 30)        # Reusable sampler. 
end; 
```


#### 
and use the prior
$$\gamma \sim Beta(1/2, 1/2)$$


#### Analytic Posterior
Using this prior, the posterior density given $n$ trials and $s$ successes is
$$\gamma \sim Beta(1/2 + s, 1/2  + n - s)$$


```julia
begin 
	y = sample_data()
	n = length(y)
	s = sum(y)
	posterior = Beta(1/2 + s, 1/2 + n - s)
	max_a_post = mode(posterior) 
end;
```


#### Quadratic Approximation
Firstly fitted to the proportion γ directly.


```julia
begin
	γ̂ = mode(posterior)
	S = inv(-hessian(x -> logpdf(posterior, x), γ̂))
	quap = Normal(γ̂, S^.5) 
end;
```


#### Reparameterizing
If the goal is an approximation by a normal distribution, we might prefer a parameter defined on the whole real line. Let $θ = logit(γ)$, i.e. $γ = \sigma(\theta)$ where $\sigma$ is the standard logistic function. Then the prior for θ is 
$$p_{\theta}(\theta) = 
p_\gamma(\gamma) \bigg |  \frac{d\gamma}{d\theta}  \bigg | =
Beta \big (\sigma(\theta) \ | \  1/2, 1/2 \big )\sigma(\theta)(1 - \sigma(\theta))$$
and the likelihood is the standard product of Bernoulli densities.


```julia
begin
	σ(θ) = 1 / (1 + exp(-θ))
	logit(γ) = log(γ/(1 - γ))
	# Derivatives of above for change of variables.
	dσ(θ) = σ(θ) * (1 - σ(θ))
	dlogit(γ) = (1 / (γ * (1 - γ)))
end;
```


```julia
begin
	# Log pdf of the prior distribution.
	prior(θ) = logpdf(Beta(1/2, 1/2), σ(θ)) + log(dσ(θ))
	
	# Log likelihood of the data given the parameter θ
	likelihood(θ) = sum(logpdf(Bernoulli(σ(θ)), yᵢ) for yᵢ in y)
	# Ustandardised log posterior
	posterior2(θ) = prior(θ) + likelihood(θ)
	# Maximise posterior over whole real line (approximated by (-10, 10))
	res = optimize(θ -> -posterior2(θ), -10, 10)
	θ̂ = Optim.minimizer(res)
	Sθ = -inv(hessian(posterior2, θ̂))
	quap2 = Normal(θ̂, Sθ^.5)
end;
```


#### Better PDF
In the plot below it is clear that the approximation made on the logit scale is much closer to the true posterior.


```julia
begin
	fig1 = Figure()
	ax = Axis(fig1[1,1], xlabel = "γ", ylabel = "probability") 
	xlims!(ax, 0.7, 1.05)
	γ_grid = 0.7:.001:1.05
	
	lines!(ax, γ_grid, posterior, label = "analytic posterior", linewidth = 2)
	lines!(ax, γ_grid, quap, 
		label = "quadratic approximation on γ", linewidth = 2)
	lines!(ax, γ_grid, γ -> if γ < 1 pdf(quap2, logit(γ)) * dlogit(γ) else 0 end, 
		label = "quadratic approximation on θ = logit(γ)"
	) 
	
	axislegend(ax, position = :lt)
	save("images/fig1.svg", fig1)
end;
```



{{< figure
    src = "images/fig1.svg"
    alt = ":("
    width = "100%"
    >}}

#### Better Summary Statistics
In the following plot we have the maximum a posteriori (MAP) and 89% prediction interval for $\gamma$, for each version (true, transformed, untransformed).
The MAP for the transformed approximation is closer to the true value, but more importantly the 89% prediction interval is much more realistic.


```julia
begin
	# 89% confidence intervals
	α = .89
	q = [(1 - α) / 2, (1 + α) / 2]
	intervals = [quantile(posterior, q),
		quantile(quap, q),
		map(σ, quantile(quap2, q))]
	names = ["analytic", "quap γ", "quap θ"]
	fig2 = Figure()
	ax2 = Axis(fig2[1, 1], title = "89% confidence intervals")
	xlims!(ax2, .75, 1.1)
	coef_table!(ax2, intervals, names)
 	save("images/fig2.svg", fig2)
end;
```



{{< figure
    src = "images/fig2.svg"
    alt = ":("
    width = "100%"
    >}}

