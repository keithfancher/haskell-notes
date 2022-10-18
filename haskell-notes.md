# Learn Me a Haskell, Finally

Notes about things I keep forgetting, or continue to find confusing, while
learning Haskell.

This is not meant as a comprehensive guide, but rather is a scattered
collection of useful information and resources. These notes cover some
meta/toolchain stuff as well aspects of the language itself.

(Note: I'm writing this selfishly. Primarily, *I* am the document's target
audience. Apologies if I make some assumptions throughout. Also, I'm new to
Haskell, so I'm probably wrong about everything. Sorry again!)


## Table of contents

* [Getting started](#getting-started)
* [Installation / Toolchain stuff](#installation--toolchain-stuff)
  * [Initial setup](#initial-setup)
  * [Testing it out](#testing-it-out)
  * ["Real" projects](#real-projects)
  * [A special note about stack install](#a-special-note-about-stack-install)
  * [Stackage](#stackage)
  * [Editor stuff](#editor-stuff)
  * [Static analysis](#static-analysis)
* [Pattern matching](#pattern-matching)
  * [Function argument pattern matching and case statements](#function-argument-pattern-matching-and-case-statements)
  * [Guards](#guards)
  * [Pattern matching vs guards](#pattern-matching-vs-guards)
  * [Pattern matching references](#pattern-matching-references)
* [Monad stuff](#monad-stuff)
  * [Some useful monads](#some-useful-monads)
  * [Monad transformers](#monad-transformers)
* [Error handling](#error-handling)
  * [Exceptions](#exceptions)
  * [(Avoid) partial functions](#avoid-partial-functions)
  * [Stack traces?](#stack-traces)
* [Language extensions](#language-extensions)
* [Everyday programming tasks](#everyday-programming-tasks)
  * [CLI arguments](#cli-arguments)
  * [Unit testing](#unit-testing)
  * [JSON parsing](#json-parsing)
  * [Parsing in general](#parsing-in-general)
  * [HTTP requests](#http-requests)
  * [Working with text](#working-with-text)
* [Miscellaneous good reads](#miscellaneous-good-reads)


## Getting started

These notes won't be enough to get you started. If you're *brand* new, get
yourself a book:

* [Get Programming in
  Haskell](https://www.manning.com/books/get-programming-with-haskell) by Will
  Kurt. I thought this was fantastic. Great balance of theory and practice,
  fun examples, &c. The goal is to get you fluent enough to start building
  "weekend projects" in Haskell. (But also to understand what the hell you're
  doing.)
* [Programming in Haskell](https://www.cs.nott.ac.uk/~pszgmh/pih.html) by
  Graham Hutton. Just started working through this one. Drier, but a good
  complement to *Get Programming in Haskell*. Highly recommended by the
  Haskell community.
* [Haskell Programming from First Principles](https://haskellbook.com/). It's
  a damn encyclopedia, 1200 pages. I haven't read it yet.
* [Real World Haskell](http://book.realworldhaskell.org/). Free! But old.
  Could be valuable, but don't start here.
* [Learn You a Haskell](http://www.learnyouahaskell.com/). I love this book. A
  really amazing (and also free!) reference, though it too is a bit old these
  days. Still fantastic to read, but beware that certain info might be
  out-of-date.

Aside from books, this [CIS 194: Intro to
Haskell](https://www.seas.upenn.edu/~cis1940/fall16/) course is supposedly a
very good intro. And free!


## Installation / Toolchain stuff

Toolchain stuff. A nightmare in any language. And the current state of any
language's ecosystem is the result of years of evolution, working to solve
problems that beginners like us have no context for.

Oh well! Whatever! Gotta start somewhere! Here's what's worked for me so far.
Easy to set up, easy to maintain.

### Initial setup

0. Don't install GHC with your system's package manager. If you already did,
   uninstall it. Fewer surprises down the road this way.
1. Install [GHCup](https://www.haskell.org/ghcup/).
2. Fire it up (`ghcup tui`) and use it to install GHC (the compiler), HLS
   (language server, used for IDE-like features in your editor), and
   (probably) Stack. (More on Stack later.) It will install cabal
   automatically, I believe. (More on cabal later too.)
3. If you're unsure about which versions to install, you can go by what GHCup
   marks as "recommended". (Note that you can still use different versions of
   GHC in your specific projects -- you're not "locked in" with this. It's
   also very easy to install/uninstall different versions with GHCup. AND you
   can have multiple versions installed simultaneously. So don't stress about
   this step.)
4. Put `~/.ghcup/bin` in your path, probably. (Or whatever the equivalent
   directory is on your system. I'm on Ubuntu.)

That's it for initial setup. You can keep everything updated easily with
GHCup, and everything it does/installs is centralized to `~/.ghcup` (or your
system's equivalent). If you screw it all up and want to start fresh, just
blow that directory away. No harm.

### Testing it out

Now you can, e.g., run `ghci` from your terminal and poke around. Or use `ghc`
to compile/interpret your one-off Haskell scripts. This is effectively your
"system" version of GHC (thought it's confined to your home dir, not actually
installed system-wide).

That alone is probably enough to get you most of the way through beginner
tutorials, and even most beginner books.

### "Real" projects

...but eventually you'll want to build real projects. Can o' worms.

There's this whole "Stack or cabal" thing. Also, Stack *uses* cabal. Also,
"Stack" refers to a couple of distinct ideas, sort of? And so does cabal? I
don't have full context. Again, here's what got me started.

Assuming you installed Stack as part of the steps above:

1. `stack new cool-project-name`. This creates a fresh directory from a
   default project template. Good enough for the likes of us, for now.
2. `cd cool-project-name`. Poke around.
3. `stack build`. This'll install dependencies, do initial setup if any is
   required, generate some project files, etc. Note that Stack will also
   install its own version of GHC for your project, depending on the config in
   your `stack.yaml` file. More on this later. These versions go in
   `~/.stack`, or your system's equivalent. (You don't have to worry about
   them, or do anything special to use them. Stack takes care of it.)
4. `stack exec cool-project-name-exe`. Run your code! (I personally popped
   into my `package.yaml` and got rid of that `-exe` extension because it
   looked weird to me, but YMMV.)

I'm not going to go too in-depth here. I highly recommend *Get Programming in
Haskell*, which uses Stack for all the projects in the latter half of the book
and provides some great insight throughout.

Some other useful stuff:

* [The Stack intro user guide](https://docs.haskellstack.org/en/stable/GUIDE/)
* [stack.yaml versus package.yaml versus a Cabal
  file](https://docs.haskellstack.org/en/stable/stack_yaml_vs_cabal_package_file/)
* Stack uses a tool called [hpack](https://github.com/sol/hpack) to generate
  cabal files. I had to poke around in hpack's docs to solve some early
  project problems. You might too.
* ["Why is stack not
  cabal?"](https://www.fpcomplete.com/blog/2015/06/why-is-stack-not-cabal/), a
  good article with some historical context for the whole stack/cabal
  situation.

### A special note about `stack install`

Most tutorials I found suggest that you need to `stack install` your
dependencies. **This is false!** Adding a dependency to your `package.yaml`
file is *all* you need to do. Running `stack build` will do the rest. I don't
know why this rumor is so rampant.

The `stack install` command does only *one* thing: it copies binaries to your
local binary directory. That's it! Odds are you don't want to do that ever,
except for the occasional utility that you'll use *outside of the context of
your project*.

See [this excellent section of the Stack user
guide](https://docs.haskellstack.org/en/stable/GUIDE/#the-stack-install-command-and-copy-bins-option)
for more information and clarity.

### Stackage

[Stackage](https://www.stackage.org/) seems like a great idea to me. "Stable
Haskell package sets", or "A distribution of compatible Haskell packages from
Hackage that build together". In short, you get a versioned set of the
most-used Haskell packages that are known to work well together, *and* with a
given version of GHC.

By default when you `stack new` a new project, it depends on the latest LTS
release of stackage. (You can change this by passing an arg to `stack new` or
changing the resolver in your `stack.yaml` file. But you probably don't need
to.)

There's an LTS for every major GHC version and plenty of minor versions. In
practice, this has worked great for me so far. The only minor downside is
that, typically, the stackage LTS won't be on the absolute latest version of
GHC. (Since the whole package set needs to be vetted/compatible with that
version.) If you care, you can use one of the nightly stackage releases to get
the latest/greatest. If it works for you, you're good to go.

(You can use packages that aren't in stackage too, of course. But I haven't
needed to yet. It's a solid foundation.)

### Editor stuff

VS Code works pretty damn well with Haskell *almost* out of the box. Install
the main Haskell plugin and the Haskell syntax highlighting plugin and you're
good. You'll need to do a *little* bit of configuring -- basically point the
Haskell plugin to your GHCup installation, which it will use to manage the
language server installation(s). It's an easy way to get started!

Any modern vim/neovim setup will give you comparable results, with a bit more
tweaking of course. For neovim, I recommend:

* The [haskell-vim](https://github.com/neovimhaskell/haskell-vim) plugin,
  which greatly improves syntax highlighting and indentation for Haskell over
  the defaults that ship with vim.
* The [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) plugin, which
  includes predefined configs for various language servers. (Assuming you want
  the IDE-like features the language server provides.) See [the
  haskell-language-server
  section](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#hls)
  of the plugin's docs for more specific details.

These plugins (or equivalent) might be just as good with (a newish version of)
vanilla vim, but I don't know for sure. Give it a shot! YMMV.

It's also worthwhile to read up on [the
haskell-language-server](https://haskell-language-server.readthedocs.io/en/latest/features.html)
generally. Tons of cool features, but also some limitations that are good to
be aware of.

(Hopefully it goes without saying that other editors are also probably fine. I
just don't have direct experience. Emacs is always a cool choice. In my Scala
days, I had pretty good luck with IntelliJ, which has a Haskell plugin. Use
whatever works.)

### Static analysis

Especially if you're a beginner, and *especially* if you don't have people
reviewing your code, static analysis can be a big help. Suss out
anti-patterns, teach you some common idioms, prevent common classes of bugs,
etc.

Here's what I've been using so far, all of which have been (their own brand
of) helpful:

* [hlint](https://hackage.haskell.org/package/hlint), via the
  [haskell-language-server](https://haskell-language-server.readthedocs.io/en/latest/features.html#hlint-hints).
  You get this for free by default if you use the language server. If not,
  it's still worth checking out.
* [stan](https://github.com/kowainik/stan) static analyzer. Slightly different
  goals than hlint -- they play well together. Stan has an HLS plugin as well,
  but I haven't tried it yet. (Note that I had to compile my projects with GHC
  `8.10.7` to get Stan to work. This was a simple change for me, but if you
  depend on any GHC 9.x-specific stuff, you might be out of luck?)
* Not quite "static analysis," but: `stack build --pedantic`. Same as enabling
  the `-Wall` and `-Werror` GHC options. Might not want to do this all the
  time, but excellent for catching unused code and (potential) bugs. (Also see
  [this great
  article](https://lexi-lambda.github.io/blog/2018/02/10/an-opinionated-guide-to-haskell-in-2018/#warning-flags-for-a-safe-build)
  for more info on good warning flags to enable.)

And a special mention for
[ormolu](https://hackage.haskell.org/package/ormolu), the zero-config code
formatter. I also use this via the Haskell language server (it's the default
choice for formatting), but it's easy to use as a standalone tool as well. As
a beginner, I don't have strong (Haskell-specific) code formatting opinions,
but I like the ormolu philosophy and, more than that, I like that it Just
Works. (There are a million other formatters too. Choose one and use it
consistently, is the main thing.)


## Pattern matching

There are a few different pattern matching ideas in Haskell, and they tend to
be a little confused in the explanations I've read so far. Additionally, I've
never been clear which construct is preferred in which case. This section is
my attempt to sort that out.

### Function argument pattern matching and `case` statements

The "lowest level" structure is the `case` statement. These can be used
anywhere, not just in function definitions. For example, these two bits of
code are equivalent:

```haskell
-- "Normal" function arg pattern matching:
head' :: [a] -> a
head' [] = error "No head for empty lists!"
head' (x:_) = x
```

```haskell
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

```haskell
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

```haskell
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

> * In general, when in doubt, just stick with pattern matching by default, it's
>   usually nicer. If a pattern starts getting really ugly or convoluted, then
>   stop to consider how else you could write it. Besides using guards, other
>   options include extracting subexpressions as separate functions or putting
>   case expressions inside the function body in order to push some of the
>   pattern matching down onto them and out of the main definition.
> * Definitely don't use guards for things that could be trivially checked with
>   a pattern. Checking for empty lists is the classic example, use a pattern
>   match for that.
> * Definitely use guards when you need to make a choice based on some property
>   that doesn't correspond neatly to a pattern, e.g. comparing two Int values
>   to see which is larger.

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
* [Reader](https://hackage.haskell.org/package/mtl-2.3/docs/Control-Monad-Reader.html):
  used to pass around read-only "environment" to functions. ([This
  article](https://fsharpforfunandprofit.com/posts/dependencies-3/) is about
  F#'s Reader, but is still what made the concept "click" for me.)
* [Writer](https://hackage.haskell.org/package/mtl-2.3/docs/Control-Monad-Writer-Lazy.html): basically used for logging, or similar operations
* [State](https://hackage.haskell.org/package/mtl-2.3/docs/Control-Monad-State-Lazy.html): used to pass around/isolate mutable state (read/write)

### Monad transformers

Different monads don't (always? ever?) compose nicely. For that, one needs
monad transformers. The type class definition sums it up pretty well,
actually:

```haskell
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
on. See
[`Control.Exception`](https://hackage.haskell.org/package/base-4.17.0.0/docs/Control-Exception.html).

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

So what do you do instead? See [this
article](https://wiki.haskell.org/Avoiding_partial_functions) from the Haskell
wiki for good examples of alternatives to these functions. (Notably absent
from that list are [`readMaybe` and
`readEither`](https://hackage.haskell.org/package/base-4.17.0.0/docs/Text-Read.html#v:readMaybe),
which are safe alternatives to `read`.)

For lists specifically, there's a
[`Data.List.Safe`](https://hackage.haskell.org/package/listsafe-0.1.0.1/docs/Data-List-Safe.html)
option as well. But I've found the
[`Data.List.NonEmpty`](https://hackage.haskell.org/package/base-4.17.0.0/docs/Data-List-NonEmpty.html)
type to be even more useful. It's exactly what it sounds like: a list which
must have at least one element.

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

This enables "DWARF" support. See [this blog
post](https://www.haskell.org/ghc/blog/20200403-dwarf-1.html) for more
information. I don't fully understand the trade-offs here yet.

Also, some more (general) info here: https://wiki.haskell.org/Debugging.

Frankly, I'm still not sure what the "right" thing to do is, overall.
Certainly one doesn't want to always have profiling enabled. Right? (My
current approach is to hunt down and destroy any code that *might* throw an
error, which I guess is ideal anyway.)


## Language extensions

Language extensions seemed totally bananas to me at first. Maybe they still
do, but I'm getting used to it. In short, as I understand it, they're simply
"add-ons" to the official Haskell spec, which tends to be pretty conservative.
(In theory, certain extensions might make it into the next version of the
spec, so it's a kind of proving ground, perhaps?)

In practice, being able to change the behavior of the language, more or less
ad hoc, *somehow* doesn't cause horrible problems. And there are a handful of
"standard" extensions that you'll see used in almost every project.

One can enable extensions with a compiler flag, using `LANGUAGE` pragmas at
the top of a given file, or for a whole project in your
`package.yaml`/`.cabal` file. Opinions are mixed about what's the "best" way.
(I've been doing it on a per-project basis. It's just simpler.)

Here's a sampling of useful extensions I've discovered so far:

* `OverloadedStrings`: string literals play well with other types,
  specifically (for me) `Text`. This is the only extension I've used in
  *every* project.
* `DuplicateRecordFields`: allow definition of record types with
  identically-named fields.
* `NoImplicitPrelude`: don't use the default Haskell `Prelude`. Sometimes
  handy.
* `StrictData`: make fields of data types defined in the current module strict
  by default. There are all kinds of trade-offs both ways, but this is a good
  thing to know about.
* `DeriveGeneric`: I've barely scraped the surface of this one.
  [Here's](https://blog.ocharles.org.uk/posts/2014-12-16-derive-generic.html)
  an article that goes into it a bit. Mostly, I've just used this to magically
  make my types instances of `FromJSON` and `ToJSON`.

Note that, **as of GHC 9.2.1**, there's a [`GHC2021`
meta-extension](https://downloads.haskell.org/~ghc/9.2.1/docs/html/users_guide/exts/control.html#extension-GHC2021)
which is enabled by default:

> GHC blesses a number of extensions, beyond Haskell 2010, to be suitable to
> turned on by default. These extensions are considered to be stable and
> conservative.
>
> `GHC2021` is used by GHC if neither `Haskell98` nor `Haskell2010` is turned
> on explicitly. Since later versions of GHC may use a later `GHC20xx` by
> default, users are advised to declare the language set explicitly with
> `-XGHC2021`.

Some more quality reading on extensions:

* [The official GHC
  docs](https://ghc.gitlab.haskell.org/ghc/doc/users_guide/exts.html), which
  aren't so great for opinionated help but are great for reference.
* [This big-ass guide](https://limperg.de/ghc-extensions/) from 2018, which
  groups the extensions into "tracks" and gives some good info.
* [The extension
  section](https://lexi-lambda.github.io/blog/2018/02/10/an-opinionated-guide-to-haskell-in-2018/#any-flavor-you-like)
  of the "Opinionated Guide to Haskell in 2018", which is a great overview of
  the "good"/recommended extensions. (At least as of 2018. Not sure how much
  has changed of late.)


## Everyday programming tasks

All the boring everyday programming stuff you need to do is possible in
Haskell, and not any more difficult than in any other language, at least in my
(limited!) experience.

### CLI arguments

For the simplest use-cases, you can just
[`getArgs`](https://hackage.haskell.org/package/base-4.17.0.0/docs/System-Environment.html#v:getArgs),
which gives you back an `IO [String]`. Very easy to use in your `main`. For
anything more complex than that, you'd likely want to use one of the many
libraries available.

The unfortunately named
[optparse-applicative](https://www.stackage.org/package/optparse-applicative#quick-start)
is probably the most widely-used. At first glance, it looks complicated (and
it has the word "applicative" in the name), but it was actually super-simple
to get up and running. The "quick start" (linked above) will get you a long
way. It can handle pretty much any complex use-case you can throw at it, but
is also great for simple stuff.

[This blog
post](https://thoughtbot.com/blog/applicative-options-parsing-in-haskell) was
also very helpful, and has some good examples of optparse-applicative in
action.

Other resources:

* [optparse-simple](https://www.stackage.org/package/optparse-simple) is built
  on optparse-applicative, but cuts out some boilerplate for simple use-cases.
* [cmdargs](https://hackage.haskell.org/package/cmdargs) is also popular and
  supposedly good, but I haven't tried it!
* [simple-get-opt](https://hackage.haskell.org/package/simple-get-opt) is
  also, well... simple.
* Check out this
  [meta-list](https://wiki.haskell.org/Command_line_option_parsers) from the
  haskell.org wiki! Good comparison of different options.

### Unit testing

[HUnit](https://hackage.haskell.org/package/HUnit) exists, but after a short
trial run I ditched it. HUnit tests can wind up looking like an exaggerated
parody of unreadable code, e.g.:

```haskell
tests = test [ "test1" ~: "(foo 3)" ~: (1,2) ~=? (foo 3),
               "test2" ~: do (x, y) <- partA 3
                             assertEqual "for the first result of partA," 5 x
                             partB y @? "(partB " ++ show y ++ ") failed" ]
```

After poking around some more, I found [Hspec](https://hspec.github.io/). It
favors readable tests over terseness and has some other handy features, nicer
output, and integrates well with HUnit tests if you want to mix/match.

Separately, there's this odd bird called
[QuickCheck](https://hackage.haskell.org/package/QuickCheck). You can use it
by itself or with any other test library/framework. It does "random testing of
program properties". A cool idea, but I haven't used it much yet.

(My intuition here is that you'd have to be careful not to rewrite the logic
of your system-under-test in order to programmatically test that system. Many
of the QuickCheck examples I've seen do just that. But still seems handy!)

### JSON parsing

Use [Aeson](https://hackage.haskell.org/package/aeson)! (In Greek mythology,
Aeson is the father of Jason. Get it?)

### Parsing in general

Use [Parsec](https://hackage.haskell.org/package/parsec). Super cool.

### HTTP requests

For simple stuff, (probably?) use
[Network.HTTP.Simple](https://hackage.haskell.org/package/http-conduit-2.3.8/docs/Network-HTTP-Simple.html).
For less simple stuff, [wreq](https://hackage.haskell.org/package/wreq)?
However, wreq uses `lens`, which is a whole other thing to get into.

### Working with text

It's a little counter-intuitive, but as a general rule: do **not** use
`String` when working with text in Haskell. The `String` type is simply a
`List` of characters, and tends to have terrible performance with text
operations.

Instead, use
[`Data.Text`](https://hackage.haskell.org/package/text-2.0.1/docs/Data-Text.html)
(and its related modules,
[`Data.Text.IO`](https://hackage.haskell.org/package/text-2.0.1/docs/Data-Text-IO.html),
etc.). Strict by default, performant, easy to use.

It's fine (and convenient!) to use `String` while you're learning, but for
anything "real", just stick with `Text`. Like, everywhere.

(Note that the
[`OverloadedStrings`](https://ghc.gitlab.haskell.org/ghc/doc/users_guide/exts/overloaded_strings.html)
language extension makes this transition pretty painless. It simply makes
string literals in your code a bit more flexible. Recommended.)


## Miscellaneous good reads

* [An opinionated guide to Haskell in
  2018](https://lexi-lambda.github.io/blog/2018/02/10/an-opinionated-guide-to-haskell-in-2018/)
  -- really good overview, context, and advice for a beginner from someone who
  clearly has experience in the trenches. Great stuff like: "Warning flags for
  a safe build", "Libraries: a field guide", and a whole big ol' section on
  language extensions. Things have probably changed a bit since 2018, but I
  still found this really valuable.
