# Lecture 24

In order to mitigate the issue of distinguishing between constants and labels, we force the programmer to always use labels when they want offsets. This process of modifying the label offset will be done in the assembler.

The output of most assemblers is not pure machine code, instead they are object files (`merl`) that can be shared to other machines for linking etc. The object files contain binary code and auxiliary information needed by the loader and linker.

DYK that `mips.twoints` and `mips.array` have an optional second argument? This is used to specify the address to load your program. 

We can also still write programs that only work if loaded to `0x0`:

```assembly
top: lis $5
.word top
beq $0, $5, ...
```

The `beq` check only works if the program is loaded into `0x0`. This is a logic issue not handled by the assembler. So, if you want relocatable code, then always use labels to specify jump targets.

### Linker

When we work on a big project, it is good to have code in many different files. So, the linker's job is just to put all those files together to produce a final output. If the linker just takes the `merl` files and puts them together into 1, that does not work since the assembler assembled each file assuming that they go into address `0x0`. Now, we have many parts that all think they are loaded into `0x0`, so things like labels do not work. 

In addition to that, different files might also contain the same labels (e.g. "helper" for helper functions in each file). So now we have the same labels and shit too.

The linker now has to be able to intelligently merge `merl` files. It can do so with the the following:

* look at posted shit about linkers etc. 