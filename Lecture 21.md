# Lecture 21

For parameters, we will save them on the stack (putting them in registers is much faster as `sw` and `lw` is expensive).

For **function calls** they have the form $\text{factor} \rightarrow \text{ID(expr}_1,\dots,\text{expr}_n)$. Say that `f` is calling `g` here.

```assembly
code(factor) = push($29)
							 push($31)
							 code(expr1)
							 push($3)
							 ... # do this for the rest of params
							 code(exprn)
							 push($3)
							 lis $5
							 .word Ffunction_name
							 jalr $5
							 pop all args
							 pop($31)
							 pop($29)
```

For **other procedures**, they have the form $\text{procedure}\rightarrow\text{INT ID (params) \{dcls stmts RETURN expr SEMI\}}$.

```assembly
code(procedure) = sub $29, $30, $4 # set up bottom of stack frame
									code(dcls)
									push les registers
									code(stmts)
									code(expr)
									pop les registers
									add $30, $29, $4
									jr $31
```

Say we have this program:

```c
int g(int a, int b, int c){
		int d=0; int e=0; int f=0;
		...
}
```

If we are careful, then what ends up happening is that `$29` will look like this:

![image-20190718103916250](/Users/kevinhua/Library/Application Support/typora-user-images/image-20190718103916250.png)

This is because `$30` was originally pointing to the top of `c`, but we do `sub $29, $30, $4` so that `$29` ends up where it is.

So, we need to fix the offsets for the parameters in the symbol table. Since we moved `$29` up 3 (number of parameters) words up, the original offsets for `a,b,c` is now wrong. So, we just add `3 x 4` to the offsets.

---

**Note:** if we have a program like this:

```c
int f() {
  	// point 1
		g();
		g();
		g();
		g();
  	// point 2
}
```

We should save f's registers at point 1, and then restore them at point 2. This is done so that `g` is not going to be saving and restoring all the time. So only 1 cycle of save and restore of `f`'s registers is needed. This is caller-save.

---

### Optimizations

We can use heuristics to optimize some statements to produce less lines of code. So if we have something like `code(1 + 2)`:

```assembly
lis $3
.word 1
sw $3, -4($30)
sub $30, $30, $4
lis $3
.word 2
lw $5, 0($30)
add $30, $30, $4
add $3, $5, $3
```

But, if we recognize that `1` and `2` are constants, then we can just do the following:

```assembly
lis $3
.word 3
```

This is called **constant folding**.

---

We also have something called **constant propagation**:

```c
int x = 2; return x + x;
```

We can recognize that `x = 2` and do

```assembly
lis $3 .word 4
```

---

**Common subexpression elimination**. For example, if we have `(a+b)*(a+b)`, we can just evaluate `(a+b)` and then just mult twice. Instead of evaluating twice.

---

**Dead code elimination** is the process of removing all the branches of code that will never be reached. LOL

---

**Register allocation** is cheaper than pushing and popping shit from the stack. We can put our most used variabble into a register for maximal savings.

---

We want $v_0$ be the equal to $\$250$ million for NPV to be 0. We also note the following relation between levered values:
$$
v_n = \frac{v_{n+1}+f_{n+1}-t_{n+1}}{1.11}+t_n
$$


We also note that $v_{10}$ is equal to $f_{10}(1+g)/(11\%-g)$, where $g$ is the perpetual growth rate we want to obtain. So we solve for $v_{10}$ first, then solve for $g$.

---

Let $r$ be the desired non-market interest rate. Then we have:
$$
\begin{align*}
54,437,216.04 &= 200,000,000-200,000,000r(1-0.3)A^{10}_{0.06}-\frac{200,000,000}{1.06^{10}}\\
200,000,000r(1-0.3)A^{10}_{0.06} &= 200,000,000-54,437,216.04-200,000,000\times1.06^{-10}\\
r&=\frac{200,000,000-54,437,216.04-200,000,000\times1.06^{-10}}{200,000,000(1-0.3)A^{10}_{0.06}}\\
\end{align*}
$$

$$
A^{10}_{0.06}:\\
r:
$$

