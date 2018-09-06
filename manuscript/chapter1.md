# 1. Formal systems

I> ### Definition 1
I>
I> According to Wikipedia, a **formal system** is a system of abstract thought based on the model of mathematics. A formal system consists of:
I>
I> 1. A **formal language** that contains:
I>     1. A finite set of symbols, which when combined are used for constructing new formulas (finite strings of symbols)
I>     1. A grammar, which is a rule that tells us how we can construct formulas based on the symbols (well-formed formulas)
I> 1. A set of **axioms**, that is, a starting set that we take for granted without having any proofs for
I> 1. A set of **inference rules** that tell us how we can transform formulas

After a formal system is defined, other formal systems can extend it. For example, set theory is based on first-order logic, which is based on propositional logic which represents a formal system. We'll discuss this theory briefly in the next chapter.

I> ### Definition 2
I>
I> For a given formal system, the system is **incomplete** if there are statements that are true but which cannot be proved to be true inside that system. Conversely, the system is **complete** if all true statements can be proved.

The statement "This statement is not provable" can either be true or false. In the case it is true, then it is not provable. Alternatively, in the case it is false then it is provable, but we're trying to prove something that is false. Thus the system is incomplete, because some truths are unprovable.

I> ### Definition 3
I>
I> For a given formal system, the system is **inconsistent** if there is a theorem in that system that is contradictory. Conversely, the system is **consistent** if there are no theorems that are contradictory.

A simple example is the statement "This statement is false". This statement is true if and only if[^ch1n1] it is false, and therefore it is neither true nor false.

In general, we often put our focus on which parts of mathematics can be formalized in concrete formal systems, rather than trying to find a theory in which all of mathematics can be developed. The reason for that is G&#246;del's incompleteness theorem. This theorem states that there doesn't exist[^ch1n2] a formal system that is both complete and consistent. As a result, it is better to reason about a formal system outside of the system itself, i.e. as the famous saying goes to "think outside of the box". Similarly to how we sometimes do meta-thinking to improve ourselves.

In conclusion, formal systems are our attempt to abstract models, whenever we reverse engineer nature in an attempt to understand it better. They may be imperfect, but are nevertheless useful tools for reasoning.

## 1.1. MU puzzle example

The MU puzzle is a formal system which we'll have a look at as an example.

I> ### Definition 4
I>
I> We're given a starting string `MI`, combined with a few inference rules, or transformation rules:
I>
I> | **No.** | **Rule**                   | **Description**                        | **Example**        |
I> | ------- | -------------------------- | -------------------------------------- | ------------------ |
I> | 1       | x`I` {$$}\to{/$$} x`IU`    | Append `U` at a string ending in `I`   | `MI` to `MIU`      |
I> | 2       | `M`x {$$}\to{/$$} M`xx`    | Double the string after `M`            | `MIU` to `MIUIU`   |
I> | 3       | x`III`y {$$}\to{/$$} x`U`y | Replace `III` inside a string with `U` | `MUIIIU` to `MUUU` |
I> | 4       | x`UU`y {$$}\to{/$$} xy     | Remove `UU` from inside a string       | `MUUU` to `MU`     |
I>

In the inference rules the symbols `M`, `I`, and `U` are part of the system, while `x` is a variable that stands for any symbol(s). For example, `MI` matches rule 2 for `x = I`, and it can also match rule 1 for `x = M`. Another example is `MII` that matches rule 2 for `x = II`, and rule 1 for `x = MI`.

We will show (or prove) how we can get from `MI` to `MIIU` using the inference rules:

1. `MI` (axiom)
1. `MII` (rule 2)
1. `MIIII` (rule 2)
1. `MIIIIIIII` (rule 2)
1. `MUIIIII` (rule 3)
1. `MUUII` (rule 3)
1. `MII` (rule 4)
1. `MIIU` (rule 1)

We can represent the formal description of this system as follows:

1. Formal language
    1. Set of symbols is {$$}\{ M, I, U \}{/$$}
    1. A string is well-formed if the first letter is `M` and there are no other `M` letters. Examples: `M`, `MIUIU`, `MUUUIII`
1. `MI` is the starting string, i.e. axiom
1. The rules of inference are defined in the table earlier

Q> Can we get from `MI` to `MU` with this system?
Q>
Q> In order to answer this, we will use an invariant[^ch1n3] with mathematical induction to prove our claim.
Q>
Q> Note that, in order to be able to apply rule 3, we need to have the number of subsequent `I`'s to be divisible by 3. So let's have our invariant say that "There is no sequence of `I`'s in the string that with length divisible by 3":
Q>
Q> 1. For the starting axiom, we have one `I`. Invariant OK.
Q> 1. Applying rule 2 will be doubling the number of `I`'s, so we can have: `I`, `II`, `IIII`, `IIIIIII` (in particular, {$$}2^n{/$$} `I`'s). Invariant OK.
Q> 1. Applying rule 3 will be reducing the number of `I`'s by 3. But note that {$$}2^n - 3{/$$} is still not divisible by 3[^ch1n4]. Invariant OK.

So we've shown that with the starting axiom `MI` it is not possible to get to `MU`. But if we look carefully, we've used a different formal system to reason about `MU` (i.e. divisibility by 3, which is not part of the MU system). This is because the puzzle cannot be solved in its own system. Otherwise, an algorithm would keep trying different inference rules of `MU` indefinitely (not knowing that `MU` is impossible).

In general, for any formal system there's this limitation. As we've seen, G&#246;del's theorem shows that there's no formal system that can contain all possible truths, because it cannot prove some truths about its own structure.

So, having experience with different formal systems and combining them as needed can be useful.

X> ### Exercise 1
X>
X> Modify the transformation rules of the MU puzzle that will allow the transition from `MI` to `MU`.

X> ### Exercise 2
X>
X> Try to think of a real-world scenario and model it using a formal system, and then try to apply a few of the transformation rules in order to demonstrate how we can get from point A to point B.

[^ch1n1]: The word iff is an abbreviation for "If and only if".

[^ch1n2]: Note that this theorem only holds for systems that allow expressing arithmetic of natural numbers (e.g. Peano, set theory, but first-order logic also has some paradoxes if we allow self-referential statements). We will look into this systems in the next chapter.

[^ch1n3]: An invariant is a property that holds true whenever we apply any of the inference rules.

[^ch1n4]: After having introduced ourselves to proofs, you will be given an exercise to prove this fact.
