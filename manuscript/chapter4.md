# 4. Type theory

There are some type theories that can serve as an alternative foundation of mathematics, as opposed to standard set theory (i.e. ZFC). One such well known type theory is Martin-L&#246;f’s intuitionistic theory of types, which is an extension of Alonzo Church’s simply typed {$$}\lambda{/$$}-calculus. Before we begin working with Idris, we will introduce ourselves to these theories, upon which Idris is built as a language.

I> ### Definition 1
I>
I> Type theory is defined as a class of formal systems. In these theories, every object is joined with a type, and operations upon these objects are constrained by the types joined. In order to say that {$$}x{/$$} is a type of {$$}X{/$$}, we denote {$$}x : X{/$$}. Functions are a primitive concept in type theory [^ch4n1].

For example, with {$$}1 : Nat, 2 : Nat{/$$} we can say that 1 and 2 are of type {$$}Nat{/$$}, that is natural numbers. There's an operation (function) {$$}+ : Nat \to Nat \to Nat{/$$}, which means that it takes two objects of type {$$}Nat{/$$} and returns an object of type {$$}Nat{/$$}.

I> ### Definition 2
I>
I> In type theory, a type constructor is a concept that builds new types from old ones. It can be thought roughly as a function that accepts types and as a result it returns a new type.

Idris supports algebraic data types. These data types are a kind of a complex type, that is, a type constructed by combining other types. Two classes of algebraic types are **product types** and **sum types**.

I> ### Definition 3
I>
I> Algebraic data types are types where we can additionally specify the form for each of the elements. They are called “algebraic” in the sense that the data types are constructed using algebraic operations. The algebra here is sum and product:
I>
I> 1. Sum (union) is alternation. It is denoted as {$$}A \ | \ B{/$$} and it means that the value is either of type A or B, but not both.
I> 1. Product is combination. It is denoted as {$$}A B{/$$} and it means that the value is a pair where the first element is of type A, and the second element is of type B.

As an example, we can assume that we have two types: {$$}Nat{/$$} for natural numbers, and {$$}Real{/$$} for real numbers. Now, for the sum (union) we can construct a new type {$$}Nat \ | \ Real{/$$}. Valid values of this type are {$$}1 : Nat \ | \ Real{/$$}, {$$}3.14 : Nat \ | \ Real{/$$}, etc. For the product type, we can construct a new type {$$}Nat Real{/$$}. Valid values of this type are {$$}1 1.5 : Nat Real{/$$}, {$$}2 3.14 : Nat Real{/$$}, etc. With this, sums and products can be combined and thus more complex data structures can be defined.

Finally, Idris supports dependent types[^ch4n2]. These kind of types are so powerful to encode most properties of programs, and with the help of them Idris can prove invariants at compile-time. This is what makes Idris a so called proof assistant[^ch4n3]. As we will see in section 5.2, types also allow us to encode mathematical proofs, which brings computer programs closer to mathematical proofs. As a consequence, this allows us to prove properties (e.g. specifications) about our software.

Q> ### Why are types useful?
Q>
Q> Russell's paradox (per the mathematician Bertrand Russell) states the following: In a village in which there is only one barber, there is a rule according to which the barber shaves everyone who don’t shave themselves, and no-one else. Now, who shaves the barber? In order to attempt to solve the paradox, we can assume that the barber shaves himself. Then, he’s one of those who shave themselves, but the barber shaves only those who do not shave themselves, which is a contradiction. Alternatively, if we assume that the barber does not shave himself, then he is in the group of people whom which the barber shaves, which again is a contradiction.
Q>
Q> The (naive) set theoretical foundations were affected by this paradox. As a response to this, between 1902 and 1908, Bertrand Russell himself proposed different type theories in attempt to resolve the issue. By joining types to values, we avoid the paradox because in this theory every set is defined as having elements from distinct type, for example, {$$}Type 1{/$$}. Elements from {$$}Type 1{/$$} can be included in a different set, say, elements of {$$}Type 2{/$$}, and so forth. Thus, the paradox is no longer an issue since the set of elements of {$$}Type 1{/$$} cannot be contained in their own set, since the types do not match. In a way, we’re adding hierarchy to sets in order to resolve the issue of “self-referential” sets. This is also the case with Idris, where we have that {$$}Type : Type 1 : Type 2{/$$}, etc.

## 4.1. Lambda calculus

I> ### Definition 4
I>
I> Lambda calculus is a formal for expressing computation[^ch4n4].
I> Per Wikipedia, the set of symbols for this system is defined as:
I>
I> 1. There are variables {$$}v_1, v_2, \ldots{/$$}
I> 1. There are only two abstract symbols: {$$}.{/$$} and {$$}\lambda{/$$}
I> 1. There are parentheses: {$$}({/$$} and {$$}){/$$}
I>
I> The grammar rules for well-formed expressions are:
I>
I> 1. If {$$}x{/$$} is a variable, then {$$}x \in \Lambda{/$$}
I> 1. If {$$}x{/$$} is a variable and {$$}M \in \Lambda{/$$}, then {$$}(\lambda x.M) \in \Lambda{/$$} (rule of abstraction)
I> 1. If {$$}M, N \in \Lambda{/$$}, then {$$}(M N) \in \Lambda{/$$} (rule of application)

Abstraction is when we define a function to which no arguments are applied, that is, there are no function calls. Application is when we apply arguments to some defined function. Some well-formed expressions are:

1. {$$}\lambda f \ x . f \ x{/$$}
1. {$$}\lambda f \ x . f \ (f \ x){/$$}

In fact, we can encode numbers this way. The first example can be thought of as the number one, and the second as the number two. This encoding is known as the Church encoding. Operations on numbers (plus, minus, etc) can also be defined in a similar way. With the {$$}\lambda{/$$} symbol we begin an abstraction, and with the {$$}.{/$$} symbol we separate the abstraction from the function body. In other words, 1 is defined roughly as {$$}f(x){/$$}, and 2 as {$$}f(f(x)){/$$}. Note that {$$}f{/$$} and {$$}x{/$$} do not have special definitions, they are abstract objects.

X> ### Exercise 1
X>
X> Convince yourself that the expression {$$}\lambda f \ x . f \ x{/$$} is a well-formed expression by writing down each one of the grammar rules used.

### 4.1.1. Terms reduction

I> ### Definition 5
I>
I> There are two types of variables in this system: free and bound. Bound variables are those who appear within the {$$}\lambda{/$$} abstraction. Analogously, free variables are those who do not appear in the {$$}\lambda{/$$} abstraction.

For example, in the expression {$$}\lambda y . x y{/$$} we have that {$$}y{/$$} is a bound variable, and {$$}x{/$$} is a free one.

I> ### Definition 6
I>
I> The rules of terms reduction allow us to compute (simplify) lambda expressions. There are three types of reduction:
I>
I> 1. {$$}\alpha{/$$} - (alpha) reduction: Renaming bound variables
I> 1. {$$}\beta{/$$} - (beta) reduction: Applying arguments to functions
I> 1. {$$}\eta{/$$} - (eta) reduction: Two functions are "equal" iff they return the same result for all arguments

For example, for the expression {$$}(\lambda x . f \ x) y{/$$}, we can use alpha reduction to get to {$$}(\lambda z . f \ z) y{/$$}, by changing {$$}x{/$$} to {$$}z{/$$}. Using beta reduction, the expression can further be reduced to just {$$}f y{/$$}, since we “consumed” the {$$}z{/$$} by removing it from the abstraction, and wherever it occured in the body we just replaced it with what was applied to it, that is, {$$}y{/$$}. Finally, with eta reduction, we can rewrite {$$}(\lambda x . f \ x){/$$} to just {$$}f{/$$}, since they are equivalent.

Given these rules, we can define the successor function as {$$}SUCC = \lambda n\ f\ x\ . f\ (n\ f\ x){/$$}. So, now we can try to apply 1 to {$$}SUCC{/$$}:

{$$}\\ SUCC 1 =\\ (\lambda n \ f \ x . f \ (n \ f \ x)) (\lambda f \ x . f \ x) =\\ \lambda f \ x . f \ ((\lambda f \ x . f \ x) \ f \ x) =\\ \lambda f \ x . f \ (f \ x) =\\ 2{/$$}

On the second line, we’ve substituted the very own definitions of {$$}SUCC{/$$} and 1. On the third line, we applied 1 to {$$}SUCC{/$$}, that is, we “consumed” the {$$}n{/$$} by using beta reduction. Finally, in the fourth line, we applied {$$}f{/$$} and {$$}x{/$$} to a function that accepts {$$}f{/$$} and {$$}x{/$$}, so the result is just the body of that abstraction.

X> ### Exercise 2
X>
X> Evaluate {$$}SUCC 2{/$$} to find out the definition of number 3.

X> ### Exercise 3
X>
X> Come up with your own functions that operates on the Church numerals. It can be as simple as returning the same number, or a constant one.

## 4.2. Lambda calculus with types

I> ### Definition 5
I>
I> Simply typed lambda calculus is a type theory which adds types to lambda calculus. It joins the system with a unique type constructor {$$}\to{/$$} which constructs types for functions. The formal definition and the set of lambda expressions is similar to that of lambda calculus, where in addition types are added.
I>
I> The set of symbols for this system is defined as:
I>
I> 1. There are variables {$$}v_1, v_2, \ldots{/$$}
I> 1. There are only two abstract symbols: {$$}.{/$$} and {$$}\lambda{/$$}
I> 1. There are parentheses: {$$}({/$$} and {$$}){/$$}
I>
I> The grammar rules for well-formed expressions are:
I>
I> 1. If {$$}x{/$$} is a variable, then {$$}x \in \Lambda{/$$}
I> 1. If {$$}x{/$$} is a variable and {$$}M \in \Lambda{/$$}, then {$$}(\lambda x.M) \in \Lambda{/$$} (rule of abstraction)
I> 1. If {$$}M, N \in \Lambda{/$$}, then {$$}(M N) \in \Lambda{/$$} (rule of application)
I> 1. If {$$}x{/$$} is a variable, {$$}T{/$$} is a type, and {$$}M \in \Lambda{/$$}, then {$$}(\lambda x:T.M) \in \Lambda{/$$}
I> 1. If {$$}x{/$$} is a variable and {$$}T{/$$} is a type, then {$$}x:T \in \Lambda{/$$}
I>
I> The type constructors are:
I>
I> 1. For some type {$$}A{/$$}, the type constructor {$$}T{/$$} is defined as {$$}A \ | \ T \to T{/$$}

That is, an expression in this system can additionally be an abstraction with {$$}x{/$$} having joined a type (rule 4), or an expression of a variable having joined a type {$$}T{/$$} (rule 5), where our type constructor is a sum type, and it says that we either have primitive types, or a way to form new types. Now in our attempt to re-define Church numerals and the successor function, we have to be careful as the types of these definitions have to match. Let’s recall the Church numerals:

1. {$$}1 = \lambda f \ x . f \ x{/$$}
1. {$$}2 = \lambda f \ x . f \ (f \ x){/$$}

Given the definition of 1, its type must have the form {$$}(a \to b) \to a \to b{/$$} for some {$$}a{/$$} and {$$}b{/$$}. This is so because we are expecting to be able to apply {$$}x{/$$} to {$$}f{/$$}, and so if {$$}x : a{/$$} then {$$}f : a \to b{/$$} in order for our types to match correctly. With similar reasoning, we have the same type for 2. So at this point, we have the type of {$$}(a \to b) \to a \to b{/$$}. Finally, with the given definition of 2, we can note that expressions of type {$$}b{/$$} need to be able to be applied to functions of type {$$}a \to b{/$$}, since the result of applying {$$}f{/$$} to {$$}x{/$$} serves as the argument of {$$}f{/$$}. The most general way for that to be true is if {$$}a = b{/$$}. So, as a result we have the type {$$}(a \to a) \to a \to a{/$$}. We can denote this type definition to be {$$}Nat{/$$}. Now, our numbers become:

1. {$$}1 = \lambda [f:(a \to a)] \ [x : a] . f \ x : Nat{/$$}
1. {$$}2 = \lambda [f:(a \to a)] \ [x : a] . f \ (f \ x) : Nat{/$$}

The (typed) successor function is: {$$}SUCC = \lambda [n:Nat]\ [f:(a \to a)] \ [x:a] . f\ (n\ f\ x) : Nat \to Nat{/$$}

X> ### Exercise 4
X>
X> Come up with a definition of the typed number 3.

X> ### Exercise 5
X>
X> Apply the typed {$$}SUCC{/$$} to 1 and confirm it results to 2. Make sure you confirm that the types also match in the process of evaluation.

X> ### Exercise 6
X>
X> In exercise 1 of 4.1.1, you were asked to come up with a function. Try to figure out the types of this function, or if not applicable, come up with a new function and then figure out its types using the reasoning above.

## 4.3. Dependent types

## 4.4. Intuitionistic theory of types

### 4.4.1. Intuitionistic logic

## 4.5. Lambda cube

[^ch4n1]: Unlike in set theory, where they are defined in terms of relations.

[^ch4n2]: Dependent types are the reason why Idris can formally prove mathematical statements, compared to other programming languages. While useful, since we can check whether an expression fulfills a given condition at compile-time, dependent types add complexity to a type system. In order to calculate type “equality” of dependent types, computations are necessary. If we allow any values for dependent types, then solving an equality of a type may involve deciding whether two programs produce the same result. Thus, the check may become undecidable.

[^ch4n3]: In general, Idris combines a lot of functionalities from mainstream languages (Java, C, C++), and some functionalities from proof assistants, which further blurs the limit between these two kinds of software.

[^ch4n4]: A Turing machine is an abstract mathematical machine that allows computation. Roughly, it is consisted of an initial state, and a transition function for manipulating this state. For any computer algorithm, a Turing machine can express that algorithm's logic. This is what makes a machine Turing complete. (Untyped) Lambda calculus is Turing complete.
