# Lecture 20

For `Address-of`, we have 2 cases for $\text{expr}$:

* $\text{factor}\rightarrow \text{AMP lvalue/ID/STAR expr}$

  If $\text{expr = ID}$, then we just have something like `&a`, and the code is:

  ```assembly
  code(factor) = lis $3
  							 .word _____ # look up offset in symboltable
  							 add $3, $29, $3
  ```

  If $\text{expr = STAR expr}_2$, the operators just cancel each other out `&*a` so we just need

  ```assembly
  code(factor) = code(expr2)
  ```

For `delete`, this is a part of the runtime environment. The CS241 mans provide an allocation module `alloc.merl`, but we need to link that shit as well. Link this last.

---

For the heap-based shit, add to prologue:

* `import init`
* `import new`
* `import delete`

The function `init` sets up the initial data structure. This must be called exactly once at the beginning of the assembly file. The `init` takes its parameter from `$2`. If we calling `mips.array` (you determine this by looking at the parameters of `wain`) then `$2` should be the length of the array. If not, then we just put 0 into `$2` (after saving the second number first).

---

For `new`, `$1` is the parameter for how many words to request. This returns a pointer to a memory location into `$3`. If we cannot allocate shit and it fails, then 0 is put into `$3`.

```assembly
code(new int [expr]) = code(expr)  # compute result of expr
										   add $1, $3, $0  # move result to $1
										   call(new) # this calls the proc new, so you need to save $31 etc.
										   # now we need to check if new returned a 0, so we can replace it with 												our version of NULL
										   bne $3, $0, 1  # if not 0, skip next line
										   add $3, $11, $0 # save 1 which is our NULL into $3
```

For `delete`, `$1` is the pointer to be deallocated.

```assembly
code(delete [] expr) = code(expr)
											 beq $3, $11, skipDeleteX # we skip if we deleting NULL
											 add $1, $3, $0
											 call(delete)
					 skipDeleteX:
```

---

### Compiling Procedures

In the big picture, we got all the user defined procedures and shit before `wain`. 

```c
int f() {...}
int g(int h) {...}
int wain(int a, int b){...}
```

This is going to look like this in assembly:

```assembly
# prologue for main
# main function code for main
# epilogue for main

f: # procedure specific prologue and epilogue
jr $31

g: # same shit
jr $31
```

In the main procedure prologue/epilogue:

* We save `$1, $2` on the stack

* We import `print, int, new, delete`

* We set `$4, $11, etc`

* Set `$29`

* Call `init`

  ---

* Reset stack to bottom
* `jr $31`

In the procedure specific shit:

* No need to set constants, import shit etc.

* Set `$29`

* Save registers that procedure will modify/ overwrite

  ---

* Restore registers
* `jr $31`

#### Saving and Restoring Registers

Procedures should save and restore all registers that it modifies. How do we know which registers to save? If we are not sure, we just save and restore everything (except `$3` which is the output).

Our code generation uses `$1-$7`, `$10`, `$11`, `$29-$31`. If your code generation uses other registers, that's cool but keep track of registers used. We also need to save and restore `$29`.

There are also two approaches to saving registers, let us suppose `f` calls `g`:

* **Caller-save** `f` saves all the registers containing important data, then calls `g`
* **Callee-save** `g` saves all the data that it modifies

Our approach has been:

* Caller-save `$31`
* Callee-save everything else

However, who is responsible for saving `$29`? 

* Suppose that `g` saves `$29`, we prolly do something like

  ```assembly
  g: sub $29, $30, $4
  	 save g's registers
  ```

  So we have to point `$29` to `g`'s frame, but then also save registers. Which do we do first:

  1. If we save registers then set `$29`, we need to keep track of how many of `g`'s new things added to the stack, then calculate the offset based on `$30`
  2. If we set `$29` then save registers, we can just do `$29 = $30-4`, then save registers.

Also, how do we save `$29`?

* We can save it in `g`, so:

  ```assembly
  push $29
  add $29, $30, 0
  push other registers
  ```

  In this case, the stack looks like: `f's shit -> f's $29 -> g's shit`, and `g`'s `$29` points to the first element of its stack frame.

* We can save it in `f`, so

  ```assembly
  push $29
  push $31
  call g
  pop $31
  pop $29
  ```

---

What happens if WLP4 program has some shit like:

```assembly
int init() {...}
int print() {...}
```

Here, we have procedure names that match the names in the runtime environment. We are going to have duplicate labels when we put that shit into assembly. We have to reserve names with labels starting with "F" as user functions.

