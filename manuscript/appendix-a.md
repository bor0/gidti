# Appendix A: Writing a simple type-checker in Haskell

## Evaluator

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

*Rules of inference*: The semantics we use here are based on so called small-step style, which state how a term is rewritten to a specific value, written {$$}t \to v{/$$}. In contrast, big-step style states how a specific term evaluates to a final value, written {$$}t \Downarrow v{/$$}.

| Name         | Rule |
| ------------ | ---- |
| E-IfTrue     | {$$}\text{If T Then } t_2 \text{ Else } t_3 \to t_2{/$$} |
| E-IfFalse    | {$$}\text{If F Then } t_2 \text{ Else } t_3 \to t_3{/$$} |
| E-If         | {$$}\frac{t_1 \to t'}{\text{If }t_1 \text{ Then } t_2 \text{ Else } t_3 \to \text{If }t' \text{ Then } t_2 \text{ Else } t_3}{/$$} |
| E-Succ       | {$$}\frac{t_1 \to t'}{\text{Succ }t_1 \to \text{ Succ } t'}{/$$} |
| E-PredZero   | {$$}\text{Pred O} \to \text{O}{/$$} |
| E-PredSucc   | {$$}\text{Pred(Succ } k \text {)} \to k{/$$} |
| E-Pred       | {$$}\frac{t_1 \to t'}{\text{Pred }t_1 \to \text{ Pred } t'}{/$$} |
| E-IszeroZero | {$$}\text{IsZero O} \to \text{T}{/$$} |
| E-IszeroSucc | {$$}\text{IsZero(Succ } k \text {)} \to \text{F}{/$$} |
| E-IsZero     | {$$}\frac{t_1 \to t'}{\text{IsZero }t_1 \to \text{ IsZero } t'}{/$$} |

As an example, the rule `E-IfTrue` written using big-step semantics would be {$$}\frac{t_1 \Downarrow \text{T}, t2 \Downarrow v}{\text{If T} \text{ Then } t_2 \text{ Else } t_3 \Downarrow t_2}{/$$}.

Given the rules, by pattern matching them we will reduce terms. Implementation in Haskell is mostly "copy-paste" according to the rules:

```haskell
eval :: Term -> Term
eval (IfThenElse T t2 t3) = t2
eval (IfThenElse F t2 t3) = t3
eval (IfThenElse t1 t2 t3) = let t' = eval t1 in IfThenElse t' t2 t3
eval (Succ t1) = let t' = eval t1 in Succ t'
eval (Pred O) = O
eval (Pred (Succ k)) = k
eval (Pred t1) = let t' = eval t1 in Pred t'
eval (IsZero O) = T
eval (IsZero (Succ t)) = F
eval (IsZero t1) = let t' = eval t1 in IsZero t'
eval _ = error "No rule applies"
```

As an example, evaluating the following:

```haskell
Main> eval $ Pred $ Succ $ Pred O
Pred O
```

Corresponds to the following inference rules:

```text
             ----------- E-PredZero
             pred O -> O
       ----------------------- E-Succ
       succ (pred O) -> succ O
------------------------------------- E-Pred
pred (succ (pred O)) -> pred (succ O)
```

## Type-checker

*Syntax*: In addition to the previous syntax, we create a new one for types which is defined as:

```text
<type> ::= Bool | Nat
```

In Haskell:

```haskell
data Type =
    TBool
    | TNat
```

*Rules of inference*: Getting a type of a term expects a term, and either returns an error or the type derived:

```haskell
typeOf :: Term -> Either String Type
```

| Name     | Rule |
| -------- | ---- |
| T-True   | {$$}\text{T : TBool}{/$$} |
| T-False  | {$$}\text{F : TBool}{/$$} |
| T-Zero   | {$$}\text{O : TNat}{/$$} |
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

Going back to the previous example, we can now "safely" evaluate (by type-checking first), depending on the type-check results.

## Environments

Our simple language supports evaluation and type checking, but does not allow for defining constants. To do that, we will need some kind of an environment which will hold information about constants.

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

*Rules of inference (evaluator)*: `eval'` is exactly the same as `eval`, with the following additions:

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

*Rules of inference (type-checker)*: `typeOf'` is exactly the same as `typeOf`, with the only addition to support `env` (for retrieval of types for constants in an env) and the new let syntax.

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

To conclude, the evaluator and the type checker almost live in two separate worlds -- they do two separate tasks. If we want to ensure the evaluator will produce the correct results, the first thing is to assure that the type-checker returns no error. Another interesting observation is how pattern matching the data type is similar to the hypothesis part of the inference rules. The relationship is due to the Curry-Howard isomorphism. When we have a formula {$$}a \vdash b{/$$} (a implies b), and pattern match on a, it's as if we assumed a and need to show b.
