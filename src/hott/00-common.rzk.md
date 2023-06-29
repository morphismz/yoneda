# 0. Common

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Products of types

```rzk
#def prod (A B : U) : U
  := ∑ (x : A), B

-- defined to illustrate the syntax for terms in sigma types
#def diagonal (A : U) (a : A) : prod A A
  := (a , a)
```

## The type of logical equivalences between types

```rzk
#def iff (A B : U) : U
  := prod (A -> B) (B -> A)
```

## Basic function definitions

```rzk
#section basic-functions

#variables A B C D : U

#def composition
  (g : B -> C)    -- The second function.
  (f : A -> B)    -- The first function.
  : A -> C        -- The composite function.
  := \z -> g (f z)

#def triple-composition
  (h : C -> D)
  (g : B -> C)
  (f : A -> B)
  : A -> D
  := \z -> h (g (f z))

#def identity
  : A -> A
  := \a -> a

#def constant
  (b : B)         -- The constant output value.
  : A -> B
  := \a -> b

#end basic-functions
```

## Substitution

```rzk
-- Reindexing a type family along a function into the base type.
#def reindex
  (A B : U)
  (f : B -> A)
  (C : A -> U)
  : (B -> U)
  := \b -> C (f b)
```

## Final projection

```rzk
-- Constant map into a pointed type
#def const
  (A : U)
  (B : U)
  (b : B)
  : (A -> B)
  := \a -> b

-- Final projection as a constant map
#def final-projection
  (A : U)
  : A -> Unit
  := const A Unit unit

```

## Unit type

```rzk
#def ind-Unit
  (C : Unit -> U)
  (C-unit : C unit)
  (x : Unit)
  : C x
  := C-unit

#def uniq-Unit
  (x : Unit)
  : x = unit
  := refl

#def isProp-Unit
  (x y : Unit)
  : x = y
  := refl
```