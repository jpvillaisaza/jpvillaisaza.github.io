---
title: Introduction to Lenses
author: Juan Pedro Villa Isaza
icon: fas fa-search
tags: haskell programming
---

Lenses.

```haskell
data Lens a b = Lens
  { get :: a -> b
  , set :: b -> a -> a  
  }
```
