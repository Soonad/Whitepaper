# Moonad: a Peer-to-Peer Operating System

**Abstract.** We present the design of a minimal, decentralized operating system, aiming to solve different problems with existing computer architectures, programming languages, package managers and operating systems. Instead of assembly, its low-level machine language is based on a massively parallel graph-reduction system, with an accompanying new hardware architecture. Instead of C, it is built on top of a proof and programming language which can use Curry-Howard's isomorphism to state and prove mathematical theorems about its own programs. Instead of usual package manager, a type-queryable repository of code is used. A distributed file system and smart-contract network are built-in for varying usages, and state-of-art insights from programming language theory are used. This leads to the design of a massively parallel operating system which improves existing ones in several senses, which will elaborate through the paper.

## Why?

Moonad comes from the realization that our computers, operating systems, programming languages and package managers are much more flawed and primitive than they should be. The last few decades of research gave us breakthroughs on all those fields, yet most of those are barely used in practice. For every software-related fiasco news you see, there is often an existing technology capable of preventing it trivially, yet we still struggle with those problems in a daily basis. Moonad aims to bring all those insights into a real product that "just works", in the form of a new, modern operating system. We'll now list several of those serious, yet solveable problems with existing technology.

From a **security** perspective, we had a single, small malicious package [cause an entire ecossystem to collapse](https://www.npmjs.com/package/left-pad), [twice](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident). We have been dealing with SQL and JavaScript injections since the birth of the internet. It has been 2k years since the first mathematical proof was published, yet we still lose absurd amounts of money for bugs like the [heartbleed](http://heartbleed.com) and [TheDAO](https://medium.com/@ogucluturk/the-dao-hack-explained-unfortunate-take-off-of-smart-contracts-2bd8c8db3562). Faulty software and malwares are widespread on the industry, sometimes causing [airplanes to crash](https://www.ndtv.com/world-news/boeing-737-max-crash-was-software-to-blame-and-what-comes-next-2007893), hospitals to [stop operating](https://www.forbes.com/sites/leemathews/2018/11/28/ransomware-attack-disrupts-emergency-services-at-ohio-hospital/). Imagine a world without bugs and malwares? As far-fetched as that seems, this is actually perfectly possible with existing technologies, if we just use them. Here, we will explain how an operating system built on top of a proof language can improve or even solve all those problems.

From a **performance** perspective, our most widely used programming languages have several, unavoidable inefficiencies that, by design, make them many times slower than existing alternatives. Our CPUs are built using sequential models of computation that have hit a bottleneck and stagnated. The recent rise of GPUs for high-performance computing doesn't fully solve this problem because the way we're used to build software still relies profoundly on sequential idioms such as `for-loops`. Even if our languages were more GPU-friendly, the necessity for ASICs in applications such as crypto-currency mining shows that even GPUs are still not making perfect use of the underlying computing capabilities of our physical world. Imagine if we had processors for arbitrary programs that couldn't be subsumed by ASICs? Imagine if our programming languages could compute optimally in parallel without any programmer effort? Here, we will propose a general-purpose, massively parallel computer architecture that is almost optimal up to the laws of physics, in a sense that will be made precise later on, as well as detail how future programming languages can be designed to make the best use of those capacities.

From a **productivity** perspective, we are extremelly inefficient. When it comes to features, our programming languages lag behind theory. Java took decades to implement lambdas. JavaScript took years to figure out monads (and failed). Modern languages such as Go struggle with aspects as simple as polymorphic datatypes. On top of that, we never wasted as much developer time. By searching, for example, [`"def mergesort" filetype:py`](https://www.google.com/search?ei=zmOOXNKxDY_E5OUPiLq5mAs&q=%22def+mergesort%22+filetype%3Apy&oq=%22def+mergesort%22+filetype%3Apy&gs_l=psy-ab.3..0i71l8.7645.7645..7650...0.0..0.0.0.......0....2j1..gws-wiz.6UEn0LSx0FI), we get 454 results: thousands of re-implementations of the same particular function, in the same particular language; and chances are this number is hugely underestimated. Imagine if all that effort was spent somewhere else? Our developers could be creating new technology, researching space travel, solving climate change, curing cancer, yet they are, almost without exception, reinventing the wheel. Here, we will explain how several type-theory insights can lead us to unprecedented developer productive gains.

Note that this kind of improductivity doesn't affect only developers. Mathematical publishing is in an even worse state. In the apex of the compute era, findings and publications are still communicated through primitive, computer-inaccessible, text-based languages. Once you realize proofs are just code, the entire academic system becomes a huge, inefficient and paid package manager. Citations can be seen as manual imports and peer-reviewing is just an error-prone, human-laboured type-checker. Errors in earlier works propagate upwards, paywalls make knowledge private. The entire system extremely fragile, bureaucratic and inefficient. By building an operating system on top of a proof language, the same ideas that improve developer productivity also apply to mathematics as a whole, leading to an unprecedented leap in scientific progress. 

Finally, Moonad has many other minor aspects that set it apart from existing operating systems. For example, instead of local files, it uses decentralized storages (IPFS and Swarm). This means that, whenever you save a file locally, it is backed up, encrypted, in a network of computers. This has many consequences. For one, there isn't such a thing as "local files" anymore: once you save a file, you're able to access it from anywhere. If your hardware is damaged, nothing is lost. Backups become obsolete. Sharing a file or a photo with someone becomes as easy as sending its path. This also has means that publishing a site is straightforward: all you have to do is make its directory public. Naturally, DDOSes also become a thing of the past.

Moonad also comes with Ethereum built-in. This has many consequences. For example, instead of local users, we have Ethereum accounts. This makes website logins and passwords obsolete, since they all can use signatures from the same crypto account. Similarly, the entire credit-card payment system is replaced by direct crypto payments. You don't need to trust a site not to misuse or steal your credentials. Credit-card disputes become a thing of the past. Ethereum's benefits go much beyound payments, though. Important business agreements can be formalized through automatically enforced smart-contracts. Virtual assets such as items in online games are allowed to have a much more real value, since there isn't a centralized company capable of "changing the rules of the game" or even turning it down. Finally, any kind of censure in applications such as torrent websites become impossible.

Now, we will explain several components of the operating system, and elaborate on how they solve existing problems

## Nasic: a massively parallel, low-level machine language

### Motivation

Operating systems often make a distinction between high and low-level languages. This enables a separation of concerns: while high-level languages focus on usability and user-friendliness, their low-level counterparts focus on speed and efficiency, often resembling the underlying computer architecture, which is usually be modelled by Turing machines. While that model has worked well for the last few decades, it hit a bottleneck as the speed of sequential CPUs stagnated. Meanwhile, the computing power of GPUs, FPGAs, ASICs and other parallel architectures keeps growing exponentially, but, sadly, modern programming languages can't make use of those advancements. Functional programming languages, which are based on alternative model, the lambda calculus, were sold as a solution, but never fulfilled its promisses due to the inherent complexity of beta-reduction. 

In 1997, a very simple graph-rewrite system with only 3 symbols and 6 rules has been shown to be a universal model of computation [1]. This system, interaction combinators, is remarkable for having the best properties of Turing machines and the lambda calculus. Like the former, it can be evaluated as a series of atomic, local operations with a clear physical implementation. Like the later, it is inherently parallel, but in a more robust manner, admitting desirable computational properties such as strong confluence, optimal sharing, zero-cost garbage-collection and so on. For those reasons, we use a slightly modified version of symmetric interaction combinators, Nasic, as our lowest-level machine language. This allows our applications to target a computing model that is ready for the upcoming future of massively parallel architectures.

### Specification

While conventional low-level languages describe they programs as a series of statements in an assembly-like language, Nasic programs are simply graphs of a specific format, on which every node has exactly 3 outgoing edges, and is annotated with a symbolic label (e.g., a 32-bit int).

(image)

Any fully connected arrangement of those nodes forms a valid Nasic program. For example, those are valid Nasic programs:

(image)

The position of edges is important. For example, those graphs, while isomorphic, denote two different programs:

(image)

For that reason, the ports from which edges come must be named. The port at the top of the triangle is the `main` port. The one nearest to it in counter-clockwise direction is the `aux1` port. The other one is the `aux2` port.

(image)

Moreover, there are 2 computation rules:

(image)

Those rewrite rules dictate that, whenever a sub-graph matches the left side of a rule, it must be replaced by its right side. For example, the graph below:

(image)

Is rewritten as:

(image)

Because (...). Rewritteable nodes are called redexes, or active pairs. 

This completes Nasic's specification.

### Explanation

As unintuitive as it may seem, this simple graph-rewrite system is extremelly powerful, because it captures the two fundamental rules of computation: annihilation and commutation. The first rule covers a wide array of computational phenomena such as communication, functions, datatypes, pattern-matching and garbage-collection. The second rule covers memory allocation, copying and repetitive behaviors such as loops and recursion. We will explain how it works through a set of examples:

(...)

## Formality: a minimal programming and proof language

### Motivation

While it has been known for almost a century that mathematics and programming are, actually, the same thing - an observation known as the Curry-Howard correspondence - they're still regarded and practiced as separate subjects. This is harmful for both sides. In one side, mathematicians have the power of rigor: there is nothing more undeniably correct than a mathematical proof. Yet, ironically, those proofs are often written, checked and reviewed by humans in an extremelly error-prone process. On the other side, programmers have the power of automation: there is nothing more capable of performing a huge set of tasks without making a single mistake than a computer program. Yet, ironically, most of those programs are written in an unrigorous, bug-ridden fashion. Formality lies in the intersection between both fields: it is a programming language, on which developers are capable of implementing everyday algorithms and data structures, but it is also a proof language, on which mathematicians are capable of proposing and proving theorems. 

Formality aims to be as simple as technically possible: our reference implementation takes about 500 lines of code. This doesn't make it less powerful: due to its inductive lambda encodings, Formality is capable of expressing every datatype included in a traditional proof language. This extreme expressivity also makes it very efficient, since it is capable of being compiled to all sorts of Nasic graphs. As a consequence, it isn't terminating: users must pick a specific termination checker if they worry about mathematical consistency. Different termination checkers should forbid and allow different programming styles. Users not interested by mathematical consistency can simply dispense it, though, getting to work with a very high-level, massively parallel functional programming language.

### Specification

Formality can be described in a single sentence: extended-scope, Church-style Calculus of Constructions with global recursive definitions and optional computational irrelevance annotations. To elaborate what this means, let's start by defining its initial syntax:

```haskell
name ::=
  (any character in the set "a-z", "A-Z", "_", ".", "~") 

term ::=
    Type               -- type of types            
  | [name] term        -- unannotated function
  | [name : term] term -- annotated function
  | {name : term} term -- function type
  | (term term)        -- function application
  | name               -- variable
```

(... reduction rules ...)

(... typing rules ...)


### Explanation

## References

[1] https://pdfs.semanticscholar.org/6cfe/09aa6e5da6ce98077b7a048cb1badd78cc76.pdf
