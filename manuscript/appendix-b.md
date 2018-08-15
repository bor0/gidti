# Appendix B: Codegen targets, compilation, and FFI

This section is only relevant to programmers that have experience with some of the programming languages: C, JavaScript, C#, PHP, Python. Feel free to skip it, as it will only demonstrate how Idris can interact with these programming languages.

In the following examples we will see how we can compile Idris code. A given program in Idris can be compiled to a binary executable or a back-end for some other programming language. If we decide to compile to a binary executable, then the C back-end will be used by default.

## Codegen

The keywords `module` and `import` allow us to specify a name of the current executing code context and load other modules by referring to their names respectively. We can implement our own back-end for a given programming language, for which we need to create a so-called Codegen (`CG`) program in Idris. An empty `CG` program would look like this:

```
module IRTS.CodegenEmpty(codegenEmpty) where

import IRTS.CodegenCommon

codegenEmpty :: CodeGenerator
codegenEmpty ci = putStrLn "Not implemented"
```

The package `IRTS` (which is a collection of modules) is built-in in Idris and it stands for Idris Run-Time System. It is a data structure where we need to implement Idris commands and give definitions for how they map to the target language. For example, a `putStr` may map to `printf` in C.

## Compilation

In the following examples we will show how Idris generates a binary executable and JavaScript code, as well as the difference between total and partial functions and how Idris handles both cases. We will define a dependent type `Vect`, which will allow us to work with lists and additionally have the length of the list at the type level:

```
data Vect : Nat -> Type -> Type where
   VNil  : Vect Z a
   VCons : a -> Vect k a -> Vect (S k) a
```

In the code above we define `Vect` as a data structure with two constructors: empty (`VNil`) or element construction (`VCons`). An empty vector is of type `Vect 0 a`, which can be `Vect 0 Int`, `Vect 0 Char`, etc. One example of a vector is `VCons 1 VNil : Vect 1 Integer`, where a list with a single element is represented and we note how the type contains the information about the length of the list. Another example is: `VCons 1.0 (VCons 2.0 (VCons 3.0 VNil)) : Vect 3 Double`.

In the following example we will see how total and partial functions can both pass at compile-time, but the latter can cause a run-time error.

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

We've defined functions `list_to_vect` and `vect_to_list` that convert between dependently typed vector and a list. Further, we have another function that calls these two functions together. Note how we commented the total keyword and the second pattern match for the purposes of this example. Now, if we check values for this partial function:

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

```
boro@bor0:~/idris-test$ idris --codegen node test.idr -o test.js
boro@bor0:~/idris-test$ node test.js 
Following greet, enter any number of chars
What is your name? Hello
/Users/boro/idris-test/test.js:177
    $cg$7 = new $HC_2_1$Prelude__List___58__58_($cg$2.$1, new $HC_2_1$Prelude__List___58__58_($cg$9.$1, $HC_0_0$Prelude__List__Nil));
                                                                                                    ^
 
TypeError: Cannot read property '$1' of undefined
    at Main__greet (/Users/boro/idris-test/test.js:177:101)
    at Main__main (/Users/boro/idris-test/test.js:187:12)
    at $_0_runMain (/Users/boro/idris-test/test.js:231:25)
    at Object.<anonymous> (/Users/boro/idris-test/test.js:235:1)
    at Object.<anonymous> (/Users/boro/idris-test/test.js:236:3)
    at Module._compile (module.js:660:30)
    at Object.Module._extensions..js (module.js:671:10)
    at Module.load (module.js:573:32)
    at tryModuleLoad (module.js:513:12)
    at Function.Module._load (module.js:505:3)
boro@bor0:~/idris-test$ node test.js 
Following greet, enter any number of chars
What is your name? Hi
Hello Hi
```

We get a run-time error from JavaScript. If we do the same with the C back-end:

```
boro@bor0:~/Desktop/idris-test$ idris --codegen C test.idr -o test
boro@bor0:~/Desktop/idris-test$ ./test
Following greet, enter any number of chars
What is your name? Hello
Segmentation fault: 11
boro@bor0:~/Desktop/idris-test$ ./test
Following greet, enter any number of chars
What is your name? Hi
Hello Hi
```

It causes a segmentation fault, which is a run-time error. As a conclusion, if we use partial functions then we need to do additional checks in the code to cover the cases for potential run-time errors. Alternatively, if we want to take full advantage of the safety that the type system offers, we should define all functions as total.

By defining the function `list_to_vect` to be total, we specify that every input has to have an output. All the remaining checks are done at compile-time by Idris and with that we're guaranteed that all callers of this function satisfy the types.

## Foreign Function Interface

In this example we'll introduce the FFI system, which stands for  Foreign Function Interface. It is a system that allows us to call functions written in other programming languages.

We can define the file `test.c` as follows:

```
#include "test.h"

int succ(int i) {
	return i+1;
}
```

Together with `test.h`:

```
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

This can be useful if there's a need to use a library that's already written in another programming language.
