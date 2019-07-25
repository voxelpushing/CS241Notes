# Lecture 23

### Implicit Memory Management

Garbage collection and shit yanno.

**Example**

```java
public int f() {
  MyClass shit = new MyClass();
  ...
}  // ob is out of scope, no more references to heap obj, reclaim it 
```

```java
public int f() {
  MyClass shit2 = null;
  if (x == y) {
    MyClass shit1 = new MyClass();
    shit2 = shit1;
  }
  // the scope of shit1 is limited to the if statement
  // However, shit2 is still storing the address of the object, so you can't reclaim it right away
}
// When the scope is done, reclaim shit2
```

#### Garbage Collection Techniques

1. **Mark & sweep**: we scan the entire stack, looking at pointers. For each pointer found, mark the heap block that it's pointing to. If that heap block contains pointers, follow those as well to mark them. Then, we scan the heap, reclaim any blocks not marked and clear all marks.
2. **Reference counting:** for each heap block, keep track of the numer of pointers that point to it. You need to watch every pointer and update the reference count (each tiem a pointer is reassigned, we ++ or - -). If the count reaches 0, we reclaim it. The problem is when we have cyclic references, where 1 object has a reference to another that has a reference to the original. Now, neither will ever have count 0 even if they may not be accessible.
3. **Copying GC**: we have our heap divided into two halves `from` and `to`. We allocate memory only in `from`. When `from` fills up, then all the reachable data is copied into `to`, and the roles are reversed. This method has **compaction**, we are guaranteed that after each copy, the reachable data will be in contiguous memory. So, we do not have fragmentation. 

### Linkers and Loaders

`mips.twoints` and `mips.array` are loaders. It basically loads (copies) the program into RAM to start executing it. However, it may load your program into a memory location $\alpha$ other than `0x0`. This causes problems:

* Labels (since they are just offsets from `0x0`) may be resolved to the wrong memory addresses, and the loader will need to fix this.
* It is also hard to tell the difference between `.word f` where `f` is a label and `.word 4` where `4` is a constant.
* We get $\alpha$ upon the first line of the program, but this may change everytime the program is run
* The `merl` files are what help us solve this, they contain instructions that augment your original assembly file and changes the memory addresses that need to be updated
* `$0` is the $\alpha$ that your program has been loaded with. The assembler will create the `merl` file that the loader will then use to increment all your memory addresses by $\alpha$