# 8. Families of maps

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Fiber of total map

We now calculate the fiber of the map on total spaces associated to a family of
maps.

```rzk
#def total-map-family-of-maps
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))             -- a family of maps
  : (Σ (x : A) , B x) → (Σ (x : A) , C x)      -- the induced map on total spaces
  := \ z → (first z , f (first z) (second z))

#def total-map-to-fiber
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))             -- a family of maps
  (w : (Σ (x : A) , C x))
  : fib (B (first w)) (C (first w)) (f (first w)) (second w) →
    ( fib (Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f) w)
  :=
    \ (b , p) →
      ( (first w , b) ,
        eq-eq-fiber-Σ A C (first w) (f (first w) b) (second w) p)

#def total-map-from-fiber
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))             -- a family of maps
  (w : (Σ (x : A) , C x))
  : (fib (Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f) w)
    → fib (B (first w)) (C (first w)) (f (first w)) (second w)
  :=
    \ (z , p) →
      idJ
      ( (Σ (x : A) , C x) ,
        ((total-map-family-of-maps A B C f) z) ,
        \ w' p' → fib (B (first w')) (C (first w')) (f (first w')) (second w') , ((second z) , refl) ,
        w ,
        p)

#def total-map-to-fiber-retraction
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))
  (w : (Σ (x : A) , C x))
  : has-retraction
    ( fib (B (first w)) (C (first w)) (f (first w)) (second w))
    ( fib (Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f) w)
    ( total-map-to-fiber A B C f w)
  :=
    ( total-map-from-fiber A B C f w ,
      \ (b , p) →
        idJ
        ( (C (first w)) ,
          (f (first w) b) ,
          \ w1 p' →
            ((total-map-from-fiber A B C f ((first w , w1)))
              ((total-map-to-fiber A B C f (first w , w1)) (b , p')))
            =_{(fib (B (first w)) (C (first w)) (f (first w)) (w1))}
            (b , p') ,
          refl ,
          (second w) ,
          p))

#def total-map-to-fiber-section
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))             -- a family of maps
  (w : (Σ (x : A) , C x))
  : has-section
    ( fib (B (first w)) (C (first w)) (f (first w)) (second w))
    ( fib (Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f) w)
    ( total-map-to-fiber A B C f w)
  :=
    ( total-map-from-fiber A B C f w ,
      \ (z , p) →
        idJ
        ( (Σ (x : A) , C x) ,
          ((first z , f (first z) (second z))) ,
          \ w' p' →
            ( (total-map-to-fiber A B C f w')
              ( (total-map-from-fiber A B C f w') (z , p'))) =
            ( z , p') ,
          refl ,
          w ,
          p))

#def total-map-to-fiber-is-equiv
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))             -- a family of maps
  (w : (Σ (x : A) , C x))
  : is-equiv
    ( fib (B (first w)) (C (first w)) (f (first w)) (second w))
    ( fib (Σ (x : A) , B x) (Σ (x : A) , C x)
      ( total-map-family-of-maps A B C f) w)
    ( total-map-to-fiber A B C f w)
  :=
    ( total-map-to-fiber-retraction A B C f w ,
      total-map-to-fiber-section A B C f w)

#def total-map-fiber-equiv
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))             -- a family of maps
  (w : (Σ (x : A) , C x))
  : Equiv
    ( fib (B (first w)) (C (first w)) (f (first w)) (second w))
    ( fib (Σ (x : A) , B x) (Σ (x : A) , C x)
      ( total-map-family-of-maps A B C f) w)
  := (total-map-to-fiber A B C f w , total-map-to-fiber-is-equiv A B C f w)
```

## Families of equivalences

A family of equivalences induces an equivalence on total spaces and conversely.
It will be easiest to work with the incoherent notion of two-sided-inverses.

```rzk
#def invertible-family-total-inverse
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))
  (invfamily : (a : A) → has-inverse (B a) (C a) (f a))
  : (Σ (x : A) , C x) → (Σ (x : A) , B x)
  := \ (a , c) → (a , (has-inverse-inverse (B a) (C a) (f a) (invfamily a)) c)

#def invertible-family-total-retraction
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))
  (invfamily : (a : A) → has-inverse (B a) (C a) (f a))
  : has-retraction
    ( Σ (x : A) , B x)
    ( Σ (x : A) , C x)
    ( total-map-family-of-maps A B C f)
  :=
    ( invertible-family-total-inverse A B C f invfamily ,
      \ (a , b) →
        (eq-eq-fiber-Σ A B a
          ( (has-inverse-inverse (B a) (C a) (f a) (invfamily a)) (f a b)) b
          ( (first (second (invfamily a))) b)))

#def invertible-family-total-section
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))
  (invfamily : (a : A) → has-inverse (B a) (C a) (f a))
  : has-section (Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f)
  :=
    ( invertible-family-total-inverse A B C f invfamily ,
      \ (a , c) →
        ( eq-eq-fiber-Σ A C a
          ( f a ((has-inverse-inverse (B a) (C a) (f a) (invfamily a)) c)) c
          ( (second (second (invfamily a))) c)))

#def invertible-family-total-invertible
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))
  (invfamily : (a : A) → has-inverse (B a) (C a) (f a))
  : has-inverse
    ( Σ (x : A) , B x)
    ( Σ (x : A) , C x)
    ( total-map-family-of-maps A B C f)
  :=
    ( invertible-family-total-inverse A B C f invfamily ,
      ( second (invertible-family-total-retraction A B C f invfamily) ,
        second (invertible-family-total-section A B C f invfamily)))

#def family-of-equiv-total-equiv
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))
  (familyequiv : (a : A) → is-equiv (B a) (C a) (f a))
  : is-equiv
    ( Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f)
  :=
    is-equiv-has-inverse
    ( Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f)
    ( invertible-family-total-invertible A B C f
      ( \ a → has-inverse-is-equiv (B a) (C a) (f a) (familyequiv a)))

#def total-equiv-family-equiv
  ( A : U)
  ( B C : A → U)
  ( familyeq : (a : A) → Equiv (B a) (C a))
  : Equiv (Σ (x : A) , B x) (Σ (x : A) , C x)
  :=
    ( total-map-family-of-maps A B C (\ a → first (familyeq a)) ,
      family-of-equiv-total-equiv A B C
        ( \ a → first (familyeq a))
        ( \ a → second (familyeq a)))
```

The one-way result: that a family of equivalence gives an invertible map (and
thus an equivalence) on total spaces.

```rzk
#def total-has-inverse-family-equiv
  ( A : U)
  ( B C : A → U)
  ( f : (a : A) → (B a) → (C a))
  ( familyequiv : (a : A) → is-equiv (B a) (C a) (f a))
  : has-inverse (Σ (x : A) , B x) (Σ (x : A) , C x) (total-map-family-of-maps A B C f)
  :=
    invertible-family-total-invertible A B C f
    ( \ a → has-inverse-is-equiv (B a) (C a) (f a) (familyequiv a))
```

For the converse, we make use of our calculation on fibers. The first
implication could be proven similarly.

```rzk
#def total-contr-map-family-of-contr-maps
  ( A : U)
  ( B C : A → U)
  ( f : (a : A) → (B a) → (C a))                         -- a family of maps
  ( totalcontrmap :
    is-contr-map
      ( Σ (x : A) , B x)
      ( Σ (x : A) , C x)
      ( total-map-family-of-maps A B C f))
  ( a : A)
  : is-contr-map (B a) (C a) (f a)
  :=
    \ c →
      is-contr-is-equiv-to-contr
        ( fib (B a) (C a) (f a) c)
        ( fib (Σ (x : A) , B x) (Σ (x : A) , C x)
          ( total-map-family-of-maps A B C f) ((a , c)))
        ( total-map-fiber-equiv A B C f ((a , c)))
        ( totalcontrmap ((a , c)))

#def total-equiv-family-of-equiv
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))                         -- a family of maps
  (totalequiv : is-equiv
                ( Σ (x : A) , B x)
                ( Σ (x : A) , C x)
                ( total-map-family-of-maps A B C f))
  (a : A)
  : is-equiv (B a) (C a) (f a)
  :=
    is-equiv-is-contr-map (B a) (C a) (f a)
    ( total-contr-map-family-of-contr-maps A B C f
      ( is-contr-map-is-equiv
        ( Σ (x : A) , B x) (Σ (x : A) , C x)
        ( total-map-family-of-maps A B C f) totalequiv) a)
```

In summary, a family of maps is an equivalence iff the map on total spaces is an
equivalence.

```rzk
#def total-equiv-iff-family-of-equiv
  (A : U)
  (B C : A → U)
  (f : (a : A) → (B a) → (C a))
  : iff
      ( (a : A) → is-equiv (B a) (C a) (f a))
      ( is-equiv (Σ (x : A) , B x) (Σ (x : A) , C x)
        ( total-map-family-of-maps A B C f))
  := (family-of-equiv-total-equiv A B C f , total-equiv-family-of-equiv A B C f)
```

## Codomain based path spaces

```rzk
#def equiv-rev
  (A : U)
  (x y : A)
  : Equiv (x = y) (y = x)
  := (rev A x y , ((rev A y x , rev-rev A x y) , (rev A y x , rev-rev A y x)))

-- An equivalence between the based path spaces.
#def equiv-based-paths
  ( A : U)
  (a : A)
  : Equiv (Σ (x : A) , x = a) (Σ (x : A) , a = x)
  := total-equiv-family-equiv A (\ x → x = a) (\ x → a = x) (\ x → equiv-rev A x a)

-- Codomain based path spaces are contractible
#def is-contr-codomain-based-paths
  (A : U)         -- The ambient type.
  (a : A)         -- The basepoint.
  : is-contr (Σ (x : A) , x = a)
  :=
    is-contr-is-equiv-to-contr (Σ (x : A) , x = a) (Σ (x : A) , a = x)
      ( equiv-based-paths A a)
      ( is-contr-based-paths A a)
```

## Pullback of a type family

A family of types over B pulls back along any function f : A → B to define a
family of types over A.

```rzk
#def pullback
  (A B : U)
  (f : A → B)
  (C : B → U)
  : A → U
  := \ a → C (f a)
```

The pullback of a family along homotopic maps is equivalent.

```rzk
#section is-equiv-pullback-htpy

#variables A B : U
#variables f g : A → B
#variable α : homotopy A B f g
#variable C : B → U
#variable a : A

#def pullback-homotopy
  : (pullback A B f C a) → (pullback A B g C a)
  := \ c → transport B C (f a) (g a) (α a) c

#def pullback-homotopy-inverse
  : (pullback A B g C a) → (pullback A B f C a)
  := \ c → transport B C (g a) (f a) (rev B (f a) (g a) (α a)) c

#def pullback-homotopy-has-retraction
  : has-retraction
    ( pullback A B f C a)
    ( pullback A B g C a)
    ( pullback-homotopy)
  :=
    ( pullback-homotopy-inverse ,
      \ c →
        concat
        ( pullback A B f C a)
        ( transport B C (g a) (f a)
          ( rev B (f a) (g a) (α a))
          ( transport B C (f a) (g a) (α a) c))
        ( transport B C (f a) (f a)
          ( concat B (f a) (g a) (f a) (α a) (rev B (f a) (g a) (α a))) c)
        ( c)
        ( transport-concat-rev B C (f a) (g a) (f a) (α a)
          ( rev B (f a) (g a) (α a)) c)
        ( transport2 B C (f a) (f a)
        ( concat B (f a) (g a) (f a) (α a) (rev B (f a) (g a) (α a))) refl
        ( right-inverse-concat B (f a) (g a) (α a)) c))

#def pullback-homotopy-has-section
  : has-section (pullback A B f C a) (pullback A B g C a)
    ( pullback-homotopy)
  :=
    ( pullback-homotopy-inverse ,
      \ c →
        concat (pullback A B g C a)
          ( transport B C (f a) (g a) (α a)
            ( transport B C (g a) (f a) (rev B (f a) (g a) (α a)) c))
          ( transport B C (g a) (g a)
            ( concat B (g a) (f a) (g a) (rev B (f a) (g a) (α a)) (α a)) c)
          ( c)
          ( transport-concat-rev B C (g a) (f a) (g a)
            ( rev B (f a) (g a) (α a)) (α a) c)
          ( transport2 B C (g a) (g a)
            ( concat B (g a) (f a) (g a) (rev B (f a) (g a) (α a)) (α a)) refl
            ( left-inverse-concat B (f a) (g a) (α a)) c))

#def is-equiv-pullback-homotopy uses (α)
  : is-equiv
    (pullback A B f C a) (pullback A B g C a) (pullback-homotopy)
  := ( pullback-homotopy-has-retraction , pullback-homotopy-has-section)

#end is-equiv-pullback-htpy
```

The total space of a pulled back family of types maps to the original total
space.

```rzk
#def pullback-comparison-map
  ( A B : U)
  ( f : A → B)
  ( C : B → U)
  : (Σ (a : A) , (pullback A B f C) a) → (Σ (b : B) , C b)
  := \ (a , c) → (f a , c)
```

Now we show that if a family is pulled back along an equivalence, the total
spaces are equivalent by proving that the comparison is a contractible map. For
this, we first prove that each fiber is equivalent to a fiber of the original
map.

```rzk
#def pullback-comparison-fiber
  (A B : U)
  (f : A → B)
  (C : B → U)
  (z : Σ (b : B) , C b)
  : U
  :=
    fib (Σ (a : A) , (pullback A B f C) a) (Σ (b : B) , C b)
      ( pullback-comparison-map A B f C) z

#def pullback-comparison-fiber-to-fiber
  (A B : U)
  (f : A → B)
  (C : B → U)
  (z : Σ (b : B) , C b)
  : (pullback-comparison-fiber A B f C z) → (fib A B f (first z))
  :=
    \ (w , p) →
      idJ
      ( (Σ (b : B) , C b) ,
        (pullback-comparison-map A B f C w) ,
        \ z' p' →
          ( fib A B f (first z')) ,
        (first w , refl) ,
        z ,
        p)

#def from-base-fiber-to-pullback-comparison-fiber
  (A B : U)
  (f : A → B)
  (C : B → U)
  (b : B)
  : (fib A B f b) → (c : C b) → (pullback-comparison-fiber A B f C (b , c))
  :=
    \ (a , p) →
        idJ
        ( B ,
          f a ,
          \ b' p' →
            (c : C b') → (pullback-comparison-fiber A B f C ((b' , c))) ,
          \ c → ((a , c) , refl) ,
          b ,
          p)

#def pullback-comparison-fiber-to-fiber-inv
  (A B : U)
  (f : A → B)
  (C : B → U)
  (z : Σ (b : B) , C b)
  : (fib A B f (first z)) → (pullback-comparison-fiber A B f C z)
  :=
    \ (a , p) →
      from-base-fiber-to-pullback-comparison-fiber A B f C
      ( first z) (a , p) (second z)

#def pullback-comparison-fiber-to-fiber-retracting-homotopy
  (A B : U)
  (f : A → B)
  (C : B → U)
  (z : Σ (b : B) , C b)
  ((w , p) : pullback-comparison-fiber A B f C z)
  : ( (pullback-comparison-fiber-to-fiber-inv A B f C z)
      ( (pullback-comparison-fiber-to-fiber A B f C z) (w , p))) = (w , p)
  :=
    idJ
    ( (Σ (b : B) , C b) ,
      (pullback-comparison-map A B f C w) ,
      \ z' p' →
        ((pullback-comparison-fiber-to-fiber-inv A B f C z')
          ((pullback-comparison-fiber-to-fiber A B f C z') (w , p'))) = (w , p') ,
      refl ,
      z ,
      p)

#def pullback-comparison-fiber-to-fiber-section-homotopy-map
  (A B : U)
  (f : A → B)
  (C : B → U)
  (b : B)
  ((a , p) : fib A B f b)
  : (c : C b) →
      ((pullback-comparison-fiber-to-fiber A B f C (b , c))
        ((pullback-comparison-fiber-to-fiber-inv A B f C (b , c)) (a , p))) =
      (a , p)
  :=
    idJ
    ( B ,
      f a ,
      \ b' p' → (c : C b') →
        ( (pullback-comparison-fiber-to-fiber A B f C (b' , c))
          ( (pullback-comparison-fiber-to-fiber-inv A B f C (b' , c)) (a , p'))) =
        ( a , p') ,
      \ c → refl ,
      b ,
      p)

#def pullback-comparison-fiber-to-fiber-section-homotopy
  (A B : U)
  (f : A → B)
  (C : B → U)
  (z : Σ (b : B) , C b)
  ((a , p) : fib A B f (first z))
  : ((pullback-comparison-fiber-to-fiber A B f C z)
      ((pullback-comparison-fiber-to-fiber-inv A B f C z) (a , p))) = (a , p)
  :=
    pullback-comparison-fiber-to-fiber-section-homotopy-map A B f C
    ( first z) (a , p) (second z)

#def equiv-pullback-comparison-fiber
  (A B : U)
  (f : A → B)
  (C : B → U)
  (z : Σ (b : B) , C b)
  : Equiv (pullback-comparison-fiber A B f C z) (fib A B f (first z))
  :=
    ( pullback-comparison-fiber-to-fiber A B f C z ,
      ( (pullback-comparison-fiber-to-fiber-inv A B f C z ,
        pullback-comparison-fiber-to-fiber-retracting-homotopy A B f C z) ,
        ( pullback-comparison-fiber-to-fiber-inv A B f C z ,
          pullback-comparison-fiber-to-fiber-section-homotopy A B f C z)))
```

As a corollary, we show that pullback along an equivalence induces an
equivalence of total spaces.

```rzk
#def total-equiv-pullback-is-equiv
  (A B : U)
  (f : A → B)
  (is-equiv-f : is-equiv A B f)
  (C : B → U)
  : Equiv (Σ (a : A) , (pullback A B f C) a) (Σ (b : B) , C b)
  :=
    ( pullback-comparison-map A B f C ,
      is-equiv-is-contr-map
        ( Σ (a : A) , (pullback A B f C) a)
        ( Σ (b : B) , C b)
        ( pullback-comparison-map A B f C)
        ( \ z →
          ( is-contr-is-equiv-to-contr
            ( pullback-comparison-fiber A B f C z)
            ( fib A B f (first z))
            ( equiv-pullback-comparison-fiber A B f C z)
            ( is-contr-map-is-equiv A B f is-equiv-f (first z)))))
```

## Fundamental theorem of identity types

```rzk
#section fundamental-thm-id-types

#variable A : U
#variable a : A
#variable B : A → U
#variable f : (x : A) → (a = x) → B x

#def fund-id-fam-of-eqs-implies-sum-over-codomain-contr
  : ((x : A) → (is-equiv (a = x) (B x) (f x))) → (is-contr (Σ (x : A) , B x))
  :=
    ( \ familyequiv →
      ( equiv-with-contractible-domain-implies-contractible-codomain
        ( Σ (x : A) , a = x) (Σ (x : A) , B x)
        ( ( total-map-family-of-maps A ( \ x → (a = x)) B f) ,
          ( is-equiv-has-inverse (Σ (x : A) , a = x) (Σ (x : A) , B x)
            ( total-map-family-of-maps A ( \ x → (a = x)) B f)
            ( total-has-inverse-family-equiv A
              ( \ x → (a = x)) B f familyequiv)))
        ( is-contr-based-paths A a)))

#def fund-id-sum-over-codomain-contr-implies-fam-of-eqs
  : (is-contr (Σ (x : A) , B x)) → ((x : A) → (is-equiv (a = x) (B x) (f x)))
  :=
    ( \ is-contr-B →
      ( \ x →
        ( total-equiv-family-of-equiv A
          ( \ x' → (a = x'))
          B
          f
          ( is-equiv-are-contr (Σ (x' : A) , (a = x')) (Σ (x' : A) , (B x'))
            ( is-contr-based-paths A a) is-contr-B
            ( total-map-family-of-maps A ( \ x' → (a = x')) B f))
          x)))

-- This allows us to apply "based path induction"
-- to a family satisfying the fundamental theorem.
-- Please suggest a better name.
#def ind-based-path
  (familyequiv : (z : A) → (is-equiv (a = z) (B z) (f z)))
  (P : (z : A) → B z → U)
  (p0 : P a (f a refl))
  (u : A)
  (p : B u)
  : P u p
  :=
    ind-sing
      ( Σ (v : A) , B v)
      ( a , f a refl)
      ( \ (u' , p') → P u' p')
      ( contr-implies-singleton-induction-pointed
        ( Σ (z : A) , B z)
        ( fund-id-fam-of-eqs-implies-sum-over-codomain-contr familyequiv)
        ( \ (x', p') → P x' p'))
      ( p0)
      (u , p)

#end fundamental-thm-id-types
```

For all `x` , `y` in `A`, `ap_{e ,x ,y}` is an equivalence.

```rzk
#def emb-is-equiv
  (A B : U)
  (e : A → B)
  (is-equiv-e : is-equiv A B e)
  : (Emb A B)
  :=
    ( e ,
      \ x y →
          ( fund-id-sum-over-codomain-contr-implies-fam-of-eqs
  -- By the fundamental theorem of identity types, it will suffice to show
  -- contractibility of sigma_{t : A} e x = e t
  -- for the family of maps ap_e, which is of type
  -- (\t:A) → (x = t) → (e x = e t)
            A
            x
            ( \ t → (e x = e t))
            ( \ t → (ap A B x t e)) -- the family of maps ap_e
            ( (is-contr-is-equiv-to-contr
  -- Contractibility of sigma_{t : A} e x = e t will follow since
  -- total (\ t → rev B (e x) = (e t)), mapping from sigma_{t : A} e x = e t to
  -- sigma_{t : A} e t = e x
  -- is an equivalence, and `Σ_{t : A} e t = e x ~ fib (e , e x)` is
  -- contractible since e is an equivalence.
                (Σ (y' : A) , (e x = e y')) -- source type
                (Σ (y' : A) , (e y' = e x)) -- target type
                ((total-map-family-of-maps A
                  ( \ y' → (e x) = (e y'))
                  ( \ y' → (e y') = (e x))
                  ( \ y' → (rev B (e x) (e y')))) , -- a) total map

                ( -- b) proof that total map is equivalence
                  ( first
                    ( total-equiv-iff-family-of-equiv A
                    ( \ y' → (e x) = (e y'))
                    ( \ y' → (e y') = (e x))
                    ( \ y' → (rev B (e x) (e y')))))
                  ( \ y' → (is-equiv-rev B (e x) (e y')))))
                ( -- fiber of e at e(x) is contractible
                  (is-contr-map-is-equiv A B e is-equiv-e) (e x))))) (y))
                  -- evaluate at y

#def is-emb-is-equiv
  (A B : U)
  (e : A → B)
  (is-equiv-e : is-equiv A B e)
  : is-emb A B e
  := (second (emb-is-equiv A B e is-equiv-e))
```

## 2-of-3 for equivalences

```rzk
-- It might be better to redo this without appealing to results about
-- embeddings so that this could go earlier.
#def RightCancel-is-equiv
  (A B C : U)
  (f : A → B)
  (g : B → C)
  (is-equiv-g : is-equiv B C g)
  (gfisequiv : is-equiv A C (composition A B C g f))
  : is-equiv A B f
  :=
    ( ( composition B C A
        (is-equiv-retraction A C (composition A B C g f) gfisequiv) g ,
        (second (first gfisequiv))) ,
      ( composition B C A
        (is-equiv-section A C (composition A B C g f) gfisequiv) g ,
        \ b →
          inv-ap-is-emb
            B C g
            ( is-emb-is-equiv B C g is-equiv-g)
            ( f ((is-equiv-section A C (composition A B C g f) gfisequiv) (g b)))
            b
            ( (second (second gfisequiv)) (g b))))

#def LeftCancel-is-equiv
  (A B C : U)
  (f : A → B)
  (is-equiv-f : is-equiv A B f)
  (g : B → C)
  (gfisequiv : is-equiv A C (composition A B C g f))
  : is-equiv B C g
  :=
    ( ( composition C A B
          f (is-equiv-retraction A C (composition A B C g f) gfisequiv) ,
        \ b →
          triple-concat B
            ( f ((is-equiv-retraction A C (composition A B C g f) gfisequiv)
              (g b)))
            ( f ((is-equiv-retraction A C (composition A B C g f) gfisequiv)
              (g (f ((is-equiv-section A B f is-equiv-f) b)))))
            ( f ((is-equiv-section A B f is-equiv-f) b))
            ( b)
            ( ap B B
              ( b)
              ( f ((is-equiv-section A B f is-equiv-f) b))
              ( \ b0 →
                ( f ((is-equiv-retraction A C
                      ( composition A B C g f) gfisequiv) (g b0))))
              ( rev B (f ((is-equiv-section A B f is-equiv-f) b)) b
                ( (second (second is-equiv-f)) b)))
            ( ( homotopy-whisker B A A B
                ( \ a →
                  ( is-equiv-retraction A C
                    ( composition A B C g f) gfisequiv) (g (f a)))
                ( \ a → a)
                ( second (first gfisequiv))
                ( is-equiv-section A B f is-equiv-f)
                f) b)
            ( (second (second is-equiv-f)) b)) ,
      ( composition C A B
        ( f)
        ( is-equiv-section A C (composition A B C g f) gfisequiv) ,
        ( second (second gfisequiv))))
```

## Maps over product types

For later use, we specialize the previous results to the case of a family of
types over a product type.

```rzk
#section fibered-map-over-product

#variables A A' B B' : U
#variable C : A → B → U
#variable C' : A' → B' → U
#variable f : A → A'
#variable g : B → B'
#variable h : (a : A) → (b : B) → (c : C a b) → C' (f a) (g b)

#def total-map-fibered-map-over-product
  : (Σ (a : A) , (Σ (b : B) , C a b)) → (Σ (a' : A') , (Σ (b' : B') , C' a' b'))
  := \ (a , (b , c)) → (f a , (g b , h a b c))

#def pullback-is-equiv-base-is-equiv-total-is-equiv
  (totalisequiv : is-equiv
                    (Σ (a : A) , (Σ (b : B) , C a b))
                    (Σ (a' : A') , (Σ (b' : B') , C' a' b'))
                    total-map-fibered-map-over-product)
  (is-equiv-f : is-equiv A A' f)
  : is-equiv
    ( Σ (a : A) , (Σ (b : B) , C a b))
    ( Σ (a : A) , (Σ (b' : B') , C' (f a) b'))
    ( \ (a , (b , c)) → (a , (g b , h a b c)))
  :=
    RightCancel-is-equiv
    ( Σ (a : A) , (Σ (b : B) , C a b))
    ( Σ (a : A) , (Σ (b' : B') , C' (f a) b'))
    ( Σ (a' : A') , (Σ (b' : B') , C' a' b'))
    ( \ (a , (b , c)) → (a , (g b , h a b c)))
    ( \ (a , (b' , c')) → (f a , (b' , c')))
    ( second (total-equiv-pullback-is-equiv
                A
                A'
                f
                is-equiv-f
                ( \ a' → (Σ (b' : B') , C' a' b'))))
    ( totalisequiv)

#def pullback-is-equiv-bases-are-equiv-total-is-equiv
  (totalisequiv : is-equiv
                    ( Σ (a : A) , (Σ (b : B) , C a b))
                    ( Σ (a' : A') , (Σ (b' : B') , C' a' b'))
                    total-map-fibered-map-over-product)
  (is-equiv-f : is-equiv A A' f)
  (is-equiv-g : is-equiv B B' g)
  : is-equiv
    ( Σ (a : A) , (Σ (b : B) , C a b))
    ( Σ (a : A) , (Σ (b : B) , C' (f a) (g b)))
    ( \ (a , (b , c)) → (a , (b , h a b c)))
  :=
    RightCancel-is-equiv
    ( Σ (a : A) , (Σ (b : B) , C a b))
    ( Σ (a : A) , (Σ (b : B) , C' (f a) (g b)))
    ( Σ (a : A) , (Σ (b' : B') , C' (f a) b'))
    ( \ (a , (b , c)) → (a , (b , h a b c)))
    ( \ (a , (b , c)) → (a , (g b , c)))
    ( family-of-equiv-total-equiv A
      ( \ a → (Σ (b : B) , C' (f a) (g b)))
      ( \ a → (Σ (b' : B') , C' (f a) b'))
      ( \ a (b , c) → (g b , c))
      ( \ a →
        ( second
          ( total-equiv-pullback-is-equiv
              B
              B'
              g
              is-equiv-g
              ( \ b' → C' (f a) b')))))
    ( pullback-is-equiv-base-is-equiv-total-is-equiv totalisequiv is-equiv-f)

#def fibered-map-is-equiv-bases-are-equiv-total-map-is-equiv
  (totalisequiv : is-equiv
                  ( Σ (a : A) , (Σ (b : B) , C a b))
                  ( Σ (a' : A') , (Σ (b' : B') , C' a' b'))
                  total-map-fibered-map-over-product)
  (is-equiv-f : is-equiv A A' f)
  (is-equiv-g : is-equiv B B' g)
  (a0 : A)
  (b0 : B)
  : is-equiv (C a0 b0) (C' (f a0) (g b0)) (h a0 b0)
  :=
    total-equiv-family-of-equiv B
    ( \ b → C a0 b)
    ( \ b → C' (f a0) (g b))
    ( \ b c → h a0 b c)
    ( total-equiv-family-of-equiv
      A
      ( \ a → (Σ (b : B) , C a b))
      ( \ a → (Σ (b : B) , C' (f a) (g b)))
      ( \ a (b , c) → (b , h a b c))
      ( pullback-is-equiv-bases-are-equiv-total-is-equiv
          totalisequiv is-equiv-f is-equiv-g)
      a0)
    b0

#end fibered-map-over-product
```
