# Covariantly functorial type families

These formalisations correspond to Section 8 of the RS17 paper.

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Prerequisites

- `hott/*` - We require various prerequisites from homotopy type theory, for
  instance the notion of contractible types.
- `3-simplicial-type-theory.md` — We rely on definitions of simplicies and their
  subshapes.
- `5-segal-types.md` - We make use of the notion of Segal types and their
  structures.

## Dependent hom types

In a type family over a base type, there is a dependent hom type of arrows that
live over a specified arrow in the base type.

```rzk title="RS17, Section 8 Prelim"
-- The type of dependent arrows in C over f from u to v
#def dhom
  (A : U)            -- The base type.
  (x y : A)          -- Two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (C : A → U)        -- A type family.
  (u : C x)          -- A lift of the domain.
  (v : C y)          -- A lift of the codomain.
  : U
  := (t : Δ¹) → C (f t) [
        t ≡ 0₂ ↦ u ,
        t ≡ 1₂ ↦ v
    ]
```

It will be convenient to collect together dependent hom types with fixed domain
but varying codomain.

```rzk
#def dhom-from
  (A : U)            -- The base type.
  (x y : A)          -- Two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (C : A → U)        -- A type family.
  (u : C x)          -- A lift of the domain.
  : U
     := (Σ (v : C y) , dhom A x y f C u v)
```

There is also a type of dependent commutative triangles over a base commutative
triangle.

```rzk
#def dhom2
  (A : U)              -- The base type.
  (x y z : A)            -- Three points in the base.
  (f : hom A x y)          -- An arrow in the base.
  (g : hom A y z)          -- An arrow in the base.
  (h : hom A x z)          -- An arrow in the base.
  (alpha : hom2 A x y z f g h)  -- A composition witness in the base.
  (C : A → U)          -- A type family.
  (u : C x)            -- A lift of the initial point.
  (v : C y)            -- A lift of the second point.
  (w : C z)            -- A lift of the third point.
  (ff : dhom A x y f C u v)    -- A lift of the first arrow.
  (gg : dhom A y z g C v w)    -- A lift of the second arrow.
  (hh : dhom A x z h C u w)    -- A lift of the diagonal arrow.
  : U
  := ((t1 , t2) : Δ²) → C (alpha (t1 , t2)) [
        t2 ≡ 0₂ ↦ ff t1 ,
        t1 ≡ 1₂ ↦ gg t2 ,
        t2 ≡ t1 ↦ hh t2
      ]
```

## Covariant families

A family of types over a base type is covariant if every arrow in the base has a
unique lift with specified domain.

```rzk title="RS17, Definition 8.2"
#def is-covariant
  (A : U)
  (C : A → U)
  : U
  := (x : A) → (y : A) → (f : hom A x y) → (u : C x)
    → is-contr (dhom-from A x y f C u)

-- Type of covariant families over a fixed type
#def covariant-family (A : U) : U
  := (Σ (C : ((a : A) → U)) , is-covariant A C)
```

The notion of having a unique lift with a fixed domain may also be expressed by
contractibility of the type of extensions along the domain inclusion into the
1-simplex.

```rzk
#def has-unique-lifts-with-fixed-domain
  (A : U)
  (C : A → U)
  : U
  := (x : A) → (y : A) → (f : hom A x y) → (u : C x)
    → is-contr ((t : Δ¹) → C (f t) [ t ≡ 0₂ ↦ u ])
```

These two notions of covariance are equivalent because the two types of lifts of
a base arrow with fixed domain are equivalent. Note that this is not quite an
instance of Theorem 4.4 but its proof, with a very small modification, works
here.

```rzk
#def equiv-lifts-with-fixed-domain
  (A : U)
  (C : A → U)
  (x y : A)
  (f : hom A x y)
  (u : C x)
  : Equiv
      ((t : Δ¹) → C (f t) [ t ≡ 0₂ ↦ u ])
      (dhom-from A x y f C u)
  :=
    ( \ h → (h 1₂ , \ t → h t) ,
      ( ( \ fg t → (second fg) t , \ h → refl) ,
        ( ( \ fg t → (second fg) t , \ h → refl))))
```

By the equivalence-invariance of contractibility, this proves the desired
logical equivalence

```rzk title="RS17, Proposition 8.4"
#def has-unique-lifts-with-fixed-domain-iff-is-covariant
  (A : U)
  (C : A → U)
  : iff
      (has-unique-lifts-with-fixed-domain A C)
      (is-covariant A C)
  :=
    ( \ C-has-unique-lifts x y f u →
      is-contr-is-equiv-from-contr
        ( (t : Δ¹) → C (f t) [ t ≡ 0₂ ↦ u ])
        ( dhom-from A x y f C u)
        ( equiv-lifts-with-fixed-domain A C x y f u)
        ( C-has-unique-lifts x y f u),
      \ C-is-covariant x y f u →
      is-contr-is-equiv-to-contr
        ( (t : Δ¹) → C (f t) [ t ≡ 0₂ ↦ u ])
        ( dhom-from A x y f C u)
        ( equiv-lifts-with-fixed-domain A C x y f u)
        ( C-is-covariant x y f u))
```

## Representable covariant families

If A is a Segal type and a : A is any term, then hom A a defines a covariant
family over A, and conversely if this family is covariant for every a : A, then
A must be a Segal type. The proof involves a rather lengthy composition of
equivalences.

```rzk
#def dhom-representable
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  (v : hom A a y)        -- A lift of the codomain.
  : U
  := dhom A x y f (\ z → hom A a z) u v

-- By uncurrying (RS 4.2) we have an equivalence:
#def uncurried-dhom-representable
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  (v : hom A a y)        -- A lift of the codomain.
  : Equiv (dhom-representable A a x y f u v)
  (((t , s) : Δ¹×Δ¹) → A
    [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
      (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
      (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ])
  := curry-uncurry 2 2 Δ¹ ∂Δ¹ Δ¹ ∂Δ¹ (\ t s → A)
    (\ (t , s) → recOR
      ( (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
        (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
        (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
        (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ))

#def dhom-from-representable
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : U
  := dhom-from A x y f (\ z → hom A a z) u

-- By uncurrying (RS 4.2) we have an equivalence:
#def uncurried-dhom-from-representable
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv (dhom-from-representable A a x y f u)
    (Σ (v : hom A a y) , (((t , s) : Δ¹×Δ¹) → A
      [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
        (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
        (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
        (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
  := total-equiv-family-equiv (hom A a y) (\ v → dhom-representable A a x y f u v)
    (\ v → (((t , s) : Δ¹×Δ¹) → A
      [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
        (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
        (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
        (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    (\ v → uncurried-dhom-representable A a x y f u v)

#def square-to-hom2-pushout
  (A : U)
  (w x y z : A)
  (u : hom A w x)
  (f : hom A x z)
  (g : hom A w y)
  (v : hom A y z)
  : (((t , s) : Δ¹×Δ¹) → A [(t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ g t ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ])
  → (Σ (d : hom A w z) , product (hom2 A w x z u f d) (hom2 A w y z g v d))
  := \ sq → ((\ t → sq (t , t)) , (\ (t , s) → sq (s , t) , \ (t , s) → sq (t , s)))

#def hom2-pushout-to-square
  (A : U)
  (w x y z : A)
  (u : hom A w x)
  (f : hom A x z)
  (g : hom A w y)
  (v : hom A y z)
  : (Σ (d : hom A w z) , product (hom2 A w x z u f d) (hom2 A w y z g v d))
  → (((t , s) : Δ¹×Δ¹) → A [(t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ g t ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ])
  := \ (d , (alpha1 , alpha2)) (t , s) → recOR (t ≤ s ↦ alpha1 (s , t) ,
                        s ≤ t ↦ alpha2 (t , s))
#def Eq-square-hom2-pushout
  (A : U)
  (w x y z : A)
  (u : hom A w x)
  (f : hom A x z)
  (g : hom A w y)
  (v : hom A y z)
  : Equiv (((t , s) : Δ¹×Δ¹) → A [(t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ g t ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ])
    (Σ (d : hom A w z) , product (hom2 A w x z u f d) (hom2 A w y z g v d))
  := (square-to-hom2-pushout A w x y z u f g v ,
    ((hom2-pushout-to-square A w x y z u f g v , \ sq → refl) ,
    (hom2-pushout-to-square A w x y z u f g v , \ alphas → refl)))

#def representable-dhom-from-uncurry-hom2
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv (Σ (v : hom A a y) , (((t , s) : Δ¹×Δ¹) → A [(t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    (Σ (v : hom A a y) , (Σ (d : hom A a y) , product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))
  := total-equiv-family-equiv (hom A a y)
    (\ v → (((t , s) : Δ¹×Δ¹) → A [(t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    (\ v → (Σ (d : hom A a y) , product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))
    (\ v → Eq-square-hom2-pushout A a x a y u f (id-arr A a) v)

#def representable-dhom-from-hom2
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv (dhom-from-representable A a x y f u)
    (Σ (d : hom A a y) , (Σ (v : hom A a y) , product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))
  := triple-comp-equiv
    (dhom-from-representable A a x y f u)
    (Σ (v : hom A a y) , (((t , s) : Δ¹×Δ¹) → A [(t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    (Σ (v : hom A a y) , (Σ (d : hom A a y) , product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))
    (Σ (d : hom A a y) , (Σ (v : hom A a y) , product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))
    (uncurried-dhom-from-representable A a x y f u)
    (representable-dhom-from-uncurry-hom2 A a x y f u)
    (fubini-Σ (hom A a y) (hom A a y)
      (\ v d → product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))

#def representable-dhom-from-hom2-dist
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv (dhom-from-representable A a x y f u)
    (Σ (d : hom A a y) , (product (hom2 A a x y u f d) (Σ (v : hom A a y) , hom2 A a a y (id-arr A a) v d)))
  := right-cancel-equiv
    (dhom-from-representable A a x y f u)
    (Σ (d : hom A a y) , (product (hom2 A a x y u f d) (Σ (v : hom A a y) , hom2 A a a y (id-arr A a) v d)))
    (Σ (d : hom A a y) , (Σ (v : hom A a y) , product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))
    (representable-dhom-from-hom2 A a x y f u)
    (total-equiv-family-equiv (hom A a y)
      (\ d → (product (hom2 A a x y u f d) (Σ (v : hom A a y) , hom2 A a a y (id-arr A a) v d)))
      (\ d → (Σ (v : hom A a y) , product (hom2 A a x y u f d) (hom2 A a a y (id-arr A a) v d)))
      (\ d → (distributive-product-Σ (hom2 A a x y u f d) (hom A a y) (\ v → hom2 A a a y (id-arr A a) v d))))
```

Now we introduce the hypothesis that A is Segal type.

```rzk
#def Segal-representable-dhom-from-path-space
  (A : U)            -- The ambient type.
  (is-segal-A : is-segal A)    -- A proof that A is a Segal type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv (dhom-from-representable A a x y f u)
    (Σ (d : hom A a y) , (product (hom2 A a x y u f d) (Σ (v : hom A a y) , (v = d))))
  := right-cancel-equiv
    (dhom-from-representable A a x y f u)
    (Σ (d : hom A a y) , (product (hom2 A a x y u f d) (Σ (v : hom A a y) , (v = d))))
    (Σ (d : hom A a y) , (product (hom2 A a x y u f d) (Σ (v : hom A a y) , hom2 A a a y (id-arr A a) v d)))
    (representable-dhom-from-hom2-dist A a x y f u)
    (total-equiv-family-equiv (hom A a y)
      (\ d → (product (hom2 A a x y u f d) (Σ (v : hom A a y) , (v = d))))
      (\ d → (product (hom2 A a x y u f d) (Σ (v : hom A a y) , hom2 A a a y (id-arr A a) v d)))
      (\ d → (total-equiv-family-equiv (hom2 A a x y u f d)
        (\ alpha → (Σ (v : hom A a y) , (v = d)))
        (\ alpha → (Σ (v : hom A a y) , hom2 A a a y (id-arr A a) v d))
        (\ alpha → (total-equiv-family-equiv (hom A a y)
          (\ v → (v = d))
          (\ v → hom2 A a a y (id-arr A a) v d)
          (\ v → (Eq-Segal-homotopy-hom2 A is-segal-A a y v d)))))))


#def codomain-based-paths-contraction
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  (d : hom A a y)
  : Equiv (product (hom2 A a x y u f d) (Σ (v : hom A a y) , (v = d))) (hom2 A a x y u f d)
  := equiv-projection-contractible-fibers (hom2 A a x y u f d) (\ alpha → (Σ (v : hom A a y) , (v = d)))
    (\ alpha → is-contr-codomain-based-paths (hom A a y) d)

#def is-segal-representable-dhom-from-hom2
  (A : U)            -- The ambient type.
  (is-segal-A : is-segal A)    -- A proof that A is a Segal type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv (dhom-from-representable A a x y f u)
    (Σ (d : hom A a y) , (hom2 A a x y u f d))
  := comp-equiv
    (dhom-from-representable A a x y f u)
    (Σ (d : hom A a y) , (product (hom2 A a x y u f d) (Σ (v : hom A a y) , (v = d))))
    (Σ (d : hom A a y) , (hom2 A a x y u f d))
    (Segal-representable-dhom-from-path-space A is-segal-A a x y f u)
    (total-equiv-family-equiv (hom A a y)
      (\ d → product (hom2 A a x y u f d) (Σ (v : hom A a y) , (v = d)))
      (\ d → hom2 A a x y u f d)
      (\ d → codomain-based-paths-contraction A a x y f u d))

#def is-segal-representable-dhom-from-contractible
  (A : U)            -- The ambient type.
  (is-segal-A : is-segal A)    -- A proof that A is a Segal type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : is-contr (dhom-from-representable A a x y f u)
  := is-contr-is-equiv-to-contr (dhom-from-representable A a x y f u)
        (Σ (d : hom A a y) , (hom2 A a x y u f d))
        (is-segal-representable-dhom-from-hom2 A is-segal-A a x y f u)
        (is-segal-A a x y u f)
```

Finally, we see that covariant hom families in a Segal type are covariant.

```rzk title="RS17, Proposition 8.13(<-)"
#def is-segal-representable-is-covariant
  (A : U)
  (is-segal-A : is-segal A)
  (a : A)
  : is-covariant A (\ x → hom A a x)
  := \ x y f u → is-segal-representable-dhom-from-contractible A is-segal-A a x y f u
```

The proof of the claimed converse result given in the original source is
circular - using Proposition 5.10, which holds only for Segal types - so instead
we argue as follows:

```rzk title="RS17, Proposition 8.13(→)"
#def representable-is-covariant-is-segal
  (A : U)
  (repiscovfam : (a : A) → is-covariant A (\ x → hom A a x))
  : is-segal A
  := \ x y z f g → is-contr-base-is-contr-Σ
    (Σ (h : hom A x z) , hom2 A x y z f g h)
    (\ hk → Σ (v : hom A x z) , hom2 A x x z (id-arr A x) v (first hk))
    (\ hk → (first hk , \ (t , s) → first hk s))
    (is-contr-is-equiv-to-contr
      (Σ (hk : Σ (h : hom A x z) , hom2 A x y z f g h) , Σ (v : hom A x z) , hom2 A x x z (id-arr A x) v (first hk))
      (dhom-from-representable A x y z g f)
      (inv-equiv (dhom-from-representable A x y z g f)
        (Σ (hk : Σ (h : hom A x z) , hom2 A x y z f g h) , Σ (v : hom A x z) , hom2 A x x z (id-arr A x) v (first hk))
        (comp-equiv
          (dhom-from-representable A x y z g f)
          (Σ (h : hom A x z) , (product (hom2 A x y z f g h) (Σ (v : hom A x z) , hom2 A x x z (id-arr A x) v h)))
          (Σ (hk : Σ (h : hom A x z) , hom2 A x y z f g h) , Σ (v : hom A x z) , hom2 A x x z (id-arr A x) v (first hk))
          (representable-dhom-from-hom2-dist A x y z g f)
          (associative-Σ
            (hom A x z)
            (\ h → hom2 A x y z f g h)
            (\ h _ → Σ (v : hom A x z) , hom2 A x x z (id-arr A x) v h))))
      (repiscovfam x y z g f))
```

While not needed to prove Proposition 8.13, it is interesting to observe that
the dependent hom types in a representable family can be understood as extension
types as follows.

```rzk
#def cofibration-union-test
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv
    ( ((t , s) : ∂□) → A
      [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ])
    ( ((t , s) : 2 × 2 | (t ≡ 1₂) ∧ (Δ¹ s)) → A
      [ (t ≡ 1₂) ∧ (s ≡ 0₂) ↦ a ,
        (t ≡ 1₂) ∧ (s ≡ 1₂) ↦ y ])
  :=
    cofibration-union (2 × 2)
    ( \ (t , s) → (t ≡ 1₂) ∧ Δ¹ s)
    ( \ (t , s) →
      (t ≡ 0₂) ∧ (Δ¹ s) ∨ (Δ¹ t) ∧ (s ≡ 0₂) ∨ (Δ¹ t) ∧ (s ≡ 1₂))
    ( \ (t , s) → A)
    ( \ (t , s) →
      recOR
        ( (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
          (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
          (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t))

#def base-hom-rewriting
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv
    (((t , s) : 2 × 2 | (t ≡ 1₂) ∧ (Δ¹ s)) → A
      [ (t ≡ 1₂) ∧ (s ≡ 0₂) ↦ a ,
        (t ≡ 1₂) ∧ (s ≡ 1₂) ↦ y ])
    (hom A a y)
  :=
    ( \ v → (\ r → v ((1₂ , r))) ,
      ( ( \ v → \ (t , s) → v s ,
          \ v → refl) ,
        ( \ v → \ (t , s) → v s ,
          \ v → refl)))

#def base-hom-expansion
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv
    ( ((t , s) : ∂□) → A
      [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
        (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
        (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t])
    (hom A a y)
  :=
    comp-equiv
    ( ((t , s) : ∂□) → A
        [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
          (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
          (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ] )
    ( ((t , s) : 2 × 2 | (t ≡ 1₂) ∧ (Δ¹ s)) → A
      [ (t ≡ 1₂) ∧ (s ≡ 0₂) ↦ a ,
        (t ≡ 1₂) ∧ (s ≡ 1₂) ↦ y ])
    ( hom A a y)
    ( cofibration-union-test A a x y f u)
    ( base-hom-rewriting A a x y f u)

#def representable-dhom-from-expansion
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv
    (Σ (sq : ((t , s) : ∂□) → A
        [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
          (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
          (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]) ,
        (((t , s) : Δ¹×Δ¹) → A
          [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (t ≡ 1₂) ∧ (Δ¹ s) ↦ (sq (1₂ , s)) ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    (Σ (v : hom A a y) ,
      (((t , s) : Δ¹×Δ¹) → A
          [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
  :=
    total-equiv-pullback-is-equiv
      ( ((t , s) : ∂□) → A
        [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
          (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
          (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ] )
      (hom A a y)
      (first (base-hom-expansion A a x y f u))
      (second (base-hom-expansion A a x y f u))
      ( \ v →
        ( ((t , s) : Δ¹×Δ¹) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))

#def representable-dhom-from-composite-expansion
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv (dhom-from-representable A a x y f u)
      (Σ (sq : ((t , s) : ∂□) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]) ,
        ( ((t , s) : Δ¹×Δ¹) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (t ≡ 1₂) ∧ (Δ¹ s) ↦ (sq (1₂ , s)) ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
  :=
    right-cancel-equiv
    ( dhom-from-representable A a x y f u)
    ( Σ (sq : ((t , s) : ∂□) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]) ,
        ( ((t , s) : Δ¹×Δ¹) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (t ≡ 1₂) ∧ (Δ¹ s) ↦ (sq (1₂ , s)) ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    ( Σ (v : hom A a y) , (((t , s) : Δ¹×Δ¹) → A
        [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
          (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
          (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
          (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    ( uncurried-dhom-from-representable A a x y f u)
    ( representable-dhom-from-expansion A a x y f u)

#def representable-dhom-from-cofibration-composition
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv
    (((t , s) : Δ¹×Δ¹) → A
        [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
          (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
          (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t] )
    (Σ (sq : ((t , s) : ∂□) → A
        [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]) ,
        (((t , s) : Δ¹×Δ¹) → A
          [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (t ≡ 1₂) ∧ (Δ¹ s) ↦ (sq (1₂ , s)) ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
  :=
    cofibration-composition (2 × 2) Δ¹×Δ¹ ∂□
    ( \ (t , s) →
      (t ≡ 0₂) ∧ (Δ¹ s) ∨ (Δ¹ t) ∧ (s ≡ 0₂) ∨ (Δ¹ t) ∧ (s ≡ 1₂))
    ( \ ts → A)
    ( \ (t , s) →
      recOR ( (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t))

#def representable-dhom-from-as-extension-type
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A a x)        -- A lift of the domain.
  : Equiv
      (dhom-from-representable A a x y f u)
      (((t , s) : Δ¹×Δ¹) → A
              [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
                (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t] )
  :=
    right-cancel-equiv
    ( dhom-from-representable A a x y f u)
    ( ((t , s) : Δ¹×Δ¹) → A
        [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
          (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
          (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t] )
    ( Σ (sq : ((t , s) : ∂□) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]) ,
        (((t , s) : Δ¹×Δ¹) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (t ≡ 1₂) ∧ (Δ¹ s) ↦ (sq (1₂ , s)) ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ a ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ f t ]))
    ( representable-dhom-from-composite-expansion A a x y f u)
    ( representable-dhom-from-cofibration-composition A a x y f u)
```

## Covariant lifts, transport, and uniqueness

In a covariant family C over a base type A , a term u : C x may be transported
along an arrow f : hom A x y to give a term in C y.

```rzk title="RS17, covariant transport from beginning of Section 8.2"
#def covariant-transport
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C : A → U)
  (is-covariant-C : is-covariant A C)
  (u : C x)
   : C y
   := first (contraction-center (dhom-from A x y f C u) (is-covariant-C x y f u))
```

```rzk title="RS17, covariant lift from beginning of Section 8.2"
#def covariant-lift
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C : A → U)
  (is-covariant-C : is-covariant A C)
  (u : C x)
  : (dhom A x y f C u (covariant-transport A x y f C is-covariant-C u))
   := second (contraction-center (dhom-from A x y f C u) (is-covariant-C x y f u))

#def covariant-uniqueness
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C : A → U)
  (is-covariant-C : is-covariant A C)
  (u : C x)
  (lift : dhom-from A x y f C u)
  : (covariant-transport A x y f C is-covariant-C u) = (first lift)
  :=
    first-path-Σ
    ( C y)
    ( \ v → dhom A x y f C u v)
    ( contraction-center (dhom-from A x y f C u) (is-covariant-C x y f u))
    ( lift)
    ( contracting-htpy (dhom-from A x y f C u) (is-covariant-C x y f u) lift)
```

## Covariant functoriality

The covariant transport operation defines a covariantly functorial action of
arrows in the base on terms in the fibers. In particular, there is an identity
transport law.

```rzk
#def d-id-arr
  (A : U)
  (x : A)
  (C : A → U)
  (u : C x)
  : dhom A x x (id-arr A x) C u u
  := \ t → u
```

```rzk title="RS17, Proposition 8.16, Part 2"
-- Covariant families preserve identities
#def id-arr-covariant-transport
  (A : U)
  (x : A)
  (C : A → U)
  (is-covariant-C : is-covariant A C)
  (u : C x)
  : (covariant-transport A x x (id-arr A x) C is-covariant-C u) = u
  :=
    covariant-uniqueness
      A x x (id-arr A x) C is-covariant-C u (u , d-id-arr A x C u)
```

## Natural transformations

A fiberwise map between covariant families is automatically "natural" commuting
with the covariant lifts.

```rzk title="RS17, Proposition 8.17"
-- Covariant naturality
#def covariant-fiberwise-transformation-application
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C D : A → U)
  (is-covariant-C : is-covariant A C)
  (ϕ : (z : A) → C z → D z)
  (u : C x)
  : dhom-from A x y f D (ϕ x u)
  := (ϕ y (covariant-transport A x y f C is-covariant-C u) ,
  \ t → ϕ (f t) (covariant-lift A x y f C is-covariant-C u t))

#def naturality-covariant-fiberwise-transformation
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C D : A → U)
  (is-covariant-C : is-covariant A C)
  (DisCov : is-covariant A D)
  (ϕ : (z : A) → C z → D z)
  (u : C x)
  : (covariant-transport A x y f D DisCov (ϕ x u)) =
    (ϕ y (covariant-transport A x y f C is-covariant-C u))
  :=
    covariant-uniqueness A x y f D DisCov (ϕ x u)
      ( covariant-fiberwise-transformation-application
          A x y f C D is-covariant-C ϕ u)
```

## Contravariant families

A family of types over a base type is contravariant if every arrow in the base
has a unique lift with specified codomain.

```rzk
#def dhom-to
  (A : U)            -- The base type.
  (x y : A)          -- Two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (C : A → U)        -- A type family.
  (v : C y)          -- A lift of the domain.
  : U
     := (Σ (u : C x) , dhom A x y f C u v)
```

```rzk title="RS17, Definition 8.2, dual form"
#def is-contravariant
  (A : U)
  (C : A → U)
  : U
  := (x : A) → (y : A) → (f : hom A x y) → (v : C y)
    → is-contr (dhom-to A x y f C v)

-- Type of contravariant families over a fixed type
#def contravariant-family (A : U) : U
  := (Σ (C : A → U) , is-contravariant A C)
```

The notion of having a unique lift with a fixed codomain may also be expressed
by contractibility of the type of extensions along the codomain inclusion into
the 1-simplex.

```rzk
#def has-unique-lifts-with-fixed-codomain
  (A : U)
  (C : A → U)
  : U
  := (x : A) → (y : A) → (f : hom A x y) → (v : C y)
    → is-contr ((t : Δ¹) → C (f t) [ t ≡ 1₂ ↦ v ])
```

These two notions of covariance are equivalent because the two types of lifts of
a base arrow with fixed codomain are equivalent. Note that this is not quite an
instance of Theorem 4.4 but its proof, with a very small modification, works
here.

```rzk
#def equiv-lifts-with-fixed-codomain
  (A : U)
  (C : A → U)
  (x y : A)
  (f : hom A x y)
  (v : C y)
  : Equiv
      ((t : Δ¹) → C (f t) [ t ≡ 1₂ ↦ v ])
      (dhom-to A x y f C v)
  :=
    ( \ h → (h 0₂ , \ t → h t) ,
      ( ( \ fg t → (second fg) t , \ h → refl) ,
        ( ( \ fg t → (second fg) t , \ h → refl))))
```

By the equivalence-invariance of contractibility, this proves the desired
logical equivalence

```rzk title="RS17, Proposition 8.4"
#def has-unique-lifts-with-fixed-codomain-iff-is-contravariant
  (A : U)
  (C : A → U)
  : iff
      (has-unique-lifts-with-fixed-codomain A C)
      (is-contravariant A C)
  :=
    ( \ C-has-unique-lifts x y f v →
      is-contr-is-equiv-from-contr
        ( (t : Δ¹) → C (f t) [ t ≡ 1₂ ↦ v ])
        ( dhom-to A x y f C v)
        ( equiv-lifts-with-fixed-codomain A C x y f v)
        ( C-has-unique-lifts x y f v),
      \ is-contravariant-C x y f v →
      is-contr-is-equiv-to-contr
        ( (t : Δ¹) → C (f t) [ t ≡ 1₂ ↦ v ])
        ( dhom-to A x y f C v)
        ( equiv-lifts-with-fixed-codomain A C x y f v)
        ( is-contravariant-C x y f v))
```

## Representable contravariant families

If A is a Segal type and a : A is any term, then the family \ x → hom A x a
defines a contravariant family over A , and conversely if this family is
contravariant for every a : A , then A must be a Segal type. The proof involves
a rather lengthy composition of equivalences.

```rzk
#def dhom-contra-representable
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A x a)        -- A lift of the domain.
  (v : hom A y a)        -- A lift of the codomain.
  : U
  := dhom A x y f (\ z → hom A z a) u v

-- By uncurrying (RS 4.2) we have an equivalence:
#def uncurried-dhom-contra-representable
  (A : U)            -- The ambient type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (u : hom A x a)        -- A lift of the domain.
  (v : hom A y a)        -- A lift of the codomain.
  : Equiv (dhom-contra-representable A a x y f u v)
  (((t , s) : Δ¹×Δ¹) → A [(t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ f t ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ a ])
  := curry-uncurry 2 2 Δ¹ ∂Δ¹ Δ¹ ∂Δ¹ (\ t s → A)
    (\ (t , s) → recOR ((t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ f t ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ a ))

#def dhom-to-representable
  (A : U)            -- The ambient type.
  (a x y : A)        -- The representing object and two points in the base.
  (f : hom A x y)    -- An arrow in the base.
  (v : hom A y a)    -- A lift of the codomain.
  : U
  := dhom-to A x y f (\ z → hom A z a) v

-- By uncurrying (RS 4.2) we have an equivalence:
#def uncurried-dhom-to-representable
  (A : U)            -- The ambient type.
  (a x y : A)        -- The representing object and two points in the base.
  (f : hom A x y)    -- An arrow in the base.
  (v : hom A y a)    -- A lift of the codomain.
  : Equiv (dhom-to-representable A a x y f v)
    (Σ (u : hom A x a) ,
        (((t , s) : Δ¹×Δ¹) → A
                    [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ f t ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ a ]))
  :=
    total-equiv-family-equiv
    ( hom A x a)
    ( \ u → dhom-contra-representable A a x y f u v)
    ( \ u →
      (((t , s) : Δ¹×Δ¹) → A
          [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
            (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
            (Δ¹ t) ∧ (s ≡ 0₂) ↦ f t ,
            (Δ¹ t) ∧ (s ≡ 1₂) ↦ a ]))
    ( \ u → uncurried-dhom-contra-representable A a x y f u v)

#def representable-dhom-to-uncurry-hom2
  (A : U)            -- The ambient type.
  (a x y : A)        -- The representing object and two points in the base.
  (f : hom A x y)    -- An arrow in the base.
  (v : hom A y a)    -- A lift of the codomain.
  : Equiv
    (Σ (u : hom A x a) , (((t , s) : Δ¹×Δ¹) → A
                    [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ f t ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ a ]))
    (Σ (u : hom A x a) ,
      (Σ (d : hom A x a) ,
        product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d) ))
  :=
    total-equiv-family-equiv (hom A x a)
    ( \ u →
      (((t , s) : Δ¹×Δ¹) → A
                    [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
                      (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
                      (Δ¹ t) ∧ (s ≡ 0₂) ↦ f t ,
                      (Δ¹ t) ∧ (s ≡ 1₂) ↦ a ]))
    ( \ u →
      (Σ (d : hom A x a) ,
          product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d) ))
    ( \ u → Eq-square-hom2-pushout A x a y a u (id-arr A a) f v)

#def representable-dhom-to-hom2
  (A : U)          -- The ambient type.
  (a x y : A)      -- The representing object and two points in the base.
  (f : hom A x y)  -- An arrow in the base.
  (v : hom A y a)  -- A lift of the codomain.
  : Equiv
    (dhom-to-representable A a x y f v)
    (Σ (d : hom A x a) ,
      (Σ (u : hom A x a) ,
        product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d) ))
  :=
    triple-comp-equiv
    ( dhom-to-representable A a x y f v)
    ( Σ (u : hom A x a) ,
        (((t , s) : Δ¹×Δ¹) → A
            [ (t ≡ 0₂) ∧ (Δ¹ s) ↦ u s ,
              (t ≡ 1₂) ∧ (Δ¹ s) ↦ v s ,
              (Δ¹ t) ∧ (s ≡ 0₂) ↦ f t ,
              (Δ¹ t) ∧ (s ≡ 1₂) ↦ a ]))
    ( Σ (u : hom A x a ) ,
      (Σ (d : hom A x a) ,
        product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d)))
    ( Σ (d : hom A x a ) ,
      (Σ (u : hom A x a) ,
        product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d)))
    ( uncurried-dhom-to-representable A a x y f v)
    ( representable-dhom-to-uncurry-hom2 A a x y f v)
    ( fubini-Σ (hom A x a) (hom A x a)
      (\ u d → product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d)))

#def representable-dhom-to-hom2-swap
  (A : U)          -- The ambient type.
  (a x y : A)      -- The representing object and two points in the base.
  (f : hom A x y)  -- An arrow in the base.
  (v : hom A y a)  -- A lift of the codomain.
  : Equiv
    ( dhom-to-representable A a x y f v)
    ( Σ (d : hom A x a) , (Σ (u : hom A x a) , product (hom2 A x y a f v d) (hom2 A x a a u (id-arr A a) d) ))
  := comp-equiv
      (dhom-to-representable A a x y f v)
      (Σ (d : hom A x a) , (Σ (u : hom A x a) , product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d) ))
      (Σ (d : hom A x a) , (Σ (u : hom A x a) , product (hom2 A x y a f v d) (hom2 A x a a u (id-arr A a) d) ))
      (representable-dhom-to-hom2 A a x y f v)
      (total-equiv-family-equiv (hom A x a)
        (\ d → (Σ (u : hom A x a) , product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d) ))
        (\ d → (Σ (u : hom A x a) , product (hom2 A x y a f v d) (hom2 A x a a u (id-arr A a) d) ))
        (\ d → total-equiv-family-equiv (hom A x a)
          (\ u → product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d))
          (\ u → product (hom2 A x y a f v d) (hom2 A x a a u (id-arr A a) d))
          (\ u → sym-product (hom2 A x a a u (id-arr A a) d) (hom2 A x y a f v d))))

#def representable-dhom-to-hom2-dist
  (A : U)            -- The ambient type.
  (a x y : A)        -- The representing object and two points in the base.
  (f : hom A x y)    -- An arrow in the base.
  (v : hom A y a)    -- A lift of the codomain.
  : Equiv (dhom-to-representable A a x y f v)
    (Σ (d : hom A x a ) , (product (hom2 A x y a f v d)
      (Σ (u : hom A x a ) , hom2 A x a a u (id-arr A a) d)))
  := right-cancel-equiv
    (dhom-to-representable A a x y f v)
    (Σ (d : hom A x a ) , (product (hom2 A x y a f v d)
      (Σ (u : hom A x a ) , hom2 A x a a u (id-arr A a) d)))
    (Σ (d : hom A x a) , (Σ (u : hom A x a) , product (hom2 A x y a f v d) (hom2 A x a a u (id-arr A a) d)))
    (representable-dhom-to-hom2-swap A a x y f v)
    (total-equiv-family-equiv (hom A x a)
      (\ d → (product (hom2 A x y a f v d)
      (Σ (u : hom A x a ) , hom2 A x a a u (id-arr A a) d)))
      (\ d → (Σ (u : hom A x a) , product (hom2 A x y a f v d) (hom2 A x a a u (id-arr A a) d) ))
      (\ d → (distributive-product-Σ (hom2 A x y a f v d) (hom A x a) (\ u → hom2 A x a a u (id-arr A a) d))))
```

Now we introduce the hypothesis that A is Segal type.

```rzk
#def Segal-representable-dhom-to-path-space
  (A : U)            -- The ambient type.
  (is-segal-A : is-segal A)    -- A proof that A is a Segal type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (v : hom A y a)        -- A lift of the codomain.
  : Equiv (dhom-to-representable A a x y f v)
    (Σ (d : hom A x a) , (product (hom2 A x y a f v d) (Σ (u : hom A x a) , (u = d))))
  := right-cancel-equiv
    (dhom-to-representable A a x y f v)
    (Σ (d : hom A x a) , (product (hom2 A x y a f v d) (Σ (u : hom A x a) , (u = d))))
    (Σ (d : hom A x a) , (product (hom2 A x y a f v d) (Σ (u : hom A x a) , (hom2 A x a a u (id-arr A a) d))))
    (representable-dhom-to-hom2-dist A a x y f v)
    (total-equiv-family-equiv (hom A x a)
      (\ d → (product (hom2 A x y a f v d) (Σ (u : hom A x a) , (u = d))))
      (\ d → (product (hom2 A x y a f v d) (Σ (u : hom A x a) , hom2 A x a a u (id-arr A a) d)))
      (\ d → (total-equiv-family-equiv (hom2 A x y a f v d)
        (\ α → (Σ (u : hom A x a) , (u = d)))
        (\ α → (Σ (u : hom A x a) , hom2 A x a a u (id-arr A a) d))
        (\ α → (total-equiv-family-equiv (hom A x a)
          (\ u → (u = d))
          (\ u → hom2 A x a a u (id-arr A a) d)
          (\ u → (Eq-Segal-homotopy-hom2' A is-segal-A x a u d)))))))

#def is-segal-representable-dhom-to-hom2
  (A : U)            -- The ambient type.
  (is-segal-A : is-segal A)    -- A proof that A is a Segal type.
  (a x y : A)          -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (v : hom A y a)        -- A lift of the codomain.
  : Equiv (dhom-to-representable A a x y f v)
    (Σ (d : hom A x a) , (hom2 A x y a f v d))
  := comp-equiv
    (dhom-to-representable A a x y f v)
    (Σ (d : hom A x a) , (product (hom2 A x y a f v d) (Σ (u : hom A x a) , (u = d))))
    (Σ (d : hom A x a) , (hom2 A x y a f v d))
    (Segal-representable-dhom-to-path-space A is-segal-A a x y f v)
    (total-equiv-family-equiv (hom A x a)
      (\ d → product (hom2 A x y a f v d) (Σ (u : hom A x a) , (u = d)))
      (\ d → hom2 A x y a f v d)
      (\ d → codomain-based-paths-contraction A x y a v f d))

#def is-segal-representable-dhom-to-contractible
  (A : U)                -- The ambient type.
  (is-segal-A : is-segal A)    -- A proof that A is a Segal type.
  (a x y : A)            -- The representing object and two points in the base.
  (f : hom A x y)        -- An arrow in the base.
  (v : hom A y a)        -- A lift of the codomain.
  : is-contr (dhom-to-representable A a x y f v)
  := is-contr-is-equiv-to-contr (dhom-to-representable A a x y f v)
        (Σ (d : hom A x a) , (hom2 A x y a f v d))
        (is-segal-representable-dhom-to-hom2 A is-segal-A a x y f v)
        (is-segal-A x y a f v)
```

Finally, we see that contravariant hom families in a Segal type are
contravariant.

```rzk title="RS17, Proposition 8.13(<-), dual"
#def is-segal-representable-is-contravariant
  (A : U)
  (is-segal-A : is-segal A)
  (a : A)
  : is-contravariant A (\ x → hom A x a)
  := \ x y f v → is-segal-representable-dhom-to-contractible A is-segal-A a x y f v
```

The proof of the claimed converse result given in the original source is
circular - using Proposition 5.10, which holds only for Segal types - so instead
we argue as follows:

```rzk title="RS17, Proposition 8.13(→), dual"
#def representable-is-contravariant-is-segal
  (A : U)
  (repiscontrafam : (a : A) → is-contravariant A (\ x → hom A x a))
  : is-segal A
  := \ x y z f g → is-contr-base-is-contr-Σ
    (Σ (h : hom A x z) , hom2 A x y z f g h)
    (\ hk → Σ (u : hom A x z) , hom2 A x z z u (id-arr A z) (first hk))
    (\ hk → (first hk , \ (t , s) → first hk t))
    (is-contr-is-equiv-to-contr
      (Σ (hk : Σ (h : hom A x z) , hom2 A x y z f g h) , Σ (u : hom A x z) , hom2 A x z z u (id-arr A z) (first hk))
      (dhom-to-representable A z x y f g)
      (inv-equiv (dhom-to-representable A z x y f g)
        (Σ (hk : Σ (h : hom A x z) , hom2 A x y z f g h) , Σ (u : hom A x z) , hom2 A x z z u (id-arr A z) (first hk))
        (comp-equiv
          (dhom-to-representable A z x y f g)
          (Σ (h : hom A x z) , (product (hom2 A x y z f g h) (Σ (u : hom A x z) , hom2 A x z z u (id-arr A z) h)))
          (Σ (hk : Σ (h : hom A x z) , hom2 A x y z f g h) , Σ (u : hom A x z) , hom2 A x z z u (id-arr A z) (first hk))
          (representable-dhom-to-hom2-dist A z x y f g)
          (associative-Σ
            (hom A x z)
            (\ h → hom2 A x y z f g h)
            (\ h _ → Σ (u : hom A x z) , hom2 A x z z u (id-arr A z) h))))
      (repiscontrafam z x y f g))
```

## Contravariant lifts, transport, and uniqueness

In a contravariant family C over a base type A, a term v : C y may be
transported along an arrow f : hom A x y to give a term in C x.

```rzk title="RS17, contravariant transport from beginning of Section 8.2"
#def contravariant-transport
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C : A → U)
  (is-contravariant-C : is-contravariant A C)
  (v : C y)
   : C x
   := first (contraction-center (dhom-to A x y f C v) (is-contravariant-C x y f v))
```

```rzk title="RS17, contravariant lift from beginning of Section 8.2"
#def contravariant-lift
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C : A → U)
  (is-contravariant-C : is-contravariant A C)
  (v : C y)
  : (dhom A x y f C (contravariant-transport A x y f C is-contravariant-C v) v)
  :=
    second
      (contraction-center (dhom-to A x y f C v) (is-contravariant-C x y f v))

#def contravariant-uniqueness
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C : A → U)
  (is-contravariant-C : is-contravariant A C)
  (v : C y)
  (lift : dhom-to A x y f C v)
  : (contravariant-transport A x y f C is-contravariant-C v) = (first lift)
  := first-path-Σ
    (C x)
    (\ u → dhom A x y f C u v)
    (contraction-center (dhom-to A x y f C v) (is-contravariant-C x y f v))
    lift
    (contracting-htpy (dhom-to A x y f C v) (is-contravariant-C x y f v) lift)
```

## Contravariant functoriality

The contravariant transport operation defines a comtravariantly functorial
action of arrows in the base on terms in the fibers. In particular, there is an
identity transport law.

```rzk title="RS17, Proposition 8.16, Part 2, dual"
-- Comtravariant families preserve identities
#def id-arr-contravariant-transport
   (A : U)
  (x : A)
   (C : A → U)
  (is-contravariant-C : is-contravariant A C)
  (u : C x)
  : (contravariant-transport A x x (id-arr A x) C is-contravariant-C u) = u
  := contravariant-uniqueness A x x (id-arr A x) C is-contravariant-C u (u , d-id-arr A x C u)
```

## Contravariant natural transformations

A fiberwise map between contrvariant families is automatically "natural"
commuting with the contravariant lifts.

```rzk title="RS17, Proposition 8.17, dual"
-- Contravariant naturality
#def contravariant-fiberwise-transformation-application
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C D : A → U)
  (is-contravariant-C : is-contravariant A C)
  (ϕ : (z : A) → C z → D z)
  (v : C y)
  : dhom-to A x y f D (ϕ y v)
  := (ϕ x (contravariant-transport A x y f C is-contravariant-C v) ,
  \ t → ϕ (f t) (contravariant-lift A x y f C is-contravariant-C v t))

#def naturality-contravariant-fiberwise-transformation
  (A : U)
  (x y : A)
  (f : hom A x y)
  (C D : A → U)
  (is-contravariant-C : is-contravariant A C)
  (DisContra : is-contravariant A D)
  (ϕ : (z : A) → C z → D z)
  (v : C y)
  : (contravariant-transport A x y f D DisContra (ϕ y v)) =
      (ϕ x (contravariant-transport A x y f C is-contravariant-C v))
  := contravariant-uniqueness A x y f D DisContra (ϕ y v)
    (contravariant-fiberwise-transformation-application A x y f C D is-contravariant-C ϕ v)
```
