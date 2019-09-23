# Moonad: a Peer-to-Peer Operating System

Victor Maia, John Burnham

**Abstract.** We present Moonad, a peer-to-peer operating system. Its machine
language, FM-Net, consists of a massively parallel, beta-optimal
graph-reduction system based on interaction nets. Its user-facing language,
Formality, is purely functional and features a powerful, elegant type-system
capable of stating and proving theorems about its own programs through
inductive λ-encodings. Algorithms, theorems and proofs are stored in an
immutable, decentralized database of terms, Formbase, capable of tracking
imports and citations. Online interactions are made exclusively through a
worldwide, ever-growing log of events, Unilog, under Nakamoto Consensus. An
decentralized application platform is then designed as a transaction function,
Phos, over Unilog events. On top of those, we build a front-end interface,
Moonad, where users can create, deploy and use decentralized applications, pose
and solve mathematical problems, and exchange scarce virtual assets.

## Motivation

Moonad comes from the realization that computers, operating systems and
programming languages are lagging behind theory. Through the last decades, we
have seen significant scientific breakthroughs that are barely used in
practice. For example, we succesfuly built Ethereum, a worldwide computer
relying on a byzantine-tolerant chain of signed transactions, yet most sites
still store passwords in plaintext and demand your credit card credentials for
online purchases. Similarly, we have proof languages capable of unifying the
field of mathematics and programming as one, yet our software was never so
vulnerable. And we also had tremendous progress in compiler design, specially
through linear types, that is barely exploited. We believe this can be
attributed to presentation: while we have the tech, we don't have the user
experience. Moonad aims to assemble all those amazing technologies as a unified
package that "just works", to become the Windows/MacOS of decentralized
operating systems, and to ultimately answer the question: "how can type-theory
be used to make the world a better place?"

### Example: formal proofs for absolute software security

Not long ago, we had a single, small malicious package cause the entire web
ecosystem [to collapse][left-pad], [twice][event-stream]. More than [300,000
sites](https://wpvulndb.com/statistics) online today are vulnerable to SQL
injections. Bugs like [heartbleed](http://heartbleed.com) and [TheDAO][DAO]
cause enormous monetary losses. A single [CPU exploit][meltdown-spectre] slowed
processors by 30%, slowing down our progress as a society. Faulty software
routinelly kills people, causing [airplanes to crash][737-max] and hospitals to
[stop operating][ohio-hospital].

Moonad's user-facing language, Formality, has a type-system so expressive that
is capable of stating and proving mathematical theorems about its own programs.
In other words, its types can be seen as a language of specfications that can
be mechanically checked by the compiler, allowing you to enforce arbitrary
guarantees on your program's behavior. For example, in an untyped language, you
could write an algorithm such as:

```javascript
// Returns the element at index `idx` of an array `arr`
function get_at(i, arr) {
  ... code here ...
}
```

But this has no guarantees. You could call it with nonsensical arguments such
as `get_at("foo", 42)`, and the compiler would do nothing to stop you, leading
to (possibly catastrophical) runtime errors. Traditional typed languages
improve the situation, allowing you to "enrich" that definition with simple or
even polymorphic types, such as:

```javascript
function get_at<A>(i : Num, arr : Array<A>) : A {
  ... code here ...
}
```

This has some important guarantees. For example, it is impossible to call
`get_at` with a String index, preventing a wide class of runtime errors. But
this guarantee is incomplete. You can still, for example, call `get_at` with an
index out of bounds. In fact, a similar error caused the catastrophic
[heartbleed](http://heartbleed.com) vulnerability. Formality types are
arbitrarily expressive, allowing you to capture all demands you'd expect from
`get_at`:

```javascript
get_at*L : {A: Type, i: Fin(L), arr: Vector(A, L)} -> [x: A ~ At(A,L,i,arr,x)]
  ... code here ...
```

Here, we use a bound, `L`, to enforce that the index is not larger than the
size of the array, and we also demand that the returned element, `x: A`, is
exactly the one at index `i`. This specification is complete, in the
mathematical sense. It is not only impossible to get a runtime error by calling
`get_at`, but it is also impossible to write `get_at` incorrectly to begin
with. Note that this is different from, for example, `assert`s or bound
checking. Here, the compiler statically reasons about the flow or your program,
assuring that it can't go wrong with zero runtime costs.

In a similar fashion, you could use types to ensure that the balance of a
smart-contract never violates certain invariants, or that an airplane
controller won't push its nose down to a crashing position. Of course, you
don't *need* to write types so precise. If your software doesn't demand
security, you could go all way down to having fully untyped programs. The point
is that the ability of expressing properties so precisely is immensely
valuable, specially when developing critical software that demands all the
security one can afford.

### Example: a language that is fast "by design"

Some languages are inherently slow, by design. JavaScript, for example, is
slower than C: all things equal, its mandatory garbage collector will be an
unavoidable disadvantage. Even C, arguably the fastest modern language, has its
flaws, as its ultra-sequential model of computation has hit a hard barrier.
Formality is meant to be as fast as theoretically possible. For example, it has
affine lambdas, allowing it to be garbage-collection-free. It has a strongly
confluent interaction-net runtime, allowing it to be evaluated in massively
parallel architectures. It is a functional language that doesn’t require bruijn
bookkeeping, making it the fastest “closure chunker” around. It is lazy, has a
clear cost model for blockchains and a minuscle (448 LOC) runtime, making it
extremelly portable.

If we designed our operating system using a classical language, it'd be fast
today, but, on the long-term, it would become obsolete. Instead, we decided to
rely on a new model of computation that isn't as fast today, since it is
competing against decades of optimizations, but that has the best computational
characteristics imaginable, making it extremelly future-proof and giving it a
virtually unbounded room for improvements. In other words, Formality is fast
"by design".

### Example: TODO

TODO

## FM-Net: a massively parallel, low-level machine language

### Motivation

Operating systems often make a distinction between high and low-level languages.
This enables a separation of concerns: while high-level languages focus on
usability and user-friendliness, their low-level counterparts focus on speed and
efficiency, often resembling the underlying computer architecture, which is
usually be modelled by Turing machines. While that model has worked well for the
last few decades, it hit a bottleneck as the speed of sequential CPUs stagnated.
Meanwhile, the computing power of GPUs, FPGAs, ASICs and other parallel
architectures keeps growing exponentially, but, sadly, modern programming
languages can't make effective use of those advancements. Functional programming
languages, which are based on alternative model, the lambda calculus have been
sold as a solution, but never fulfilled its promises, partly due to the inherent
complexity of beta-reduction.

In 1997, a very simple graph-rewrite system with only 3 symbols and 6 rules was
shown to be a universal model of computation [^2]. This system,
interaction combinators, is remarkable for having the best properties of Turing
machines and the lambda calculus. Like the former, it can be evaluated as a
series of atomic, local operations with a clear physical implementation. Like
the later, it is inherently parallel, but in a more robust manner, admitting
desirable computational properties such as strong confluence, optimal sharing,
zero-cost garbage-collection and so on. For those reasons, we use a slightly
modified version of symmetric interaction combinators, Formality-Core as our
lowest-level machine language. This allows our applications to target a
computing model that is ready for the upcoming future of massively parallel
architectures.

Our implementation has advantages over previous interaction net graph-reduction
systems (such as the Bologna Optimal Higher-order Machine) in that it requires
no book-keeping, requires only 128 bits per lambda or pair, has unboxed 32-bit
integers and constant-time beta reduction. This is possible because
Formality-Core is not based on classic logic, but, instead, Elementary Affine
Logic. All programs written in Formality are guaranteed to halt, and enjoy key
computational characteristics that differ from classic functional languages.
This directly enables many of the runtime features and optimizations (such as
no book-keeping) described above, although without loss of generality as
Formality-Core can model Turing-Complete programs via corecursion. In other
words, Turing-Complete computation can be performed by writing an iteration
step function in Formality-Core and looping this function infinitely in your
runtime system. We view this design decision as a principled acknowledgement
that in practice no real software is Turing-Complete; actual physical systems
can only be finite state machines.

### Specification

TODO: port from http://docs.formality-lang.org/en/latest/theory/Formality-Net.html

## Formality: a minimal programming and proof language

### Motivation

The Curry-Howard correspondence states that there is a structural congruence
between computer programs and proofs in formal logic.  While in theory this
implies that software developers and mathematicians are engaged in substantially
the same activity, in practice there are many barriers preventing the
unification of the two fields. Programming languages with type systems
expressive enough to take advantage of Curry-Howard, such as Haskell, Agda and
Coq generally have substantial runtime overhead, which makes them challenging to
use, particularly at the systems programming level. On the other hand, systems
languages such as Rust, Nim or ATS, have shown that type systems can greatly
enhance usability and correctness of low-level software, but either have type
systems too weak to fully exploit Curry-Howard (Rust, Nim), or introduce
syntactic complexity that limits adoption (ATS). In other words, there are lots
of slow languages with great type systems and lots of fast languages with lousy
type systems.

As programmers and as mathematicians, we don't want to compromise.
On one side, mathematicians have the power of rigor: there is nothing more
undeniably correct than a mathematical proof. Yet ironically, those proofs are
often written, checked and reviewed by humans in an extremelly error-prone
process. On the other side, programmers have the power of automation: there is
nothing more capable of performing a huge set of tasks without making a single
mistake than a computer program. Yet, equally ironically, most of those programs
are written in an unrigorous, bug-ridden fashion. Formality lies in the
intersection between both fields: it is a programming language in which
developers are capable of implementing everyday algorithms and data structures,
but it is also a proof language, on which mathematicians are capable of
proposing and proving theorems.

### Specification

**TODO**

## Formbase: a global, immutable database of terms

Formbase is just a simple API:

- save_file(name, code): saves a file on the server, returns an unique name.

- load_file(unique_name): loads a file from the server.

- load_citations(unique_name): loads the list of files that import this one.

**TODO**

## Unilog: a public log of human events

Unilog is essentially Bitcoin without the coin. It is an ever-growing log of transactions (strings), ordered by Nakamoto Consensus.

**TODO**

## Phos: an application platform

Phos is a transaction function, `phos : Transaction -> PhosWorld -> PhosWorld`, that interprets Unilog events as 

**TODO**

## Moonad

An user-facing browser that allows us to access applications by typing Term@X on the URL bar. If the term has a App type (essentially an effect monad that gives it access to system resources), an application is rendered. Applications can be offline, or they can query the public stream of events, allowing online interactions.

**TODO**

## MoonadStore

The default, "home-page", feature-rich DApp that serves for multiple purposes.

- Sending and Receiving Phos Coins (wallet)

- Finding and publishing cool applications

- Browsing the "blockchain" (i.e., UnilogExplorer)

- Placing bounties on types (Provit)

**TODO**


## References

[left-pad]: https://www.npmjs.com/package/left-pad
[event-stream]: https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident
[DAO]: https://medium.com/@ogucluturk/the-dao-hack-explained-unfortunate-take-off-of-smart-contracts-2bd8c8db3562
[meltdown-spectre]: https://www.intelligonetworks.com/blog/meltdown/spectre-will-cost-the-world-30-system-performance
[737-max]: https://www.ndtv.com/world-news/boeing-737-max-crash-was-software-to-blame-and-what-comes-next-2007893
[ohio-hospital]:https://www.forbes.com/sites/leemathews/2018/11/28/ransomware-attack-disrupts-emergency-services-at-ohio-hospital
[mergesort]: https://www.google.com/search?ei=zmOOXNKxDY_E5OUPiLq5mAs&q=%22def+mergesort%22+filetype%3Apy&oq=%22def+mergesort%22+filetype%3Apy&gs_l=psy-ab.3..0i71l8.7645.7645..7650...0.0..0.0.0.......0....2j1..gws-wiz.6UEn0LSx0FI,

[^1]: Some readers might object here that while formal proofs can ensure that a
  program's implementation matches its specification, it cannot eliminate errors
  inherent to the specification. This is true, unless there is a
  formalizable meta-specification of the specification, in which case we
  can construct a proof showing consistency between the two. Otherwise, the
  specification of the specification is the mind of the programmer or user
  themselves, and in this domain there may be many bugs which are not amenable
  to elimination via formal methods.

[^2]: Yves Lafont, *Interation Combinators*, Institut de Mathematiques de Luminy, 1997 (https://pdfs.semanticscholar.org/6cfe/09aa6e5da6ce98077b7a048cb1badd78cc76.pdf)

