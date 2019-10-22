## Formality: an efficient proof language

A proof language is as a programming language with a type system that is sufficiently expressive to state and prove mathematical theorems about its own programs, exploiting [Curry-Howard's correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence). Formality is a new proof language designed to achieve 3 main goals:

1. Efficiency: it should be as fast as possible. 

2. Simplicity: it should be implemented in as few lines of code as possible.

3. Accessibility: it should be usable by non-experts as much as possible.

Existing proof languages such as Agda, Coq, Lean, Isabelle and Idris don't address those goals with sufficient priority. They are often very slow, have big implementations, and demand expert knowledge from its users. Formality achieves efficiency by compiling to an efficient high-order machine featuring optimal beta-reduction, no garbage-collection and massive parallelism. It achieves simplicity by using as few core primitives as possible: in fact, every algorithm and data-structure can be represented with just lambdas, including inductive datatypes. It achieves accessibility by developing a syntax that is familiar to regular developers, drawing insipration on popular languages like Python, Rust and TypeScript.

---

## Overview

Formality is a dependently typed programming language based on extrinsic type theory. It is based on elementary terms are just annotated versions of the Elementary Affine Lambda-Calculus (EALC), which is itself a terminating subset of the lambda-calculus with great computational characteristics. In particular, EALC is compatible with the most efficient version of the optimal reduction algorithm [citation]. This is important because, while asymptically optimal, most implementations of the algorithm carry an extra, significant constant overhead caused by a book-keeping machinery. This made them too slow in practice, decreasing interest on the area. By relying on EALC, we can avoid the book-keeping overhead, making it much faster than all alternatives. Below is a table comparing the amount of native operations (graph rewrites) required to reduce λ-terms in other implementations and Formality:

 Term | GeomOpt | GeomImpl | YALE | IntComb | Formality
  --- |     --- |      --- |  --- |     --- |       ---
 22II |     204 |       56 |   38 |      66 |        21
222II |     789 |      304 |  127 |     278 |        50
  3II |      75 |       17 |   17 |      32 |         9
 33II |     649 |      332 |   87 |     322 |        49
322II |    7055 |     4457 |  383 |    3268 |        85
223II |    1750 |     1046 |  213 |     869 |        69
 44II |    3456 |     2816 |  148 |    2447 |        89

Not only Formality requires orders of magnitue less native operations, but its native operations are much simpler. 

Unlike most proof languages, Formality doesn't include a datatype system, since not only it is very complex to implement, which goes against our second goal (simplicity), but can also cause more serious issues. For example, it was discovered that type preservation does not hold in Coq due to its treatment of coinductive datatypes [citation], and both Coq and Agda were found to be incompatible with Homotopy Type Theory due to the K-axiom, used by native pattern-matching on dependent datatypes [citation]. Instead, Formality relies on lambda encodings, an alternative to a datatype system that use plain lambdas to encode arbitrary data. While attractive, this approach wasn't adopted in practical type theories for several reasons:

1. There is no encoding that provides both recursion and fast pattern-matching.

    - Church encodings provide recursion, but pattern-matching is slow (`O(n)`).

    - Scott encodings provide fast pattern-matching, but no recursion.

    - Alternative encodings that provide both consume too much memory.

2. We cannot prove `0 != 1` with the usual definition of `!=`.

3. Induction is not derivable.

Formality solves the first issue by simply using Church encodings for recursion and Scott encodings for data, the second issue by collapsing universes, and the last issue by combining self-types and type-level recursion [citation]. It is well-known that, in most proof languages, collapsing universes and type-level recursion can lead to logical paradoxes, but the remarkable coincidence is that the same feature that allows the language to be compatible with optimal reductions, Elementary Affine Logic, also makes its type-theory sound in the presence of those [proof]. This allows the theory to be remarkably simple and powerful.

---

## Formality's Interaction Nets

(Adapt sections from http://docs.formality-lang.org/en/latest/theory/Formality-Net.html.)

TODO: mention our amazing plans and ideas for FPGA compilation.

## Elementary Affine Lambda Calculus

The Elementary Affine Lambda Calculus is a subset of the λ-calculus which is both terminating and compatible with the book-keeping free optimal reduction algorithm computed by Formality's Interaction Net. Its terms are defined by the following syntax:

```
name ::=
  <any alphanumeric string>


term ::=
  -- Lambdas
  (name) => term        -- lambda term
  term(term)            -- lambda application

  -- Boxes
  # term                -- boxed term
  dup name = term; term -- boxed duplication

  -- Language
  name                  -- variable
```

The first 3 constructors are the usual λ-calculus. The last two introduce boxes, which just wrap terms, and duplications, which copies the contents of a box. The EALC has the following reduction rules:

```
1. application: ((x) => a)(b) ~> [b/x]a
2. duplication: dup x = #a; b ~> [a/x]b
3. appdup-swap: ((dup x = a; b) c) ~> dup x = a; (b c)
4. dupdup-swap: dup x = (dup y = a; b); c ~> dup y = a; dup x = b; c
```

The first rule is the usual beta-reduction. The second rule allows us to copy the contents of a box. The last two are just permutations, allowing inner `dup`s to be exposed. An EALC term is said to be stratified when it follows the following conditions:

1. Lambda-bound variables can only be used at most once.

2. There must be exactly 0 boxes between a variable bound by a lambda and its occurrence.

3. There must be exactly 1 box between a variable bound by a duplication and its occurrence.

The first condition says that lambdas must be affine. The second condition says that lambdas can't substitute inside boxes. The third condition says that you can't unbox, or add boxes, to a term by duplicating it. Stratified EALC terms are strongly normalizing. Proof: 

- Define the level of a term as the amount of boxes surrounding it.

- Define the number of redex in a level by counting its reducible apps/dups.

- Prove that level is reduction-invariant.

- Prove that applications reduce the number of redexes in a level.

- Prove that duplications reduce the number of redexes in a level.

- Prove that reduction eventually consumes all redexes on level 0.

- Prove that, if level N is normalized, reduction eventually consumes all redexes on level N+1. 

TODO: mention impressive performance results obtained by runtime fusion:

- https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07

- https://medium.com/@maiavictor/calling-a-function-a-googol-times-53933c072e3a

## Formality Type Theory

Formality Type Theory (FM-TT) extends the EALC with type annotations. Its syntax is defined as follows:

```
name ::=
  <any alphanumeric string>

term ::=
  -- Lambdas
  (name : term) -> term -- lambda type (dependent product)
  (name : term) => term -- lambda term
  term(term)            -- lambda application

  -- Boxes
  ! term                -- boxed type
  # term                -- boxed term
  dup name = term; term -- boxed duplication

  -- Selfies
  $name term             -- self type
  new(term) term         -- self term
  %term                  -- self elimination

  -- Language
  name                   -- variable
  Type                   -- universe

file ::=
  name term term; clos   -- top-level definition
  \eof                   -- end-of-file
```

For simplicity, we'll use `(x0 : A0, x1, A1, ...) => t` as a synonym for `(x0 : A0) => (x1 : A1) => ... => t`, and `f(x,y,z)` as a synonym for `f(x)(y)(z)`. When `x` isn't used in `B`, we'll write `(x : A) -> B` as `A -> B` instead. We will also sometimes omit types of lambda-bound arguments when they can be inferred and write `(x0) => t` instead. Our typing rules are:

```
-- Lambdas

Γ |- A : Type    Γ, x : A |- B : Type
------------------------------------- lambda type
Γ |- (x : A) -> B : Type

Γ |- A : Type    Γ, x : A |- t : B
---------------------------------- lambda term
Γ |- (x : A) => t : (x : A) -> B

Γ |- f : (x : A) -> B    Γ |- a : A
----------------------------------- lambda application
Γ |- f(a) : B[x <- a]

-- Boxes

Γ |- A : Type
-------------- boxed type
Γ |- !A : Type

Γ |- t : A
------------ boxed term
Γ |- #A : !A

Γ |- t : !A    Γ, x : A |- u : B
-------------------------------- boxed duplication
Γ |- dup x = t; u : B

-- Selfies

Γ, s : $x A |- A : Type
----------------------- self type
Γ |- $x A : Type

Γ, |- t : A[x <- t]    Γ |- $x A : Type
--------------------------------------- self term
Γ |- new($x A) t : $x A

Γ |- t : $x A
------------------ self elimination
Γ |- t : A[x <- t]

-- Language

(x : T) ∈ Γ
----------- variable
Γ |- x : T

Ø
---------------- type-in-type
Γ |- Type : Type
```

Those primitives allow FM-TT to be a simple type-theory featuring inductive datatypes. For

**Theorem: induction principle.**

Induction can be proved on FM-TT as follows:

```
Nat : Type
  $ self
  ( P    : Nat -> Type
  , succ : (r : Nat, i : P(r)) -> P(succ(r))
  , zero : P(zero)
  ) -> P(self)

succ : (n : Nat) -> Nat
  new(Nat) (P, succ, zero) => succ(n, (%n)(P, succ, zero))

zero : Nat
  new(Nat) (P, succ, zero) => zero

nat_ind : (n : Nat, P : Nat -> Type, s : (n : Nat, i : P(n)) -> P(succ(n)), z : P(zero)) -> P(n)
  (n) => %n
```

This defines an inductive lambda encoded datatype, Nat. The key to understand why this works is to realize that `Nat` is defined as its own induction scheme. Mutual recursion is needed because the induction scheme of any datatype refers to its constructors, which then refer to the defined datatype. Self types are used because the type returned by the induction scheme can refer to the term being induced. Together, those two components allow us to define inductive datatypes with just plain lambdas, and to induce on `n : Nat` you don't require any additional proof, all you need to do is eliminate its self-type with `%n`. Note that this `Nat` is complicated by the fact it is recursive (Church encoded). We could have a non-recursive (Scott encoded) `Nat` as:

```javascript
Nat Type
  $ self
  ( ~P   : Nat -> Type
  , succ : (n : Nat) -> P(succ(n))
  , zero : P(zero)
  ) -> P(self)

succ(n : Nat) -> Nat
  new(Nat) (~P, succ, zero) => succ(n)

zero Nat
  new(Nat) (~P, succ, zero) => zero

nat_ind(n : Nat, ~P : Nat -> Type, s : (n : Nat, i : P(n)) -> P(succ(n)), z : P(zero)) -> P(n)
  (n) => %n
```

Which is simpler and has the benefit of having constant-time pattern-matching. Even simpler types include Bool:

```
Bool Type
  $ self
  ( ~P    : Bool -> Type
  , true  : P(true)
  , false : P(false)
  ) -> P(self)

true Bool
  new(Bool) (~P, true, false) => true

false Bool
  new(Bool) (~P, true, false) => false

bool_induction(b : Bool, ~P : Bool -> Type, t : P(true), f : P(false)) -> P(b)
  (b) => (%b)
```

Unit:

```
Unit Type
  $ self
  ( ~P    : Unit -> Type
  , unit  : P(unit)
  ) -> P(self)

unit Unit
  new(Unit) (~P, unit) => unit

unit_induction(b : Unit, ~P : Unit -> Type, u : P(unit)) -> P(u)
  (u) => (%u)
```

And Empty:

```
Empty Type
  $ self
  ( ~P : Empty -> Type
  ) -> P(self)

empty_induction(b : Empty, ~P : Empty -> Type) -> P(b)
  (e) => (%e)
```

Notice how, for example, `bool_induction` is equivalent to the elimination principle of the Bool datatype on Coq or Agda, except that this was obtained without needing a complex datatype system to be implemented as a native language feature. This type theory is capable of expressing any inductive datatype seen on traditional proof languages. For example, Vector (a List with statistically known length) can, too, be encoded as its dependent elimination:

```
Vector(~A : Type, len : Nat) -> Type
  $self
  ( ~P    : (len : Nat) -> Vector(A, len) -> Type
  , vcons : (~len : Nat, x : A, xs : Vector(A, len)) -> P(succ(len), vcons(~A, ~len, x, xs))
  , vnil  : P(zero, vnil(~A))
  ) -> P(len, self)

vcons(~A : Type, ~len : Nat, head : A, tail : Vector(A, len)) -> Vector(A, succ(len))
  new(Vector(A, succ(len))) (~P, vcons, vnil) => vcons(~len, head, tail)

vnil(~A : Type) -> Vector(A, zero)
  new(Vector(A, zero)) (~P, vcons, vnil) => vnil
```

And an identity type for propositional equality is just the J axiom wrapped by a self type:

```
Id(A : Type, a : A, b : A) -> Type
  $self
  ( ~P   : (b : A, eq : Id(A, a, b)) -> Type
  , refl : P(a, refl(~A, ~a))
  ) -> P(b, self)

refl(~A : Type, ~a : A) -> Id(A, a, a)
  new(Id(A, a, a)) (~P, refl) => refl
```

We're also able to express datatypes that traditional proof languages can't. For example, intervals, i.e., booleans that can only be eliminated if both provided cases are equal, can be written as:

```
Interval Type
  $self
  ( ~P : (i : Interval) -> Type
  , I0 : P(i0)
  , I1 : P(i1)
  , SG : I0 == I1
  ) -> P(self)

i0 Interval
  new(Interval) (~P, i0, i1, sg) => i0

i1 Interval
  new(Interval) (~P, i0, i1, sg) => i1
```

And we can easily prove troublesome theorems like `0 != 1`: 

```
true_isnt_false(e : Id(Bool, true, false)) -> Empty
  (%e)(~{b, e.b} (%e.b)(~{b}Type, Unit, Empty), unit)
```

That much type-level power comes at a cost: if Formality was based on the conventional lambda calculus, it wouldn't be sound. Since it is based on EALC, a terminating untyped language, its normalization is completely independent from types, so it is impossible to derive paradoxes by exploiting non-terminating computations. A good exercise is attempting to write `λx. (x x) λx. (x x)` on EALC, which is impossible. The tradeoff is that EALC is considerably less computationally powerful than the lambda calculus, imposing severe restriction on how terms can be duplicated. This limitation isn't very problematic in practice since the language is still capable of implementing any algorithm a normal programming language, but doing so involves a resource-usage discipline that can be annoying to users. Formality is currently being implemented in Agda, on which the consistency argument will be formalized.

## Formality Core

Formality Lang extends Formality with native pairs and numbers. That is done for efficiency reasons: native pairs use 1/3 of the space of lambda encoded pairs, and native numbers can be stored unboxed on interaction net nodes, using orders of magnitude less space than lambda encoded numbers.

## Formality Lang

Formality Lang adds a series of user-friendly syntax sugars on top of Formality Core, making it much more usable and accessible. It includes syntaxes for datatype definitions and pattern-matching, loops, recursion, branching and so on. Example:

```javascript
T Nat
| succ(n : Nat)
| zero

T Bool
| true
| false

T Unit
| unit

T Empty

T Vector (A : Type) (len : Nat)
| vcons(~len : Nat, head : A, tail : Vector(A, len)) : Vector(A, succ(len))
| vnil                                               : Vector(A, zero)

T Equal (A : Type, a : A) (b : A)
| eq_refl : Equal(A, a, a)

T Interval
| i0
| i1
| i0 == i1

true_isnt_false(e : Id(Bool, true, false)) -> Empty
  case e
  | eq_refl => unit
  : case e.b
    | true  => Unit
    | false => Empty
    : Type
```

TODO: more examples.
