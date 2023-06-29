# Cocartesian families

These formalizations capture cocartesian families as treated in BW23.

The goal, for now, is not to give a general structural account as in the paper
but rather to provide the definitions and results that are necessary to prove
the cocartesian Yoneda Lemma.

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Prerequisites

- `hott/*` - We require various prerequisites from homotopy type theory, for
  instance the axiom of function extensionality.
- `3-simplicial-type-theory.md` — We rely on definitions of simplicies and their
  subshapes.
- `4-extension-types.md` — We use the fubini theorem and extension
  extensionality.
- `5-segal-types.md` - We make heavy use of the notion of Segal types
- `8-covariant.md` - We use covariant type families.
- `10-rezk-types.md`- We use Rezk types.



## (Iso-)Inner families

This is a (tentative and redundant) definition of (iso-)inner families.
In the future, hopefully, these can be replaced by instances of
orthogonal and LARI families.

```rzk
#def totalType
  (B : U)
  (P : B -> U)
  : U
  := ∑ (b : B), P b

#def isInnerFam
  (B : U)
  (P : B -> U)
  : U
  := prod (prod (isSegal B) (isSegal (totalType B P))) ((b : B) -> (isSegal (P b)))

#def isIsoInnerFam
  (B : U)
  (P : B -> U)
  : U
  := prod (prod (isRezk B) (isRezk (totalType B P))) ((b : B) -> (isSegal (P b)))
```

## Cocartesian arrows

Here we define the proposition that a dependent arrow in a family is
cocartesian. This is an alternative version using unpacked extension types, as
this is preferred for usage.

```rzk
-- [BW23, Definition 5.1.1]
#def isCocartArr
  (B : U)
  (b b' : B)
  (u : hom B b b')
  (P : B -> U)
  (e : P b)
  (e' : P b')
  (f : dhom B b b' u P e e')
  : U
  := (b'' : B)
  -> (v : hom B b' b'')
  -> (w : hom B b b'')
  -> (sigma : hom2 B b b' b'' u v w)
  -> (e'' : P b'')
  -> (h : dhom B b b'' w P e e'')
  -> isContr
      ( ∑ ( g : dhom B b' b'' v P e' e'') ,
          ( dhom2 B b b' b'' u v w sigma P e e' e'' f g h))

```

## Cocartesian lifts

The following is the type of cocartesian lifts of a fixed arrow in the base with
a given starting point in the fiber.

```rzk
-- [BW23, Definition 5.1.2]
#def CocartLift
    (B : U)
    (b b' : B)
    (u : hom B b b')
    (P : B -> U)
    (e : P b)
    : U
    := ∑(e' : P b'), ∑(f : dhom B b b' u P e e'), isCocartArr B b b' u P e e' f

```

#def cocart-is-prop
    (B : U)
    (BisRezk : isRezk B)
    (b b' : B)
    (u : hom B b b')
    (P : B -> U)
    (TPisRezk : isRezk (totalType B P))
    (PisfibRezk : (b : B) -> isRezk (P b))
    (e : P b)
    (e' : P b')
    (f : dhom B b b' u P e e')
    (fiscocart : isCocartArr B b b' u P e e' f)
    : isContr(CocartLift B b b' u P e)
    := ( (e', f, fiscocart),
        \d -> \g ->

## Initial objects

```rzk
#def isInitial
    (A : U)
    (a : A)
    : U
    := (x : A) -> isContr(hom A a x)
```

In a Segal type, initial objects are isomorphic.

```rzk
#def initial-iso
  (A : U)
  (AisSegal : isSegal A)
  (a b : A)
  (ainitial : isInitial A a)
  (binitial : isInitial A b)
  : Iso A AisSegal a b
  :=
    ( first (ainitial b) ,
      ( ( first (binitial a) ,
          contractible-connecting-htpy
            ( hom A a a)
            ( ainitial a)
            ( Segal-comp A AisSegal a b a
              ( first (ainitial b))
              ( first (binitial a)))
            ( id-arr A a)) ,
        ( first (binitial a) ,
          contractible-connecting-htpy
            ( hom A b b)
            ( binitial b)
            ( Segal-comp A AisSegal b a b
              ( first (binitial a))
              ( first (ainitial b)))
            ( id-arr A b))))
```

## Final objects

```rzk
#def isFinal
  (A : U)
  (a : A)
  : U
  := (x : A) -> isContr(hom A x a)
```

In a Segal type, final objects are isomorphic.

```rzk
#def final-iso
  (A : U)
  (AisSegal : isSegal A)
  (a b : A)
  (afinal : isFinal A a)
  (bfinal : isFinal A b)
  (iso : Iso A AisSegal a b)
  : Iso A AisSegal a b
  :=
    ( first (bfinal a) ,
      ( ( first (afinal b) ,
          contractible-connecting-htpy
            ( hom A a a)
            ( afinal a)
            ( Segal-comp A AisSegal a b a
              ( first (bfinal a))
              ( first (afinal b)))
            ( id-arr A a)) ,
        ( first (afinal b) ,
          contractible-connecting-htpy
            ( hom A b b)
            ( bfinal b)
            ( Segal-comp A AisSegal b a b
              ( first (afinal b))
              ( first (bfinal a)))
            ( id-arr A b))))
```