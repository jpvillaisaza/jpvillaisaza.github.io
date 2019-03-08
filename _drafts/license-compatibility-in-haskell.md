---
title: License Compatibility in Haskell
author: Juan Pedro Villa Isaza
tags: haskell programming
---

There are several tools to check for license compatibility issues, but
one of the most simple ones is to directly use Stack and some of its
commands, which is actually a very nice way of taking care of the
dependencies in your projects.

There are two commands we'll have to deal with. The first one is the
one to list dependencies, which is stack list-dependencies.

For a minimal project, we can do:

```
$ stack list-dependencies
```

If we want to know the license for each package, then:

```
$ stack list-dependencies --license
```

And then we can grep things if we want to find out more. For example,
suppose we want to know whether we have a gpl dependency:

```
$ stack list-dependencies --license | grep -i gpl
```

In some cases this is enough. If we depend on a package A and that
package is GPL licensed, then we get that in the output. But in some
other cases, we depend on package A and package A depends on some
other package that is GPL licensed, but we don't know where the
dependency comes from.

Without changing tools, we can use stack dot to figure it out.

```
$ stack dot --external | grep -i b
```

And we'll see, in dot format, how b is related to all dependencies.
