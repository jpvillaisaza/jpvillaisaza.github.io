---
title: Minimal Cabal Files
author: Juan Pedro Villa Isaza
tags: haskell programming
---

Every Haskell package has a [Cabal][cabal] file, which contains
metadata about the package. Have you ever wondered what a minimal
Cabal file looks like? In other words, what is the minimum amount of
data you absolutely need to include in a Cabal file so that it yields
a fully functional Haskell package?

Well, let's make an experiment and find out!

First of all, here are the versions of Cabal and GHC I'm using:

```
$ cabal --version
cabal-install version 1.24.0.2
compiled using version 1.24.2.0 of the Cabal library
$ ghc --version
The Glorious Glasgow Haskell Compilation System, version 8.0.2
```

Let's create a directory for experiments and move to it:

```
$ mkdir experiments
$ cd experiments/
```

Every time we do something, we'll try to build the package using
Cabal:

```
$ cabal build
Package has never been configured...
cabal: No Cabal file found.
Please create a package description file <pkgname>.cabal
```

Also, every time we get an error, we'll try to do the minimum amount
of work required to fix it, and then try to build the package again.
In this case, let's simply create a Cabal file:

```
$ touch experiments.cabal
```

And build (from now on, I'll omit some of the output or replace it
with ellipses):

```
$ cabal build
... Using 'build-type: Custom' but there is no Setup.hs...
```

But we don't want a custom build, so let's be specific about what we
want:

```
$ echo "build-type: Simple" >> experiments.cabal
```

And try again:

```
$ cabal build
... No 'name' field.
... No 'version' field.
```

Every Haskell package has a name and a version, so let's add both to
the Cabal file:

```
$ echo "name: experiments" >> experiments.cabal
$ echo "version: 1.0.0" >> experiments.cabal
```

This fixes some of the errors:

```
$ cabal build
... No executables, libraries (...) found. Nothing to do.
```

But we have to give Cabal something to do. Let's do that by declaring
a library section:

```
$ echo "library" >> experiments.cabal
```

And try again:

```
$ cabal build
... A package using section syntax must specify at least
... 'cabal-version: >= 1.2'.
```

Adding a library section requires at least Cabal 1.2. Before
specifying a Cabal version, let's take a look at the Cabal file so
far:

```
$ cat experiments.cabal
build-type: Simple
name: experiments
version: 1.0.0
library
```

We can't keep appending things to the Cabal file (try it!), so we'll
have to open an editor and add a Cabal version. After reorganizing the
file, it should look like this:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.2

library
```

Let's build:

```
$ cabal build
Resolving dependencies...
Configuring experiments-1.0.0...
Building experiments-1.0.0...
Preprocessing library experiments-1.0.0...
```

It works!

No, not really. Let's try to open GHCi:

```
$ cabal repl
... Not in scope: ‘System.IO.hSetBuffering’
... No module named ‘System.IO’ is imported.
```

We can't open GHCi because we're missing the base package, which we'll
have to add as a build dependency:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.2

library
  build-depends: base
```

Let's try again:

```
$ cabal repl
Warning: No exposed modules
...
Prelude>
```

We can open GHCi now, but there are no exposed modules. We should be
able to put Haskell code somewhere, so let's add an `Experiments`
module:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.2

library
  exposed-modules: Experiments
  build-depends: base
```

And a corresponding `Experiments.hs` file:

```haskell
module Experiments where
```

Let's try again:

```
$ cabal repl
...
*Experiments>
```

It works! Kind of...

Just for fun, let's try adding an executable component:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.2

library
  exposed-modules: Experiments
  build-depends: base

executable experiments
  main-is: Main.hs
```

The `Main.hs` file can look something like this:

```haskell
main = putStrLn "Put it to the test!"
```

Everything works even though the executable has no dependencies:

```
$ cabal run
...
Put it to the test!
```

An executable usually depends on the library defined within the same
package, so let's do that:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.2

library
  exposed-modules: Experiments
  build-depends: base

executable experiments
  main-is: Main.hs
  build-depends: experiments
```

And build:

```
$ cabal build
... The field 'build-depends: experiments' refers to a library which
... is defined within the same package. To use this feature the
... package must specify at least 'cabal-version: >= 1.8'.
```

OK, we need at least Cabal 1.8:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.8

library
  exposed-modules: Experiments
  build-depends: base

executable experiments
  main-is: Main.hs
  build-depends: experiments
```

Let's try again:

```
$ cabal build
... Failed to load interface for ‘Prelude’
... It is a member of the hidden package ‘base-4.9.1.0’.
```

We broke it!

We're just missing the base package, but why did it work before then?
Before Cabal 1.8, build dependencies were global, so the executable
had access to the base package because it was listed as a dependency
of the library.

Here's the Cabal file after fixing this:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.8

library
  exposed-modules: Experiments
  build-depends: base

executable experiments
  main-is: Main.hs
  build-depends: base, experiments
```

It works! Almost... If we don't specify a default language, Cabal will
default to Haskell 98, which is probably not what we want. Let's test
that this is the case by adding an empty type to the `Experiments`
module:

```haskell
module Experiments where

data Empty
```

This doesn't work in Haskell 98 unless we use a language extension:

```
$ cabal build
... ‘Empty’ has no constructors (EmptyDataDecls permits this)
```

But it works in Haskell 2010, which is what we want right now:

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.8

library
  exposed-modules: Experiments
  build-depends: base
  default-language: Haskell2010

executable experiments
  main-is: Main.hs
  build-depends: base, experiments
```

Let's build:

```
$ cabal build
... To use the 'default-language' field the package needs to
... specify at least 'cabal-version: >= 1.10'.
```

We need at least Cabal 1.10, which is actually what we get if we use
`cabal init` to generate a Cabal file. Here's the resulting file (note
that the default language is specified for all components):

```
name: experiments
version: 1.0.0
build-type: Simple
cabal-version: >= 1.10

library
  exposed-modules: Experiments
  build-depends: base
  default-language: Haskell2010

executable experiments
  main-is: Main.hs
  build-depends: base, experiments
  default-language: Haskell2010
```

And this is, finally, a minimal Cabal file which yields a fully
functional Haskell package with a library and an executable (which we
added just for fun).


### Conclusion

Now, why would you want a minimal Cabal file? Well... I guess you
typically wouldn't, but now you know how one of them looks like... In
fact, I can think of at least one reason why I'd want one.
The [Haskell Programming from First Principles][haskell-book] book
recommends having a permanent Haskell package for experiments, which
eliminates the overhead of creating a package each time you just want
to make an experiment with Haskell. I like my Haskell experiments
package to have a minimal Cabal file: just enough to play with
Haskell.

Additionally, knowing what a minimal Cabal file looks like is
interesting and means you know how to build a Haskell package from
scratch, which can help to better understand Cabal and Haskell. Or
not.

[cabal]: https://www.haskell.org/cabal/
[haskell-book]: http://haskellbook.com/
