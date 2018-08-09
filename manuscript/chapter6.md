# 6. Programming in Idris

In this chapter we will provide several examples to demonstrate the power of Idris. We will start doing mathematical proofs. There are several built-ins in Idris that will help us achieve this, so for each section we will introduce the relevant definitions.

## 6.1. Weekdays

In this section we will introduce a way to represent weekdays, and then do some proofs with them. Before starting with the examples, we will list the relevant definitions.

I> ### Definition 1
I>
I> The equality data type is roughly defined as follows:
I>
I> ```
I> data (=) : a -> b -> Type where
I>     Refl : x = x
I> ```
I>
I> We can use the type constructor `Refl` to prove equalities.

I> ### Definition 2
I>
I> The function `the` accepts `A : Type` and `value : A` and then returns `value : A`. We can use it to manually assign a type to an expression.

X> ### Exercise 1
X>
X> Check the documentation of the equality type with `:doc (=)`.

X> ### Exercise 2
X>
X> Evaluate `the Nat 3` and `the Integer 3` and note the differences.

We start with the following data structure, for weekdays:

```
data Weekday = Mon | Tue | Wed | Thu | Fri | Sat | Sun
```

The `Weekday` type has 7 constructors, one for each weekday. The data type and its type constructors do not accept any parameters.

### 6.1.1. First proof (auto-inference)

We will prove that after Monday comes Tuesday. We start by implementing a function that, given a weekday, returns the next weekday:

```
total next_day : Weekday -> Weekday
next_day Mon = Tue
next_day Tue = Wed
next_day Wed = Thu
next_day Thu = Fri
next_day Fri = Sat
next_day Sat = Sun
next_day Sun = Mon
```

Given these definitions, we can now state the proof as follows:

```
our_first_proof : next_day Mon = Tue
our_first_proof = ?prf
```

The type of `our_first_proof` is `next_day Mon = Tue`. Since `next_day` is total, Idris will be able to evaluate `next_day Mon` at compile-time. We used a hole for the definition since we still don't know what the proof will look like. If we run this code in Idris, it will tell us we have a hole `prf` to fill:

```
Type checking ./first_proof.idr
Holes: Main.prf
Idris> :t prf
--------------------------------------
prf : Tue = Tue
```

Checking the type of `prf`, we notice how Idris evaluated the left part of the equation at compile-time. Now, in order to prove that `Tue = Tue`, we can just use `Refl`:

```
our_first_proof : next_day Mon = Tue
our_first_proof = Refl
```

Reloading the file in Idris:

```
Idris> :r
Type checking ./first_proof.idr
Idris> 
```

Which means that we've successfully proven that `next_day Mon = Tue`, that is, after Monday comes Tuesday!

X> ### Exercise 3
X>
X> Remove one or more pattern match definitions of `next_day` and observe the error that Idris will produce. Afterwards, alter the function to not be total, and observe the following error.

### 6.1.2. Second proof (rewrite)

In addition to the previous proof, let's implement a function that accepts a `Weekday` and returns `True` if it's Monday, and `False` otherwise.

```
is_it_monday : Weekday -> Bool
is_it_monday Mon = True
is_it_monday _   = False
```

For the sake of example, we will prove that for any given day, if it's Monday then `is_it_monday` will return `True`, and `False` otherwise. It's pretty obvious by the definition of `is_it_monday`, but proving that is a whole different story. We can attempt to prove it as follows:

```
our_second_proof : (day : Weekday) -> day = Mon -> is_it_monday day = True
our_second_proof day day_eq_Mon = Refl
```

Note the type definition. We gave a name of the `Weekday` so that we can refer to it in the rest of the type definition.

If we run this code in Idris, it will produce an error at compile-time since it cannot deduce that `True` is equal to `is_it_monday day`. In the previous proof example, Idris was able to infer everything by the definitions at compile-time. However, at this point we need to help Idris do the inference since it cannot derive the proof based only on the definitions. We can change the `Refl` to a hole `?prf`:

```
  day : Weekday
  day_eq_Mon : day = Mon
--------------------------------------
prf : is_it_monday day = True
```

We see that along with `prf` we also get `day` and `day_eq_Mon` in the list of givens, per the left hand side of the function definition of `our_second_proof`.

Q> How do we replace something we have in the givens, with the goal?
Q>
Q> If we only had a way to tell Idris that it just needs to replace `day` with `day = Mon` to get to `is_it_monday Mon = True`, it will be able to infer the rest.

I> ### Definition 3
I>
I> The `rewrite` keyword can be used to rewrite expressions. If we have `X : x = y`, then the syntax `rewrite X in Y` will replace all occurences of `x` with `y` in `Y`.

With the power to do rewrites, we can attempt the proof as follows:

```
our_second_proof : (day : Weekday) -> day = Mon -> is_it_monday day = True
our_second_proof day day_eq_Mon = rewrite day_eq_Mon in ?prf
```

Idris produces:

```
Idris> :t prf
  day : Weekday
  day_eq_Mon : day = Mon
  _rewrite_rule : day = Mon
--------------------------------------
prf : True = True
```

Changing `?prf` to `Refl` completes the proof.

### 6.1.3. Third proof (impossible)

As an inspiration from the previous section, we will now prove that `is_it_monday Tue = True` is a contradiction.

Per intuitionistic logic, in order to prove that {$$}P{/$$} is a contradiction, we need to prove {$$}P \to \bot{/$$}. Idris provides an empty data type (without type constructors, i.e. proofs) for that, which is called `Void`.

So, to prove that `is_it_monday Tue = True` is a contradiction, we will do the following:

```
our_third_proof : is_it_monday Tue = True -> Void
our_third_proof mon_is_Tue = ?prf
```

Checking the type of the hole:

```
  mon_is_Tue : False = True
--------------------------------------
prf : Void
```

Q> How do we prove `prf`, given that there are no type constructors for `Void`?
Q>
Q> Seems that at this point we are stuck. We need to find a way to tell Idris this proof is impossible.

I> ### Definition 4
I>
I> The `impossible` keyword can be used to prove statements that are not true.

We will slightly rewrite the function:

```
our_third_proof : is_it_monday Tue = True -> Void
our_third_proof Refl impossible
```

With this syntax, we're telling Idris that the reflexivity of `False = True` is impossible, and thus the proof is complete.

X> ### Exercise 4
X>
X> Check the documentation of `Void`.

## 6.2. Natural numbers

As we have seen, natural numbers are very powerful. In this section we will proofs related to them, and also do a little bit of induction.

We will start with the following definitions for natural numbers:

```
data MyNat = Zero | Succ MyNat

total mynat_plus : MyNat -> MyNat -> MyNat
mynat_plus Zero     m = m
mynat_plus (Succ k) m = Succ (mynat_plus k m)
```

Note how the definition of `MyNat` is recursive compared to `Weekday`. A consequence of that is that we may need to use induction for some proofs.

X> ### Exercise 5
X>
X> Compare the addition definition to definition 19 of chapter 3.

### 6.2.1. First proof (auto-inference)

We will prove that {$$}0 + a = a{/$$}, given the definitions for natural numbers and addition.

For that, we need to implement a function that accepts a natural number `a`, and returns the proposition that `mynat_plus Zero a = a`.

```
total our_first_proof : (a : MyNat) -> mynat_plus Zero a = a
our_first_proof a = ?prf
```

If we check the type of the hole, we get that the goal is `prf : a = a`, so changing the hole to a `Refl` completes the proof.

### 6.2.2. Second proof (introduction of new givens)

Let's assume a simple case, where we want to prove that a natural number exists:

```
total our_second_proof : MyNat
our_second_proof = ?prf
```

Now, if we check the type of the hole, we get:

```
Idris> :t prf
--------------------------------------
prf : MyNat
Holes: Main.prf
```

Q> By the definition of `MyNat` we are sure that there exists a type constructor of `MyNat`, but how do we tell Idris?

I> ### Definition 5
I>
I> The `let` keyword that we introduced earlier allows us to add a new given to the list of givens.

Now we can slightly rewrite our code:

```
total our_second_proof : MyNat
our_second_proof = let the_number = Zero in ?prf
```

Checking the type:

```
  the_number : MyNat
--------------------------------------
prf : MyNat
```

Changing `prf` to `the_number` concludes the proof.

### 6.2.3. Third proof (induction)

We will prove that {$$}a + 0 = a{/$$}. We can try to use the same approach as in 6.2.1:

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof a = Refl
```

If we try to run the code above, Idris will produce an error that there is a type mismatch between `a` and `mynat_plus a Zero`.

Q> It seems that we have just about all the definitions we need, but we're missing a piece. But, how do we re-use our definitions?
Q>
Q> To prove that {$$}a + 0 = a{/$$}, we can use mathematical induction starting with the definitions we already have as the base case, and building on top of that until {$$}a + 0 = a{/$$}. That is, we need to prove that {$$}0 + 0 = 0 \to (a - a) + 0 = a - a \to ... \to (a - 1) + 0 = a - 1 \to a + 0 = a{/$$}.

From here, we rewrite our function to contain a base case and an inductive step:

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof Zero     = ?base
our_third_proof (Succ k) = ?inductive_hypothesis
```

Checking the types of the holes:

```
Idris> :t base
--------------------------------------
base : Zero = Zero
Holes: Main.inductive_hypothesis, Main.base
Idris> :t inductive_hypothesis
  k : MyNat
--------------------------------------
inductive_hypothesis : Succ (mynat_plus k Zero) = Succ k
Holes: Main.inductive_hypothesis, Main.base
```

For the base case, we can just use `Refl`, but for the inductive step we need to do something different. Now we need to find a way to assume (add to list of givens) {$$}a + 0 = a{/$$}, and show that it follows {$$}(a + 1) + 0 = a + 1{/$$} from that assumption. Since we pattern match on `Succ k`, we can use recursion on `k` along with `let` to generate the hypothesis:

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof Zero     = Refl
our_third_proof (Succ k) = let inductive_hypothesis = our_third_proof k in ?conclusion
```

Our proof givens and goals now become:

```
  k : MyNat
  inductive_hypothesis : mynat_plus k Zero = k
--------------------------------------
conclusion : Succ (mynat_plus k Zero) = Succ k
```

To prove the conclusion, we can simply rewrite the inductive hypothesis in the goal and we are done.

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof Zero     = Refl
our_third_proof (Succ k) = let inductive_hypothesis = our_third_proof k in rewrite inductive_hypothesis in Refl
```

This concludes the proof.

X> ### Exercise 6
X>
X> Note the similarity between this proof and the proof in section 3.4.2.

## 6.3. Monoids

A Monoid is an abstract algebraic structure. Monoids, as with other algebraic structures are useful because they share a set of properties in which programmers can later take for granted.

I> ### Definition 6
I>
I> Monoid is consisted of a set {$$}S{/$$}, together with a binary operator {$$}\oplus{/$$} such that the following properties are fulfilled:
I>
I> 1. Closure {$$}\forall a, b \in S a \oplus b \in S{/$$}, which means that the result of the operation also belongs in the same set {$$}S{/$$}
I> 1. Associattivity {$$}\forall a, b, c \in S (a \oplus b) \oplus c = a \oplus (b \oplus c){/$$}, which means tthat the order of evaluation does not matter. This is useful for parallel processing
I> 1. Identity {$$}\exists e \forall a a \oplus e = e \oplus a = a{/$$}, which means that there exists an element such that it doesn't alter the value when used with the operator. This can useful with recursion for example, as the base case where the recursion terminates

For example, the set of natural numbers with the addition operator is a monoid:

1. Closure: The sum of two natural numbers is also a natural number
1. Associativitty: The order of addition will not change the result, i.e. {$$}(a + b) + c = a + (b + c){/$$}
1. Identity: The identity element zero fulfills this property, so we have that {$$}a + 0 = 0 + a = a{/$$}

Now we can implement the interface as follows:

```
interface Monoid a where
    mempty : a -- Identity
    mappend : a -> a -> a -- Associative operation
```

Let's see the implementation for the `Monoid Nat`, that is, {$$}(\Nat, +){$$}:

```
implementation Monoid Integer where
    mempty = 0
    mappend a b = a + b
```

For example

```
Idris> mappend 1 2
3 : Integer
```

Prove properties.

## 6.4. Ordering

6.9. b <= a -> max a b = b
First we need to figure out the type of our function
Intuitively, we can approach as follows

```
proof1 : (a : Nat) -> (b : Nat) -> a <= b -> maximum a b = b
proof1 a b a_lt_b = ?x
```

However this won't work since a <= b is a Bool, not a Type. Need to think of something else. Fortunately Idris has a data structure called LTE which accepts two Nat numbers. So we write LTE a b to denote that a <= b

```
proof1 : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
proof1 a b a_lt_b = ?x
```

This compiles and now we have to figure out the hole. If we check its type, we get
```
  a : Nat
  b : Nat
  a_lt_b : LTE a b
--------------------------------------
x : maximum a b = b
```
So the question is, given a, b, a_lt_b, how can we conclude x? First, we can simplify

```
proof1 : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
proof1 Z Z _              = Refl
proof1 Z (S k) _          = Refl
proof1 (S k) (S j) a_lt_b = ?a
```
Now we get
```
  k : Nat
  j : Nat
  a_lt_b : LTE (S k) (S j)
--------------------------------------
a : S (maximum k j) = S j
```

Cool, this gives us something to work with. We can use induction on k and j:
```
proof1 : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
proof1 Z Z _              = Refl
proof1 Z (S k) _          = Refl
proof1 (S k) (S j) a_lt_b = let IH = (proof1 k j ?a) in rewrite IH in Refl
```
Now we get
```
Holes: Main.a
*test> :t a
  k : Nat
  j : Nat
  a_lt_b : LTE (S k) (S j)
--------------------------------------
a : LTE k j
Holes: Main.a
```
If only we had a way to convert the type of `a_lt_b` to a. Luckily, Idris has built-in fromLteSucc:
```
*test> :doc fromLteSucc
Prelude.Nat.fromLteSucc : LTE (S m) (S n) -> LTE m n
    If two numbers are ordered, their predecessors are too
    
    The function is Total
Holes: Main.a
*test> 
```

So our final proof is

```
total
proof1 : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
proof1 Z Z _              = Refl
proof1 Z (S k) _          = Refl
proof1 (S k) (S j) a_lt_b = let IH = (proof1 k j (fromLteSucc a_lt_b)) in rewrite IH in Refl
More simply
total
proof1 : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
proof1 Z Z _              = Refl
proof1 Z (S k) _          = Refl
proof1 (S k) (S j) a_lt_b = rewrite (proof1 k j (fromLteSucc a_lt_b)) in Refl
```

## 6.5. Trees

Q> What is a tree structure?
Q>
Q> Woot.

We can define a tree structure similarly using the following implementation:

```
data Tree = Empty | Leaf Int | Node Tree Tree
```

Which states that a tree is defined as one of:

1. `Empty`, which has no values
1. `Leaf`, which has a single value
1. `Node` (points to another `Empty` or `Leaf`)

As mentioned above, functions (or operations) on the algebraic data types can be defined by using pattern match to extract values. As an example, let's take a look at the depth function which calculates the depth of a tree:

```
depth : Tree -> Int
depth Empty = 0
depth (Leaf n) = 1
depth (Node l r) = 1 + max (depth l) (depth r)
```

If we pass a `Tree` to the function `depth`, then Idris will pattern match the types and we can extract the values. For example, in the `Node` case, we pattern match to extract the sub-trees `l` and `r` for further processing.
