# Appendix B: Theorem provers

## Metamath

Metamath is a programming language that can express theorems accompanied by a proof checker. The interesting thing about this language is its simplicity. We start by defining a formal system (variables, symbols, axioms and rules of inference) and proceed with building new theorems based on the formal system.

As we've seen, proofs in mathematics (and Idris to some degree) are usually done at a very high level. Even though the foundations are formal systems, it is very difficult to do proofs at a low level. However, we will show that there are such programming languages like Metamath that work at the lowest level, that is formal systems.

The most basic concept in Metamath is the substitution method. Metamath uses an RPN stack[^apbn1] to build hypotheses and then rewrites using the rules of inference in order to reach a conclusion. Metamath has a very simple syntax. A token is a Metamath token if it starts with `$`, otherwise, it is a user-generated token. Here is a list of Metamath tokens:

1. `$c` defines constants
1. `$v` defines variables
1. `$f` defines the type of variables (floating hypothesis)
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

In the following example, we'll define a formal system and demonstrate the use of the rule modus ponens in order to get to a new theorem, based on our initial axioms.

```text
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

We created constants (strings) `->`, `wff`, etc. that we will use in our system. Further, we defined `p` and `q` to be variables. The strings `wp` and `wq` specify that `p` and `q` are `wff` (well-formed formulas) respectively. The definition of modus ponens says that for a given {$$}p{/$$} (`mp1`) and a given {$$}p \to q{/$$} (`mp2`) we can conclude {$$}q{/$$} (`mp`) i.e. {$$}p, p \to q \vdash q{/$$}. Note that outside of this block, only `mp` is visible per the rules above. Our initial axioms state that `I`, `J`, and `p -> q` are well-formed formulas.

Having defined our formal system, we can proceed with the proof:

```text
$( For givens I and I -> J, we prove that we can conclude J. Note: We use block scoping here since we don't want the hypothesis proof_I and proof_I_imp_J to be visible outside of the block $)
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

With the code above, we assume `proof_I` and `proof_I_imp_J` in some scope/context. Further, with `proof_J` we want to show that we can conclude `J`. To start the proof, we put `I` and `J` on the stack by using the commands `wI` and `wJ`. Now that our stack contains `[ 'wff I', 'wff J' ]`, we can use `proof_I` to use the first parameter from the stack to conclude `|- I`. Since `proof_I_imp_J` accepts two parameters, it will use the first two parameters from the stack, i.e. `wff I` and `wff J` to conclude `|- I -> J`. Finally, with `mp` we use `|- I` and `|- I -> J` from the stack to conclude that `|- J`.

Note how we separated `wff` from `|-`. Otherwise, if we just used `|-` then all of the formulas would be true, which does not make sense.

## Simple Theorem Prover

In this section we'll put formal systems into action by building a proof tree generator in Haskell. We should be able to specify axioms and inference rules, and then query the program so that it will produce all valid combinations of inference in attempt to reach the target result.

We start by defining our data structures:

```haskell
-- | A rule is a way to change a theorem.
data Rule a = Rule { name :: String , function :: a -> a }
    deriving (Show)

-- | A theorem is consisted of an initial axiom and rules
-- (ordered set) applied
data Thm a = Thm {
    axiom :: a,
	rulesThm :: [Rule a],
	result :: a
	} deriving (Show)

-- | A prover system is consisted of a bunch of axioms and
-- rules to apply between them
data ThmProver a = ThmProver {
    axioms :: [Thm a],
	rulesThmProver :: [Rule a]
	} deriving (Show)

-- | An axiom is just a theorem already proven
mkAxiom :: a -> Thm a
mkAxiom a = Thm a [] a
```

To apply a rule to a theorem, we create a new theorem whose result is all the rules applied to the target theorem:

```haskell
-- | Applies a single rule to a theorem
thmApplyRule :: Thm a -> Rule a -> Thm a
thmApplyRule theorem rule =
    Thm (axiom theorem) (rulesThm theorem ++ [rule])
    ((function rule) (result theorem))
```

We will need a function that will apply all the rules to all theorems consisted:

```haskell
-- | Applies all prover's rules to a list of theorems
thmApplyRules :: ThmProver a -> [Thm a] -> [Thm a]
thmApplyRules prover (thm:thms) =
    map (thmApplyRule thm) (rulesThmProver prover) ++
	(thmApplyRules prover thms)
thmApplyRules _ _ = []
```

In order to find a proof, we search through the theorem results and see if the target is there. If it is, we just return. Otherwise, we recursively go through the theorems and apply rules in order to attempt to find the target theorem.

```haskell
-- | Finds a proof by constructing a proof tree by
-- iteratively applying theorem rules
findProofIter :: (Ord a, Eq a) =>
    ThmProver a -> a -> Int -> [Thm a] -> Maybe (Thm a)
findProofIter _ _ 0 _ = Nothing
findProofIter prover target depth foundProofs =
    case (find (\x -> target == result x) foundProofs) of
    Just prf -> Just prf
    Nothing  ->
        let theorems = thmApplyRules prover foundProofs
            proofsSet = fromList (map result foundProofs)
            theoremsSet = fromList (map result theorems) in
        if (union proofsSet theoremsSet) == proofsSet
        -- The case where no new theorems were produced,
		-- that is, A union B = A
        then Nothing
        -- Otherwise keep producing new proofs
        else findProofIter prover target (depth - 1)
		     (mergeProofs foundProofs theorems)
```

Where `mergeProofs` is a function that given 2 lists of theorems, it will return them merged, avoiding duplicates. An example usage:

```haskell
muRules :: [Rule String]
muRules = [
    Rule   "One"   (\thm ->
	    if (isSuffixOf "I" thm) then (thm ++ "U") else thm)
    , Rule "Two"   (\thm ->
	    case (matchRegex (mkRegex "M(.*)") thm) of
            Just [x] -> thm ++ x
            _        -> thm)
    , Rule "Three" (\thm ->
	    subRegex (mkRegex "III") thm "U")
    , Rule "Four"  (\thm ->
	    subRegex (mkRegex "UU") thm "")
    ]

testProver = ThmProver (map mkAxiom ["MI"]) muRules
```

As a result of `findProofIter testProver "MIUIU" 5 (axioms testProver)`, we'll get that for a starting theorem `MI`, we apply rule "One" and rule "Two" (in that order) to get to `MIUIU` (our target proof that we've specified).

[^apbn1]: Reverse Polish Notation is a mathematical notation where functions follow their arguments. For example, to represent {$$}1 + 2{/$$}, we would write {$$}1 \ 2 \ +{/$$}.
