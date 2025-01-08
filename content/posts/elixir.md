+++
title = 'Elixir'
date = 2024-12-07T09:23:50+11:00
draft = true
+++

Elixir is a dynamically typed and functional language.
To me it feels like Python + Haskell, while at the same time 
being very different from both of those languages.

### How is is similar to Python?
The type system  allows you to write a function like 
`def join(x, y)...` without mentioning what types `x` and `y` will have.
So long as whatever operations occur in the function body are defined for 
these types, the code will run.

### How is it similar to Haskell?
It is a functional language, i.e. one with immutable data structures.
You cannot replace the word "bad" with the word good in the list 
`["nato", "tastes", "bad"]`, all you can do is create a new list 
with this update effected.
Elixir also makes use of higher order functions, especially with the `Enum`
module for doing things to a collection of items that can be examined one by one.


### How is it different?
A somewhat unusual feature is that functions are identified by name *and arity*, 
that is, the number of arguments that they take.
So 
```elixir
def greet(first_name) do
    "Hi #{first_name}!"
end

def greet(first_name, last_name) do 
    "Hello #{first_name} {last_name}."
end
```
will run.

While Elixir does have some higher order functions, the absence of direct composition, currying and partial application have the effect of making the code
longer but simpler. I'm sure that people will have different views on this one
though.




### Euler
The [first 50 Project Euler problems](https://projecteuler.net/archives)
are my go to for playing with a new programming language. Here are some
of the more illuminating ones as regards features that highlight the 
character of Elixir.



```elixir
defmodule Euler50 do
  require Logger
  import Enum

  @moduledoc """
  Solutions to the first 50 problems.
  My goal is to write the simplest functions possible, such that each takes no more than a minute to run.
  Crazy how often reduce gets used.

  TODO: Move private helper functions to ater the main ones.
  """

  @doc """
  Sieve of Eratosthenes, used to make a list of primes up to a given number.
  """
  def primes(m) do
    # The inital map of p -> prime-ness
    initial =
      for x <- 2..m do
        {x, true}
      end
      |> Map.new()

    # Strike out all multiples of p starting from 2p.
    strike_out = fn p, primes ->
      reduce((2 * p)..m//p, primes, fn c, acc -> Map.replace(acc, c, false) end)
    end

    # Strike out multiples of primes up to sqrt(m).
    reduce(2..trunc(m ** 0.5), initial, strike_out)
    |> filter(fn {_, v} -> v end)
    |> map(&elem(&1, 0))
  end

  @doc """
  Pythagorean triplet generation by Berggren's transformations that I actually don't need for problem 39.
  """
  def berggren(x) do
    berggren1 = [[-1, 2, 2], [-2, 1, 2], [-2, 2, 3]]
    berggren2 = [[1, 2, 2], [2, 1, 2], [2, 2, 3]]
    berggren3 = [[1, -2, 2], [2, -1, 2], [2, -2, 3]]

    [berggren1, berggren2, berggren3]
    |> map(fn a -> map(a, fn row -> zip_with(row, x, &(&1 * &2)) |> sum end) end)
  end

  @doc """
  Prime factors of an number by trial division.
  """
  def prime_factors(n, d \\ 2) do
    cond do
      n < d -> []
      rem(n, d) == 0 -> [d | prime_factors(div(n, d))]
      1 -> prime_factors(n, d + 1)
    end
  end

  @doc """
  Problem 1: Multiples of 3 or 5.
  """
  def p1() do
    1..999 |> filter(fn n -> rem(n, 3) * rem(n, 5) == 0 end) |> sum()
  end

  @doc """
  Problem 2: Even Fibonacci numbers.
  """
  def p2 do
    count_even_fibs_loop(1, 1, 0)
  end

  defp count_even_fibs_loop(a, b, total) do
    cond do
      b > 4_000_000 -> total
      rem(b, 2) == 0 -> count_even_fibs_loop(b, a + b, b + total)
      true -> count_even_fibs_loop(b, a + b, total)
    end
  end

  @doc """
  Problem 3: Largest prime factor.
  """
  def p3 do
    largest_prime_factor(600_851_475_143)
  end

  def largest_prime_factor(n, d \\ 3) do
    cond do
      rem(n, d) == 0 -> largest_prime_factor(div(n, d), d)
      d * d > n -> n
      true -> largest_prime_factor(n, d + 2)
    end
  end

  @doc """
  4: Largest palindrome product.
  """
  def p4 do
    for x <- 100..999, y <- x..999 do
      Integer.to_string(x * y)
    end
    |> filter(fn s -> s == String.reverse(s) end)
    |> max_by(&Integer.parse/1)
    |> String.to_integer()
  end

  @doc """
  5: Smallest multiple.
  """
  def p5 do
    2..20
    |> map(&prime_factors/1)
    |> map(&frequencies/1)
    |> reduce(%{}, fn x, acc -> Map.merge(acc, x, fn _, v1, v2 -> Kernel.max(v1, v2) end) end)
    |> map(fn {k, v} -> k ** v end)
    |> product
  end

  @doc """
  6: Sum square difference.
  """
  def p6 do
    sum(1..100) ** 2 - sum(map(1..100, &(&1 ** 2)))
  end

  @doc """
  7: 10 001th prime.
  """
  def p7 do
    is_prime? = fn
      n -> all?(map(2..round(n ** 0.5), &(rem(n, &1) > 0)))
    end

    Stream.iterate(2, &(&1 + 1))
    |> Stream.filter(is_prime?)
    |> take(10000)
    |> List.last()
  end

  @doc """
  8: Largest product in series.
  """
  def p8 do
    File.read!("./data/p8.txt")
    |> String.replace("\n", "")
    |> String.graphemes()
    |> map(&String.to_integer/1)
    |> chunk_every(13, 1)
    |> map(&product/1)
    |> max
  end

  @doc """
  9: Special Pythagorean triplet.
  """
  def p9 do
    {a, b, c} =
      for a <- 2..1000, b <- (a + 1)..1000 do
        {a, b, (a ** 2 + b ** 2) ** 0.5}
      end
      |> filter(fn {a, b, c} -> a + b + c == 1000 and abs(round(c) - c) < 10 ** -5 end)
      |> hd

    (a * b * c) |> round
  end

  @doc """
  10: Summation of primes.
  """
  def p10 do
    primes(2 * 10 ** 6) |> sum
  end

  @doc """
  11: Largest product in a grid.
  """
  def p11 do
    matrix = p11_load_matrix()

    [
      p11_max_horizontal_product(matrix),
      matrix |> p11_transpose |> p11_max_horizontal_product,
      matrix |> p11_all_diagonals |> p11_max_horizontal_product,
      matrix |> map(&reverse(&1)) |> p11_all_diagonals |> p11_max_horizontal_product
    ]
    |> max
  end

  defp p11_transpose(rows) do
    0..(length(rows) - 1) |> map(fn i -> map(rows, fn row -> at(row, i) end) end)
  end

  defp p11_diagonal(rows, i) do
    i..0//-1 |> map(fn j -> at(at(rows, j), i - j) end)
  end

  defp p11_all_diagonals(rows) do
    0..(length(rows) - 1) |> map(&p11_diagonal(rows, &1))
  end

  defp p11_max_horizontal_product(rows) do
    for row <- map(rows, &chunk_every(&1, 4, 1)), chunk <- row do
      product(chunk)
    end
    |> max
  end

  defp p11_load_matrix() do
    File.read!("./data/p11.txt")
    |> String.split("\n")
    |> map(&String.split(&1))
    |> map(fn line -> map(line, &String.to_integer/1) end)
  end

  @doc """
  12: Highly divisble triangular number.
  """
  def p12 do
    triangles =
      Stream.unfold({1, 1}, fn
        {n, t} -> {{n, t}, {n + 1, t + n + 1}}
      end)
      |> Stream.map(fn {_, t} -> t end)

    totient = fn
      n ->
        prime_factors(n)
        |> frequencies
        |> map(fn {_, v} -> v + 1 end)
        |> product
    end

    triangles |> Stream.filter(fn n -> totient.(n) > 500 end) |> Stream.take(1) |> to_list |> hd
  end

  @doc """
  13: Large sum
  """
  def p13 do
    File.read!("./data/p13.txt")
    |> String.split("\n")
    |> map(fn line -> map(reverse(String.graphemes(line)), &String.to_integer/1) end)
    |> p13_transpose
    |> p13_add_cols(0)
    |> reverse
    |> map(&Integer.to_string(&1))
    |> join
    |> String.slice(0..9)
    |> String.to_integer()
  end

  defp p13_transpose(rows) do
    0..(length(hd(rows)) - 1) |> map(fn i -> map(rows, fn row -> at(row, i) end) end)
  end

  defp p13_add_cols([], carry) do
    [carry]
  end

  defp p13_add_cols([column | tail], carry) do
    s = sum(column) + carry
    [rem(s, 10) | p13_add_cols(tail, div(s, 10))]
  end

  # Get the length of the Collatz sequence starting from n.
  defp p14_get(n, collatz) do
    if Map.has_key?(collatz, n) do
      {collatz, n}
    else
      m =
        if rem(n, 2) == 0 do
          div(n, 2)
        else
          3 * n + 1
        end

      {foo, x} = p14_get(m, collatz)
      new_collatz = Map.put(foo, n, 1 + Map.fetch!(foo, x))
      {new_collatz, n}
    end
  end

  @doc """
  14: Longest Collatz Sequence
  """
  def p14 do
    collatz =
      reduce(1..(10 ** 6), %{1 => 1}, fn
        x, acc ->
          {acc2, _} = p14_get(x, acc)
          acc2
      end)

    max_by(1..(10 ** 6), fn n -> Map.fetch!(collatz, n) end)
  end

  @doc """
  15: Lattice paths.
  """
  def p15 do
    initial_grid =
      Map.merge(
        for y <- 0..20 do
          {{0, y}, 1}
        end
        |> Map.new(),
        for x <- 0..20 do
          {{x, 0}, 1}
        end
        |> Map.new()
      )

    count_paths_to = fn
      {x, y}, grid -> Map.fetch!(grid, {x - 1, y}) + Map.fetch!(grid, {x, y - 1})
    end

    grid =
      reduce(
        for x <- 1..20, y <- 1..20 do
          {x, y}
        end,
        initial_grid,
        fn z, acc ->
          Map.put_new(acc, z, count_paths_to.(z, acc))
        end
      )

    grid |> Map.fetch!({20, 20})
  end

  @doc """
  16: Power digit sum
  """
  def p16 do
    (2 ** 1000) |> to_string |> String.graphemes() |> map(&String.to_integer/1) |> sum
  end

  def words(1000) do
    "one thousand"
  end

  @doc """
  17: Number letter counts.
  """
  def words(n) when n < 20 do
    %{
      0 => "zero",
      1 => "one",
      2 => "two",
      3 => "three",
      4 => "four",
      5 => "five",
      6 => "six",
      7 => "seven",
      8 => "eight",
      9 => "nine",
      10 => "ten",
      11 => "eleven",
      12 => "twelve",
      13 => "thirteen",
      14 => "fourteen",
      15 => "fifteen",
      16 => "sixteen",
      17 => "seventeen",
      18 => "eighteen",
      19 => "nineteen"
    }
    |> Map.fetch!(n)
  end

  def words(n) when rem(n, 10) == 0 and n < 100 do
    %{
      20 => "twenty",
      30 => "thirty",
      40 => "forty",
      50 => "fifty",
      60 => "sixty",
      70 => "seventy",
      80 => "eighty",
      90 => "ninety"
    }
    |> Map.fetch!(n)
  end

  def words(n) when n < 100 do
    words(div(n, 10) * 10) <> "-" <> words(rem(n, 10))
  end

  def words(n) when rem(n, 100) == 0 and n < 1000 do
    words(div(n, 100)) <> " hundred"
  end

  def words(n) do
    words(div(n, 100) * 100) <> " and " <> words(rem(n, 100))
  end

  def p17 do
    1..1000
    |> map(fn
      n ->
        words(n)
        |> String.replace(" ", "")
        |> String.replace("-", "")
        |> String.length()
    end)
    |> sum
  end

  @doc """
  18: Maximum path sum 1
  """
  def p18 do
    triangle =
      File.read!("./data/p18.txt")
      |> String.split("\n")
      |> map(fn line -> map(String.split(line), &String.to_integer/1) end)

    paths_by_indicies =
      reduce(
        1..14,
        [[0]],
        fn _, acc ->
          flat_map(acc, fn [x | xs] -> [[x] ++ [x | xs], [x + 1] ++ [x | xs]] end)
        end
      )
      |> map(&reverse/1)

    # Convert a path described by indicies to the actual elements in the triangle.
    map(
      paths_by_indicies,
      fn path -> zip_with(triangle, path, fn row, i -> fetch!(row, i) end) end
    )
    |> map(&sum/1)
    |> max
  end

  @doc """
  19: Counting Sundays
  """
  def p19 do
    twentieth_century = Date.range(~D[1901-01-01], ~D[2000-12-31])

    twentieth_century
    |> filter(fn date -> Date.day_of_week(date) == 7 and date == Date.beginning_of_month(date) end)
    |> length
  end

  @doc """
  20: Factorial digit sum
  """
  def p20 do
    reduce(1..100, 1, &*/2) |> to_string |> String.graphemes() |> map(&String.to_integer/1) |> sum
  end

  @doc """
  21: Amicable numbers.
  """
  def p21 do
    sum_of_divisors = fn n ->
      (2..round(n ** 0.5)
       |> filter(&(rem(n, &1) == 0))
       |> flat_map(fn d -> [d, div(n, d)] end)
       |> uniq
       |> sum) + 1
    end

    amicable = fn n ->
      m = sum_of_divisors.(n)

      if m > 10000 or n == m do
        false
      else
        sum_of_divisors.(m) == n
      end
    end

    2..10000 |> filter(&amicable.(&1)) |> sum
  end

  @doc """
  22: Names Scores
  """
  def p22 do
    name_values =
      File.read!("./data/p22.txt")
      |> String.replace(~s("), "")
      |> String.split(",")
      |> sort
      |> map(fn name -> name |> String.to_charlist() |> map(fn c -> c - 64 end) end)
      |> map(&sum/1)

    zip_with(1..length(name_values), name_values, &*/2) |> sum
  end

  @doc """
  23: Abundant Numbers.
  """
  def p23 do
    sum_of_divisors = fn n ->
      (2..round(n ** 0.5)
       |> filter(&(rem(n, &1) == 0))
       |> flat_map(fn d -> [d, div(n, d)] end)
       |> uniq
       |> sum) + 1
    end

    abundants = 3..28123 |> filter(fn n -> sum_of_divisors.(n) > n end)

    sum_of_two_abundants =
      for x <- abundants, y <- abundants, x <= y do
        x + y
      end

    MapSet.difference(
      MapSet.new(1..28123),
      MapSet.new(sum_of_two_abundants)
    )
    |> sum
  end

  @doc """
  24: Lexicographic permutations
  """
  def p24 do
    reduce(2..(10 ** 6), 0..9 |> to_list, fn _, acc -> p24_next_permutation(acc) end)
    |> map(&Integer.to_string/1)
    |> join
  end

  defp p24_next_permutation(s) do
    k = 8..0//-1 |> find(fn k -> at(s, k) < at(s, k + 1) end)
    l = 9..(k + 1)//-1 |> find(fn l -> at(s, k) < at(s, l) end)
    sk = at(s, k)
    sl = at(s, l)
    s2 = s |> List.update_at(k, fn _ -> sl end) |> List.update_at(l, fn _ -> sk end)
    start = s2 |> take(k + 1)
    rest = s2 |> drop(k + 1)
    start ++ reverse(rest)
  end

  @doc """
  25: 1000-digit Fibonacci Number
  """
  def p25 do
    p25_fib(1, 1, 2)
  end

  defp p25_fib(a, b, n) do
    if b |> to_string |> String.graphemes() |> length == 1000 do
      n
    else
      p25_fib(b, a + b, n + 1)
    end
  end

  # Slightly dodgy algorithm since it will give the prefix as well as the cyclic part, e.g
  # 6 goes to [1, 6].
  defp p26_cycle(d, r, seen_before, res) do
    cond do
      r == 0 -> []
      member?(seen_before, r) -> res
      d > r -> p26_cycle(d, 10 * r, seen_before, res)
      true -> p26_cycle(d, rem(r, d), [r | seen_before], [div(r, d) | res])
    end
  end

  @doc """
  26: Reciprocal cycles
  """
  def p26 do
    2..1000 |> max_by(fn n -> length(p26_cycle(n, 1, [], [])) end)
  end

  @doc """
  27: Quadratic Primes.
  """
  def p27 do
    # A pretend infinite list.
    primes = primes(10 ** 6) |> MapSet.new()

    count_prime_run = fn
      {a, b} ->
        Stream.iterate(1, &(&1 + 1))
        |> Stream.map(fn n -> n ** 2 + a * n + b end)
        |> Stream.take_while(&MapSet.member?(primes, &1))
        |> to_list
        |> length
    end

    {a, b} =
      for a <- -1000..1000, b <- -1000..1000 do
        {a, b}
      end
      |> max_by(count_prime_run)

    a * b
  end

  @doc """
  Problem 28: Number Spiral Diagonals
  """
  def p28 do
    corner_sum = fn k ->
      n = 2 * k + 1
      4 * n ** 2 - 6 * n + 6
    end

    (1..500 |> map(corner_sum) |> sum) + 1
  end

  @doc """
  Problem Distinct Powers
  """
  def p29 do
    for a <- 2..100, b <- 2..100 do
      a ** b
    end
    |> uniq
    |> length
  end

  @doc """
  Problem 30: Digit Fifth Powers
  """
  def p30 do
    digit_power_sum = fn
      n ->
        n
        |> to_string
        |> String.graphemes()
        |> map(&String.to_integer/1)
        |> map(&(&1 ** 5))
        |> sum
    end

    # Set an arbitrary upper bound of 1_000_000.
    2..1_000_000 |> filter(fn n -> digit_power_sum.(n) == n end) |> sum
  end

  @doc """
  Problem 31: Coin Sums
  """
  def p31() do
    p31_change(200, [1, 2, 5, 10, 20, 50, 100, 200])
  end

  defp p31_change(0, _) do
    1
  end

  defp p31_change(n, _) when n < 0 do
    0
  end

  defp p31_change(_, []) do
    0
  end

  defp p31_change(n, coins = [coin | rest]) do
    p31_change(n - coin, coins) + p31_change(n, rest)
  end

  @doc """
  Problem 32: Pandigital Products
  """
  def p32() do
    # Guess an upper bound for each factor.
    m = 2000

    for a <- 1..m, b <- (a + 1)..m do
      {a * b,
       [a, b, a * b]
       |> map(fn n -> to_string(n) end)
       |> join}
    end
    |> filter(fn {_, b} -> sort(to_charlist(b)) == ~c"123456789" end)
    |> map(fn {n, _} -> n end)
    |> uniq
    |> sum
  end

  @doc """
  Problem 33: Digit Cancelling Fractions
  """
  def p33 do
    frac2 =
      for a <- 10..99, b <- 10..99, a < b do
        {a, b}
      end

    compare = fn {a, b}, {c, d} -> a * d == b * c end

    # Cancel the leading digits.
    cancel11 = fn {a, b} ->
      compare.({rem(a, 10), rem(b, 10)}, {a, b}) and div(a, 10) == div(b, 10)
    end

    # Keep the leading digits.
    cancel22 = fn {a, b} ->
      compare.({div(a, 10), div(b, 10)}, {a, b}) and rem(a, 10) == rem(b, 10) and
        not (rem(a, 10) * rem(b, 10) == 0)
    end

    # Keep the leading digit of the numerator and the second digit of the denominator.
    cancel12 = fn {a, b} ->
      compare.({div(a, 10), rem(b, 10)}, {a, b}) and rem(a, 10) == div(b, 10)
    end

    # Keep the second digit of the numerator and the leading digit of the denominator.
    cancel21 = fn {a, b} ->
      compare.({rem(a, 10), div(b, 10)}, {a, b}) and div(a, 10) == rem(b, 10)
    end

    {a, b} =
      reduce(
        frac2
        |> filter(fn {a, b} ->
          [cancel11, cancel22, cancel21, cancel12] |> map(fn f -> f.({a, b}) end) |> any?
        end),
        {1, 1},
        fn {a, b}, {c, d} -> {a * c, b * d} end
      )

    # Notice that the numerator cleanly divides the denominator, so no need to fuss with simplifying fractions.
    div(b, a)
  end

  def factorial(n) when n < 2 do
    1
  end

  def factorial(n) do
    reduce(2..n, 1, &(&1 * &2))
  end

  @doc """
  Problem 34: Digit Factorials
  """
  def p34 do
    3..100_000
    |> filter(fn n ->
      n
      |> Integer.to_string()
      |> String.graphemes()
      |> map(&String.to_integer/1)
      |> map(&factorial/1)
      |> sum == n
    end)
    |> sum
  end

  @doc """
  Problem 35: Circular Primes
  """
  def p35 do
    primes =
      primes(10 ** 6)
      |> map(&Integer.to_string/1)
      |> filter(fn p -> uniq(String.graphemes(p)) -- ["1", "3", "7", "9"] == [] end)
      |> MapSet.new()

    rotations = fn s ->
      l = String.length(s) - 1

      0..l
      |> map(fn i -> [String.slice(s, i, l + 1), String.slice(s, 0, i)] end)
      |> map(&join/1)
    end

    circular = fn p ->
      rotations.(p) |> all?(fn rotation -> member?(primes, rotation) end)
    end

    # Add 2 for circular primes 2 and 5.
    (primes |> filter(circular) |> length) + 2
  end

  @doc """
  Problem 36: Double-base Palindromes
  """
  def p36 do
    1..1_000_000
    |> map(&Integer.to_string/1)
    |> filter(fn s -> s == String.reverse(s) end)
    |> map(fn s -> Integer.to_string(String.to_integer(s), 2) end)
    |> filter(fn s -> s == String.reverse(s) end)
    |> map(&String.to_integer(&1, 2))
    |> sum
  end

  @doc """
  Problem 37: Truncatable Primes
  """
  def p37 do
    primes =
      primes(750_000)
      |> map(&Integer.to_string/1)
      # |> filter(fn p -> uniq(String.graphemes(p)) -- ["1", "3", "7", "9"] == [] end)
      |> MapSet.new()

    truncations = fn s ->
      l = String.length(s) - 1
      [s] ++ (1..l |> flat_map(fn i -> [String.slice(s, 0, i), String.slice(s, i, l)] end))
    end

    primes
    |> filter(fn p -> all?(truncations.(p), &member?(primes, &1)) end)
    |> map(&String.to_integer/1)
    |> sum
  end

  defp p38_helper(start, n, acc) do
    s = acc |> map(&Integer.to_string/1) |> join

    if String.length(s) < 9 do
      p38_helper(start, n + 1, [start * n | acc])
    else
      acc
    end
  end

  def p38_multiples(start) do
    p38_helper(start, 1, [])
  end

  @doc """
  Problem 38: Pandigital Multiples
  """
  def p38 do
    9..10000
    |> filter(fn n ->
      p38_multiples(n) |> map(&Integer.to_string/1) |> join |> String.graphemes() |> sort |> join ==
        "123456789"
    end)
    |> map(fn n ->
      p38_multiples(n) |> reverse |> map(&Integer.to_string/1) |> join
    end)
    |> sort
    |> reverse
    |> hd
    |> String.to_integer()
  end

  @doc """
  Problem 39: Integer Right Triangles
  """
  def p39() do
    triples =
      for a <- 1..999, b <- (a + 1)..999, a + b do
        c2 = a ** 2 + b ** 2
        c = round(c2 ** 0.5)

        if abs(c ** 2 - c2) < 10 ** -10 and a + b + c <= 1000 do
          {a, b, c}
        end
      end
      |> filter(fn x -> x end)

    {perimeter, _} =
      reduce(
        triples,
        %{},
        fn {a, b, c}, acc -> acc |> Map.update(a + b + c, 1, fn count -> count + 1 end) end
      )
      |> max_by(fn {_, count} -> count end)

    perimeter
  end

  defp p40_champernowne(i, s, _, l) when i < l do
    s
  end

  defp p40_champernowne(i, s, n, l) do
    s_new = Integer.to_string(n)
    p40_champernowne(i, s <> s_new, n + 1, l + String.length(s_new))
  end

  @doc """
  Problem 40: Champernowne's Constant
  """
  def p40 do
    s = "." <> p40_champernowne(10 ** 6, "", 1, 0)

    [1, 10, 100, 1000, 10_000, 100_000, 1_000_000]
    |> map(&String.at(s, &1))
    |> map(&String.to_integer/1)
    |> product()
  end

  def p41_perms_from(p) do
    if p == sort(p) |> reverse do
      []
    else
      k = (length(p) - 1)..0//-1 |> find(fn k -> at(p, k) < at(p, k + 1) end)
      l = length(p)..(k + 1)//-1 |> find(fn l -> at(p, k) < at(p, l) end)
      pk = at(p, k)
      pl = at(p, l)
      p2 = p |> List.update_at(k, fn _ -> pl end) |> List.update_at(l, fn _ -> pk end)
      start = p2 |> take(k + 1)
      rest = p2 |> drop(k + 1)
      p_new = start ++ reverse(rest)
      [p | p41_perms_from(p_new)]
    end
  end

  @doc """
  Problem 41: Pandigital Primes
  """
  def p41 do
    prime? = fn
      n -> all?(map(3..round(n ** 0.5)//2, &(rem(n, &1) > 0)))
    end

    f = fn n ->
      p41_perms_from("123456789" |> String.slice(0, n) |> String.graphemes())
      # |> map(&join/1)
      |> filter(fn p -> member?(["1", "3", "7", "9"], hd(reverse(p))) end)
      |> map(&join/1)
      |> map(&String.to_integer/1)
      |> reverse
      |> find(prime?)
    end

    # We cheat, knowing that the answer has 7 digits. How to you check all 8 and 9 digit
    # values in under a minute? Or how do you know there won't be one?
    f.(7)
  end

  @doc """
  Problem 42: Coded Triangle Numbers
  """
  def p42 do
    encode = fn s -> String.to_charlist(s) |> map(fn c -> c - 64 end) |> sum end

    encoded_words =
      File.read!("./data/p42.txt")
      |> String.replace(~s("), "")
      |> String.split(",")
      |> map(encode)

    # Integers 1, 2, ...
    triangles =
      Stream.unfold(1, fn n -> {n, n + 1} end)
      # Triangle numbers 1, 3, ...
      |> Stream.map(fn n -> div(n * (n + 1), 2) end)
      # Only the ones we need
      |> take_while(fn n -> n < max(encoded_words) end)
      |> MapSet.new()

    encoded_words |> filter(&member?(triangles, &1)) |> length
  end

  defp p43_permutations do
    Stream.unfold(
      "0123456789",
      fn
        "9876543210" -> nil
        p -> {p, p43_next_perm(p)}
      end
    )
  end

  defp p43_next_perm(s) do
    p = String.graphemes(s)
    k = (length(p) - 1)..0//-1 |> find(fn k -> at(p, k) < at(p, k + 1) end)
    l = length(p)..(k + 1)//-1 |> find(fn l -> at(p, k) < at(p, l) end)
    pk = at(p, k)
    pl = at(p, l)
    p2 = p |> List.update_at(k, fn _ -> pl end) |> List.update_at(l, fn _ -> pk end)
    start = p2 |> take(k + 1)
    rest = p2 |> drop(k + 1)
    (start ++ reverse(rest)) |> join
  end

  @doc """
  Problem 43: Sub-string Divisibility
  """
  def p43() do
    # The substring divisibility property
    special? = fn p ->
      # f = fn {i, m} -> rem(String.slice(p, i, 3) |> String.to_integer(), m) == 0 end

      f = fn {i, m} -> rem(String.slice(p, i, 3) |> String.to_integer(), m) == 0 end
      zip(1..7, [2, 3, 5, 7, 11, 13, 17]) |> map(f) |> all?
    end

    p43_permutations() |> Stream.filter(special?) |> map(&String.to_integer/1) |> sum
  end

  @doc """
  Problem 44: Pentagonal Numbers
  """
  def p44 do
    pentagonals =
      Stream.unfold(1, fn n -> {n, n + 1} end)
      |> Stream.map(fn n -> div(n * (3 * n - 1), 2) end)
      |> take(3000)
      |> MapSet.new()

    {a, b} =
      for a <- pentagonals, b <- pentagonals, a < b do
        {a, b}
      end
      |> find(fn {a, b} -> member?(pentagonals, a + b) and member?(pentagonals, b - a) end)

    b - a
  end

  @doc """
  Problem 45: Triangular, Pentagonal, and Hexagonal
  """
  def p45 do
    triangulars =
      Stream.unfold(1, fn n -> {n, n + 1} end) |> Stream.map(fn n -> div(n * (n + 1), 2) end)

    int_sqrt = fn n -> round(n ** 0.5) end
    square? = fn n -> abs(int_sqrt.(n) ** 2 - n) < 10 ** -10 end
    pentagonal? = fn n -> square?.(1 + 24 * n) and rem(1 + int_sqrt.(1 + 24 * n), 6) == 0 end
    hexagonal? = fn n -> square?.(1 + 8 * n) and rem(1 + int_sqrt.(1 + 8 * n), 4) == 0 end

    triangulars
    |> Stream.drop(285)
    |> Stream.filter(fn n -> pentagonal?.(n) and hexagonal?.(n) end)
    |> take(1)
    |> hd
  end

  @doc """
  Problem 46: Goldbach's Other Conjecture
  """
  def p46 do
    primes = primes(10000) |> MapSet.new()
    squares = 1..1000 |> map(fn n -> n ** 2 end)

    goldbach =
      for p <- primes, s <- squares do
        p + 2 * s
      end
      |> filter(&(rem(&1, 2) == 1))
      |> MapSet.new()

    3..10000//2 |> find(fn n -> not (MapSet.member?(primes, n) or member?(goldbach, n)) end)
  end

  def p47_prime_fac(n, d \\ 2) do
    cond do
      n < d -> []
      rem(n, d) == 0 -> [d | p47_prime_fac(div(n, d))]
      1 -> p47_prime_fac(n, d + 1)
    end
  end

  @doc """
  Problem 47: Distinct Primes Factors
  """
  def p47 do
    count_distinct_prime_factors = fn n -> p47_prime_fac(n) |> uniq |> length end

    chunk_fun = fn element, acc ->
      cond do
        length(acc) == 4 -> {:cont, hd(reverse(acc)), []}
        count_distinct_prime_factors.(element) == 4 -> {:cont, [element | acc]}
        true -> {:cont, []}
      end
    end

    after_fun = fn
      [] -> {:cont, []}
      acc -> {:cont, acc, []}
    end

    Stream.unfold(1, fn n -> {n, n + 1} end)
    |> Stream.chunk_while([], chunk_fun, after_fun)
    |> take(1)
    |> hd
  end

  defp pow10(n, 1) do
    n
  end

  defp pow10(n, k) do
    rem(n * pow10(n, k - 1), 10 ** 10)
  end

  @doc """
  Problem 48: Self Powers
  """
  def p48 do
    pow10(7, 7)
    1..1000 |> map(&pow10(&1, &1)) |> sum |> rem(10 ** 10)
  end

  @doc """
  Problem 49: Prime Permutations
  """
  def p49 do
    find_geometric_sequence = fn xs ->
      for a <- xs, b <- xs, a < b do
        d = b - a
        Stream.iterate(a, &(&1 + d)) |> Stream.take_while(&member?(xs, &1)) |> to_list
      end
    end

    primes(9999)
    |> filter(&(&1 >= 1000))
    |> group_by(fn n -> Integer.to_string(n) |> String.graphemes() |> sort end)
    |> Map.values()
    |> filter(&(length(&1) >= 3))
    |> flat_map(find_geometric_sequence)
    |> filter(&(length(&1) > 2))
    # The first one is the one we are looking for.
    |> hd
    |> map(&Integer.to_string/1)
    |> join
    |> String.to_integer()
  end

  defp tails([]) do
    []
  end

  defp tails(x) do
    [x | tails(tl(x))]
  end

  @doc """
  Problem 50: Consecutive Prime Sum
  """
  def p50 do
    m = 10 ** 6
    primes_set = primes(m)
    primes = primes_set |> sort

    {p, _} =
      tails(primes)
      |> Stream.map(&scan(&1, {0, 0}, fn p, {total, count} -> {total + p, count + 1} end))
      |> Stream.flat_map(&take_while(&1, fn {total, _} -> total < m end))
      |> Stream.filter(fn {total, _} -> member?(primes_set, total) end)
      |> max_by(fn {_, count} -> count end)

    p
  end
end
```