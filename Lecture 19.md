# Lecture 19

So far, the following is our code:

```assembly
code so far:
import print
lis $4
.word 4
lis $11
.word 1
lis $10
.word print
sub $29, $30, $4
# Allocate space for all vars on stack
# ma code
add $30, $29, $4
jr $31
```

When we do the test for `if` and `while` statements, we need to do **Boolean testing** (put 1 into `$3` if true), which have the following rules:

* $\text{test} \rightarrow \text{expr}_1 < \text{expr}_2$, will be checked as:

```assembly
code(test) = code(expr1)
						 add $5, $3, $0 # $3 <- expr1
						 code(expr2)    # $5 <- $3
						 slt $3, $5, $3 # $3 <- expr2
```

* $\text{test} \rightarrow \text{expr}_1 > \text{expr}_2$ can just be checked as $\text{expr}_2 < \text{expr}_1$ as in the first case
* $\text{test} \rightarrow \text{expr}_1 \not= \text{expr}_2$, will be chcked as:

```assembly
code(test) = code(expr1)
						 add $5, $3, $0
						 code(expr2)
						 slt $6, $3, $3 # since at most one of these comparisons is true
						 slt $7, $5, $3 # we can just add them to see if true or not (to get 1)
						 add $3, $6, $7
```

* $\text{test} \rightarrow \text{expr}_1 == \text{expr}_2$ can just be treated as a $!\text{expr}_1 \not= \text{expr}_2$. So we just add the line `sub $3, $11, $3` to the end of the code above. This flips the bits and inverts the result. (`$11` always holds the value of 1.)

---

#### If-statements

These have the form $\text{stmt} \rightarrow \text{IF test stmts}_1 \text{, ELSE stmts}_2$.

```assembly
code(stmt) = code(test)
						 beq $3, $0, else # if 3 is 0 then test is false so go to else
						 code(stmts1)
						 beq $0, $0, endif # we do not want to run else so we skip to endif
			 else: code(stmts2)
			endif:
```

Since we have labels, we need to have a counter so that if we have multiple if statements we do not have duplicate labels. So, we could have `x` as the counter, incrementing each time we have a new statement, and use `elseX`, `endifX` etc. Note that we could also have the else code come before the if code. This will run faster if some condition is gonna be false the majority of the time. Compilers actually check these things and chooses an optimal one in each case.

---

#### While-loops

These have the form $\text{stmt} \rightarrow \text{WHILE } (\text{test}) \,\{\text{stmts}\}$. Like above, we also use a counter `Y` to generate unique labels.

```assembly
code(stmt) = loopY: code(test)
									  beq $3, $0, doneY # break loop if condition fails
									  code(stmts)
									  beq $0, $0, loopY # always return to the top
						 doneY:
```

---

### Pointers

When it comes to pointers, we need to support all of the following:

* `NULL`
* dereferencing
* address of
* comparisons
* pointer arithmetic
* allocation / deallocation
* assignments through pointers

---

The rule is $\text{factor}\rightarrow \text{NULL}$. `NULL` needs to have the following behavior

```c
int *p = NULL;
if (p) {...} // is false
if (*p) {...} // crashes as we cannot dereference NULL
```

So, we can just add 1 into `$3`. So if this is dereferenced, it is not word aligned and will crash.

```
code(factor) = add $3, $11, $0
```

---

**Dereference** has the form $\text{factor}\rightarrow *\text{expr}$, where `expr` is a valid address.

```assembly
code(factor) = code(expr)
							 lw $3, 0($3) # put shit in memory address described by $3 into $3
```

---

**Comparisons** are the same form as the integer comparisons with 1 difference. Since pointers are always positive, we use `sltu` instead of `slt`. We need to also check that what is being compared are of `int*` type. So, we should have a type as a field for each node in our parse tree.

---

**Pointer arithmetic** has the form of $\text{expr}_1 \rightarrow \text{expr}_2 \pm \text{term}$. What this results as is dependent on the types of the RHS. The following are possible if we have a $+$:

| `expr2` | `term`  | Result                                                       |
| ------- | ------- | ------------------------------------------------------------ |
| `int`   | `int`   | `int` like usual                                             |
| `int *` | `int`   | We need to do (`expr2` + 4`term`), as 4 is the size of the type that `int *` points to |
| `int`   | `int *` | Similarly, we do (4`expr2` + `term`)                         |

So for the second case (basically same for thrid case):

```assembly
code(expr1) = code(expr2)
						  push $3				# put expr2 onto stack
						  code(term)
						  mult $3, $4	  # multiply term by 4
						  mflo $3
						  pop $5
						  add $3, $3, $5 # add it all up
```

Now, if we have a $-$:

| `expr2` | `term`  | Result                                      |
| ------- | ------- | ------------------------------------------- |
| `int`   | `int`   | `int` like usual                            |
| `int *` | `int`   | `expr2`$-$ 4`term`                          |
| `int *` | `int *` | (`expr2`$-$ `term`) / 4, getting the offset |

---

**Assignments through pointer dereferencing**. When this happens, the RHS is the value to the stored and the LHS is the address to store the value. We have two cases:
$$
\begin{align*}
&\text{stmt} \rightarrow \text{ID BECOMES expr}_2 \text{ SEMI}\\
&\text{stmt} \rightarrow \text{STAR expr}_1 \text{ BECOMES epxr}_2 \text{ SEMI}
\end{align*}
$$
All the shit before `BECOMES` is the lvalue, which we use as the address to store the value of `expr2`.

```assembly
code(stmt) = code(expr2)
						 push $3
						 code(expr1) # without *
						 pop $5
						 sw $5, 0($3)
```

---

**Address of** has two cases:
$$
\begin{align*}

\end{align*}
$$
