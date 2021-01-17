# Appendices

## Appendix A: Writing a simple type checker in Haskell

This appendix provides a short introduction to the design of type checkers. It is based on the examples of (and may serve as a good introduction to) the book Types and Programming Languages.

### Evaluator

*Syntax*: The syntax per Backus-Naur form is defined as:

```text
<term>  ::= <bool> | <num> | If <bool> Then <expr> Else <expr> | <arith>
<bool>  ::= T | F | IsZero <num>
<num>   ::= O
<arith> ::= Succ <num> | Pred <num>
```

For simplicity we represent all of them in a single `Term`:

```haskell
data Term =
    T
    | F
    | O
    | IfThenElse Term Term Term
    | Succ Term
    | Pred Term
    | IsZero Term
    deriving (Show, Eq)
```

*Rules of inference*

| Name         | Rule |
| ------------ | ---- |
| E-IfTrue     | {$$}\frac{v_2 \to v_2'}{\text{If T} \text{ Then } v_2 \text{ Else } v_3 \to v_2'}{/$$} |
| E-IfFalse    | {$$}\frac{v_3 \to v_3'}{\text{If F} \text{ Then } v_2 \text{ Else } v_3 \to v_3'}{/$$} |
| E-If         | {$$}\frac{v_1 \to v_1'}{\text{If } v_1 \text{ Then } v_2 \text{ Else } v_3 \to \text{If } v_1' \text{ Then } v_2 \text{ Else } v_3}{/$$} |
| E-Succ       | {$$}\frac{v_1 \to v_1'}{\text{Succ }v_1 \to \text{ Succ } v_1'}{/$$} |
| E-PredZero   | {$$}\frac{}{\text{Pred O} \to \text{O}}{/$$} |
| E-PredSucc   | {$$}\frac{}{\text{Pred(Succ } v \text {)} \to v}{/$$} |
| E-Pred       | {$$}\frac{v \to v'}{\text{Pred }v \to \text{ Pred } v'}{/$$} |
| E-IszeroZero | {$$}\frac{}{\text{IsZero O} \to \text{T}}{/$$} |
| E-IszeroSucc | {$$}\frac{}{\text{IsZero(Succ } v \text {)} \to \text{F}}{/$$} |
| E-IsZero     | {$$}\frac{v \to v'}{\text{IsZero }v \to \text{ IsZero } v'}{/$$} |
| E-Zero       | {$$}\frac{}{O}{/$$} |
| E-True       | {$$}\frac{}{T}{/$$} |
| E-False      | {$$}\frac{}{F}{/$$} |

Recall that {$$}\frac{x}{y}{/$$} can be thought of as the implication {$$}x \to y{/$$} at the metalanguage level, where the actual arrow {$$}\to{/$$} is the implication at the object level (in this case, simply `eval`).

Given these rules, we will reduce terms by pattern matching on them. Implementation in Haskell is mostly "copy-paste" according to the rules:

```haskell
eval :: Term -> Term
eval (IfThenElse T v2 _) = v2
eval (IfThenElse F _ v3) = v3
eval (IfThenElse v1 v2 v3) = let v1' = eval v1 in IfThenElse v1' v2 v3
eval (Succ v1) = let v1' = eval v1 in Succ v1'
eval (Pred O) = O
eval (Pred (Succ v)) = v
eval (Pred v) = let v' = eval v in Pred v'
eval (IsZero O) = T
eval (IsZero (Succ t)) = F
eval (IsZero v) = let v' = eval v in IsZero v'
eval T = T
eval F = F
eval O = O
eval _ = error "No rule applies"
```

As an example, evaluating `eval $ Pred $ Succ $ Pred O` corresponds to the following inference rules:

```text
             ----------- E-PredZero
             pred O -> O
       ----------------------- E-Succ
       succ (pred O) -> succ O
------------------------------------- E-Pred
pred (succ (pred O)) -> pred (succ O)
```

### Type checker

*Syntax*: In addition to the previous syntax, we create a new one for types:

```text
<type> ::= Bool | Nat
```

In Haskell:

```haskell
data Type = TBool | TNat
```

*Rules of inference*: Getting a type of a term expects a term, and either returns an error or the type derived:

```haskell
typeOf :: Term -> Either String Type
```

| Name     | Rule |
| -------- | ---- |
| T-True   | {$$}\frac{}{\text{T : TBool}}{/$$} |
| T-False  | {$$}\frac{}{\text{F : TBool}}{/$$} |
| T-Zero   | {$$}\frac{}{\text{O : TNat}}{/$$} |
| T-If     | {$$}\frac{t_1\text{ : Bool},  t_2\text{ : }T, t_3\text{ : }T}{\text{If }t_1 \text{ Then } t_2 \text{ Else } t_3\text{ : }T}{/$$} |
| T-Succ   | {$$}\frac{t\text{ : TNat }}{\text{Succ } t \text{ : TNat}}{/$$} |
| T-Pred   | {$$}\frac{t\text{ : TNat }}{\text{Pred } t \text{ : TNat}}{/$$} |
| T-IsZero | {$$}\frac{t\text{ : TNat }}{\text{IsZero } t \text{ : TBool}}{/$$} |

Code in Haskell:

```haskell
typeOf T = Right TBool
typeOf F = Right TBool
typeOf O = Right TNat
typeOf (IfThenElse t1 t2 t3) =
    case typeOf t1 of
        Right TBool ->
            let t2' = typeOf t2
                t3' = typeOf t3 in
                if t2' == t3'
                then t2'
                else Left "Types mismatch"
        _ -> Left "Unsupported type for IfThenElse"
typeOf (Succ k) =
    case typeOf k of
        Right TNat -> Right TNat
        _ -> Left "Unsupported type for Succ"
typeOf (Pred k) =
    case typeOf k of
        Right TNat -> Right TNat
        _ -> Left "Unsupported type for Pred"
typeOf (IsZero k) =
    case typeOf k of
        Right TNat -> Right TBool
        _ -> Left "Unsupported type for IsZero"
```

Going back to the previous example, we can now "safely" evaluate (by type checking first), depending on the type check results.

### Environments

Our simple language supports evaluation and type checking but does not allow for defining constants. To do that, we will need some kind of an environment that will hold information about constants.

```haskell
type TyEnv = [(String, Type)] -- Type env
type TeEnv = [(String, Term)] -- Term env
```

We also extend our data type to contain `TVar` for defining variables, and meanwhile also introduce the `Let ... in ...` syntax:

```haskell
data Term =
    ...
    | TVar String
    | Let String Term Term
```

Here are the rules for variables:

| Name             | Rule |
| ---------------- | ---- |
| Add binding      | {$$}\frac{\Gamma, a \text{ : }T}{\Gamma \vdash a \text{ : }T}{/$$} |
| Retrieve binding | {$$}\frac{a \text{ : }T \in \Gamma}{\Gamma \vdash a \text{ : }T}{/$$} |

Haskell definitions:

```haskell
addType :: String -> Type -> TyEnv -> TyEnv
addType varname b env = (varname, b) : env

getTypeFromEnv :: TyEnv -> String -> Maybe Type
getTypeFromEnv [] _ = Nothing
getTypeFromEnv ((varname', b) : env) varname =
    if varname' == varname then Just b else getTypeFromEnv env varname
```

We have the same exact functions for terms:

```haskell
addTerm :: String -> Term -> TeEnv -> TeEnv
getTermFromEnv :: TeEnv -> String -> Maybe Term
```

*Rules of inference (evaluator)*: `eval'` is the same as `eval`, with the following additions:

1. New parameter (the environment) to support retrieval of values for constants
2. Pattern matching for the new `Let ... in ...` syntax

```haskell
eval' :: TeEnv -> Term -> Term
eval' env (TVar v) = case getTermFromEnv env v of
    Just ty -> ty
    _       -> error "No var found in env"
eval' env (Let v t t') = eval' (addTerm v (eval' env t) env) t'
```

We will modify `IfThenElse` slightly to allow for evaluating variables:

```haskell
eval' env (IfThenElse T t2 t3) = eval' env t2
eval' env (IfThenElse F t2 t3) = eval' env t3
eval' env (IfThenElse t1 t2 t3) =
    let t' = eval' env t1 in IfThenElse t' t2 t3
```

The remaining definitions can be copy-pasted.

*Rules of inference (type checker)*: `typeOf'` is the same as `typeOf`, with the only addition to support `env` (for retrieval of types for constants in an env) and the new let syntax.

```haskell
typeOf' :: TyEnv -> Term -> Either String Type
typeOf' env (TVar v) = case getTypeFromEnv env v of
    Just ty -> Right ty
    _       -> Left "No type found in env"
typeOf' env (Let v t t') = case typeOf' env t of
    Right ty -> typeOf' (addType v ty env) t'
    _        -> Left "Unsupported type for Let"
```

For the remaining cases, the pattern matching clauses need to be updated to pass `env` where applicable.

To conclude, the evaluator and the type checker almost live in two separate worlds -- they do two separate tasks. If we want to ensure the evaluator will produce the correct results, the first thing is to assure that the type checker returns no error. Another interesting observation is how pattern matching the data type is similar to the hypothesis part of the inference rules. The relationship is due to the Curry-Howard isomorphism. When we have a formula {$$}a \vdash b{/$$} ({$$}a{/$$} implies {$$}b{/$$}), and pattern match on {$$}a{/$$}, it's as if we assumed {$$}a{/$$} and need to show {$$}b{/$$}.

## Appendix B: Theorem provers

### Metamath

Metamath is a programming language that can express theorems accompanied by a proof checker. The interesting thing about this language is its simplicity. We start by defining a formal system (variables, symbols, axioms and rules of inference) and proceed with building new theorems based on the formal system.

As we've seen, proofs in mathematics (and Idris to some degree) are usually done at a very high level. Even though the foundations are formal systems, it is very difficult to do proofs at a low level. However, we will show that there are such programming languages like Metamath that work at the lowest level, that is formal systems.

The most basic concept in Metamath is the substitution method[^apbn1]. Metamath uses an RPN stack[^apbn2] to build hypotheses and then rewrites using the rules of inference in order to reach a conclusion. Metamath has a very simple syntax. A token is a Metamath token if it starts with `$`, otherwise, it is a user-generated token. Here is a list of Metamath tokens:

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

We created constants (strings) `->`, `wff`, etc. that we will use in our system. Further, we defined `p` and `q` to be variables. The strings `wp` and `wq` specify that `p` and `q` are `wff` (well-formed formulas) respectively. The definition of modus ponens says that for a given {$$}p{/$$} (`mp1`) and a given {$$}p \to q{/$$} (`mp2`) we can conclude {$$}q{/$$} (`mp`) i.e. {$$}p, p \to q \vdash q{/$$}. Note that outside of this block, only `mp` is visible per the rules above. Our initial axioms state that `I`, `J`, and `p -> q` are well-formed formulas. We separate `wff` from `|-`, because otherwise if we just used `|-` then all of the formulas would be true, which does not make sense.

Having defined our formal system, we can proceed with the proof:

```text
${ $( Use block scoping to hide the hypothesis outside of the block $)
    $( Hypothesis: Given I and I -> J $)
    proof_I $e |- I $.
    proof_I_imp_J $e |- ( I -> J ) $.
    $( Goal: Proof that we can conclude J $)
    proof_J $p |- J $=
        wI  $( Stack: [ 'wff I' ] $)
        wJ  $( Stack: [ 'wff I', 'wff J' ] $)
        $( We specify wff for I and J before using mp, since the types have to match $)
        proof_I       $( Stack: [ 'wff I', 'wff J', '|- I' ] $)
        proof_I_imp_J $( Stack: [ 'wff I', 'wff J', '|- I', '|- ( I -> J )' ] $)
        mp  $( Stack: [ '|- J' ] $)
    $.
$}
```

With the code above, we assume `proof_I` and `proof_I_imp_J` in some scope/context. Further, with `proof_J` we want to show that we can conclude `J`. To start the proof, we put `I` and `J` on the stack by using the commands `wI` and `wJ`. Now that our stack contains `[ 'wff I', 'wff J' ]`, we can use `proof_I` to use the first parameter from the stack to conclude `|- I`. Since `proof_I_imp_J` accepts two parameters, it will use the first two parameters from the stack, i.e. `wff I` and `wff J` to conclude `|- I -> J`. Finally, with `mp` we use `|- I` and `|- I -> J` from the stack to conclude that `|- J`.

### Simple Theorem Prover

In this section we'll put formal systems into action by building a proof tree generator in Haskell. We should be able to specify axioms and inference rules, and then query the program so that it will produce all valid combinations of inference in an attempt to reach the target result.

We start by defining our data structures:

```haskell
-- | A rule is a way to change a theorem
data Rule a = Rule { name :: String, function :: a -> a }
-- | A theorem is consisted of an axiom and list of rules applied
data Thm a = Thm { axiom :: a, rulesThm :: [Rule a], result :: a }
-- | Proof system is consisted of axioms and rules between them
data ThmProver a = ThmProver { axioms :: [Thm a], rules :: [Rule a] }
-- | An axiom is just a theorem already proven
mkAxiom a = Thm a [] a
```

To apply a rule to a theorem, we create a new theorem whose result is all the rules applied to the target theorem. We will also need a function that will apply all the rules to all consisted theorems:

```haskell
thmApplyRule :: Thm a -> Rule a -> Thm a
thmApplyRule thm rule = Thm (axiom thm) (rulesThm thm ++ [rule])
    ((function rule) (result thm))

thmApplyRules :: ThmProver a -> [Thm a] -> [Thm a]
thmApplyRules prover (thm:thms) = map (thmApplyRule thm) (rules prover)
    ++ (thmApplyRules prover thms)
thmApplyRules _ _ = []
```

In order to find a proof, we search through the theorem results and see if the target is there. If it is, we just return. Otherwise, we recursively go through the theorems and apply rules in order to attempt to find the target theorem.

```haskell
-- | Construct a proof tree by iteratively applying theorem rules
findProofIter :: (Ord a, Eq a) =>
    ThmProver a -> a -> Int -> [Thm a] -> Maybe (Thm a)
findProofIter _ _ 0 _ = Nothing
findProofIter prover target depth foundProofs =
    case (find (\x -> target == result x) foundProofs) of
    Just prf -> Just prf
    Nothing  -> let theorems = thmApplyRules prover foundProofs
        proofsSet = fromList (map result foundProofs)
        theoremsSet = fromList (map result theorems) in
        -- The case where no new theorems are produced, A union B = A
        if (union proofsSet theoremsSet) == proofsSet then Nothing
        -- Otherwise keep producing new proofs
        else findProofIter prover target (depth - 1)
             (mergeProofs foundProofs theorems)
```

Where `mergeProofs` is a function that merges 2 lists of theorems, avoiding duplicates. An example of usage:

```haskell
muRules = [
  Rule "One" (\t -> if (isSuffixOf "I" t) then (t ++ "U") else t)
  , Rule "Two"   (\t ->
    case (matchRegex (mkRegex "M(.*)") t) of
      Just [x] -> t ++ x
      _        -> t)
  , Rule "Three" (\t -> subRegex (mkRegex "III") t "U")
nn, Rule "Four" (\t -> subRegex (mkRegex "UU") t "")
    ]

let testProver = ThmProver (map mkAxiom ["MI"]) muRules in
  findProofIter testProver "MIUIU" 5 (axioms testProver)
```

As a result we get that for a starting theorem `MI`, we apply rule "One" and rule "Two" (in that order) to get to `MIUIU` (our target proof that we've specified).

[^apbn1]: Using this method makes it easy to follow any proof _mechanically_. However, this is very different from understanding the _meaning_ of a proof, which in some cases may take a long time of studying.

[^apbn2]: Reverse Polish Notation is a mathematical notation where functions follow their arguments. For example, to represent {$$}1 + 2{/$$}, we would write {$$}1 \ 2 \ +{/$$}.

## Appendix C: IO, Codegen targets, compilation, and FFI

This section is most relevant to programmers that have experience with programming languages such as C, Haskell, JavaScript. It will demonstrate how Idris can interact with the outside world (IO) and these programming languages.

In the following examples, we will see how we can compile Idris code. A given program in Idris can be compiled to a binary executable or a back-end for some other programming language. If we decide to compile to a binary executable, then the C back-end will be used by default.

### IO

IO stands for Input/Output. Examples of a few IO operations are: write to a disk file, talk to a network computer, launch rockets.

Functions can be roughly categorized into two parts: **pure** and **impure**.

1. Pure functions are functions that will produce the same result every time they are called
1. Impure functions are functions that might return a different result on a function call

An example of a pure function is {$$}f(x) = x + 1{/$$}. An example of an impure function is {$$}f(x) = \text{launch} \ x \ \text{rockets}{/$$}. Since this function causes side-effects, sometimes the launch of the rockets may not be successful (e.g. the case where we have no more rockets to launch).

Computer programs are not usable if there is no interaction with the user. One problem that arises with languages such as Idris (where expressions are mathematical and have no side effects) is that IO contains side effects. For this reason, such interactions will be encapsulated in a data structure that looks something like:

```
data IO a -- IO operation that returns a value of type a
```

The concrete definition for `IO` is built within Idris itself, that is why we will leave it at the data abstraction as defined above. Essentially `IO` describes all operations that need to be executed. The resulting operations are executed externally by the Idris Run-Time System (or `IRTS`). The most basic IO program is:

```
main : IO ()
main = putStrLn "Hello world"
```

The type of `putStrLn` says that this function receives a `String` and returns an `IO` operation.

```
Idris> :t putStrLn
putStrLn : String -> IO ()
```

We can read from the input similarly:

```
getLine : IO String
```

In order to combine several IO functions, we can use the `do` notation as follows:

```
main : IO ()
main = do
    putStr "What's your name? "
    name <- getLine
    putStr "Nice to meet you, "
    putStrLn name
```

In the REPL, we can say `:x main` to execute the IO function. Alternatively, if we save the code to `test.idr`, we can use the command `idris test.idr -o test` in order to output an executable file that we can use on our system. Interacting with it:

```shell
boro@bor0:~$ idris test.idr -o test
boro@bor0:~$ ./test
What's your name? Boro
Nice to meet you, Boro
boro@bor0:~$
```

Let's slightly rewrite our code by abstracting out the concatenation function:

```
concat_string : String -> String -> String
concat_string a b = a ++ b

main : IO ()
main = do
    putStr "What's your name? "
    name <- getLine
    let concatenated = concat_string "Nice to meet you, " name
    putStrLn concatenated
```

Note how we use the syntax `let x = y` with pure functions. In contrast, we use the syntax `x <- y` with impure functions.

The `++` operator is a built-in one used to concatenate lists. A `String` can be viewed as a list of `Char`. In fact, Idris has functions called `pack` and `unpack` that allow for conversion between these two data types:

```
Idris> unpack "Hello"
['H', 'e', 'l', 'l', 'o'] : List Char
Idris> pack ['H', 'e', 'l', 'l', 'o']
"Hello" : String
```

### Codegen

The keywords `module` and `import` allow us to specify a name of the currently executing code context and load other modules by referring to their names respectively. We can implement our own back-end for a given programming language, for which we need to create a so-called Codegen (`CG`) program. An empty `CG` program would look like this:

```
module IRTS.CodegenEmpty(codegenEmpty) where

import IRTS.CodegenCommon

codegenEmpty :: CodeGenerator
codegenEmpty ci = putStrLn "Not implemented"
```

Since Idris is written in Haskell, the package `IRTS` (Idris Run-Time System) is a Haskell collection of modules. It contains data structures where we need to implement Idris commands and give definitions for how they map to the target language. For example, a `putStr` could map to `printf` in C.

### Compilation

We will show how Idris can generate a binary executable and JavaScript code, as well as the difference between total and partial functions and how Idris handles both cases. We define a dependent type `Vect`, which will allow us to work with lists and additionally have the length of the list at the type level:

```
data Vect : Nat -> Type -> Type where
    VNil  : Vect Z a
    VCons : a -> Vect k a -> Vect (S k) a
```

In the code above we define `Vect` as a data structure with two constructors: empty (`VNil`) or element construction (`VCons`). An empty vector is of type `Vect 0 a`, which can be `Vect 0 Int`, `Vect 0 Char`, etc. One example of a vector is `VCons 1 VNil : Vect 1 Integer`, where a list with a single element is represented and we note how the type contains the information about the length of the list. Another example is: `VCons 1.0 (VCons 2.0 (VCons 3.0 VNil)) : Vect 3 Double`.

We will see how total and partial functions can both pass the compile-time checks, but the latter can cause a run-time error.

```
--total
list_to_vect : List Char -> Vect 2 Char
list_to_vect (x :: y :: []) = VCons x (VCons y VNil)
--list_to_vect _ = VCons 'a' (VCons 'b' VNil)

vect_to_list : Vect 2 Char -> List Char
vect_to_list (VCons a (VCons b VNil)) = a :: b :: []

wrapunwrap : String -> String
wrapunwrap name = pack (vect_to_list (list_to_vect (unpack name)))

greet : IO ()
greet = do
    putStr "What is your name? "
    name <- getLine
    putStrLn ("Hello " ++ (wrapunwrap name))

main : IO ()
main = do
    putStrLn "Following greet, enter any number of chars"
    greet
```

We've defined functions `list_to_vect` and `vect_to_list` that convert between dependently typed vectors and lists. Further, we have another function that calls these two functions together. Note how we commented out the total keyword and the second pattern match for the purposes of this example. Now, if we check values for this partial function:

```
Idris> list_to_vect []
list_to_vect [] : Vect 2 Char
Idris> list_to_vect ['a']
list_to_vect ['a'] : Vect 2 Char
Idris> list_to_vect ['a','b']
list_to_vect 'a' (VCons 'b' VNil) : Vect 2 Char
Idris> list_to_vect ['a','b','c']
list_to_vect ['a', 'b', 'c'] : Vect 2 Char
```

It is obvious that we only get a value for the test `['a', 'b']` case. We can note that for the remaining cases the value is not calculated (computed). If we go further and investigate what happens at run-time:

```
Idris> :exec
Following greet, enter any number of chars
What is your name? Hello
Idris> :exec
Following greet, enter any number of chars
What is your name? Hi
Hello Hi
Idris>
```

Idris stopped the process execution. Going one step further, after we compile:

```shell
boro@bor0:~$ idris --codegen node test.idr -o test.js
boro@bor0:~$ node test.js
Following greet, enter any number of chars
What is your name? Hello
/Users/boro/test.js:177
    $cg$7 = new $HC_2_1$Prelude__List___58__58_($cg$2.$1, new $HC_2_1$Prelude__List___58__58_($cg$9.$1, $HC_0_0$Prelude__List__Nil));
                                                                                                    ^

TypeError: Cannot read property '$1' of undefined
...
boro@bor0:~$ node test.js
Following greet, enter any number of chars
What is your name? Hi
Hello Hi
```

We get a run-time error from JavaScript. If we do the same with the C back-end:

```shell
boro@bor0:~$ idris --codegen C test.idr -o test
boro@bor0:~$ ./test
Following greet, enter any number of chars
What is your name? Hello
Segmentation fault: 11
boro@bor0:~$ ./test
Following greet, enter any number of chars
What is your name? Hi
Hello Hi
```

It causes a segmentation fault, which is a run-time error. As a conclusion, if we use partial functions then we need to do additional checks in the code to cover the cases for potential run-time errors. Alternatively, if we want to take full advantage of the safety that the type system offers, we should define all functions as total.

By defining the function `list_to_vect` to be total, we specify that every input has to have an output. All the remaining checks are done at compile-time by Idris and with that, we're guaranteed that all callers of this function satisfy the types.

### Foreign Function Interface

In this example, we'll introduce the FFI system, which stands for Foreign Function Interface. It allows us to call functions written in other programming languages.

We can define the file `test.c` as follows:

```c
#include "test.h"

int succ(int i) {
    return i+1;
}
```

Together with `test.h`:

```c
int succ(int);
```

Now we can write a program that calls this function as follows:

```
module Main

%include C "test.h"
%link C "test.o"

succ : Int -> IO Int
succ x = foreign FFI_C "succ" (Int -> IO Int) x

main : IO ()
main = do x <- succ 1
    putStrLn ("succ 1 =" ++ show x)
```

With the code above we used the built-in function `foreign` together with the built-in constant `FFI_C` which are defined in Idris as follows:

```
Idris> :t foreign
foreign : (f : FFI) -> ffi_fn f -> (ty : Type) -> {auto fty : FTy f [] ty} -> ty
Idris> FFI_C
MkFFI C_Types String String : FFI
```

This can be useful if there's a need to use a library that's already written in another programming language. Alternatively, with IRTS we can export Idris functions to C and call them from a C code. We can define `test.idr` as follows:

```
nil : List Int
nil = []

cons : Int -> List Int -> List Int
cons x xs = x :: xs

show' : List Int -> IO String
show' xs = do { putStrLn "Ready to show..." ; pure (show xs) }

testList : FFI_Export FFI_C "testHdr.h" []
testList = Data (List Int) "ListInt" $ Fun nil "nil" $ Fun cons "cons" $ Fun show' "showList" $ End
```

Running `idris test.idr --interface -o test.o` will generate two files: `test.o` (the object file) and `testHdr.h` (the header file). Now we can input the following code in some file, e.g. `test_idris.c`:

```c
#include "testHdr.h"

int main() {
    VM* vm = idris_vm();
    ListInt x = cons(vm, 10, cons(vm, 20, nil(vm)));
    printf("%s\n", showList(vm, x));
    close_vm(vm);
}
```

We will now compile and test everything together:

```shell
boro@bor0:~$ ${CC:=cc} test_idris.c test.o `${IDRIS:-idris} $@ --include` `${IDRIS:-idris} $@ --link` -o test
boro@bor0:~$ ./test
Ready to show...
[10, 20]
```

With this approach, we can write verified code in Idris and export its functionality to another programming language.
