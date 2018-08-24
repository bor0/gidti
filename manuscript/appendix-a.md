# Appendix A: Metamath

Metamath is a programming language that can express theorems accompanied by a proof checker. The interesting thing about this language is its simplicity. We start by defining a formal system (variables, symbols, axioms and rules of inference) and proceed with building new theorems based on the formal system.

As we've seen, proofs in mathematics (and Idris to some degree) are usually done at a very high level. Even though the foundations are formal systems, it is very difficult to do proofs at a low level. However, we will show that there are such programming languages like Metamath that work at the lowest level, that is formal systems.

The most basic concept in Metamath is the substitution method. Metamath uses an RPN stack[^apan1] to build hypotheses and then rewrites using the rules of inference in order to reach a conclusion. Metamath has a very simple syntax. A token is a Metamath token if it starts with `$` and is a user-generated token otherwise. The following is a list of Metamath's tokens:

1. `$c` defines constants
1. `$v` defines variables
1. `$f` defines type of variables (floating hypothesis)
1. `$e` defines required arguments (essential hypotheses)
1. `$a` defines axioms
1. `$p` defines proofs
1. `$=` and `$.` start and end body of a proof
1. `$(` and `$)` start and end code comments
1. `${` and `$}` start and end proof blocks

Besides these tokens, there are several rules:

1. A hypothesis is either defined by using the token `$e` or `$f`
1. For every variable in `$e`, `$a` or `$p`, there has to be a `$f` token defined, that is any variable in essential hypothesis/axiom/proof must have defined a type
1. An expression that contains `$f`, `$e` or `$d` is active in the given block from the start of the definition until the end of the block. An expression that contains `$a` or `$p` is active from the start of the definition until the end of the file
1. Proof blocks have an effect on the access of definitions, i.e. scoping. For a given code in a block, only `$a` and `$p` remain visible outside of the block

In the following example we'll define a formal system and demonstrate the use of the rule modus ponens in order to get to a new theorem, based on our initial axioms.

```
$( Declaration of constants $)
$c -> ( ) wff |- I J $.

$( Declaration of variables $)
$v p q $.

$( Properties of variables, i.e. they are well-formed formulas $)
wp $f wff p $. $( wp is a "command" we can use in RPN that represents a well-formed p $)
wq $f wff q $.

$( Modus ponens definition $)
${
    mp1 $e |- p $.
    mp2 $e |- ( p -> q ) $.
    mp  $a |- q $.
$}

$( Definition of initial axioms $)
wI  $a wff I $. $( wI is a "command" that we can use in RPN that represents a well-formed I $)
wJ  $a wff J $.
wim $a wff ( p -> q ) $.
```

We created constants (strings) `->`, `wff`, etc. that we will use in our system. Further, we defined `p` and `q` to be variables. The strings `wp` and `wq` specify that `p` and `q` are `wff` (well-formed formulas) respectively. The definition of modus ponens says that for a given {$$}p{/$$} (`mp1`) and a given {$$}p \to q{/$$} (`mp2`) we can conclude {$$}q{/$$} (`mp`). Note that outside of this block, only `mp` is visible per the rules above. Our initial axioms state that `I`, `J`, and `p -> q` are well-formed formulas.

Having defined our formal system, we can proceed with the proof:

```
$( For given I and I -> J, we prove that we can conclude J. Note: We use block scoping here since we don't want the hypothesis proof_I and proof_I_imp_J to be visible outside of the block $)
${
    $( Given I and I -> J $)
    proof_I $e |- I $.
    proof_I_imp_J $e |- ( I -> J ) $.
    $( Proof that we can conclude J $)
    proof_J $p |- J $=
        wI  $( Stack: [ 'wff I' ] $)
        wJ  $( Stack: [ 'wff I', 'wff J' ] $)
        $( Note: We've specified wff for I and J before using mp, since the types have to match $)
        proof_I       $( Stack: [ 'wff I', 'wff J', '|- I' ] $)
        proof_I_imp_J $( Stack: [ 'wff I', 'wff J', '|- I', '|- ( I -> J )' ] $)
        mp  $( Stack: [ '|- J' ] $)
    $.
$}
```

With the code above, we state that `proof_I` and `proof_I_imp_J` are given. Further, with `proof_J` we want to show that we can conclude `J`. To start the proof, we put `I` and `J` on the stack by using the commands `wI` and `wJ`. Now that our stack contains `[ 'wff I', 'wff J' ]`, we can use `proof_I` to use the first parameter from the stack to conclude `|- I`. Since `proof_I_imp_J` accepts two parameters, it will use the first two parameters from the stack, i.e. `wff I` and `wff J` to conclude `|- I -> J`. Finally, with `mp` we use `|- I` and `|- I -> J` from the stack to conclude that `|- J`.

Note how we separated `wff` from `|-`. Otherwise, if we just used `|-` then all of the formulas would be true, which does not make sense. Furthermore, the reason that we have implication `->` and entailment `|-` is that the former is a well-formed formula (that is, the expression belongs to the object language), while the latter is not a well-formed formula, rather an expression in the meta language and works upon proofs (instead of objects). This distinction allows us to represent the relation between hypothesis and conclusions in the object language.

[^apan1]: Reverse Polish Notation is a mathematical notation where functions follow their arguments. For example, to represent {$$}1 + 2{/$$}, we would write {$$}1 \ 2 \ +{/$$}.
