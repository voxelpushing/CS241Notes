#  Lecture 22

Remember register allocation, the shit where you put variables in unused registers instead of the stack to make the runtime quicker? Yeah. That shit has 1 problem where you can't get the register for some shit in the registers. So, for those variables, you need to put that shit onto the stack.

#### Strength Reduction

`add` usually runs slightly faster than `mult`, cause when you for example multiply by 2, you gotta do liek

```assembly
lis $5
.word 2
mult $3, $5
mflo $3
```

You can just do `add $3, $3, $3` though to get this done. Moreover, you can just shift your bits in order to multiply by 2, which is even faster.

#### Procedure-specific Optimization

You can **inline** simple procedures, just like the `inline` keyword in C++. LOL Will wants to sue me. If you save more code by inlining everywhere, then do it. Just decide if you want to do inlining based on the amount of code it saves you.

We can do **tail recursion**. This has a couple of benefits:

* We can reuse the stack frame, since the recursive call is the last thing to do in the current stack frame, so we don't give a fuck about the shit in the frame right now
* We can use `jr` instead of `jalr`

#### Memory-management Heap

We use the heap if we want data that outlives its scope. For example,

```cpp
C *f(){
  C *d = new C;
  return d;
}
```

We also need to free/delete all the shit that we allocate. When we do `malloc/delete`, there are a variety of implementations. In order to keep track of the free space, we have a linked list of pointers to free areas of the heap.

At the beginning, if the whole heap is free, we only have 1 entry in the list since every block is free. In each entry, we save the amount of space in the block, as well as a pointer to the next free block.

Let's say that we want to request 16 bytes, we actually allocate 20 bytes, so that we have 4 more bytes to store an `int`, the `int` stores the size of the used block and is stored at index -1. 

When we `free` shit, that is maintained in a `free` linked list. If we have a block freed, and then that piece of memeory is adjacent to another block already in the linked list, then we have to merge the blocks.

Repeated allocation and deallocation are going to leave lots of "holes" in the memory. This is because for example when we deallocate some small amount of memory, we cannot use that small free block since its often too small for other allocations. Repeat this, and we have lots of holes everywhere. This is known as **fragmentation**. There are heuristics for combatting the shit.