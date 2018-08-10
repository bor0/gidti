# 6. Programming in Idris

In this chapter we will provide several examples to demonstrate the power of Idris. We will do mathematical proofs. There are a lot of built-ins in Idris that will help us achieve our goals. So in each section we will introduce the relevant definitions.

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

## 6.1. Weekdays

In this section we will introduce a way to represent weekdays, and then do some proofs with them.

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

Given these definitions, we can write the proof as follows:

```
our_first_proof : next_day Mon = Tue
our_first_proof = ?prf
```

Note how we used a function at the type level, for the type of `our_first_proof` which is `next_day Mon = Tue`. Since `next_day` is total, Idris will be able to evaluate `next_day Mon` at compile-time. We used a hole for the definition since we still don't know what the proof will look like. If we run this code in Idris, it will tell us we have a hole `prf` to fill:

```
Type checking ./first_proof.idr
Holes: Main.prf
Idris> :t prf
--------------------------------------
prf : Tue = Tue
```

Checking the type of `prf`, we notice how Idris evaluated the left part of the equation at compile-time. In order to prove that `Tue = Tue`, we can just use `Refl`:

```
our_first_proof : next_day Mon = Tue
our_first_proof = Refl
```

Reloading the file in Idris:

```
Idris> :r
Type checking ./first_proof.idr
```

Which means that we've successfully proven that `next_day Mon = Tue`, that is, after Monday comes Tuesday!

X> ### Exercise 3
X>
X> Remove one or more pattern match definitions of `next_day` and observe the error that Idris will produce. Afterwards, alter the function to not be total, and observe the error.

### 6.1.2. Second proof (rewrite)

In addition to the previous proof, we'll implement a function that accepts a `Weekday` and returns `True` if it's Monday, and `False` otherwise.

```
is_it_monday : Weekday -> Bool
is_it_monday Mon = True
is_it_monday _   = False
```

For the sake of example, we will prove that for any given day, if it's Monday then `is_it_monday` will return `True`, and `False` otherwise. It's obvious by the definition of `is_it_monday`, but proving that is a whole different story. The type definition that we need to prove is:

```
our_second_proof : (day : Weekday) -> day = Mon -> is_it_monday day = True
```

We gave a name of the first parameter `day : Weekday` so that we can refer to it in the rest of the type definition. The second parameter says that `day = Mon`, and the return value is `is_it_monday day = True`. We can treat the first and the second parameter as givens, since we are allowed to assume them (per definition of implication). With that, we proceed to the function definition:

```
our_second_proof day day_eq_Mon = Refl
```

In this definition, `day` and `day_eq_Mon` are our assumptions (givens). If we run this code in Idris, it will produce an error at compile-time since it cannot deduce that `True` is equal to `is_it_monday day`. In the previous proof example, Idris was able to infer everything by the definitions at compile-time. However, at this point we need to help Idris do the inference since it cannot derive the proof based only on the definitions. We can change the `Refl` to a hole `?prf`:

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

Changing `?prf` to `Refl` completes the proof. We just proved that {$$}\forall x \in \text{Weekdays}, x = \text{Mon} \to IsItMonday(x){/$$}. We assumed {$$}x = \text{Mon}{/$$} is true (by pattern matching against `day_eq_Mon` in our definition), and then used rewriting to alter `x`.

### 6.1.3. Third proof (impossible)

As an inspiration from the previous section, we will prove that `is_it_monday Tue = True` is a contradiction.

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
Q> Seems that at this point we are stuck. We need to find a way to tell Idris that this proof is impossible.

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

As we have seen, natural numbers are very powerful. In this section we will do proofs related to them, and also do a bit of induction. We will start with the following definitions for natural numbers:

```
data MyNat = Zero | Succ MyNat

total mynat_plus : MyNat -> MyNat -> MyNat
mynat_plus Zero     m = m
mynat_plus (Succ k) m = Succ (mynat_plus k m)
```

Note how the definition of `MyNat` is recursive compared to `Weekday`. A consequence of that is that we may need to use induction for some proofs.

X> ### Exercise 5
X>
X> Compare the addition definition to definition 19 in chapter 3.

### 6.2.1. First proof (auto-inference and existence)

We will prove that {$$}0 + a = a{/$$}, given the definitions for natural numbers and addition.

For that, we need to implement a function that accepts a natural number `a`, and returns the proposition that `mynat_plus Zero a = a`.

```
total our_first_proof : (a : MyNat) -> mynat_plus Zero a = a
our_first_proof a = ?prf
```

If we check the type of the hole, we get that the goal is `prf : a = a`, so changing the hole to a `Refl` completes the proof. Idris was able to derive the proof by directly substituting definitions.

To prove the existence of a successor, i.e. `Succ x`, per intuitionistic logic we need to construct a pair where the first element is `x`, and the second element is `Succ x`:

```
total our_first_proof_2 : MyNat -> (MyNat, MyNat)
our_first_proof_2 x = (x, Succ x)
```

We just proved that {$$}\exists x \in \text{MyNat}, Succ(x){/$$}.

### 6.2.2. Second proof (introduction of new givens)

An alternative way to prove that a natural number exists is as follows:

```
total our_second_proof : MyNat
our_second_proof = ?prf
```

If we check the type of the hole, we get:

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

We can slightly rewrite our code:

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

X> ### Exercise 6
X>
X> Simplify `our_second_proof` without the usage of `let`.
X>
X> Hint: Providing a valid type constructor that satisfies (inhabits) the type is a constructive proof.

X> ### Exercise 7
X>
X> Construct a proof similar to `our_second_proof` without defining a function for it, and by using the function `the`.

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

For the base case, we can just use `Refl`, but for the inductive step we need to do something different. We need to find a way to assume (add to list of givens) {$$}a + 0 = a{/$$}, and show that it follows {$$}(a + 1) + 0 = a + 1{/$$} from that assumption. Since we pattern match on `Succ k`, we can use recursion on `k` along with `let` to generate the hypothesis:

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof Zero     = Refl
our_third_proof (Succ k) = let inductive_hypothesis = our_third_proof k in ?conclusion
```

Our proof givens and goals become:

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

X> ### Exercise 8
X>
X> Observe the similarity between this proof and the proof in section 3.4.2.

### 6.2.4. Ordering

Idris has a built-in data type for ordering of natural numbers `LTE`, which stands for less than or equal to. This data type has two constructors:

1. `LTEZero`, used to prove that zero is less than or equal to any natural number
1. `LTESucc`, used to prove that {$$}a \leq b \to S(a) \leq S(b){/$$}

If we check the type of `LTEZero`, we will get the following:

```
Idris> :t LTEZero
LTEZero : LTE 0 right
```

So, `LTEZero` does not accept any arguments, but at the type level it can be passed `right`. With the usage of implicits, we can construct a very simple proof to show that {$$}0 <= 1{/$$}:

```
Idris> LTEZero {right = S Z}
LTEZero : LTE 0 1
```

Similarly, with `LTESucc` we can do the same:

```
Idris> LTESucc {left = Z} {right = S Z}
LTESucc : LTE 0 1 -> LTE 1 2
```

X> ### Exercise 9
X>
X> Check the documentation of `GTE`, and then evaluate `GTE 2 2`. Observe what Idris returns, and think how `GTE` can be implemented in terms of `LTE`.

X> ### Exercise 10
X>
X> We used the built-in type `LTE` defined for `Nat`. Try to come up with a `LTE` definition for `MyNat`.

### 6.2.5. Safe division for natural numbers

Idris provides a function called `divNat` that divides two numbers. Checking the documentation:

```
Idris> :doc divNat
Prelude.Nat.divNat : Nat -> Nat -> Nat
    
    The function is not total as there are missing cases
```

We can try to use it a couple of times:

```
Idris> divNat 4 2
2 : Nat
Idris> divNat 4 1
4 : Nat
Idris> divNat 4 0
divNat 4 0 : Nat
```

As expected, partial functions do not cover all inputs and thus `divNat` does not return a computed value when we divide by zero.

Q> How do we make a function like `divNat` total?
Q>
Q> The only way to make this work is to pass a proof to `divNat` that says that the divisor is not zero. Idris has a built-in function for that, called `divNatNZ`.

We can check the documentation of this function as follows:

```
Idris> :doc divNatNZ
Prelude.Nat.divNatNZ : Nat -> (y : Nat) -> Not (y = 0) -> Nat
    Division where the divisor is not zero.
    The function is Total
```

This function is total, but we need to also provide a parameter (proof) that the divisor is not zero. Fortunately, Idris also provides a function called `SIsNotZ`, which accepts any natural number (through implicit argument `x`) and returns a proof for `x + 1` is not zero.

We can try to construct a few proofs:

```
Idris> SIsNotZ {x = 0}
SIsNotZ : (1 = 0) -> Void
Idris> SIsNotZ {x = 1}
SIsNotZ : (2 = 0) -> Void
Idris> SIsNotZ {x = 2}
SIsNotZ : (3 = 0) -> Void
```

Great. It seems we have everything we need. We can safely divide as follows:

```
Idris> divNatNZ 4 2 (SIsNotZ {x = 1})
2 : Nat
Idris> divNatNZ 4 1 (SIsNotZ {x = 0})
4 : Nat
Idris> divNatNZ 4 0 (SIsNotZ {x = ???})
4 : Nat
```

We cannot construct a proof for the third case, and so it will never be able to divide by zero, which is not allowed anyway.

X> ### Exercise 11
X>
X> Implement `SuccIsNotZ` for `MyNat` that works similarly to `SIsNotZ`.

### 6.2.6. Maximum of two numbers

I> ### Definition 6
I>
I> The maximum of two numbers, a and b, is defined as:
I>
I> {$$}max(a, b) = \left\{ \begin{array}{ll} b\text{, if } a \leq b \\   a\text{, otherwise} \end{array} \right.{/$$}

In this section we will try to prove that {$$}b \leq a \to b = max(a, b){/$$}. Idris already has a built-in function `maximum`, so we can re-use that. Next, we need to figure out the type of the function to approach the proof. Intuitively, we can try the following:

```
our_proof : (a : Nat) -> (b : Nat) -> a <= b -> maximum a b = b
our_proof a b a_lt_b = ?prf
```

However this won't work since `a <= b` is a `Bool`, not a `Type`. At the type level, we need to rely on `LTE` which is a `Type`.

```
our_proof : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
our_proof a b a_lt_b = ?prf
```

This compiles and we have to figure out the hole. If we check its type, we get

```
  a : Nat
  b : Nat
  a_lt_b : LTE a b
--------------------------------------
prf : maximum a b = b
```

This looks a bit complicated, so we can further simplify by breaking the proof into several cases (by adding pattern matching):

```
our_proof : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
our_proof Z Z _              = Refl
our_proof Z (S k) _          = Refl
our_proof (S k) (S j) a_lt_b = ?prf
```

We get the following:

```
  k : Nat
  j : Nat
  a_lt_b : LTE (S k) (S j)
--------------------------------------
prf : S (maximum k j) = S j
```

It seems like we made progress, as this gives us something to work with. We can use induction on `k` and `j`, and use a hole for the third parameter to ask Idris what type we need to satisfy:

```
our_proof : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
our_proof Z Z _              = Refl
our_proof Z (S k) _          = Refl
our_proof (S k) (S j) a_lt_b = let I_H = (our_proof k j ?a) in rewrite IH in Refl
```

The hole produces the following:

```
Holes: Main.a
Idris> :t a
  k : Nat
  j : Nat
  a_lt_b : LTE (S k) (S j)
--------------------------------------
a : LTE k j
Holes: Main.a
```

Q> How do we go from {$$}S(a) \leq S(b) \to a \leq b{/$$}?
Q>
Q> It seems pretty obvious that if we know that {$$}1 \leq 2{/$$}, then also {$$}0 \leq 1{/$$}, but we still need to find out how to tell Idris that this is true. For this, Idris has a built-in function `fromLteSucc`:
Q>
Q> ```
Q> Idris> :doc fromLteSucc
Q> Prelude.Nat.fromLteSucc : LTE (S m) (S n) -> LTE m n
Q>     If two numbers are ordered, their predecessors are too
Q> ```

It seems we have everything we need to conclude our proof. We can proceed as follows:

```
total
our_proof : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
our_proof Z Z _              = Refl
our_proof Z (S k) _          = Refl
our_proof (S k) (S j) a_lt_b = let IH = (our_proof k j (fromLteSucc a_lt_b)) in rewrite IH in Refl
```

Or, a more simplified version:

```
total
our_proof : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
our_proof Z Z _              = Refl
our_proof Z (S k) _          = Refl
our_proof (S k) (S j) a_lt_b = rewrite (our_proof k j (fromLteSucc a_lt_b)) in Refl
```

X> ### Exercise 12
X>
X> Use `fromLteSucc` with implicits to construct some proofs.

## 6.3. Trees

A tree structure is a way to represent hierarchical data. We can define a tree structure using the following implementation:

```
data Tree = Leaf | Node Nat Tree Tree
```

Which states that a tree is defined as one of:

1. `Leaf`, which has no values
1. `Node`, which holds a number and points to two other trees (which can be either `Node`s or `Leaf`s)

For example, to represent the following tree, we can use the expression `Node 2 (Node 1 Leaf Leaf) (Node 3 Leaf Leaf)`:

```
  2
 / \
1   3
```

I> ### Definition 7
I>
I> The depth of a tree is defined as the number of links between nodes.

We can implement the recursive function `depth` as follows:

```
depth : Tree -> Nat
depth Leaf = 0
depth (Node n l r) = 1 + maximum (depth l) (depth r)
```

If we pass a `Tree` to the function `depth`, then Idris will pattern match the types and we can extract the values. For example, in the `Node` case, we pattern match to extract the sub-trees `l` and `r` for further processing. For the `Leaf` case, since it's just an empty leaf there are no links to it.

### 6.4.1. The depth of any tree is >= 0

We can approach the proof as follows:

```
depth_tree_gt_0 : (tr : Tree) -> GTE (depth tr) 0
depth_tree_gt_0 tr = ?prf
```

For the hole, we get:

```
  tr : Tree
--------------------------------------
prf : LTE 0 (depth tr)
```

Doesn't seem like we have enough information. We can proceed by proof with cases:

```
depth_tree_gt_0 : (tr : Tree) -> GTE (depth tr) 0
depth_tree_gt_0 Leaf             = ?prf1
depth_tree_gt_0 (Node v tr1 tr2) = ?prf2
```

The holes are:

```
Idris> :t prf1
--------------------------------------
prf1 : LTE 0 0
Holes: Main.prf2, Main.prf1
Idris> :t prf2
  tr1 : Tree
  tr2 : Tree
  _t : Nat
--------------------------------------
prf2 : LTE 0 (S (maximum (depth tr1) (depth tr2)))
```

For the first case, it's pretty easy. We just use the constructor `LTEZero` with implicit `right = 0`:

```
depth_tree_gt_0 : (tr : Tree) -> GTE (depth tr) 0
depth_tree_gt_0 Leaf             = LTEZero {right = 0}
depth_tree_gt_0 (Node v tr1 tr2) = ?prf2
```

For the second case, it also seems we need to use `LTEZero`, but the second argument is `(S (maximum (depth tr1) (depth tr2)))`.

```
depth_tree_gt_0 : (tr : Tree) -> GTE (depth tr) 0
depth_tree_gt_0 Leaf             = LTEZero {right = 0}
depth_tree_gt_0 (Node v tr1 tr2) = LTEZero {right = 1 + maximum (depth tr1) (depth tr2)}
```

And thus, we have proven that the depth of any tree is greater or equal to zero.
