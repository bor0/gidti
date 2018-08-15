# Introduction

Software proof of correctness is a proof that the software is functioning according to the given specifications. Writing correct code in software engineering is a complex task. Often, the written code produces inaccurate results. There are several ways to solve this problem. In practice, the approach is to usually write code tests, which means that we are writing another code that tests our original code. However, these tests are testing certain cases. With mathematical proofs, we cover all possible cases and we are more confident that the code does exactly what it is intended for.

Idris is a general purpose functional programming[^ch1n1] language that supports dependent types. The features of Idris are influenced by Haskell, another general purpose functional programming language. However, Haskell does not support dependent types. Idris shares many features with the programming language Haskell, especially in the part of syntax and types, where Idris has a more advanced type system. There are several other programming languages that have support for dependent types[^ch1n2], however, I chose Idris for its very readable syntax.

The first version of Idris was released in 2009 and is developed by The Idris Community. Seen as a programming language, it is a functional language implemented with dependent types. Seen as a logical system, it implements intuitionistic type theory, which we will cover in details in the following chapters. We will show how these two views relate to each other in section 5.2, with the Curry-Howard correspondence.

Idris allows us to express mathematical statements. By mechanically examining these statements, it helps us find formal proof of the program's formal specification.

To fully understand how proofs in Idris work, we will start with the foundations, by defining, in order: formal systems, classical mathematical logic, lambda calculus, intuitionistic logic, and type theory (which is a more "up-to-date" version of classical mathematical logic).

[^ch1n1]: The core concept of functional programming languages is a mathematical function.

[^ch1n2]: Several other languages with dependent types support are Coq, Agda, Lean.
