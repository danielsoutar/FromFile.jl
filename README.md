# FromFile.jl

This package exports a macro `@from`, which can be used to import objects from files.

The hope is that you will never have to write `include` again.

## Installation
```
] add FromFile
```

## Usage

Objects in other files may be imported in the following way:

```julia
# file1.jl
using FromFile
@from "file2.jl" import foo

bar() = foo()

#file2.jl
foo() = println("hi")
```

File systems may be navigated: `@from "../folder/file.jl" import foo`

The usual import syntax is supported; the only difference is that the objects are looked up in the file requested: `@from "file.jl" using MyModule`; `@from "file.jl" import MyModule: foo`; `@from "file.jl" import foo, bar`.

Using `@from` to access a file multiple times (for example calling `@from "file.jl" import foo` in multiple files) will access the same objects each time; i.e. without the duplication issues that `include("file.jl")` would introduce.

## Specification

FromFile.jl is a draft implementation of [this specification](./SPECIFICATION.md), for improving import systems as discussed in [Issue 4600](https://github.com/JuliaLang/julia/issues/4600).

## Tips

When porting a project to use `FromFile`, it can be a bit daunting to figure out the inheritance of different objects. This is because in Julia, one doesn't need to specify which object comes from which file. To print out which file defines which object, you can execute the following bash snippet at the root of your project:
```bash
for f in $(find . -name '*.jl'); do echo $f && cat $f | vim - -nes -c '%s/#.*//ge' -c '%s/"""\_.\{-}"""//ge' -c '%v/^\S\+/d_' -c '%g/^\(end\|@from\|using\|export\|import\|include\|begin\|let\)\>/d_' -c '%g/.*/exe "norm >>"' -c ':%p' -c ':q!' | tail -n +2; done | less
```
This will print out each file, and the objects defined in that file.
