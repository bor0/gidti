# Appendix C: IO, Codegen targets, compilation, and FFI

This section is most relevant to programmers that have experience with programming languages such as C, Haskell, JavaScript. It will demonstrate how Idris can interact with the outside world (IO) and these programming languages.

In the following examples, we will see how we can compile Idris code. A given program in Idris can be compiled to a binary executable or a back-end for some other programming language. If we decide to compile to a binary executable, then the C back-end will be used by default.

## IO

IO stands for Input/Output. Examples of a few IO operations are: write to a disk file, talk to a network computer, launch rockets.

Functions can be roughly categorized into two parts: **pure** and **impure**.

1. Pure functions are functions that will produce the same result every time they are called
1. Impure functions are functions that might return a different result on a function call

An example of a pure function is {$$}f(x) = x + 1{/$$}. An example of an impure function is {$$}f(x) = \text{launch} \ x \ \text{rockets}{/$$}. Since this function causes side-effects, sometimes the launch of the rockets may not be successful (e.g. the case where we have no more rockets to launch).

Computer programs are not usable if there is no interaction with the user. One problem arises with languages such as Idris (where expressions are mathematical and have no side effects) is that IO contains side effects. For this reason, such interactions will be encapsulated in a data structure that looks something like:

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

## Codegen

The keywords `module` and `import` allow us to specify a name of the currently executing code context and load other modules by referring to their names respectively. We can implement our own back-end for a given programming language, for which we need to create a so-called Codegen (`CG`) program. An empty `CG` program would look like this:

```
module IRTS.CodegenEmpty(codegenEmpty) where

import IRTS.CodegenCommon

codegenEmpty :: CodeGenerator
codegenEmpty ci = putStrLn "Not implemented"
```

Since Idris is written in Haskell, the package `IRTS` (Idris Run-Time System) is a Haskell collection of modules. It contains data structures where we need to implement Idris commands and give definitions for how they map to the target language. For example, a `putStr` could map to `printf` in C.

## Compilation

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

We've defined functions `list_to_vect` and `vect_to_list` that convert between dependently typed vectors and lists. Further, we have another function that calls these two functions together. Note how we commented the total keyword and the second pattern match for the purposes of this example. Now, if we check values for this partial function:

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

## Foreign Function Interface

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
