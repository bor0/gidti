# Conclusion

We've seen how powerful types are. They allow us to put additional constraints on values. This helps with reasoning about our programs since a whole class of non-valid programs will not be accepted by the type checker. For example, if we look at the following function, we immediately know by its type that it returns a natural number:

```
f : Nat -> Nat
f _ = 6
```

Note that there are many constructors (proofs) for `Nat`. For example, we also have that `f _ = 7`. Depending on whether we need 6 or 7 in practice has to be additionally checked. But we're certain that it's a natural number (by the type).

Most programming languages that have a type system have the same expressive power as that of propositional logic (no quantifiers). Dependent types are powerful because they allow us to express quantifiers, which increases the power of expressiveness. As a result, we can write any kind of mathematical proofs.

In mathematics, everything has a precise definition. Same is the case with Idris. We had to give clear definitions of our functions and types prior to proving any theorems.

Q> If Idris proves software correctness, what proves the correctness of Idris?
Q>
Q> Trusted Computing Base (TCB) can be thought of as the "axioms" of Idris, that is, we choose to trust Idris and the way it rewrites, inserts a new given, etc.

The most challenging part is to come up with the precise properties (specifications) that we should prove in order to claim correctness for our software.

The beauty of all of this is that almost everything is just about types and finding inhabitants (values) of types.

* * *
