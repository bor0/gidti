# 3. Classic mathematical logic

All engineering disciplines involve some usage of logic. The foundations of Idris, as we will see later, are based on a system that implements (or encodes) classic mathematical logic so that we can easily "map" this logic and its inference rules to computer programs.

We will give a short introduction to two theories that are relevant to this chapter.

I> ### Definition 1
I>
I> The ZFC set theory, named after mathematicians Ernst Zermelo and Abraham Fraenkel, is a type of a formal system. The C stands for choice, after the axiom of choice[^ch3n1]. ZFC is the most common **foundation of mathematics**. It is a branch of mathematical logic that works with **sets**, which are collections of objects. It consists of 9 axioms, but we will not introduce them as they are out of scope for this book.

I> ### Definition 2
I>
I> Peano's axioms is a system of axioms that describes the natural numbers. It consists of 9 axioms, but we will name only a few:
I>
I> 1. 0 (zero) is a natural number
I> 1. For every number {$$}x{/$$}, we have that {$$}S(x){/$$} is a natural number, namely the successor function.
I> 1. For every number {$$}x{/$$}, we have that {$$}x = x{/$$}, namely that equality is reflexive.

We will discuss these concepts next in a bit more details.

## 3.1. Hierarchy of mathematical logic and definitions

At its core, mathematical logic deals with mathematical concepts expressed using formal logical systems. In this section we'll take a look at the hierarchy of these logical systems. The reason why we have different levels of hierarchies is that at each level we have more power in expressiveness. Further, these logical systems are what will allow us to produce proofs.

I> ### Definition 3
I>
I> The propositional branch of logic is concerned with the study of **propositions**, which are statements that are either {$$}\top{/$$} (true) or {$$}\bot{/$$} (false). Propositions are formed by other propositions with the use of logical connectives. The most basic logical connectives are {$$}\land{/$$} (and), {$$}\lor{/$$} (or), {$$}\to{/$$} (implication), and {$$}\lnot{/$$} (negation). The connectives are commutative, meaning {$$}a \oplus b = b \oplus a{/$$}, that is the order is irrelevant). These are their values:
I>
I> | {$$}a{/$$} | {$$}b{/$$} | {$$}a \land b{/$$} | {$$}a \lor b{/$$} | {$$}a \to b{/$$} | {$$}\lnot a{/$$} |
I> | --- | --- | --- | --- | --- | --- |
I> | {$$}\top{/$$} | {$$}\top{/$$} | {$$}\top{/$$} | {$$}\top{/$$} | {$$}\top{/$$} | {$$}\bot{/$$} |
I> | {$$}\top{/$$} | {$$}\bot{/$$} | {$$}\bot{/$$} | {$$}\top{/$$} | {$$}\bot{/$$} | {$$}\bot{/$$} |
I> | {$$}\bot{/$$} | {$$}\top{/$$} | {$$}\bot{/$$} | {$$}\top{/$$} | {$$}\top{/$$} | {$$}\top{/$$} |
I> | {$$}\bot{/$$} | {$$}\bot{/$$} | {$$}\bot{/$$} | {$$}\bot{/$$} | {$$}\top{/$$} | {$$}\top{/$$} |

In other words:

1. {$$}\top \land \top{/$$} = {$$}\top{/$$}, other combinations are {$$}\bot{/$$}.
1. {$$}\bot \lor \bot{/$$} = {$$}\bot{/$$}, other combinations are {$$}\top{/$$}.
1. {$$}\top \to \bot{/$$} = {$$}\bot{/$$}, other combinations are {$$}\top{/$$}.
1. {$$}\lnot \bot{/$$} = {$$}\top{/$$}, {$$}\lnot \top{/$$} = {$$}\bot{/$$}.

We can also use variables to represent statements. For example, we can say `a = Salad is organic`, and thus {$$}a{/$$} is a true statement. Another statement is `a = Rock is organic`, and thus {$$}a{/$$} is a false statement. The statement `a = Hi there!` is neither a true nor a false statement, and thus
is not a proposition.

I> ### Definition 4
I>
I> An argument is a list of propositions. An argument is valid iff in the case where all of the propositions are true, the conclusion is also true.

For example, given the two propositions {$$}a \lor b{/$$} and {$$}\lnot b{/$$}, we can conclude {$$}a{/$$}.

I> ### Definition 5
I>
I> The first-order logic logical system extends propositional logic by additionally covering **predicates** and **quantifiers**. A predicate {$$}P(x){/$$} takes as an input {$$}x{/$$}, and produces either true or false as an output. There are two quantifiers introduced: {$$}\forall{/$$} (universal quantifier) and {$$}\exists{/$$} (existential quantifier).

One example of a predicate is `P(x) = x is organic`, with {$$}P(Salad) = \top{/$$}, but {$$}P(Rock) = \bot{/$$}.

In the following example the universal quantifier says that the predicate will hold for **all** possible choices of {$$}x{/$$}: {$$}\forall x P(x){/$$}. Alternatively, the existential quantifier says that the predicate will hold for **at least one** choice of {$$}x{/$$}: {$$}\exists x P(x){/$$}.

Another example of combining a predicate with the universal quantifier is `P(x) = x is a mammal`, then {$$}\forall x P(x){/$$} is true, for all {$$}x{/$$} ranging over the set of humans.

I> ### Definition 6
I>
I> The higher-order logical system (second-order logic, third-order-logic, ..., higher-order (nth-order) logic) extends the quantifiers that range over individuals.

For example, the second-order logic quantifies over sets. Third-order logic quantifies over sets of sets, and so on.

I> ### Definition 7
I>
I> The ninth axiom in Peano's axioms is the induction axiom. It states the following: if {$$}P{/$$} is a predicate where {$$}P(0){/$$} is true, and for every {$$}P(n){/$$} we can prove that {$$}P(n+1){/$$}, then {$$}P(n){/$$} is true for all natural numbers.

Peano's axioms are expressed using a combination of first-order and second-order logic. This concept consists of a set of axioms for the natural numbers, and all of them are statements in first-order logic. An exception of this is the induction axiom, which is in second-order since it quantifies over predicates. The base axioms can be augmented with arithmetical operations of addition, multiplication and the order relation, which can also be
defined using first-order axioms.

I> ### Definition 8
I>
I> The turnstile symbol is similar to implication. It is denoted as {$$}\Gamma \vdash A{/$$}, where {$$}\Gamma{/$$} is a set of statements and {$$}A{/$$} is a conclusion. It is {$$}\top{/$$} iff it is impossible for all statements in {$$}\Gamma{/$$} to be {$$}\top{/$$}, and {$$}A{/$$} to be {$$}\bot{/$$}. In the Metamath appendix we'll cover an interesting difference between implication and this symbol.

X> ### Exercise 1
X>
X> Come up with a few propositions and combine them using some of the connectives explained above.

X> ### Exercise 2
X>
X> Think of a real-world predicate and express the truthness of it using the {$$}\forall{/$$} and {$$}\exists{/$$} symbols.

## 3.2. Set theory abstractions

Like in programming, building abstractions in mathematics is of equal importance. However, the best way to understand something is to get to the bottom of it. We'll start by working from the foundations level upwards. So we will start with the most basic object (the unordered collection) and work our way up to defining functions. Functions are an important core concept of Idris, however, as we will see in the theory that Idris relies on, functions are used as a primitive notion (an axiom) instead of being built on top of something else.

I> ### Definition 9
I>
I> A set is an **unordered** collection of objects. The objects can be anything. It is usually denoted by comma separating the list of objects and enclosing them using curly braces.

For example, one set of fruits is {$$}\{ apple, banana \}{/$$}. Since it is an unordered collection we have that {$$}\{ apple, banana \} = \{ banana, apple \}{/$$}.

I> ### Definition 10
I>
I> Set membership states that a given object is belonging to a set. It is denoted using the {$$}\in{/$$} operator.

For example, {$$}apple \in \{ apple, banana \}{/$$} says that `apple` is in that set.

I> ### Definition 11
I>
I> An {$$}n{/$$}-tuple is an **ordered collection** of {$$}n{/$$} objects. As with sets, the objects can be anything. It is usually denoted by comma separating the list of objects and enclosing them using parenthesis.

We can represent it as follows: {$$}(a_1, a_2, ..., a_n){/$$}. In order to represent this to its set alternative, we can use the set {$$}\{ \{ 1, \{ a_1 \} \}, \{ 2, \{ a_2 \} \}, \ldots, \{ n, \{ a_n \} \} \}{/$$}. This will now allow us to extract the {$$}k{/$$}-th element of the tuple, by picking {$$}x{/$$} such that {$$}\{ k, \{ x \} \} \in A{/$$}. Having done that, now we have that {$$}(a, b) = (c, d) \equiv a = c \land b = d{/$$}, that is, two tuples are equal iff their first and second elements respectively are equal. This is what makes them ordered.

One valid tuple is {$$}(1 pm, 2 pm, 3 pm){/$$} which represents 3 hours of a day sequentially.

I> ### Definition 12
I>
I> An n-ary relation is just a set of {$$}n{/$$}-tuples with different values.

For example, the "is bigger than" relation represents a 2-tuple (pair), for the following set: {$$}\{ (cat, mouse), (mouse, cheese), (cat, cheese) \}{/$$}.

I> ### Definition 13
I>
I> {$$}A{/$$} is a subset of {$$}B{/$$} if all elements of {$$}A{/$$} are found in {$$}B{/$$} (but not necessarily vice-versa). We denote it as such: {$$}A \subseteq B{/$$}.

For example, the expressions {$$}\{ 1, 2 \} \subseteq \{ 1, 2, 3 \}{/$$} and {$$}\{ 1, 2, 3 \} \subseteq \{ 1, 2, 3 \}{/$$} are both true. But this expression is not true: {$$}\{ 1, 2, 3 \} \subseteq \{ 1, 2 \}{/$$}.

I> ### Definition 14
I>
I> A Cartesian product is defined as the set {$$}\{ (a, b) \}{/$$} such that for all {$$}a{/$$} and {$$}b{/$$}, we have that {$$}a \in A \land b \in B{/$$}. It is denoted as {$$}A \times B{/$$}.

For example if {$$}A = \{ a, b \}{/$$} and {$$}B = \{ 1, 2, 3 \}{/$$} then the combinations are: {$$}A \times B = \{ (a, 1), (a, 2), (a, 3), (b, 1), (b, 2), (b, 3) \}{/$$}.

I> ### Definition 15
I>
I> **Functions** are defined in terms of relations[^ch3n2]. A binary (2-tuple) set {$$}F{/$$} represents a mapping[^ch3n3] from some set {$$}A{/$$} to some set {$$}B{/$$}, where {$$}F{/$$} is a subset of the Cartesian product of {$$}A{/$$} and {$$}B{/$$}. That is, a function {$$}f{/$$} from {$$}A{/$$} to {$$}B{/$$} is denoted {$$}f : A \to B{/$$} and is a subset of {$$}F{/$$}, i.e. {$$}f \subseteq F{/$$}. There is one more constraint that functions have, namely, that they cannot produce 2 or more different values for a single input.

For example, the function {$$}f(x) = x + 1{/$$} is a function that, given a number, returns it increased by one. So {$$}f(1) = 2{/$$}, {$$}f(2) = 3{/$$}, etc. Another way to represent this function is using the 2-tuple set: {$$}f = \{ (1, 2), (2, 3), (3, 4), \ldots \}{/$$}.

One simple way to think of functions is in form of tables. For a function accepting a single parameter {$$}f(x){/$$}, we have a two-column table where the first column is the input, and the second column is the output. For a function accepting two parameters {$$}x{/$$} and {$$}y{/$$}, {$$}f(x, y){/$$}, we have a three-column table where the first and second columns are the inputs, and the third one is the output. Thus, to display the function discussed above in a form of table, it would look like this:

| {$$}x{/$$} | {$$}f(x){/$$} |
| --- | --- |
| a | 1 |
| b | 2 |

X> ### Exercise 3
X>
X> Think of a set of objects and try to express that some object belongs to that set.

X> ### Exercise 4
X>
X> Think of a set of objects whose order matters and try to express it in terms of an ordered collection.

X> ### Exercise 5
X>
X> Think of a relation (for example, a relation between two persons in a family tree) and express it using the notation described.

X> ### Exercise 6
X>
X> Come up with two subset expressions, one that is true and another one that is false.

X> ### Exercise 7
X>
X> Think of two sets, and combine them using the definition of Cartesian product. Afterwards, think of two subset expressions, one that is true and another one that is false.

X> ### Exercise 8
X>
X> Think of a valid function and represent it using the table approach.

X> ### Exercise 9
X>
X> Write down the corresponding input and output sets for the function you implemented in exercise 6.

## 3.3. Substitution and proofs

Substitution lies at the heart of mathematics. A similar statement can be made about programming, but we will cover this in the later chapters with so called **pure** and **impure** functions. Substitution can be applied in different contexts involving formal objects containing symbols. It consists of systematically replacing occurrences of some symbol by a given value.

For example, let's assume we have the following as givens:

1. An inference rule that states: If {$$}a = b{/$$} and {$$}b = c{/$$}, then {$$}a = c{/$$}
1. Two axioms that state: {$$}1 = 2{/$$} and {$$}2 = 3{/$$}

Then we can use the following proof to claim that {$$}1 = 3{/$$}:

1. {$$}1 = 2{/$$} (axiom)
1. {$$}2 = 3{/$$} (axiom)
1. {$$}1 = 2{/$$} and {$$}2 = 3{/$$} (from 1 and 2 combined)
1. {$$}1 = 3{/$$}, from 3 and the inference rule

We know that in general, {$$}1 = 3{/$$} does not make any sense. But, in the context of the givens above, this proof is valid.

X> ### Exercise 10
X>
X> TODO

X> ### Exercise 11
X>
X> Come up with several axioms and inference rules and try to do a proof similar to the example above.

## 3.4. Mathematical proofs

I> ### Definition 16
I>
I> A proof is defined as an inferential **argument** for a set of given mathematical propositions.

I> ### Definition 17
I>
I> Modus ponens (method of affirming) and modus tollens (method of denying) are two inference rules in logic. Their definition is as follows:
I> 1. Modus ponens states: If we are given {$$}p \to q{/$$} and {$$}p{/$$}, then we can conclude {$$}q{/$$}
I> 1. Modus tollens states: If we are given {$$}p \to q{/$$} and {$$}\lnot q{/$$}, then we can conclude {$$}\lnot p{/$$}

As we've defined before, an argument is a list of statements. There are several ways to do mathematical proofs. One of them is by using the so-called
three-column proofs.

For example, given {$$}A \lor B{/$$}, {$$}B \to C{/$$}, {$$}\lnot C{/$$}, prove {$$}A{/$$}. We can approach the proof as follows:

| No. | Step | Reasoning |
| --- | --- | --- |
| 1 | {$$}A \lor B{/$$} | Given |
| 2 | {$$}B \to C{/$$} | Given |
| 3 | {$$}\lnot C{/$$} | Given |
| 4 | {$$}(B \to C) \land \lnot C{/$$} | 2 and 3 |
| 5 | {$$}\lnot B{/$$} | Modus tollens rule on 4, i.e. {$$}(p \to q \land \not q) \to \not p{/$$} |
| 6 | {$$}(A \lor B) \land \lnot B{/$$} | 1 and 5 |
| 7 | {$$}A{/$$} | 6, where {$$}p \land \lnot p{/$$} is a contradiction, i.e. invalid argument |

However, this type of proof contains many details. Ideally, the proof should be short, clear and concise about what we want to prove.

Another proof technique that we can use is by constructing a truth table. The way truth tables are constructed for a given statement is to break it down into atoms and then include every subset of the expression.

For example, to prove the statement {$$}A \land B \to B{/$$}, we can approach as follows:

| {$$}A{/$$} | {$$}B{/$$} | {$$}A \land B{/$$} | {$$}A \land B \to B{/$$} |
| --- | --- | --- | --- |
| {$$}\bot{/$$} | {$$}\bot{/$$} | {$$}\bot{/$$} | {$$}\top{/$$} |
| {$$}\bot{/$$} | {$$}\top{/$$} | {$$}\bot{/$$} | {$$}\top{/$$} |
| {$$}\top{/$$} | {$$}\bot{/$$} | {$$}\bot{/$$} | {$$}\top{/$$} |
| {$$}\top{/$$} | {$$}\top{/$$} | {$$}\top{/$$} | {$$}\top{/$$} |

Note that wherever {$$}A \land B{/$$} is true, then so is {$$}A \land B \to B{/$$}, which means that this is a valid logical argument, according to the definition 4.

However, if our statements involve the usage of quantifiers, then doing proofs with truth tables is impossible. Therefore, we will try to prove the same statement by means of a formal proof.

To prove that {$$}A \land B \to B{/$$}, we start by assuming that {$$}A \land B{/$$} is true, since otherwise the statement is vacuously true by definition for implication. If {$$}A \land B{/$$} is true, then both {$$}A{/$$} and {$$}B{/$$} are true by definition of `and`, that is, we can conclude {$$}B{/$$}.

Do not worry if the previous paragraph sounded too magical. There is not much magic involved. Usually it comes down to using a few rules (or "tricks", if you will) for how we can use given information and achieve our goal. We will summarize these proof techniques next.

## 3.4.1 Proof techniques

In order to prove a goal of a given form:

| Goal form | Technique |
| --- | --- |
| {$$}P \to Q{/$$} | Assume that {$$}P{/$$} is true and prove {$$}Q{/$$} |
| {$$}\lnot P{/$$} | Assume that {$$}P{/$$} is true and arrive at a contradiction |
| {$$}P_1 \land P_2 \land \ldots \land P_n{/$$} | Prove each one of {$$}P_1, P_2, \ldots, P_n{/$$} separately |
| {$$}P_1 \lor P_2 \lor \ldots \lor P_n{/$$} | Use proof by cases, where in each case you prove one of {$$}P_1, P_2, \ldots, P_n{/$$} |
| {$$}P \leftrightarrow Q{/$$} | Prove both {$$}P \to Q{/$$} and {$$}Q \to P{/$$} |
| {$$}\forall x P(x){/$$} | Assume that {$$}x{/$$} is an arbitrary object and prove that {$$}P(x){/$$} |
| {$$}\exists x P(x){/$$} | Find an {$$}x{/$$} such that {$$}P(x){/$$} is true |
| {$$}\exists! x P(x){/$$}[^ch3n4] | Prove {$$}\exists x P(x){/$$} (existence) and {$$}\forall x \forall y (P(x) \land P(y) \to x = y){/$$} (uniqueness) separately |

In order to use a given of form:

| Given form | Technique |
| --- | --- |
| {$$}P \to Q{/$$} | If {$$}P{/$$} is also given, then conclude that Q (by modus ponens) |
| {$$}\lnot P{/$$} | If {$$}P{/$$} can be proven true, then conclude a contradiction |
| {$$}P_1 \land P_2 \land \ldots \land P_n{/$$} | Treat each one of {$$}P_1, P_2, \ldots, P_n{/$$} as a given |
| {$$}P_1 \lor P_2 \lor \ldots \lor P_n{/$$} | Use proof by cases, where in each case you assume one of {$$}P_1, P_2, \ldots, P_n{/$$} |
| {$$}P \leftrightarrow Q{/$$} | Conclude both {$$}P \to Q{/$$} and {$$}Q \to P{/$$} |
| {$$}\forall x P(x){/$$} | For any {$$}x{/$$}, conclude that {$$}P(x){/$$} |
| {$$}\exists x P(x){/$$} | Introduce a new variable, say {$$}x_0{/$$} so that {$$}P(x_0){/$$} is true |
| {$$}\exists! x P(x){/$$}[^ch3n4] | Introduce a new variable, say {$$}x_0{/$$} so that {$$}P(x_0){/$$} is true. Can also use that {$$}\forall x \forall y (P(x) \land P(y) \to x = y){/$$} |

X> ### Exercise 12
X>
X> We've used the rules modus tollens and modus ponens without giving an actual proof for them. Try to prove by yourself that these two rules hold, either by constructing a truth table or a three-column proof:
X>
X> 1. Modus tollens: {$$}(p \to q, \lnot q) \to \lnot p{/$$}
X> 1. Modus ponens: {$$}(p \to q, p) \to q{/$$}

## 3.4.2 Mathematical induction

I> ### Definition 18
I>
I> Mathematical induction is a proof method that is used to prove that a predicate {$$}P(n){/$$} is true for all natural numbers {$$}n{/$$}. It consists of proving two parts: a base case and an inductive step. For the base case we need to show that what we want to prove {$$}P(n){/$$} is true for some starting value, {$$}k{/$$}, which is usually zero. For the inductive step, we need to prove that {$$}P(n) \to P(n+1){/$$}, that is, if we assume that {$$}P(n){/$$} is true, then {$$}P(n+1){/$$} must follow as a consequence. After having proven these two parts, we can conclude that {$$}P(n){/$$} holds for all natural numbers. So the formula that we need to prove is {$$}P(0) \land ( P(n) \to P(n+1) ){/$$}.

To understand why mathematical induction works, as an example it is best to visualize dominoes arranged in a sequence. If we push the first domino, it will push the second, which will push the third, and so on to infinity. That is, if we position the dominoes such that if one falls it will push the next one, i.e. {$$}P(n){/$$} implies {$$}P(n+1){/$$}, and we push the first one {$$}P(0){/$$}, then all the dominoes will fall, i.e. {$$}P(n){/$$} is true in general.

I> ### Definition 19
I>
I> We are given this recursive definition for adding numbers:
I>
I> 1. {$$}n = 0 + n{/$$}
I> 1. {$$}S(m) + n = S(m + n){/$$}, where {$$}S{/$$} is the successor function, that is {$$}S(0) = 1, S(1) = 2{/$$}, etc.

For example, in order to prove that {$$}\forall n, n + 0 = n{/$$} in the system of Peano's axioms, we can proceed by induction (this is an axiom). For the base case, we have that {$$}0 + 0 = 0{/$$}, which is true (by definition of adding numbers, for {$$}n = 0{/$$}). For the inductive step, we first assume that {$$}n + 0 = n{/$$} is true, and prove that {$$}S(n) + 0 = S(n){/$$}. By definition of addition, we have {$$}S(n) + 0 = S(n + 0){/$$}. Now if we use the inductive hypothesis we have {$$}S(n + 0) = S(n){/$$}, which is what we needed to show. With this example, we can see how induction and natural numbers are closely related to each other. Note how we proved {$$}n + 0 = n{/$$}, given {$$}n = 0 + n{/$$}. That is, we proved that addition with 0 is commutative.

I> ### Definition 20
I>
I> {$$}a{/$$} is divisible by {$$}b{/$$} if there exists a natural number {$$}k{/$$} so that {$$}a = bk{/$$}.

X> ### Exercise 13
X>
X> Come up with a predicate, and then prove its truthness using mathematical induction.

X> ### Exercise 14
X>
X> Prove that {$$}2^n - 3{/$$} is not divisible by 3. Hint: Use {$$}n = 2{/$$} as the base case.

[^ch3n1]: This axiom states that the Cartesian product of non-empty sets is non-empty.

[^ch3n2]: It is worth noting that in set theory, {$$}P{/$$} would be a subset of a relation, i.e. {$$}P \subseteq A \times \{ T, F \}{/$$}, where {$$}A{/$$} is a set of some inputs, for example `Salad` and `Rock` in the example. When working with other systems we need to be careful, as this is not the case with first-order logic. In the case of first-order logic, we have {$$}P(Salad) = \top{/$$}, {$$}P(Rock) = \bot{/$$}, etc as atomic statements, not mathematical functions (i.e. they cannot be broken down into smaller statements). This is what makes first-order logic independent of set theory.

[^ch3n3]: In other words, a function is a subset of all combinations of ordered pairs whose first element is an element of {$$}A{/$$} and second element is an element of {$$}B{/$$}.

[^ch3n4]: The notation {$$}\exists!{/$$} stands for unique existential quantifier. It means that **only one** object fulfills the predicate, as opposed to {$$}\exists{/$$}, which states that **at least one** object fulfills the predicate.
