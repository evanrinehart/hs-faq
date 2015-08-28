# FAQ

Haskell is very different from standard programming languages currently in use.
While there is a huge universe of new ideas worth exploring in Haskell, it can
be daunting to carry out tasks which you might consider "simple" in a language
like PHP. Here is a grocery list of tips and rules of thumb for how to
accomplish these things in Haskell. Sometimes the answer is a straightforward
translation to a feature in Haskell that does the same thing, but might seem
obscure. On the other hand there might be a better way to think about your
problem in the context of purity, static typing, and or laziness.

## How do I convert a number to a string?

- For basic decimal rendering of an integer just use `show`.
- For careful formating of a float there are a variety of utility functions
in the Numeric module. You can also use the `printf` from Text.Printf.
- If you want Text instead of a String then there is Data.Text.Format.

## How do I convert a string to a number?

- The standard function `read` can convert a string to a type that implements
the Read class. However this should be considered quick and dirty because a
badly formatted string will cause a crash.
- The module Text.Read contains a function `readMaybe` which will return Nothing
instead of crashing and is a better idea for production code.
- For parsing numbers contained within other textual data, the parsing library
you use will have appropriate combinators for various numeric formats.

## How do I convert one type of number to another?

- To go from an integral type to another type use `fromIntegral`.
- To go from a fractional type to an integral type use one of `floor` `ceiling`
`truncate` or `round`.
- To go from a fractional type to another fractional type use `realToFrac`.

## How do I catch exceptions or throw exceptions?

- Depending on the kind of catching behavior you want, you need to use
`finally`, `bracket`, `try`, `catch`, or something else from Control.Exception.
- To throw an exception in IO code use `throwIO`.
- It is possible to throw exceptions from pure code but it's a better
idea and is better style to use Maybe, Either, or an error monad in this case.
- Don't try to catch exceptions caused by evaluating `undefined` or `error`.
This is because `undefined` and `error` should only be used in cases that are
impossible or where recovery is inappropriate. Pure code that needs to signal
a recoverable failure should use Maybe, Either, or an error monad.
- It is worth reading Control.Exception to understand all the implications of
exception safety. In particular GHC supports throwing exceptions into another
running thread.

## How do I write a foreach loop?

- In the case that you want to create a new list from an existing list by
applying a function at each position, use `map`. This pattern has been generalized
to any data structure that implements the Functor class whose equivalent is
called `fmap`.
- In some cases you want to `fmap` something that is not a functor for some (good)
reason. In these cases a special `map` function is typically provided for that
structure.
- In the case that you want to iterate through a list and execute some effects
for each elements, then use `forM_`. Note that this doesn't let you "break".
- In case you want to `forM_` but stop if some condition is satisfied, then
use `whileM_` from the package `monad-loops`.
- In case you want to `forM_` but accumulate a state value as you traverse the
list then use `foldM`.
- `forM` and `forM_` have been generalized to work with anything that is
Traversable and Applicative. In this case use `for` and `for_`.
- A sufficiently complex looping strategy might be too obscure or even impossible
to implement with the combinators above. In this case you can directly write the
loop behavior recursively or factor out the looping strategy into it's own
recursive function.

## How do I write an if statement?

- Sometimes you want to only execute some effect in a sequence if some condition
is true. For this use `when` or `unless` from Control.Monad.
- In most other cases you want to make decisions based on pattern matching and
guards. However you can explicitly map a Bool to one of two cases using the
`if _ then _ else _` expression or the `bool` function from Data.Bool.

## How do I return from a procedure that is using do-notation "early"?

- Note that the oddly-named function `return` does not do this.
- The conventional way is to restructure your do block so that the early
exit point is at the top level. This is analogous to never having code that
follows an if-then-else in an imperative language code block. If you find
that you are doing a lot of early exiting by case matching on Nothing then you
should use the Maybe monad `>>=` operator or do notation instead. The monad
instead for Either works the same way.
- It is possible to create a monad with an explicit exit command. This will
have the same implementation and behavior as Maybe or MaybeT.
- In IO code it is possible to do this by throwing exceptions. But I don't 
encourage using exceptions just for early exit control flow, at least in
Haskell.

## How do I get a random number or random value from a set?

- In the IO system there is a implicit global generator. You can simply request
a random number from a range with randomRIO or select a random element from a
list with a combination of randomRIO and !!. You'd want to wrap the
latter procedure in it's own function, not included in the standard
System.Random module. This method is quick and dirty but requires that the code
be in IO, even though no actual IO is involved. Also if your application cares
about the reproducibility of the result this is a non-option.
- Pass a random number generator around and return an updated generator from
any functions that will use it. While this might seem ridiculous, it actually
might be simpler to implement, understand, and refactor than other solutions.
- Use a state-like monad to implicitly pass around a generator. Any code that
uses randomness would either use this monad type or else pass the generator
explicitly until you decide to execute random monad actions. You might think
this way is best as long as your entire program either uses this monad or includes
it in a transformer stack. However this has the same modularity problems as
IO and is way more complex. A random monad is a good idea for a local algorithm
whose only "effect" is to request random values, similar in spirit to use cases
for the basic state monad and writer monad.
- Use a random library. For serious generation of random results with
carefully controlled distributions and generator management, there are good
libraries on hackage. `random-fu` provides a variety of distributions and
`mwc-random` provides a high performance high quality generator for uniform
distributions.

## 
