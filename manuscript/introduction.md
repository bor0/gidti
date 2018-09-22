# Introduction

Writing correct code in software engineering is a complex and expensive task, and too often our written code produces inaccurate or unexpected results. There are several ways to deal with this problem. In practice the most common approach is to write _tests_, which means that we are writing more code to test our original code. However, these tests can only ever detect problems in specific cases. As Edsgar Dijkstra noted[^intron3], "testing shows the presence, not the absence of bugs". A less common approach is to find a _proof of correctness_ for our code. A software proof of correctness is a logical proof that the software is functioning according to given specifications. With valid proofs, we can cover all possible cases and be more confident that the code does exactly what it is intended to do.

Idris is a general purpose functional[^intron1] programming language that supports dependent types. The features of Idris are influenced by Haskell, another general purpose functional programming language. However, Haskell does not support dependent types. Idris shares many features with the programming language Haskell, especially in the part of syntax and types, where Idris has a more advanced type system. There are several other programming languages that have support for dependent types[^intron2], however, I chose Idris for its very readable syntax.

The first version of Idris was released in 2009 and is developed by The Idris Community. Seen as a programming language, it is a functional programming language implemented with dependent types. Seen as a logical system, it implements intuitionistic type theory, which we will cover in detail in the following chapters. We will show how these two views relate to each other in section 4.2, with the Curry-Howard correspondence.

Idris allows us to express mathematical statements. By mechanically examining these statements, it helps us find a formal proof of a program's formal specification.

To fully understand how proofs in Idris work we will start with the foundations by defining: formal systems, classic mathematical logic, lambda calculus, intuitionistic logic, and type theory (which is a more "up-to-date" version of classical mathematical logic).

[^intron1]: The core concept of functional programming languages is the _pure function_. These are functions that depend only on their inputs and do not have side effects.

[^intron2]: Several other languages with dependent types support are Coq, Agda, Lean.

[^intron3]: J.N. Buxton and B. Randell, eds, Software Engineering Techniques, April 1970, p. 16. Report on a conference sponsored by the NATO Science Committee, Rome, Italy, 27–31 October 1969