# Lecture 18

For simplicity, we will store all local variables and parameters on the stack. During code generation, we are only translating to equivalent MIPS code.

**Example:**

```c
int wain(int a, int b) {return a;}
```

We have the following MIPS code for storing the params on the stack:

```assembly
sw $1, -4($30)
sw $2, -8($30)
lis $4
.word 4
sub $30, $30, $4
sub $30, $30, $4
lw $3, 4($30) # 4 comes from the lookup in the symbol table
add $30, $30, $4 # pop the shit off the stack 
add $30, $30, $4
jr $31
```

Symbol table:

| Name | Type  | Offset from `$30`                                            |
| ---- | ----- | ------------------------------------------------------------ |
| `a`  | `int` | 4 (since `$30` points to the <br />beginning of `b`, so only 4 to get `a`) |
| `b`  | `int` | 0                                                            |

---

**Example:**

```c
int wain(int a, int b) {int c = 0; return a;}
```

We cannot use `$30` to keep track of where the variables are stored since they change with different numbers of paramters and variables. We need to have processed all declarations. Thus, we introduce new conventions:

* `$4` always contains 4
* `$29` points to the bottom of the stack frame. Offsets calculated against `$29` will be constant
* `$3` is for output of all expressions

```assembly
lis $4
.word 4
sub $29, $30, $4 # set up bottom of stack frame
sw $1, -4($30) # push a
sub $30, $30, $4
sw $2, -4($30) # push b
sub $30, $30, $4
sw $0, -4($30) # initialize c and push to stack
sub $30, $30, $4
lw $3, 0($29) # put a to return register
add $30, $30, $4
add $30, $30, $4
add $30, $30, $4
jr $31
```

Symbol table (`$29` points to the highest memory address that is part of our stack frame, 即 `a`, everything else is in lower memory addresses, so the offsets are negative):

| Name | Type  | Offset from `$29` |
| ---- | ----- | ----------------- |
| `a`  | `int` | 0                 |
| `b`  | `int` | -4                |
| `c`  | `int` | -8                |

**Note:** we output the assembly language as a string that will be fed into the assembler.

---

**More complicated Example:**

```c
int wain(int a, int b) {return a + b;}
```

In general, for each grammar rule $A\rightarrow \alpha$, we want to build the code for $A$ ($\text{code}(A)$) from the code for $\alpha$. Recursive descent again just like checking well-typedness.

When doing these computations, we need a place to store the intermediate computations. We could use registers:

* For `a + b`, we have:

```assembly
code(a) # $3 <- a
add $5, $3, 0 # $5 <- $3
code(b) # $3 <- b
add $3, $5, $3 # $3 <- $3 + $5
```

* For `a + (b + c)`, we have:

```assembly
code(a) # $3 <- a
add $5, $3, $0 # $5 <- $3
code(b) # $3 <- b
add $6, $3, $0 # $6 <- $3
code(c) # $3 <- c
add $3, $6, $3 # do (b + c) first
add $3, $5, $3 # $3 <- a + (b + c)
```

​	Here we need 2 extra registers, and so if our expression is too complicated, we may run out of registers. So use the fucking stack instead...

Now, our shit looks like this for `a + (b + (c + d))`:

```assembly
code (a) # goes into $3
push $3 # put the shit onto stack
code (b)
push $3
code (c)
push $3
code (d) # d is in $3
pop $5 # put top of stack to $5
add $3, $5, $3 # $3 <- c + d
pop $5
add $3, $5, $3 # $3 <- b + (c + d)
pop $5
add $3, $5, $3 # $3 <- a + (b + (c + d))
```

In general $\text{expr}_1 \rightarrow \text{expr}_2 + \text{term}$, then we have that:

```c
code(expr1) = code(expr2)
						+ push($3)
						+ code(term)
						+ pop($5)
						+ "add $3, $5, $3";
```

This becomes one big giant string that we output.

---

#### Singleton Rules

These are easy like $S \rightarrow \text{BOF } \text{procedures} \text{ EOF }$, we then have that `code(S) = code(procedures)`. Similarly, $\text{expr} \rightarrow \text{term}$, we have `code(expr) = code(term)`.

The **runtime environment** is a set of procedures supplied by the compiler to assist theprograms in their execution. We need to make print a part of the runtime environment so we link that shit:

```
wlp4gen < source.wlp4i > source.asm
cs241.linkasm < source.asm > source.merl
linker source.merl print.merl > source.mips
mips.twoints source.mips
```

