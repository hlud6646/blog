+++
title = 'Rebuilding Grep'
date = 2024-08-02T13:20:16+10:00
draft = false
tags = ['haskell', 'software', 'AST']
+++

*Source code: [https://github.com/hlud6646/grep-haskell](https://github.com/hlud6646/grep-haskell)*

Grep seems like such an innocent tool, but it does a lot.
Rebuilding it was a hard project for me and I am proud of
what I have learned along the way. This post is a summary of those
things.
From a bird's eye perspective:

- Developing an abstract syntax tree for regular expressions;
- Parsing a provided expression into the abstract representation;
- Comparing an input to a particular expression.


## 1. An AST for Regular Expressions.
To start off, you need to define a data type with constructors which correspond
to elements of an expression.
 For example you need to represent the special 
character `\w` which means 'any alphanumeric character' and so on.
In Haskell this is a breeze:

```haskell
data Pattern
  = AlphaNumeric
  | Digit
  | Literal Char
  ...
```
One of the highlights of the language is that it supports recursive
types, so that the definition of an alternative (the abstract
representation of an expression like "(cat|dog)") is 
```haskell
  ...
  | Alternative Pattern Pattern
```

The other regex constructs have their own constructors, and we have the 
following data type:

```haskell
data Pattern
  = Empty
  | Literal Char
  | Digit
  | AlphaNum
  | Wildcard
  | PosCharClass String
  | NegCharClass String
  | StartAnchor
  | EndAnchor
  | Repeat Quantifier Pattern
  | Alternative Pattern Pattern
  | Capture Pattern
  | BackRef Int
  | Seq [Pattern]

data Quantifier = ZeroOrOne | OneOrMore
```
This is not absolutely everything that grep can do but it is
a lot.

## 2: Parsing
In Brent Yorgey's fantastic [intro to Haskell course](https://www.cis.upenn.edu/~cis1940/spring13/)
you eventually build an applicative parser from scratch.
It's a good exercise and a nice intro to parsing with one 
of the actual libraries. Here we use `megaparsec` which
, by virtue of being so powerful, is also a bit intimidating
at first. I hope to write another post on this because I found 
that there wasn't actually a good hello world for the library
anywhere. Anyway you start by importing and defining a convenience

```haskell
import Text.Megaparsec
import Text.Megaparsec.Char
import Data.Void

type Parser = Parsec Void String
```
which means that you will not be doing anything in particular with 
the error messages and that you intend to consume `String` data.

You can define parsers using the applicative syntax like 
```haskell
regularChar :: Char -> Bool
regularChar c = c `notElem` "$^+?[]|()."

pLiteral :: Parser Pattern
pLiteral = Literal <$> satisfy regularChar <?> "non special character."

pDigit :: Parser Pattern
pDigit = Digit <$ chunk "\\d"

pAlphaNum :: Parser Pattern
pAlphaNum = AlphaNum <$ chunk "\\w"
```
which might look a bit magical but is really nice to use 
once you get the hang of it.

You can also define more complicated parsers with monadic 
combinators like 

```haskell

pPosCharClass :: Parser Pattern
pPosCharClass = do
  void (char '[')
  chars <- some (satisfy regularChar)
  void (char ']')
  return $ PosCharClass chars

pNegCharClass :: Parser Pattern
pNegCharClass = do
  void (chunk "[^")
  chars <- some (satisfy regularChar)
  void (char ']')
  return $ NegCharClass chars
```

## 3. Consuming Input
The last piece is matching the parsed regex against the input text.
This is achieved by one gigantic function called `consume` that essentially
takes a pattern and an input, and optionally returns the rest (unmatched part) 
of the input. So `consume Wildcard "Hello"` should return `Just "ello"` and
`consume Digit "Hi"` should return `Nothing`. Note that this does *not* scan 
a whole line of input and try and match the regex wherever it may occur. It only
looks for matches from the start of the input. Matching anywhere in the input
is delegated to a later function.

To allow back references, this function has unfortunately to become a bit
more complicated. We need to keep a record of captures (things in parentheses
like 'Hugo' in `Hello (Hugo). I just greeted \1`) so that a back reference
 (`\1`) can be substituted for that capture.

The full function signature for `consume` is

```haskell
consume ::
  Pattern -> -- A parsed regular expression to match against.
  String -> -- The input to compare the expression to.
  [String] -> -- Accumulation of previous captures.
  Maybe (
    String, -- The part of the input consumed by this pattern.
    String, -- The rest (unconsumed part) of the input.
    [String] -- Accumulation of captures.
  )
```

I do have an uncomfortable feeling about long function signatures like this in Haskell, 
but I don't know if a type alias for the return value would make it better or worse.
The function takes (as before) a pattern and an input, but also a list of previous 
captures.
It returns not only the unmatched part of the string, but also the part just matched, 
and the accumulation of captures that was passed in, augmented by the just matched part 
if the pattern was a capture. Of course it just returns `Nothing` if the match fails.


The bulk of the work is implementing this function for all possible values of the 
`Pattern` type which we defined above.

For example
```haskell
-- Matching the empty pattern always works.
consume Empty input captures = Just ("", input, captures)
-- Matching a wildcard on the empty string fails.
consume Wildcard "" _ = Nothing
-- Matching a wildcard on a character.
consume Wildcard (s : sx) captures = Just ([s], sx, captures)
...
```
and so on. To search for a match anywhere in the input of a pattern given also 
as a string, we have the function 
```haskell
regexMatch :: String -> String -> Bool
regexMatch pattern input = isJust $ do
  case parse pRegex "<stdin>" pattern of
    Left bundle -> Nothing
    -- If the pattern begins with a start anchor, then you must match from the stat of the string.
    Right (Seq (StartAnchor : patterns)) -> consume (Seq patterns) input []
    -- Otherwise, you can try to match the pattern starting anywhere.
    Right result -> join $ find isJust (map (\i -> consume result i []) (tails input))
```

There is a bit of type juggling going on, since the `parse` function from `Megaparsec`
returns an `Either`, our `consume` function returns a `Maybe` and this `regexMatch`
function should return a `Bool`.
The solution is to catch a `Left` from `parse` (a failure) and send back a `Nothing` to 
align with the output of `consume`. Then call `isJust` on the whole thing to get a 
`Bool`.
One other thing that is happening here is that a pattern should be allowed to match 
*anywhere* inside the input, not just at the start (which is what the `consume` function
looks for). So if the pattern starts with a `StartAnchor`, just pass to `consume` without
any change. But if it doesn't, you need to check the pattern against any tail of the 
input. So to match '\d' against 'Hi12' you need to check against 'Hi12', 'i12', '12',
'2' and ''.

And that's it! Surprisingly challenging, but a working regular expression evaluator.
