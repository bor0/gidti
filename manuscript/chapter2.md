# 2. Formal systems

**_Definition 1_**: According to Wikipedia, a **formal system** is a system of abstract thought based on the model of mathematics. A formal system consists of:

1. A **formal language** that contains:
    1. A finite set of symbols, which when combined are used for constructing new formulas (finite strings of symbols)
    1. A grammar, which is a rule that tells us how we can construct formulas based on the symbols (well-formed formulas)
1. A set of **axioms**, that is, a starting set that we take for granted without having any proofs for.
1. A set of **inference rules**.

After a formal system is defined, other formal systems can extend it. For example, the ZFC set theory is based on first-order logic, which is based on propositional logic which represents a formal system. We'll discuss this theory briefly in the next chapter.

**_Definition 2_**: For a formal system, the system is **incomplete** if there are statements that are true in that system, but which cannot be proved to be true inside the system. Conversely, the system is **complete** if all true statements can be proved.

**_Example 1_**: The statement "This statement is not provable" can either be true or false. In the case it is true, then it is not provable. Alternatively, in the case it is false then it is provable, but we're trying to prove something that is false. Thus the system is incomplete, because some truths are unprovable.

**_Definition 3_**: For a formal system, the system is **inconsistent** if there is a theorem in the system that is contradictory. Conversely, the system is **consistent** if there are no theorems that are contradictory.

**_Example 2_**: A simple example is the statement "This statement is false". This statement is true if and only if it is false, and therefore it is neither true nor false.

In general, we often put our focus on which parts of mathematics can be formalized in concrete formal systems, rather than trying to find a theory in which all of mathematics can be developed. The reason for that is Gödel's incompleteness theorem. This theorem states that there doesn't exist[^1] a formal system that is both complete and consistent. As a result, it is better to reason about a formal system outside of the system itself, i.e. as the famous saying goes to think outside of the box. Similarly to how we sometimes do meta-thinking to improve ourselves.

In conclusion, formal systems are our attempt to abstract models, whenever we reverse engineer nature in an attempt to understand it better. They may be imperfect, but are nevertheless useful tools for reasoning.

## 2.1. MU puzzle example

The MU puzzle is a formal system which we'll have a look at as an example.

**_Definition 1_**: We're given a starting string `MI`, combined with a few inference rules, or transformation rules:

| **No.** | **Rule**        | **Description**                  | **Example**        |
| --- | --------------- | -------------------------------- | ------------------ |
| 1 | x`I` → x`IU`    | Add `U` at the end of any string | `MI` to `MIU`      |
| 2 | `M`x → M`xx`    | Double the string after `M`      | `MIU` to `MIUIU`   |
| 3 | x`III`y → x`U`y | Replace `III` with `U`           | `MUIIIU` to `MUUU` |
| 4 | x`UU`y → xy     | Remove `UU`                      | `MUUU` to `MU`     |

**_Example 1_**: We will show (or prove) how we can get from `MII` to `MIIU` using the inference rules:

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
    1. Set of symbols is {$$}\{ {M, I, U}\}{/$$}.
    1. A string is well-formed if the first letter is `M` and there are no other `M` letters. Examples: `M`, `MIUIU`, `MUUUIII`.
1. `MI` is the starting string, i.e. axiom.
1. The rules of inference are defined in the table above.

Question: Can we get from `MI` to `MU` with this system?

In order to answer this, we will use an invariant - a property that holds true whenever we apply some of the rules - with induction to prove our claim.

Note that, in order to be able to apply rule 3, we need to have the number of subsequent I's to be divisible by 3. So let's have our invariant say that "There is no sequence of `I`'s in the string that with length divisible by 3":

1. For the starting axiom, we have one `I`. Invariant OK.
1. Applying rule 2 will be doubling the number of `I`'s, so we can have: `I`, `II`, `IIII`, `IIIIIII` (in particular, {$$}2^n{/$$} `I`'s). Invariant OK.
1. Applying rule 3 will be reducing the number of `I`'s by 3. But note that {$$}2^n - 3{/$$} is still not divisible by 3[^2]. Invariant OK.

So we've shown that with the starting axiom `MI` it is not possible to get to `MU`.

But if we look carefully, we've used a different formal system to reason about `MU` (i.e. divisibility by 3). This is because the puzzle cannot be solved in its own system. Otherwise, an algorithm would keep trying different inference rules of `MU` indefinitely (not knowing that `MU` is impossible).

In general, for any formal system there's this limitation. As we've seen, Gödel's theorem shows that there's no formal system that can contain all possible truths, because it cannot prove some truths about its own structure.

So, having experience with different formal systems and combining them as needed can be useful.

**_Exercise 1_**: Modify the transformation rules of the MU puzzle that will allow the transition from `MI` to `MU`.

**_Exercise 2_**: Try to think of a real-world scenario and model it using a formal system, and then try to apply a few of the transformation rules in order to demonstrate how we can get from point A to point B.

[^1]: Note that this theorem only holds for systems that allow expressing arithmetic of natural numbers (e.g. Peano, ZFC, but first-order logic also has some paradoxes if we allow self-referential statements). We will look into this systems in the next chapter.
[^2]: After having introduced ourselves to proofs, you will be given an exercise to prove this fact.
