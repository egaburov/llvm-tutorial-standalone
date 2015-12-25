llvm-tutorial-standalone
------------------------

A tiny LLVM builder DSL. Basically the same code as the [Haskell Kaleidoscope
tutorial](http://www.stephendiehl.com/llvm/) uses but without going through a
frontend AST. Multiple people asked for this to be extracted.

If you want to roll a custom LLVM compiler backend this might be a good starting
point for the backend.

Install
-------

Check that your installed llvm version is precisely 3.4.

```bash
$ llvm-config --version
3.4
```

To build using stack:

```bash
$ stack build
$ stack exec main
```

To build using cabal:

```bash
$ cabal sandbox init
$ cabal install --only-dependencies
$ cabal build
```

To run:

```bash
$ cabal run
Preprocessing executable 'standalone' for tutorial-0.2.0.0...
; ModuleID = 'my cool jit'

define double @main() {
  entry:
    ret double 3.000000e+01
}

Evaluated to: 30.0
```

Usage
-----

Code is split across

* Codegen
* JIT.
* Main

The main program will use the embedded LLVM Monad to define a small program
which will add two constants together. 

```haskell
initModule :: AST.Module
initModule = emptyModule "my cool jit"

logic :: LLVM ()
logic = do
  define double "main" [] $ do
    let a = cons $ C.Float (F.Double 10)
    let b = cons $ C.Float (F.Double 20)
    res <- fadd a b
    ret res

main :: IO (AST.Module)
main = do
  let ast = runLLVM initModule logic
  runJIT ast
  return ast
```

This will generate and JIT compile into the following IR and use the LLVM
execution engine to JIT it to machien code.

```llvm
; ModuleID = 'my cool jit'

define double @main() {
entry:
  %1 = fadd double 1.000000e+01, 2.000000e+01
  ret double %1
}
```

License
-------

Released under the MIT License.
Copyright (c) 2014-2016, Stephen Diehl
