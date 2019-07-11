# Lecture 17

**Example** of `typeOf`:

```c++
string typeOf(Tree &t){
	if (t.rule == "ID name"){
    return symbolTable.lookup(name);
  }
}
```

**Operation Type Checking in wlp4**

| **Loperand** | **op** | **Roperand** | Result  |
| :----------: | :----: | :----------: | :-----: |
|    `int`     |  `+`   |    `int`     |  `int`  |
|    `int*`    |  `+`   |    `int`     | `int*`  |
|    `int`     |  `+`   |    `int*`    | `int*`  |
|    `int*`    |  `+`   |    `int*`    | `ERROR` |
|    `int`     |  `-`   |    `int`     |  `int`  |
|    `int*`    |  `-`   |    `int`     | `int*`  |
|    `int`     |  `-`   |    `int*`    | `ERROR` |
|    `int*`    |  `-`   |    `int*`    |  `int`  |

If WLP4 there is no boolean type. When we look at comparisons like `(a < b)`, we call them `well-typed` if the types match up. From this, we can extend this notion of `well-typed` to many other things. For example, in `while` loops, we need the test condition as well as the statements to by `well-typed` for the loop itself ot be `well-typed`.

Further examples:

* **Procedures:** The body needs to be `well-typed` and must return an `int`
* **Wain:** First parameter needs to be `int` or `int*`. The second must be an `int`. The body should be `well-typed`, and the return type is an `int`

We also need to consider things like `lvalues` and `rvalues`. When we do an assignment like `x=3`, the left and right hand sides are different:

* the left hand side is a memory location
* the right hand side is a value

These things are enforced by the grammar of the WLP4 language.

---

### Code Generation

For a given WLP4 program, there are essentially an infinite number of assembly programs that are equivalent. When we generate the code, we want it to be correct, easy but also efficient. Efficiency in CS241 is just measured in how many instructions there are in the assembly code.

**Example:**

Input:

```c
int wain(int a, int b) {return a;}
```

Conventions:

* Parameters of `wain` are held in `$1` and `$2`
* The output is passed in `$3`

Output:

```assembly
add $3, $1, $0
jr $31
```

Currently, the symbol table looks like:

| Name | Type  | Location |
| :--: | :---: | :------: |
| `a`  | `int` |   `$1`   |
| `b`  | `int` |   `$2`   |

We need to store the location of all the variables. We have a choice of putting them either on the stack on in registers.