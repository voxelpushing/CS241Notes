# Lecture 11

CS 241 - June 11, 2019

## Top-Down Parsing

start with $S$, apply grammar rules to get $w$

$S \Rightarrow \alpha_1 \Rightarrow \alpha_2 \Rightarrow … \Rightarrow w$

Use stack to store intermediate $\alpha_i$ in reverse, match against chars in $w$.

Invariant: consumed input + reverse (stack contents) = $\alpha_i$



When top of stack is a non-terminal $A$:

- pop $A$ and push $\alpha^R$ ($\alpha$ reverse) where $A \rightarrow alpha$ is a grammar rule

When top of stack is terminal

- pop and match with input



we use "augmented" grammar

$\Rightarrow S' \rightarrow\ \vdash S \dashv$

- add new symbols $\vdash,\dashv$
- new stack symbol $S'$

### Example

Supposed $w = abywx$

1. $S' \rightarrow\ \vdash S \dashv$
2. $S \rightarrow AyB$
3. $A \rightarrow ab$
4. $A \rightarrow cd$
5. $B \rightarrow z$
6. $B \rightarrow wx$

| Stack             | Read                  | Unread                | Action                             | Note                                            |
| ----------------- | --------------------- | --------------------- | ---------------------------------- | ----------------------------------------------- |
| $S'$              | $\epsilon$            | $\vdash abywx \dashv$ | Pop $S'$, push $\dashv, S, \vdash$ | only used TOS to choose rule                    |
| $\dashv S \vdash$ | $\epsilon$            | $\vdash abywx \dashv$ | Match $\vdash$                     |                                                 |
| $\dashv S$        | $\vdash$              | $abywx \dashv$        | Pop $S$, push $B,y,A$              |                                                 |
| $\dashv B y A$    | $\vdash$              | $abywx \dashv$        | Pop $A$, push $b,a$                | choose rule based on TOS and next input symbbol |
| $\dashv B y ba$   | $\vdash$              | $abywx \dashv$        | Match $a$                          |                                                 |
| $\dashv B y b$    | $\vdash a$            | $bywx \dashv$         | Match $b$                          |                                                 |
| $\dashv B y$      | $\vdash ab$           | $ywx \dashv$          | Match $y$                          |                                                 |
| $\dashv B$        | $\vdash aby$          | $wx \dashv$           | Pop $B$, push $x, w$               |                                                 |
| $\dashv xw$       | $\vdash aby$          | $wx \dashv$           | Match $w$                          |                                                 |
| $\dashv x$        | $\vdash abyw$         | $x \dashv$            | Match $x$                          |                                                 |
| $\dashv$          | $\vdash abywx$        | $\dashv$              | Match $\dashv$                     |                                                 |
|                   | $\vdash abywx \dashv$ |                       |                                    | Accept when stack is empty and no more input    |

What if TOS is $A$ and

$A \rightarrow ab$

$A \rightarrow a$

how do we choose?

look at 2nd input char? 

If 2nd char is $c$ then use $A \rightarrow a$

If 2nd char is $b$ then?



What if TOS is $A$ and

$A \rightarrow ab$

$A \rightarrow ad$

how do we choose?

fix, look 2 chars ahead instead of one



What if TOS is $AA$ and

$A \rightarrow ab$

$A \rightarrow cd$

$A \rightarrow abcd$

which rule to choose?

how do we decide?

could use $A \rightarrow ab$ or $A \rightarrow abcd$

choice: do not allow for this algorithm



What if more than 1 rule for $A$? How do we choose?

- Brute force, try all combinations. Very inefficient
- Nondeterminism, still need to try all combinations
- We want a deterministic algorithm

Our basic algoithm: use next symbol of input (lookahead) to help decide.



### Predictor Table

given non-terminal TOS and input symbol tells which rule to use.

|      | $\vdash$ | $a$  | $b$  | $c$  | $d$  | $w$  | $x$  | $y$  | $z$  | $\dashv$ |
| ---- | -------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | -------- |
| $S'$ | 1        |      |      |      |      |      |      |      |      |          |
| $S$  |          | 2    |      | 2    |      |      |      |      |      |          |
| $A$  |          | 3    |      | 4    |      |      |      |      |      |          |
| $B$  |          |      |      |      |      | 6    |      |      | 5    |          |

- numbers are rule numbers in the above example

- empty cells are ERROR

Descriptive ERROR messages: "Parse ERROR at row, col: expecting one of _, _, ..." (row is non-terminal, col is next input char/token)

use chars for which current TOS has entries in the table, e.g.

TOS: $A$, input: $d$ $\Rightarrow \text{ ERROR expecting a, c}$



What if we had rule

7. $A \rightarrow ad$

one cell has 2 rules



What if a cell contains more than 1 rule? Algorithm breaks.

Fix? Do not allow. Deterministic - no choice.

A grammar is called $LL(1)$ if each cell of the predictor table contains at most one entry.

### $LL(1)$

- Left-to-right scan of input
- Leftmost derivation
- 1 lookahead symbol



Automatically compute predictor tabble

$Predict(A,a)$: rules that apply when $A$ is TOS and $a$ is the next input symbol

$=\{A \rightarrow \beta \mid a \in \text{First}(\beta) \}$,  $\beta \in V^*$ where $V = \Sigma \cup N$

$\text{First}(\beta) = \{a \mid \beta \Rightarrow^* a\sigma  \}$, where $\alpha \in \Sigma$, $\beta, \sigma \in V^*$



### Example

$S \rightarrow AyB$

$A \rightarrow ab$

$A \rightarrow cd$

$B \rightarrow b$



$\text{First}(A) = \{ a, c\}$

$\text{First}(S) = \{ a, c\}$

### Example

$S \rightarrow A$

$S \rightarrow B$

$A \rightarrow ab$

$A \rightarrow cd$

$B \rightarrow b$



$\text{First}(S) = \{ a,c,b \}$

### Example

$S \rightarrow AB$

$A \rightarrow ab$

$A \rightarrow cd$

$B \rightarrow b$

$A \rightarrow \epsilon$



$\text{First}(S) = \{ a,c,b \}$

$\text{First}(A) = \{ a,c \}$

$\text{First}(B) = \{ b \}$



The rule above is not quite correct: need to consider rules that go to $\epsilon$ (nullables)