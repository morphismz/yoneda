# 7. Fibers

This is a literate `rzk` file:

```rzk
#lang rzk-1
```

## Fibers

The homotopy fiber of a map is the following type:

```rzk
-- The fiber of a map
#def fib
  (A B : U)
  (f : A → B)
  (b : B)
  : U
  := Σ (a : A) , (f a) = b

-- We calculate the transport of (a , q) : fib b along p : a = a'
#def transport-in-fiber
  (A B : U)
  (f : A → B)
  (b : B)
  (a a' : A)
  (u : (f a) = b)
  (p : a = a')
  : (transport A ( \ x → (f x) = b) a a' p u) =
    (concat B (f a') (f a) b (ap A B a' a f (rev A a a' p)) u)
  :=
    idJ
    ( A ,
      a ,
      \ a'' p' →
        (transport A (\ x → (f x) = b) a a'' p' u) =
        (concat B (f a'') (f a) b (ap A B a'' a f (rev A a a'' p')) u) ,
      ( rev ((f a) = b) (concat B (f a) (f a) b refl u) u
        ( left-unit-concat B (f a) b u)) ,
      a' ,
      p)
```

## Contractible maps

A map is contractible just when its fibers are contractible.

```rzk
-- Contractible maps
#def is-contr-map
  (A B : U)
  (f : A → B)
  : U
  := (b : B) → is-contr (fib A B f b)
```

Contractible maps are equivalences:

```rzk
#section is-contr-map-is-equiv

#variables A B : U
#variable f : A → B
#variable is-contr-f : is-contr-map A B f

-- The inverse to a contractible map
#def is-contr-map-inverse
  : B → A
  := \ b → first (contraction-center (fib A B f b) (is-contr-f b))

#def has-section-is-contr-map
  : has-section A B f
  :=
    ( is-contr-map-inverse ,
      \ b → second (contraction-center (fib A B f b) (is-contr-f b)))

#def is-contr-map-data-in-fiber uses (is-contr-f)
  (a : A)
  : fib A B f (f a)
  := (is-contr-map-inverse (f a) , (second has-section-is-contr-map) (f a))

#def is-contr-map-path-in-fiber
  (a : A)
  : (is-contr-map-data-in-fiber a) =_{fib A B f (f a)} (a , refl)
  := contractible-connecting-htpy
      ( fib A B f (f a))
      ( is-contr-f (f a))
      ( is-contr-map-data-in-fiber a)
      ( a , refl)

#def is-contr-map-has-retraction uses (is-contr-f)
  : has-retraction A B f
  :=
    ( is-contr-map-inverse ,
      \ a → ( ap (fib A B f (f a)) A
                ( is-contr-map-data-in-fiber a)
                ( (a , refl))
                ( \ u → first u)
                ( is-contr-map-path-in-fiber a)))

#def is-equiv-is-contr-map uses (is-contr-f)
  : is-equiv A B f
  := (is-contr-map-has-retraction , has-section-is-contr-map)

#end is-contr-map-is-equiv
```

## Half adjoint equivalences are contractible.

We now show that half adjoint equivalences are contractible maps.

```rzk
-- If f is a half adjoint equivalence, its fibers are inhabited.
#def is-surj-is-half-adjoint-equiv
  (A B : U)
  (f : A → B)
  (fisHAE : is-half-adjoint-equiv A B f) -- first fisHAE : has-inverse A B f
  (b : B)
  : fib A B f b
  :=
    ( (has-inverse-inverse A B f (first fisHAE)) b ,
      (second (second (first fisHAE))) b)
```

It takes much more work to construct the contracting homotopy. The bath path of
this homotopy is straightforward.

```rzk
#section half-adjoint-equivalence-fiber-data

#variables A B : U
#variable f : A → B
#variable fisHAE : is-half-adjoint-equiv A B f
#variable b : B
#variable z : fib A B f b

#def isHAE-fib-base-path
  : ((has-inverse-inverse A B f (first fisHAE)) b) = (first z)
  :=
    concat A
      ( (has-inverse-inverse A B f (first fisHAE)) b)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( first z)
      ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
        ( rev B (f (first z)) b (second z)))
      ( (first (second (first fisHAE))) (first z))

-- Specializing the above to isHAE-fib-base-path
#def isHAE-fib-base-path-transport
  : transport A (\ x → (f x) = b)
      ( (has-inverse-inverse A B f (first fisHAE)) b) (first z)
      ( isHAE-fib-base-path )
      ( (second (second (first fisHAE))) b) =
    concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
      ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
          ( rev A ((has-inverse-inverse A B f (first fisHAE)) b) (first z)
            ( isHAE-fib-base-path )))
      ( (second (second (first fisHAE))) b)
  :=
    transport-in-fiber A B f b
      ( (has-inverse-inverse A B f (first fisHAE)) b) (first z)
      ( (second (second (first fisHAE))) b)
      ( isHAE-fib-base-path )

#def isHAE-fib-base-path-rev-coherence
  : rev A ((has-inverse-inverse A B f (first fisHAE)) b) (first z)
      ( isHAE-fib-base-path) =
    concat A
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( (has-inverse-inverse A B f (first fisHAE)) b)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
        ( (first (second (first fisHAE))) (first z)))
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
          ( rev B (f (first z)) b (second z))))
  :=
    rev-concat A
      ( (has-inverse-inverse A B f (first fisHAE)) b)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( first z)
      ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
        ( rev B (f (first z)) b (second z)))
      ( (first (second (first fisHAE))) (first z))

#def isHAE-fib-base-path-transport-rev-calculation
  : concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
      ( rev A ((has-inverse-inverse A B f (first fisHAE)) b) (first z)
        ( isHAE-fib-base-path )))
    ( (second (second (first fisHAE))) b) =
    concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
      ( concat A
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z)))
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
            ( rev B (f (first z)) b (second z))))))
    ( (second (second (first fisHAE))) b)
  :=
    homotopy-concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( b)
      ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
        ( rev A ((has-inverse-inverse A B f (first fisHAE)) b) (first z)
          ( isHAE-fib-base-path )))
      ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
        ( concat A
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z)))
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))))
      ( ap-htpy A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
        ( rev A ((has-inverse-inverse A B f (first fisHAE)) b) (first z)
          ( isHAE-fib-base-path ))
        ( concat A
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z)))
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z)))))
        ( isHAE-fib-base-path-rev-coherence ))
      ( (second (second (first fisHAE))) b)

#def isHAE-fib-base-path-transport-ap-calculation
  : concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
      ( concat A
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
          ( (first (second (first fisHAE))) (first z)))
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
            ( rev B (f (first z)) b (second z))))))
    ( (second (second (first fisHAE))) b) =
    concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))))
      ( (second (second (first fisHAE))) b)
  :=
    homotopy-concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( b)
      ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
        ( concat A
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z)))
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))))
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))))
      ( ap-concat A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z)))
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
            ( rev B (f (first z)) b (second z)))))
      ( (second (second (first fisHAE))) b)

#def isHAE-fib-base-path-transport-rev-ap-rev-calculation
  : concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap A B
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
            ( rev B (f (first z)) b (second z))))))
    ( (second (second (first fisHAE))) b) =
    concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z) ((has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( f)
          ( ap B A (f (first z)) b
            ( has-inverse-inverse A B f (first fisHAE))
            ( second z))))
      ( (second (second (first fisHAE))) b)
  :=
    homotopy-concat B
      ( f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))))
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( f)
          ( ap B A
            ( f (first z)) b (has-inverse-inverse A B f (first fisHAE))
            ( second z))))
      ( concat-homotopy B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z)))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b) f
          ( ap B A (f (first z)) b
            ( has-inverse-inverse A B f (first fisHAE)) (second z)))
        ( ap-htpy A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b
              ( f (first z))
              ( has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))
          ( ap B A
            ( f (first z))
            ( b)
            ( has-inverse-inverse A B f (first fisHAE))
            ( second z))
          ( rev-ap-rev B A (f (first z)) b
            ( has-inverse-inverse A B f (first fisHAE)) (second z))))
      ( (second (second (first fisHAE))) b)

#def isHAE-fib-base-path-transport-ap-ap-calculation
  : concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( ap A B
        ( first z) ((has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap A B
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( f)
        ( ap B A (f (first z)) b
          ( has-inverse-inverse A B f (first fisHAE))
          ( second z))))
    ( (second (second (first fisHAE))) b) =
    concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( ap A B
        ( first z) ((has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap B B (f (first z)) b
        ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
        ( second z)))
    ( (second (second (first fisHAE))) b)
  :=
    homotopy-concat B
    ( f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap A B
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( f)
        ( ap B A
          ( f (first z)) b
          ( has-inverse-inverse A B f (first fisHAE)) (second z))))
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap B B (f (first z)) b
        ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
        ( second z)))
    ( concat-homotopy B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap A B
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( (has-inverse-inverse A B f (first fisHAE)) b)
        ( f)
        ( ap B A (f (first z)) b
          ( has-inverse-inverse A B f (first fisHAE)) (second z)))
      ( ap B B (f (first z)) b
        ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
        ( second z))
      ( rev-ap-comp B A B
        ( f (first z))
        ( b)
        ( has-inverse-inverse A B f (first fisHAE))
        ( f)
        ( second z)))
    ( (second (second (first fisHAE))) b)

#def isHAE-fib-base-path-transport-assoc-calculation
  : concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap B B (f (first z)) b
        ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
        ( second z)))
    ( (second (second (first fisHAE))) b) =
    concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( f)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( first z)
        ( (first (second (first fisHAE))) (first z))))
    ( concat B
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( b)
      ( ap B B (f (first z)) b
        ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
        ( second z))
      ( (second (second (first fisHAE))) b))
  :=
    associative-concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( f ((has-inverse-inverse A B f (first fisHAE)) b))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( f)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
        ( (first (second (first fisHAE))) (first z))))
    ( ap B B (f (first z)) b
      ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
      ( second z))
    ( (second (second (first fisHAE))) b)

#def isHAE-fib-base-path-transport-nat-calculation
  : concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( f)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( first z)
        ( (first (second (first fisHAE))) (first z))))
    ( concat B
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( b)
      ( ap B B (f (first z)) b
        ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
        ( second z))
      ( (second (second (first fisHAE))) b)) =
    concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( f)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( first z)
        ( (first (second (first fisHAE))) (first z))))
    ( concat B
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( b)
      ( (second (second (first fisHAE))) (f (first z)))
      ( ap B B (f (first z)) b (identity B) (second z)))
  :=
    concat-homotopy B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( b)
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( concat B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( b)
        ( ap B B
          ( f (first z))
          ( b)
          ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
          ( second z))
        ( (second (second (first fisHAE))) b))
      ( concat B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( b)
        ( (second (second (first fisHAE))) (f (first z)))
        ( ap B B (f (first z)) b (identity B) (second z)))
      ( nat-htpy B B
        ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
        ( identity B)
        ( second (second (first fisHAE)))
        ( f (first z))
        ( b)
        ( second z))

#def isHAE-fib-base-path-transport-ap-id-calculation
  : concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( f)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( first z)
        ( (first (second (first fisHAE))) (first z))))
    ( concat B
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( b)
      ( (second (second (first fisHAE))) (f (first z)))
      ( ap B B (f (first z)) b (identity B) (second z))) =
  concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( f)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( first z)
        ( (first (second (first fisHAE))) (first z))))
    ( concat B
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( b)
      ( (second (second (first fisHAE))) (f (first z)))
      ( second z))
  :=
    concat-homotopy B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( b)
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( concat B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( b)
        ( (second (second (first fisHAE))) (f (first z)))
        ( ap B B (f (first z)) b (identity B) (second z)))
      ( concat B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( b)
        ( (second (second (first fisHAE))) (f (first z)))
        (second z))
      ( concat-homotopy B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( b)
        ( (second (second (first fisHAE))) (f (first z)))
        ( ap B B (f (first z)) b (identity B) (second z))
        ( second z)
        ( ap-id B (f (first z)) b (second z)))

#def isHAE-fib-base-path-transport-reassoc-calculation
  : concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( first z)
        ( (first (second (first fisHAE))) (first z))))
    ( concat B
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( b)
      ( (second (second (first fisHAE))) (f (first z)))
      ( second z)) =
    concat B (f (first z)) (f (first z)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( (second (second (first fisHAE))) (f (first z))))
      ( second z)
  :=
    rev-associative-concat B
    ( f (first z))
    ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
    ( f (first z))
    ( b)
    ( ap A B
      ( first z)
      ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
      ( f)
      ( rev A
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( first z)
        ( (first (second (first fisHAE))) (first z))))
    ( (second (second (first fisHAE))) (f (first z)))
    ( second z)

#def isHAE-fib-base-path-transport-HAE-calculation
  : concat B (f (first z)) (f (first z)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( (second (second (first fisHAE))) (f (first z))))
    ( second z) =
    concat B (f (first z)) (f (first z)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( has-inverse-retraction-composite A B f (first fisHAE) (first z))
          ( first z) f
          ( ((first (second (first fisHAE)))) (first z))))
      ( second z)
  :=
    homotopy-concat B (f (first z)) (f (first z)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( (second (second (first fisHAE))) (f (first z))))
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap A B
        ( has-inverse-retraction-composite A B f (first fisHAE) (first z))
        ( first z)
        ( f)
        ( (first (second (first fisHAE))) (first z))))
      ( concat-homotopy B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( (second (second (first fisHAE))) (f (first z)))
        ( ap A B
          ( has-inverse-retraction-composite A B f (first fisHAE) (first z))
          ( first z) f
          ( ((first (second (first fisHAE)))) (first z)))
        ( (second fisHAE) (first z)))
      ( second z)

#def isHAE-fib-base-path-transport-HAE-reduction
  : concat B (f (first z)) (f (first z)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( ap A B
        ( first z) ((has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ((has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap A B
        ( has-inverse-retraction-composite A B f (first fisHAE) (first z))
        ( first z) f
        ( ((first (second (first fisHAE)))) (first z))))
    ( second z) =
    concat B (f (first z)) (f (first z)) b (refl) (second z)
  :=
    homotopy-concat B (f (first z)) (f (first z)) b
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( f (first z))
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( ap A B
        ( has-inverse-retraction-composite A B f (first fisHAE) (first z))
        ( first z) f
        ( ((first (second (first fisHAE)))) (first z))))
    ( refl)
    ( concat-ap-rev-ap-id A B
      ( has-inverse-retraction-composite A B f (first fisHAE) (first z))
      ( first z)
      ( f)
      ( ((first (second (first fisHAE)))) (first z)))
    ( second z)

#def isHAE-fib-base-path-transport-HAE-final-reduction uses (A)
  : concat B (f (first z)) (f (first z)) b (refl) (second z) = second z
  := left-unit-concat B (f (first z)) b (second z)

#def isHAE-fib-base-path-transport-path
  : transport A ( \ x → (f x) = b)
    ( (has-inverse-inverse A B f (first fisHAE)) b) (first z)
    ( isHAE-fib-base-path )
    ( (second (second (first fisHAE))) b) = second z
  :=
    alternating-12ary-concat ( (f (first z)) = b)
    ( transport A ( \ x → (f x) = b)
      ( (has-inverse-inverse A B f (first fisHAE)) b) (first z)
      ( isHAE-fib-base-path )
      ( (second (second (first fisHAE))) b))
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( b)
      ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
        ( rev A ((has-inverse-inverse A B f (first fisHAE)) b) (first z)
          ( isHAE-fib-base-path )))
      ( (second (second (first fisHAE))) b))
    ( isHAE-fib-base-path-transport )
    ( concat B
      ( f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
      ( ap A B (first z) ((has-inverse-inverse A B f (first fisHAE)) b) f
        ( concat A
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z)))
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))))
      ( (second (second (first fisHAE))) b))
    ( isHAE-fib-base-path-transport-rev-calculation)
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) b))
      ( b)
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) b)
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( ap B A b (f (first z)) (has-inverse-inverse A B f (first fisHAE))
              ( rev B (f (first z)) b (second z))))))
      ( (second (second (first fisHAE))) b))
    ( isHAE-fib-base-path-transport-ap-calculation )
    ( concat B
      ( f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( (has-inverse-inverse A B f (first fisHAE)) b)
          f
          ( ap B A (f (first z)) b
            ( has-inverse-inverse A B f (first fisHAE)) (second z))))
      ( (second (second (first fisHAE))) b))
    ( isHAE-fib-base-path-transport-rev-ap-rev-calculation )
    ( concat B (f (first z)) (f ((has-inverse-inverse A B f (first fisHAE)) b)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap B B (f (first z)) b
          ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
          ( second z)))
      ( (second (second (first fisHAE))) b))
    ( isHAE-fib-base-path-transport-ap-ap-calculation )
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( b)
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( concat B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f ((has-inverse-inverse A B f (first fisHAE)) b))
        ( b)
        ( ap B B
          ( f (first z)) ( b)
          ( composition B A B f (has-inverse-inverse A B f (first fisHAE)))
          ( second z))
        ( (second (second (first fisHAE))) b)))
    ( isHAE-fib-base-path-transport-assoc-calculation)
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( b)
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
        ( f)
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) (first z)
          ( (first (second (first fisHAE))) (first z))))
      ( concat B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( b)
        ( (second (second (first fisHAE))) (f (first z)))
        ( ap B B (f (first z)) b (identity B) (second z))))
    ( isHAE-fib-base-path-transport-nat-calculation)
    ( concat B
      ( f (first z))
      ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
      ( b)
      ( ap A B
        ( first z)
        ( (has-inverse-inverse A B f (first fisHAE)) (f (first z))) f
        ( rev A
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( first z)
          ( (first (second (first fisHAE))) (first z))))
      ( concat B
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( b)
        ( (second (second (first fisHAE))) (f (first z)))
        ( second z)))
    (isHAE-fib-base-path-transport-ap-id-calculation)
    ( concat B (f (first z)) (f (first z)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( (second (second (first fisHAE))) (f (first z))))
        ( second z))
    ( isHAE-fib-base-path-transport-reassoc-calculation)
    ( concat B (f (first z)) (f (first z)) b
      ( concat B
        ( f (first z))
        ( f ((has-inverse-inverse A B f (first fisHAE)) (f (first z))))
        ( f (first z))
        ( ap A B
          ( first z)
          ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
          ( f)
          ( rev A
            ( (has-inverse-inverse A B f (first fisHAE)) (f (first z)))
            ( first z)
            ( (first (second (first fisHAE))) (first z))))
        ( ap A B
          ( has-inverse-retraction-composite A B f (first fisHAE) (first z))
          ( first z) f
          ( (first (second (first fisHAE))) (first z))))
      ( second z))
    ( isHAE-fib-base-path-transport-HAE-calculation)
    ( concat B (f (first z)) (f (first z)) b (refl) (second z))
    ( isHAE-fib-base-path-transport-HAE-reduction)
    ( second z)
    ( isHAE-fib-base-path-transport-HAE-final-reduction)
```

Finally, we may define the contracting homotopy:

```rzk
#def isHAE-fib-contracting-homotopy
  : (is-surj-is-half-adjoint-equiv A B f fisHAE b) = z
  :=
    path-of-pairs-pair-of-paths A
    ( \ x → (f x) = b)
    ( (has-inverse-inverse A B f (first fisHAE)) b)
    ( first z)
    ( isHAE-fib-base-path)
    ( (second (second (first fisHAE))) b)
    ( second z)
    ( isHAE-fib-base-path-transport-path)

#end half-adjoint-equivalence-fiber-data
```

Half adjoint equivalences define contractible maps:

```rzk
#def is-contr-map-is-half-adjoint-equiv
  (A B : U)
  (f : A → B)
  (fisHAE : is-half-adjoint-equiv A B f)
  : is-contr-map A B f
  := \ b →
        ( is-surj-is-half-adjoint-equiv A B f fisHAE b ,
          \ z → isHAE-fib-contracting-homotopy A B f fisHAE b z)
```

## Equivalences are contractible maps

```rzk
#def is-contr-map-is-equiv
  (A B : U)
  (f : A → B)
  (is-equiv-f : is-equiv A B f)
  : is-contr-map A B f
  := \ b →
        ( is-surj-is-half-adjoint-equiv A B f
          ( is-half-adjoint-equiv-is-equiv A B f is-equiv-f) b ,
          \ z → isHAE-fib-contracting-homotopy A B f
            ( is-half-adjoint-equiv-is-equiv A B f is-equiv-f) b z)

#def is-contr-map-iff-is-equiv
  (A B : U)
  (f : A → B)
  : iff (is-contr-map A B f) (is-equiv A B f)
  := (is-equiv-is-contr-map A B f , is-contr-map-is-equiv A B f)
```
