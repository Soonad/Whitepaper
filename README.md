# How to improve the world with Moonad

1. Build a massively parallel processor, [LPU](https://github.com/MaiaVictor/parallel_lambda_computer_tests), based in an alien tech called [SIC](https://github.com/MaiaVictor/symmetric-interaction-calculus).

2. Build on top of it a minimalist proof language, [Formality](https://github.com/moonad/Formality), that will merge mathematics and programming.

3. Build a Linux-like operating system, [Moonad](https://github.com/moonad/moonad), replacing CPU/ASM/C by LPU/SIC/Formality.

4. Integrate with distributed storages [IPFS](https://ipfs.io)/[Swarm](https://swarm-guide.readthedocs.io/en/latest/introduction.html), and smart-contract platforms like [Ethereum](https://www.ethereum.org).

5. Build a worldwide, queryable, type-indexed repository of algorithms, proofs and DApps (like [Hoogle](https://www.haskell.org/hoogle/)).

6. People start building things on it and it will go exponential because it is better than anything that ever existed.

7. ...?

8. At this point the cancer is probably cured, mars is colonized global warming is reversed.

Wait, what?

## Why is the world broken and how can you fix it?

Programming is broken in many senses. We had a single, small malicious package cause an entire ecossystem to collapse. We got consumer processors with 4352 cores, yet most of our code is still single-threaded. It has been 2k years since the first mathematical proof was published, yet we still lose [millions](http://heartbleed.com) and [millions](https://medium.com/@ogucluturk/the-dao-hack-explained-unfortunate-take-off-of-smart-contracts-2bd8c8db3562) and [millions](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident) due to bugs that could be avoided by trivial proofs. We fragmented ourselves into dozens of different ecosystems that can't share code. Our languages lag behind theory. Java took decades to introduce lambdas. JavaScript took years to figure out monads (and failed). MySQL injection is a thing. PHP exists. Buying things online require you to give your credit card number. Developers spend most of their times reinventing the wheel and debugging errors that would never happen if we used better tools. I could go on, but you get it. We are, as a species, extremely incompetent at programming. Ants would be ashamed of us.

Mathematics is also broken because, in the apex of the compute era, publications still rely on a primitive textual system. Proofs are just mentally-evaluated pseudocode. Citations are manual imports locked behind paywalls. Peer reviewing is just a huge human-laboured, error-prone type-checker. Errors in earlier works propagate upwards. The entire system extremely fragile. Findings never come with accompanying implementations. Field-specific jargon hinder cross-field communication. If I represented humanity in an intergalactic interview, I'd never mention the academia.

In short, I believe both of those fields are due to a serious revamp, and it is time to do that. Don't underestimate how much we would leap forward if we could suddenly make developers more efficient, processors faster, scientific practice more widespread. Done properly, this could, who knows, cure of cancer, end world hunger, reverse global warming. After all, hard problems  are often solved more by plain technological advancement than by direct investment. Fortunatelly, all the problems pointed here only exist due to social limitations, not technical ones. They are solved! So why are things the way they are?

I want to fix the world by building an operating system I'm calling Moonad, which would improve dramatically the quality of life of our programmers and mathematicians, thus providing benefits for the entire humanity. It involves many building blocks, each one solving one particular problem with existing computer architectures, programming languages, package managers and so on. Most of those solutions weren't invented by me. Haskell's Hoogle is amazing, Coq's inductive datatypes are brilliant, Agda's proof search is dark magic, we have stream fusion, super-compilation, monads. Those tools already exist, they are clever, and they work. They just aren't used as much as they should. The only problem, it seems, is the industry lagged too much behind theory. We need someone to put all those findings together into a great product that feels inuitive and just works, like iOS. And that's how I want to fix the world.

## Moonad's components

I'll now list all the components of Moonad, explain how they work and what particular problem they solve.

(I'm still writing this part of the manifesto and it will take a while...)

### SIC

(Explain how register/stack machines are stupid because they are sequential and how human's intuitions about computers not necessarily correspond to the inherent nature of computation. Explain how interaction nets relates with massively parallel, optimal λ-calculus reductions. Explain what is the relationship between symmetric interaction combinators and interaction nets. Explain how one could build "general purpose ASIC", which is an obviously nonsense definition, but in the sense it'd be a general purpose computer on which properly implemented algorithms can't be improved by building ASICs since they are already physically optimal.)

### LPU

(Explain our computer architectures are stupid because information must travel a huge space from main memory to the local caches to the ALUs, clocks impose an unecessary sequentialistm, etc. Explain how memory and processors could be fused into a single grid of millions of nano-mem-cpu-cores, each one operating like a small actor, responsible for holding a SIC node, communicating with nearby nodes and performing parallel graph reductions.)

### Formality

(Explain how our programming languages are stupid because they still don't get types right, explain how formal proofs can be used to guarantee absolute correctness and replace proofs, address the "but specifications can have bugs" point, explain calmly how making such language the foundation of an operating system would make it literally immune to have viruses, malwares and all those stupid things that only exist because humans suck at programming. Explain that one doesn't need to write Agda code to use formal proofs, and we can absolutely make a Python interface for them.)

### IPFS/Swarm

(Explain how replacing local operating system by encryped decentralized operating system would make you able to access your files anywhere without paying anyone, it would allow you to "deploy" a site by merely sending your friends the path where you saved it, how it would, in short, basically merge all computers into a single universal filesystem.) 

### Ethereum

(Explain how centralized internet services are stupid because they give so much power to companies, how things like Uber, Youtube, Facebook should have DAOs of public domain, owned by nobody, providing value to the humanity as a whole...)

### Agda-inspired IDE

(Explain all the amazing, powerful code editing features we can have once types are strong enough, including things like proof search which is basically making computers do the job of a programmer...)

### Foogle

(Explain how those could allow us to build a worldwide, type-indexed, queryable package manager and repository of algorithms, proofs and DApps. Explain how this would make the entire academia obsolete, because scientific publications would be just normal files, proofs would be just code, citations would be just imports. Explain how it would save an unmeasurable amount of money because people would finally stop reinventing the wheel in different languages because they don't know someone else already solved the same problem he/she is trying to solve. Stress how much human work is wasted reinventing the wheel, and how such a thing would change everything.)
