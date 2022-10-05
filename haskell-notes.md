# Learn Me a Haskell, Finally

Notes about things I keep forgetting, or continue to find confusing, while
learning Haskell. This is not meant as a comprehensive guide, but rather is a
scattered collection of useful information and resources.

(Note: I'm writing this selfishly. Primarily, *I* am the document's target
audience. Apologies if I make some assumptions throughout.)

## Pattern matching

There are a few different pattern matching ideas in Haskell, and they tend to
be a little confused in the explanations I've read so far. Additionally, I've
never been clear which construct is preferred in which case. This section is
my attempt to sort that out.

### Function argument pattern matching and `case` statements

The "lowest level" is to use `case` statements. These can be used anywhere,
not just in function definitions. For example, these two bits of code are
equivalent:

```
-- "Normal" pattern matching:
head' :: [a] -> a
head' [] = error "No head for empty lists!"
head' (x:_) = x
```

```
-- case statement
head' :: [a] -> a
head' xs = case xs of [] -> error "No head for empty lists!"
                      (x:_) -> x
```

...and in fact, function arg pattern matching is just syntactic sugar for
`case` statements, at least according to *LYAH*. In this case, pattern
matching is more readable and (probably) generally preferred.

### Guards

The third idea -- more of a related concept than a separate "thing" -- is to
use guards. See this example from *LYAH*:

```
bmiTell :: (RealFloat a) => a -> String
bmiTell bmi
    | bmi <= 18.5 = "You're underweight, you emo, you!"
    | bmi <= 25.0 = "You're supposedly normal. Pffft, I bet you're ugly!"
    | bmi <= 30.0 = "You're fat! Lose some weight, fatty!"
    | otherwise   = "You're a whale, congratulations!"
```

"Guards are simply *constraints* for patterns..." is a great way to think
about it. They can be used in both function definitions and `case`
expressions:

> There are two places guards are allowed: function definitions and case
> expressions. In both contexts, guards appear after a pattern and before the
> body, so you use = in functions and -> in case branches, as usual.

From https://stackoverflow.com/a/40836465.

Note that you can use a combo approach: "normal" pattern matching as well as
guards.

> It's also worth mentioning that since pattern guards were added to Haskell
> 2010, you're allowed to mix patterns and guards like so:

```
accumulate_list' :: (Eq a, Num a) => [a] -> ( a -> a -> a ) -> a
accumulate_list' l f
   | []     <- l = 0          --pattern for the empty list case
   | 10 < 5      = 10         --arbitrary regular guard just because
   | (x:xs) <- l = undefined  --pattern for the non-empty case
```

From: https://stackoverflow.com/a/25493823. I find this pretty difficult to
read, personally. (Or maybe this is just a bad example?)  In either case, just
noting it here for reference.

### Pattern matching vs guards

This SO post sums up some best-practices about when to use which construct:
https://stackoverflow.com/a/4156831. Some of the most useful general
recommendations:

* In general, when in doubt, just stick with pattern matching by default, it's
  usually nicer. If a pattern starts getting really ugly or convoluted, then
  stop to consider how else you could write it. Besides using guards, other
  options include extracting subexpressions as separate functions or putting
  case expressions inside the function body in order to push some of the
  pattern matching down onto them and out of the main definition.
* Definitely don't use guards for things that could be trivially checked with
  a pattern. Checking for empty lists is the classic example, use a pattern
  match for that.
* Definitely use guards when you need to make a choice based on some property
  that doesn't correspond neatly to a pattern, e.g. comparing two Int values
  to see which is larger.

### Pattern matching references

* http://learnyouahaskell.com/syntax-in-functions
* https://stackoverflow.com/a/2225811
* https://www.haskell.org/tutorial/patterns.html
* Lesson 17.2.2 in *Get Programming with Haskell*, specifically about guards


## Monad stuff

First off, "Typeclassopedia" is really good, if a bit dense at times, and also
contains SO MANY good links to papers, articles, blog posts, etc:

https://wiki.haskell.org/Typeclassopedia

Many of my notes here are sourced from there.

What's a monad? There are a million explanations for this. *Get Programming
with Haskell* does a good job, I think. Typeclassopedia has links to others.
In particular, this simple demonstration does a pretty good job via a
"trivial" monad (essentially `Identity`):

http://blog.sigfpe.com/2007/04/trivial-monad.html

It sums things up thusly, which is not a bad way to *start* thinking about it:

> So the last question is this: why would you ever wrap data like this? In
> practice people tend not to use the trivial monad very much. Nonetheless,
> you can see how it might be used to represent tainted data. Wrapped data is
> considered tainted. Our API never lets us forget when data is tainted and
> yet it still allows us to do what we like with it.

To go back to the source, so to speak, scroll down to the bottom of the
following page for some crazy (but supposedly readable?) papers introducing
the subject:

https://homepages.inf.ed.ac.uk/wadler/topics/monads.html

And of course, some *LYAH* explanations, which are always fun:

http://learnyouahaskell.com/for-a-few-monads-more

...and the haskell.org "All About Monads" page, which has... a lot:

https://wiki.haskell.org/All_About_Monads

### Some useful monads

* Maybe: obviously!
* Either: errors!
* IO: special magic monad which (I think?) can only be unwrapped by `main`
* Reader: used to pass around read-only "environment" to functions
  (https://hackage.haskell.org/package/mtl-2.3/docs/Control-Monad-Reader.html)
  (f-sharp, but the article that made it "click" for me: https://fsharpforfunandprofit.com/posts/dependencies-3/)
* Writer: basically used for logging, or similar operations
  (https://hackage.haskell.org/package/mtl-2.3/docs/Control-Monad-Writer-Lazy.html)
* State: used to pass around/isolate mutable state (read/write)
  (https://hackage.haskell.org/package/mtl-2.3/docs/Control-Monad-State-Lazy.html)

### Monad transformers

Different monads don't (always? ever?) compose nicely. For that, one needs
monad transformers. The type class definition sums it up pretty well,
actually:

```
class MonadTrans t where
  lift :: Monad m => m a -> t m a
```

Basically, it lets you wrap a monad around another monad. For example, if you
have a function that might fail (`Either`) and which depends on some read-only
environment (`Reader`), you can use `ReaderT` and `ExceptT` monad transformers
to create a composite.

> The use of monad transformers makes it very easy to define specialized
> monads for many applications, reducing the temptation to put everything
> possibly needed into the one and only monad hand-made for the current
> application.

**IMPORTANT NOTE:** Order matters in monad composition. The "core" (inner)
monads come first, and they work their way out.

Note that many examples I've seen start with the `Identity` monad as the
inner-most layer. I don't (yet) fully understand this. It doesn't necessarily
seem required?

Related to the above, however, some special consideration is required when
wrapping the `IO` monad. In short, since `IO` can *only* be unwrapped by
`main`, it needs to be the innermost "core" of your transformer-onion. In this
case, you're using it *instead* of `Identity`. Use the `liftIO` function to
lift the IO computation into the currently running monad.

Good intro resources I've found on monad transformers:

* http://blog.sigfpe.com/2006/05/grok-haskell-monad-transformers.html
* https://github.com/mgrabmueller/TransformersStepByStep/blob/master/Transformers.lhs


## Error handling

A good, if brief overview of the different types of errors one might think
about handling (or not), and how they're represented in Haskell:

https://wiki.haskell.org/Handling_errors_in_Haskell

And a longer article, with some good insight and actually useful examples:

https://www.stackbuilders.com/blog/errors-and-exceptions-in-haskell/

As with any programming language, the trick seems to be distinguishing between
the truly "exceptional" errors/exceptions and the expected ones. The expected
ones should be encoded in your dang type signature (`Either`, etc.) if
possible.

Related, this article builds the idea of `Either` up from scratch, shows some
interesting example of error-handling cases:

https://www.schoolofhaskell.com/school/starting-with-haskell/basics-of-haskell/10_Error_Handling

### Exceptions

Haskell does have "normal" exceptions, which can be caught and thrown and so
on. See `Control.Exception`:

https://hackage.haskell.org/package/base-4.17.0.0/docs/Control-Exception.html

However, I don't yet grok how best to use these, if at all. I *think* the idea
is to keep them out of pure code entirely, and only use them when dealing with
`IO`. And related to that, the expectation seems to be that any `IO` value
could fail with an exception. (For example, a file doesn't exist, a connection
fails, etc.) So watch out!

As far as *catching* exceptions: this SO post has some good info on the
particulars of how and when to use the various exception-catching methods:
https://stackoverflow.com/a/6009807

### (Avoid) partial functions

These will fail hard, bringing your program crashing down. And from what I've
experienced, there will be no stack trace and it will be difficult to track
down exactly where in the stack the error occurred. (See the "Stack traces?"
section below.)

Examples of partial functions to avoid:

* `head`, which fails on an empty list
* `last`, same as above
* `!!`, the "access the nth elem of a list" function
* `read`, which parses a value out of a string
* ...and probably many others

See https://wiki.haskell.org/Avoiding_partial_functions for good examples of
alternatives to these functions. A useful one which is absent from that list,
`readMaybe`/`readEither`, which is a safe alternative to `read`:

https://hackage.haskell.org/package/base-4.17.0.0/docs/Text-Read.html#v:readMaybe)

For lists specifically, there's a `Data.List.Safe` option as well:

https://hackage.haskell.org/package/listsafe-0.1.0.1/docs/Data-List-Safe.html

And this is only somewhat-related, but there's the whole universe of "lens" to
explore, which is likely overkill if you just need to grab an element from a
list (but does also work for that). Plenty of useful stuff:

https://hackage.haskell.org/package/lens

The authors even include a wee bit of code you can use to create your own
lenses for arbitrary types, without adding additional dependencies. Nice!

### Stack traces?

By default, one doesn't seem to get stack traces with Haskell. (Maybe this has
something to do with lazy eval? I don't know.) You'll get a one-line error and
maybe the line number of the initial source of the error (i.e. the unit test
that was running), but nothing more. Not super helpful.

Apparently there are various compile-time options AND runtime options that can
enable stack traces. From a SO post:

> You can compile your program with profiling enabled and automatic cost
> centers for better traces: `-prof -fprof-auto` and run it with `+RTS -xc`,
> then you will get the sequence of calls that leads to your error.

Again, note that there are options for both *compiling* and *running* your
program here to get a stack trace. You need both.

Or, another option:

> With GHC 8.0.1 and better, just compile with the `-g` flag. No need for
> profiling or cost centers.

This enables "DWARF" support. See https://www.haskell.org/ghc/blog/20200403-dwarf-1.html
for more information. I don't fully understand the trade-offs here yet.

Also, some more (general) info here: https://wiki.haskell.org/Debugging.

Frankly, I'm still not sure what the "right" thing to do is, overall.
Certainly one doesn't want to always have profiling enabled. Right? (My
current approach is to hunt down and destroy any code that *might* throw an
error, which I guess is ideal anyway.)
