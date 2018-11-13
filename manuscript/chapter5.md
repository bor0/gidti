# 5. Proving in Idris

In this chapter we will provide several examples to demonstrate the power of Idris. We will do mathematical proofs. There are a lot of Idris built-ins that will help us achieve our goals and in each section we will introduce the relevant definitions.

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
X> Evaluate `the Nat 3` and `the Integer 3` and note the differences. Afterwards, try to implement `the'` that will act just like `the` and test the previous evaluations again.

X> ### Exercise 3
X>
X> Given `data Or a b = Or_introl a | Or_intror b`, show that {$$}a \to (a \lor b){/$$} and {$$}b \to (a \lor b){/$$}.
X>
X> Hint:
X>
X> ```
X> proof_1 : a -> Or a b
X> proof_1 a = Or_introl ?prf
X> ```

## 5.1. Weekdays

In this section we will introduce a way to represent weekdays and then do some proofs with them. We start with the following data structure:

```
data Weekday = Mon | Tue | Wed | Thu | Fri | Sat | Sun
```

The `Weekday` type has 7 constructors, one for each weekday. The data type and its type constructors do not accept any parameters.

### 5.1.1. First proof (auto-inference)

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

Note how we used the function `next_day Mon = Tue` at the type level, for the type of `our_first_proof`. Since `next_day` is total, Idris will be able to evaluate `next_day Mon` at compile-time. We used a hole for the definition since we still don't know what the proof will look like. If we run this code in Idris, it will tell us we have a hole `prf` to fill:

```
Type checking ./first_proof.idr
Holes: Main.prf
Idris> :t prf
--------------------------------------
prf : Tue = Tue
```

Checking the type of `prf` we notice how Idris evaluated the left part of the equation at compile-time. In order to prove that `Tue = Tue`, we can just use `Refl`:

```
our_first_proof : next_day Mon = Tue
our_first_proof = Refl
```

Reloading the file in Idris:

```
Idris> :r
Type checking ./first_proof.idr
```

The type check was successful. Per the Curry-Howard isomorphism, this means that we've successfully proven that `next_day Mon = Tue`. So, after Monday comes Tuesday!

X> ### Exercise 4
X>
X> Remove one or more pattern match definitions of `next_day` and observe the error that Idris will produce. Afterwards, alter the function so that it is not total anymore, and observe the error.

X> ### Exercise 5
X>
X> Implement `prev_day` and prove that Sunday is before Monday.

### 5.1.2. Second proof (rewrite)

In addition to the previous proof, we'll implement a function that accepts a `Weekday` and returns `True` if it's Monday and `False` otherwise.

```
is_it_monday : Weekday -> Bool
is_it_monday Mon = True
is_it_monday _   = False
```

For the sake of example, we will prove that for any given day, if it's Monday then `is_it_monday` will return `True`. It's obvious from the definition of `is_it_monday`, but proving that is a whole different story. The type definition that we need to prove is:

```
our_second_proof : (day : Weekday) -> day = Mon ->
    is_it_monday day = True
```

We gave a name of the first parameter `day : Weekday`, so that we can refer to it in the rest of the type definition. The second parameter says that `day = Mon` and the return value is `is_it_monday day = True`. We can treat the first and the second parameter as given, since we are allowed to assume them (per definition of implication). With that, we proceed to the function definition:

```
our_second_proof day day_eq_Mon = Refl
```

In this definition, `day` and `day_eq_Mon` are our assumptions (given). If we run this code in Idris, it will produce an error at compile-time since it cannot deduce that `True` is equal to `is_it_monday day`. In the previous proof example, Idris was able to infer everything from the definitions at compile-time. However, at this point we need to help Idris do the inference since it cannot derive the proof based only on the definitions. We can change the `Refl` to a hole `prf`:

```
  day : Weekday
  day_eq_Mon : day = Mon
--------------------------------------
prf : is_it_monday day = True
```

Note how checking the type of the hole lists the given/premises (above the separator), and our goal(s) (below the separator). We see that along with `prf` we also get `day` and `day_eq_Mon` in the list of given, per the left hand side of the function definition of `our_second_proof`.

Q> How do we replace something we have in the given, with the goal?
Q>
Q> If we only had a way to tell Idris that it just needs to replace `day` with `day = Mon` to get to `is_it_monday Mon = True`, it will be able to infer the rest.

I> ### Definition 3
I>
I> The `rewrite` keyword can be used to rewrite expressions. If we have `X : x = y`, then the syntax `rewrite X in Y` will replace all occurrences of `x` with `y` in `Y`.

With the power to do rewrites, we can attempt the proof as follows:

```
our_second_proof : (day : Weekday) -> day = Mon ->
    is_it_monday day = True
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

Changing `prf` to `Refl` completes the proof. We just proved that {$$}\forall x \in \text{Weekdays}, x = \text{Mon} \to IsItMonday(x){/$$}. We assumed {$$}x = \text{Mon}{/$$} is true (by pattern matching against `day_eq_Mon` in our definition), and then used rewriting to alter `x`.

X> ### Exercise 6
X>
X> Implement the function `is_it_sunday` that returns `True` if the given day is Sunday, and `False` otherwise.

X> ### Exercise 7
X>
X> In addition to Exercise 6, prove the following formula in Idris: {$$}\forall x \in \text{Weekdays}, x = \text{Sun} \to IsItSunday(x){/$$}.

### 5.1.3. Third proof (impossible)

In this section we will prove that `is_it_monday Tue = True` is a contradiction.

Per intuitionistic logic, in order to prove that {$$}P{/$$} is a contradiction, we need to prove {$$}P \to \bot{/$$}. Idris provides an empty data type `Void`. This data type has no type constructors (proofs) for it.

To prove that `is_it_monday Tue = True` is a contradiction, we will do the following:

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
I> The `impossible` keyword can be used to prove statements that are not true. With this keyword, we say that a proof for a data type cannot be constructed since there does not exist a type constructor for that particular type.

We will slightly rewrite the function:

```
our_third_proof : is_it_monday Tue = True -> Void
our_third_proof Refl impossible
```

With this syntax, we're telling Idris that the reflexivity of `False = True` is impossible and thus the proof is complete.

X> ### Exercise 8
X>
X> Check the documentation of `Void` and try to implement `Void'` yourself. Rewrite the proof above to use `Void'` instead of `Void`.

X> ### Exercise 9
X>
X> Prove that `1 = 2` is a contradiction.
X>
X> Hint: The type is `1 = 2 -> Void`

## 5.2. Natural numbers

In this section we will prove facts about natural numbers and also do some induction. Recall that a natural number is defined either as zero or as the successor of a natural number. So, `0, S 0, S (S 0), ...` are the first natural numbers. We will start with the following definitions for natural numbers:

```
data MyNat = Zero | Succ MyNat

total mynat_plus : MyNat -> MyNat -> MyNat
mynat_plus Zero     m = m
mynat_plus (Succ k) m = Succ (mynat_plus k m)
```

Note how the definition of `MyNat` is recursive compared to `Weekday`. A consequence of that is that we may need to use induction for some proofs.

X> ### Exercise 10
X>
X> Compare the addition definition to Definition 21 in chapter 2.

### 5.2.1. First proof (auto-inference and existence)

We will prove that {$$}0 + a = a{/$$}, given the definitions for natural numbers and addition.

For that, we need to implement a function that accepts a natural number `a` and returns the proposition that `mynat_plus Zero a = a`.

```
total our_first_proof : (a : MyNat) -> mynat_plus Zero a = a
our_first_proof a = ?prf
```

If we check the type of the hole, we get that the goal is `prf : a = a`, so changing the hole to a `Refl` completes the proof. Idris was able to automatically infer the proof by directly substituting definitions.

To prove the existence of a successor, i.e. `Succ x`, per intuitionistic logic we need to construct a pair where the first element is `x : MyNat` and the second element is `Succ x : MyNat`. Idris has a built-in data structure for constructing dependent pairs called `DPair`.

```
total our_second_proof : MyNat -> DPair MyNat (\_ => MyNat)
our_second_proof x = MkDPair x (Succ x)
```

We just proved that {$$}\exists x \in \text{MyNat}, Succ(x){/$$}.

X> ### Exercise 11
X>
X> Check the documentation of `DPair` and `MkDPair` and try to construct some dependent pairs.

### 5.2.2. Second proof (introduction of a new given)

An alternative way to prove that a natural number exists is as follows:

```
total our_second_proof : MyNat
our_second_proof = ?prf
```

If we check the type of the hole, we get:

```
Holes: Main.prf
Idris> :t prf
--------------------------------------
prf : MyNat
```

Q> By the definition of `MyNat` we are sure that there exists a type constructor of `MyNat`, but how do we tell Idris?

I> ### Definition 5
I>
I> The `let` keyword that we introduced earlier allows us to add a new given to the list of hypotheses.

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

X> ### Exercise 12
X>
X> Simplify `our_second_proof` without the use of `let`.
X>
X> Hint: Providing a valid type constructor that satisfies (inhabits) the type is a constructive proof.

X> ### Exercise 13
X>
X> Construct a proof similar to `our_second_proof` without defining a function for it and by using the function `the`.

### 5.2.3. Third proof (induction)

We will prove that {$$}a + 0 = a{/$$}. We can try to use the same approach as in 5.2.1:

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof a = Refl
```

If we try to run the code above, Idris will produce an error saying that there is a type mismatch between `a` and `mynat_plus a Zero`.

Q> It seems that we have just about all the definitions we need, but we're missing a piece. How do we re-use our definitions?
Q>
Q> To prove that {$$}a + 0 = a{/$$}, we can use mathematical induction starting with the definitions we already have as the base case and build on top of that until {$$}a + 0 = a{/$$}. That is, we need to prove that {$$}0 + 0 = 0 \to (a - a) + 0 = a - a \to ... \to (a - 1) + 0 = a - 1 \to a + 0 = a{/$$}.

From here, we rewrite our function to contain a base case and an inductive step:

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof Zero     = ?base
our_third_proof (Succ k) = ?ind_hypothesis
```

Note how we used pattern matching against the definition of natural numbers. Pattern matching is similar to using proof by cases. Checking the types of the holes:

```
Holes: Main.ind_hypothesis, Main.base
Idris> :t base
--------------------------------------
base : Zero = Zero
Idris> :t ind_hypothesis
  k : MyNat
--------------------------------------
ind_hypothesis : Succ (mynat_plus k Zero) = Succ k
```

For the base case we can just use `Refl`, but for the inductive step we need to do something different. We need to find a way to assume (add to list of given) {$$}a + 0 = a{/$$} and show that {$$}(a + 1) + 0 = a + 1{/$$} follows from that assumption. Since we pattern match on `Succ k`, we can use recursion on `k` along with `let` to generate the hypothesis:

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof Zero     = Refl
our_third_proof (Succ k) = let ind_hypothesis = our_third_proof k in
                           ?conclusion
```

Our proof given and goals become:

```
  k : MyNat
  ind_hypothesis : mynat_plus k Zero = k
--------------------------------------
conclusion : Succ (mynat_plus k Zero) = Succ k
```

To prove the conclusion, we can simply rewrite the inductive hypothesis in the goal and we are done.

```
total our_third_proof : (a : MyNat) -> mynat_plus a Zero = a
our_third_proof Zero     = Refl
our_third_proof (Succ k) = let ind_hypothesis = our_third_proof k in
                           rewrite ind_hypothesis in
                           Refl
```

This concludes the proof.

X> ### Exercise 14
X>
X> Observe the similarity between this proof and the proof in section 2.3.5.

### 5.2.4. Ordering

Idris has a built-in data type for ordering of natural numbers `LTE`, which stands for less than or equal to. This data type has two constructors:

1. `LTEZero`, used to prove that zero is less than or equal to any natural number
1. `LTESucc`, used to prove that {$$}a \leq b \to S(a) \leq S(b){/$$}

If we check the type of `LTEZero`, we will get the following:

```
Idris> :t LTEZero
LTEZero : LTE 0 right
```

So, `LTEZero` does not accept any arguments, but we can pass `right` at the type level. With the use of implicits, we can construct a very simple proof to show that {$$}0 \leq 1{/$$}:

```
Idris> LTEZero {right = S Z}
LTEZero : LTE 0 1
```

Similarly, with `LTESucc` we can do the same:

```
Idris> LTESucc {left = Z} {right = S Z}
LTESucc : LTE 0 1 -> LTE 1 2
```

X> ### Exercise 15
X>
X> Check the documentation of `GTE`, and then evaluate `GTE 2 2`. Observe what Idris returns and think how `GTE` can be implemented in terms of `LTE`.

X> ### Exercise 16
X>
X> We used the built-in type `LTE` which is defined for `Nat`. Try to come up with a `LTE` definition for `MyNat`.

### 5.2.5. Safe division

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

This function is total, but we need to also provide a parameter (proof) that the divisor is not zero. Fortunately, Idris also provides a function called `SIsNotZ`, which accepts any natural number (through implicit argument `x`) and returns a proof that `x + 1` is not zero.

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

We cannot construct a proof for the third case and so it will never be able to divide by zero, which is not allowed anyway.

X> ### Exercise 17
X>
X> Implement `SuccIsNotZ` for `MyNat` that works similarly to `SIsNotZ`.

X> ### Exercise 18
X>
X> Implement `divMyNatNZ` for `MyNat` that works similarly to `divNatNZ`.

### 5.2.6. Maximum of two numbers

I> ### Definition 6
I>
I> The maximum of two numbers `a` and `b` is defined as:
I>
I> {$$}max(a, b) = \left\{ \begin{array}{ll} b\text{, if } a \leq b \\   a\text{, otherwise} \end{array} \right.{/$$}

In this section we will try to prove that {$$}b \leq a \to b = max(a, b){/$$}. Idris already has a built-in function `maximum`, so we can re-use that. Next, we need to figure out the type of the function to approach the proof. Intuitively, we can try the following:

```
our_proof : (a : Nat) -> (b : Nat) -> a <= b -> maximum a b = b
our_proof a b a_lt_b = ?prf
```

However this won't work since `a <= b` is a `Bool` (per the function `<=`), not a `Type`. At the type level we need to rely on `LTE` which is a `Type`.

```
our_proof : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
our_proof a b a_lt_b = ?prf
```

This compiles and we have to figure out the hole. If we check its type, we get:

```
  a : Nat
  b : Nat
  a_lt_b : LTE a b
--------------------------------------
prf : maximum a b = b
```

This looks a bit complicated, so we can further simplify by breaking the proof into several cases by adding pattern matching for all combinations of the parameters' type constructors:

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
our_proof (S k) (S j) a_lt_b = let I_H = (our_proof k j ?prf) in
                               rewrite IH in
                               Refl
```

The hole produces the following:

```
Holes: Main.prf
Idris> :t prf
  k : Nat
  j : Nat
  a_lt_b : LTE (S k) (S j)
--------------------------------------
prf : LTE k j
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
our_proof (S k) (S j) a_lt_b = let fls = fromLteSucc a_lt_b in
                               let IH  = (our_proof k j fls) in
                               rewrite IH in
                               Refl
```

or a more simplified version:

```
total
our_proof : (a : Nat) -> (b : Nat) -> LTE a b -> maximum a b = b
our_proof Z Z _              = Refl
our_proof Z (S k) _          = Refl
our_proof (S k) (S j) a_lt_b = rewrite
                               (our_proof k j (fromLteSucc a_lt_b)) in
                               Refl
```

X> ### Exercise 19
X>
X> Use `fromLteSucc` with implicits to construct some proofs.

### 5.2.7. List of even naturals

We will prove that a list of even numbers contains no odd numbers. We will re-use the functions `even` in 4.1.4 and `even_members` in 4.1.11. We will also need another function to check if a list has odd numbers:

```
total has_odd : MyList Nat -> Bool
has_odd End         = False
has_odd (Cons x l') = if (even x) then False else has_odd l'
```

Now, to prove that a list of even numbers contains no odd numbers, we can use the following type definition:

```
even_members_list_only_even : (l : MyList Nat) ->
    has_odd (even_members l) = False
```

Note that `has_odd` is branching computation depending on the value of `even x`, so we have to pattern match with value of expressions by using the keyword `with`. The base case is simply `Refl`:

```
even_members_list_only_even End = Refl
```

However, for the inductive step, we will use `with` on `even n` and produce a proof depending on the evenness of the number:

```
even_members_list_only_even (Cons n l') with (even n) proof even_n
  even_members_list_only_even (Cons n l') | False =
      let IH = even_members_list_only_even l' in ?a
  even_members_list_only_even (Cons n l') | True  =
      ?b
```

Note how we specified `proof even_n` right after the expression in the `with` match. The `proof` keyword followed by a variable brings us the proof of the expression to the list of premises. So, the expression `with (even n) proof even_n` will pattern match on the results of `even n`, and will also bring the proof `even n` in the premises. If we now check the first hole:

```
  n : Nat
  l' : MyList Nat
  even_n : False = even n
  IH : has_odd (even_members l') = False
--------------------------------------
a : has_odd (even_members l') = False
```

That should be simple, we can just use `IH` to solve the goal. For the second hole, we have:

```
  n : Nat
  l' : MyList Nat
  even_n : True = even n
--------------------------------------
b : ifThenElse (even n) (Delay False) (Delay (has_odd
    (even_members l'))) = False
```

Seems that `if...then...else` uses a lazy structure, but nevertheless uses `(even n)` to branch the computation.

Q> How do we rewrite the inductive hypothesis to the goal in this case?
Q>
Q> It seems that we can't just rewrite here, since `even_n` has the order of the equality reversed. Idris provides a function called `sym` which takes an equality of `a = b` and converts it to `b = a`.

We can try to rewrite `sym even_n` to the goal, and it now becomes:

```
  n : Nat
  l' : MyList Nat
  even_n : True = even n
  _rewrite_rule : even n = True
--------------------------------------
b : False = False
```

Thus, the complete proof:

```
even_members_list_only_even : (l : MyList Nat) ->
    has_odd (even_members l) = False
even_members_list_only_even End = Refl
even_members_list_only_even (Cons n l') with (even n) proof even_n
  even_members_list_only_even (Cons n l') | False =
      let IH = even_members_list_only_even l' in
      IH
  even_members_list_only_even (Cons n l') | True  =
      rewrite sym even_n in
      Refl
```

Q> How did mathematical induction work in this case?
Q>
Q> Mathematical induction is defined in terms of natural numbers, but in this case we used induction to prove a fact about a list. This works because we used a more general induction called structural induction. According to Wikipedia, structural induction is used to prove that some proposition {$$}P(x){/$$} holds for all {$$}x{/$$} of some sort of recursively defined structure, such as formulas, lists, or trees. For example, for lists we used `End` as the base case, and `Cons` as the inductive step. Thus, mathematical induction is a special case of structural induction for the `Nat` type.

X> ### Exercise 20
X>
X> Rewrite `has_odd` to use `with` in the recursive case, and then repeat the proof above.

### 5.2.8. Partial orders

I> ### Definition 7
I>
I> A binary relation {$$}R{/$$} on some set {$$}S{/$$} is a partial order if the following properties are satisfied:
I>
I> 1. {$$}\forall a \in S, a R a{/$$}, i.e. reflexivity
I> 1. {$$}\forall a, b, c \in S, a R b \land b R c \to a R c{/$$}, i.e. transitivity
I> 1. {$$}\forall a, b \in S, a R b \land b R a \to a = b{/$$}, i.e. antisymmetry

Let's abstract this in Idris as an `interface`:

```
interface Porder (a : Type) (Order : a -> a -> Type) | Order where
    total proofR : Order n n -- reflexivity
    total proofT : Order n m -> Order m p -> Order n p -- transitivity
    total proofA : Order n m -> Order m n -> n = m -- antisymmetry
```

The interface `Porder` accepts a `Type` and a relation `Order`, which is a binary function. Since the interface has more than two parameters, we specify that `Order` is a determining parameter, i.e. the parameter used to resolve the instance.

Now that we have our abstract interface we can build a concrete implementation for it:

```
implementation Porder Nat LTE where
    proofR {n = Z}   =
        LTEZero
    proofR {n = S _} =
        LTESucc proofR

    proofT LTEZero           _                 =
        LTEZero
    proofT (LTESucc n_lte_m) (LTESucc m_lte_p) =
        LTESucc (proofT n_lte_m m_lte_p)

    proofA LTEZero           LTEZero           =
        Refl
    proofA (LTESucc n_lte_m) (LTESucc m_lte_n) =
        let IH = proofA n_lte_m m_lte_n in rewrite IH in Refl
```

We proved that the binary operation "less than or equal to" for `Nat`s make a `Porder`. Interfaces allow us to group one or more functions, and an implementation of a specific type is guaranteed to implement all such functions.

X> ### Exercise 21
X>
X> Convince yourself using pen and paper that {$$}\leq{/$$} on natural numbers makes a partial order, i.e. it satisfies all properties of Definition 7. Afterwards, try to understand the proofs for reflexivity, transitivity, and antisymmetry by deducing them yourself in Idris using holes.

## 5.3. Computations as types

As we stated earlier, types are first-class citizens in Idris. In this example we will see how we can convert a function that does some computation to its corresponding type definition.

Re-using the same definition of `MyVect`, we can write a function to test if all elements are same in a given list:

```
allSame : (xs : MyVect n) -> Bool
allSame Empty                = True
allSame (Cons x Empty)       = True
allSame (Cons x (Cons y xs)) = x == y && allSame xs
```

Idris will return `True` in case all elements are equal to each other, and `False` otherwise. Let's now think how we can represent this function in terms of types. We want to have a type `AllSame` that has three constructors:

1. `AllSameZero` which is a proof for `AllSame` in case of an empty list
1. `AllSameOne` which is a proof for `AllSame` in case of a single-element list
1. `AllSameMany` which is a proof for `AllSame` in case of a list with multiple elements

This is how the data type could look like:

```
data AllSame : MyVect n -> Type where
    AllSameZero : AllSame Empty
    AllSameOne  : (x : Nat) -> AllSame (Cons x Empty)
    AllSameMany : (x : Nat) -> (y : Nat) -> (ys : MyVect _) ->
        True = (x == y) -> AllSame (Cons y ys) ->
        AllSame (Cons x (Cons y ys))
```

The constructors `AllSameZero` and `AllSameOne` are easy. However, the recursive constructor `AllSameMany` is a bit trickier. It accepts two natural numbers `x` and `y`, a list `ys`, a proof that `x` and `y` are same, and a proof that `y` concatenated to `ys` is a same-element list. Given this, it will produce a proof that `x` concatenated to `y` concatenated to `ys` is also a same-element list. This type definition captures exactly the definition of a list that would contain all same elements.

Interacting with the constructors:

```
Idris> AllSameZero
AllSameZero : AllSame Empty
Idris> AllSameOne 1
AllSameOne 1 : AllSame (Cons 1 Empty)
Idris> AllSameMany 1 1 Empty Refl (AllSameOne 1)
AllSameMany 1 1 Empty Refl (AllSameOne 1) : AllSame (Cons 1
    (Cons 1 Empty))
```

The third example is a proof that the list `[1, 1]` has same elements. However, if we try to use the constructor with different elements:

```
Idris> AllSameMany 1 2 Empty
AllSameMany 1 2 Empty : (True = False) -> AllSame (Cons 2 Empty) ->
    AllSame (Cons 1 (Cons 2 Empty))
```

We see that Idris requires us to provide a proof that `True = False`, which is impossible. So for some lists the type `AllSame` cannot be constructed, but for some it can. If we now want to make a function that given a list, it maybe produces a type `AllSame`, we need to consider the `Maybe` data type first which has the following definition:

```
data Maybe a = Just a | Nothing
```

Interacting with it:

```
Idris> the (Maybe Nat) (Just 3)
Just 3 : Maybe Nat
Idris> the (Maybe Nat) Nothing
Nothing : Maybe Nat
```

We can now proceed to write our function:

```
mkAllSame : (xs : MyVect n) -> Maybe (AllSame xs)
mkAllSame Empty                = Just AllSameZero
mkAllSame (Cons x Empty)       = Just (AllSameOne x)
mkAllSame (Cons x (Cons y xs)) with (x == y) proof x_eq_y
    mkAllSame (Cons x (Cons y xs)) | False =
        Nothing
    mkAllSame (Cons x (Cons y xs)) | True  =
        case (mkAllSame (Cons y xs)) of
            Just y_eq_xs => Just (AllSameMany x y xs x_eq_y y_eq_xs)
            Nothing      => Nothing
```

Interacting with it:

```
Idris> mkAllSame (Cons 1 Empty)
Just (AllSameOne 1) : Maybe (AllSame (Cons 1 Empty))
Idris> mkAllSame (Cons 1 (Cons 1 Empty))
Just (AllSameMany 1 1 Empty Refl (AllSameOne 1)) : Maybe (AllSame
    (Cons 1 (Cons 1 Empty)))
Idris> mkAllSame (Cons 1 (Cons 2 Empty))
Nothing : Maybe (AllSame (Cons 1 (Cons 2 Empty)))
```

For lists that contain same elements it will use the `Just` constructor, and `Nothing` otherwise. Finally, we can rewrite our original `allSame` as follows:

```
allSame' : MyVect n -> Bool
allSame' xs = case (mkAllSame xs) of
    Nothing => False
    Just _  => True
```

## 5.4. Trees

A tree structure is a way to represent hierarchical data. We will work with binary trees in this section, which are trees that contain exactly two sub-trees (nodes). We can define this tree structure using the following implementation:

```
data Tree = Leaf | Node Nat Tree Tree
```

This definition states that a tree is defined as one of:

1. `Leaf`, which has no values
1. `Node`, which holds a number and points to two other trees (which can be either `Node`s or `Leaf`s)

For example, we can use the expression `Node 2 (Node 1 Leaf Leaf) (Node 3 Leaf Leaf)` to represent the following tree:

```text
  2
 / \
1   3
```

Edges can be thought of as the number of "links" from a node to its children. Node 2 in the tree above has two edges: {$$}(2, 1){/$$} and {$$}(2, 3){/$$}.

X> ### Exercise 22
X>
X> Come up with a few trees by using the type constructors above.

### 5.4.1. Depth

I> ### Definition 8
I>
I> The depth of a tree is defined as the number of edges from the node to the root.

We can implement the recursive function `depth` as follows:

```
depth : Tree -> Nat
depth Leaf         = 0
depth (Node n l r) = 1 + maximum (depth l) (depth r)
```

If we pass a `Tree` to the function `depth`, then Idris will pattern match the types and we can extract the values. For example, in the `Node` case, we pattern match to extract the sub-trees `l` and `r` for further processing. For the `Leaf` case, since it's just an empty leaf there are no links to it.

In order to prove that the `depth` of a tree is greater than or equal to zero, we can approach the proof as follows:

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

Doesn't seem like we have enough information. We can proceed with proof by cases:

```
depth_tree_gt_0 : (tr : Tree) -> GTE (depth tr) 0
depth_tree_gt_0 Leaf             = ?prf1
depth_tree_gt_0 (Node v tr1 tr2) = ?prf2
```

Checking the types of the holes:

```
Holes: Main.prf2, Main.prf1
Idris> :t prf1
--------------------------------------
prf1 : LTE 0 0
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
depth_tree_gt_0 Leaf             =
    LTEZero {right = 0}
depth_tree_gt_0 (Node v tr1 tr2) =
    LTEZero {right = 1 + maximum (depth tr1) (depth tr2)}
```

Thus, we have proven that the depth of any tree is greater or equal to zero.

### 5.4.2. Map and size

We saw how we can use `map` with lists. It would be neat if we had a way to map trees as well. The following definition will allow us to do exactly that:

```
map_tree : (Nat -> Nat) -> Tree -> Tree
map_tree _ Leaf             = Leaf
map_tree f (Node v tr1 tr2) = (Node (f v)
                                    (map_tree f tr1)
                                    (map_tree f tr2))
```

The function `map_tree` accepts a function and a `Tree` and then returns a modified `Tree` where the function is applied to all values of the nodes. In the case of `Leaf`, it just returns `Leaf`, because there's nothing to map to. In the case of a `Node`, we return a new `Node` whose value is applied to the function `f` and then recursively map over the left and right branches of the node. We can use it as follows:

```
Idris> Node 2 (Node 1 Leaf Leaf) (Node 3 Leaf Leaf)
Node 2 (Node 1 Leaf Leaf) (Node 3 Leaf Leaf) : Tree
Idris> map_tree (\x => x + 1) (Node 2 (Node 1 Leaf Leaf)
    (Node 3 Leaf Leaf))
Node 3 (Node 2 Leaf Leaf) (Node 4 Leaf Leaf) : Tree
```

I> ### Definition 9
I>
I> The size of a tree is defined as the sum of the levels of all nodes.

We will now implement `size_tree` which is supposed to return the total count of all nodes contained in a tree:

```
size_tree : Tree -> Nat
size_tree Leaf = 0
size_tree (Node n l r) = 1 + (size_tree l) + (size_tree r)
```

Trying it with a few trees:

```
Idris> size_tree Leaf
0 : Nat
Idris> size_tree (Node 1 Leaf Leaf)
1 : Nat
Idris> size_tree (Node 1 (Node 2 Leaf Leaf) Leaf)
2 : Nat
```

### 5.4.3. Length of mapped trees

We want to prove that for a given tree and _any_ function `f`, the size of that tree will be the same as the size of that tree mapped with the function `f`:

```
proof_1 : (tr : Tree) -> (f : Nat -> Nat) ->
    size_tree tr = size_tree (map_tree f tr)
```

This type definition describes exactly that. We will use proof by cases and pattern match on `tr`:

```
proof_1 Leaf _             = ?base
proof_1 (Node v tr1 tr2) f = ?i_h
```

Checking the types of the holes:

```
Holes: Main.i_h, Main.base
Idris> :t base
  f : Nat -> Nat
--------------------------------------
base : 0 = 0
Idris> :t i_h
  v : Nat
  tr1 : Tree
  tr2 : Tree
  f : Nat -> Nat
--------------------------------------
i_h: S (plus (size_tree tr1) (size_tree tr2)) =
     S (plus (size_tree (map_tree f tr1)) (size_tree (map_tree f tr2)))
```

For the base case we can just use `Refl`. However, for the inductive hypothesis, we need to do something different. We can try applying the proof recursively to `tr1` and `tr2` respectively:

```
proof_1 : (tr : Tree) -> (f : Nat -> Nat) ->
    size_tree tr = size_tree (map_tree f tr)
proof_1 Leaf _             = Refl
proof_1 (Node v tr1 tr2) f = let IH_1 = proof_1 tr1 f in
                             let IH_2 = proof_1 tr2 f in
                             ?conclusion
```

We get to the following proof state at this point:

```
Holes: Main.conclusion
Idris> :t conclusion
  v : Nat
  tr1 : Tree
  tr2 : Tree
  f : Nat -> Nat
  IH_1 : size_tree tr1 = size_tree (map_tree f tr1)
  IH_2 : size_tree tr2 = size_tree (map_tree f tr2)
--------------------------------------
conclusion : S (plus (size_tree tr1) (size_tree tr2)) =
             S (plus (size_tree (map_tree f tr1))
                 (size_tree (map_tree f tr2)))
```

From here, we can just rewrite the hypothesis:

```
proof_1 (Node v tr1 tr2) f = let IH_1 = proof_1 tr1 f in
                             let IH_2 = proof_1 tr2 f in
                             rewrite IH_1 in
                             rewrite IH_2 in
                             ?conclusion
```

At this point if we check the type of `conclusion` we will note that we can just use `Refl` to finish the proof.
