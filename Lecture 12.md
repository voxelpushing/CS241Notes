# Lecture 12

Grammar:

* $S\rightarrow AB$
* $A \rightarrow a$
* $A \rightarrow \varepsilon$
* $B \rightarrow b$

$A$ is nullable, $\text{First}(A) = \{a\}$, and $\text{First}(B)=\{b\}$.

$\text{Predict}(A,a) =\{A\rightarrow \beta \,|\, a\in \text{First}(\beta)\}$ is not quite correct.

* $ \text{First}(\beta) = \{a \,|\, \beta \Rightarrow^* a\gamma\}$, where $a\in\Sigma, \gamma \in V^*$
* We need to consider non-terminals that are *nullable* i.e. $A\Rightarrow^* \varepsilon$

Now, we have $\text{Predict}(A,a) =\{A\rightarrow \beta \,|\, a\in \text{First}(\beta)\} \cup \{A\rightarrow \beta\,|\, \text{Nullable}(\beta), a\in \text{Follow}(A)\}$, where $A$ is the top of the stack and non-terminal, and $a$ is the next input character. $\text{Nullable}(\beta)$ implies $\text{Nullable}(A)$.

Define $\text{Follow}(A) = \{b\,|\, S'\Rightarrow^* \alpha Ab\gamma\}$:

* Here, we're looking for terminals that come immediately after $A$ in $a$ in a derivation starting from $S'$

---

### Computing Nullable

Consider the grammar:

* $S\rightarrow AB$
* $A\rightarrow B$
* $B\rightarrow \varepsilon$

**Initialize**: $\text{Nullable}[A] = \text{false},\; \forall A\in N$, where $N$ is the set of non-terminals.

**Repeat**: loop over each rule $B\rightarrow B_1\dots B_k$, where $B_i \in V = \varepsilon \cup N$. If $k=0$ or $\text{Nullable}[B_1]=\dots=\text{Nullable}[B_k]=\text{true}$, then we have that $\text{Nullable}[B] = \text{true}$. We don't need to look at these rules if $\text{Nullable}[\text{terminal}] = \text{false}$.

This algorithm stops when nothing changes, i.e. we get no new information about what's nullable and what's not.

---

### Computing First

**Initialize**: $\text{First}[A] = \{\},\; \forall A\in N$.

**Pseudocode**:

$B_i \in V$

```swift
for each rule A -> B1 ... Bk
	for i = 1 ... k
		if Bi is a terminal a
			First[A] U= {a} // put a into the set
			break
		else // Bi is a non-terminal
			First[A] U= First[Bi]
				if not Nullable[Bi]
					break // break since we already have the first which is first of Bi
```

We keep on looping through this shit until nothing changes.

---

### Computing $\text{First}^*(\beta)$

Where $\beta\in V^*$, this is the "first set" for a string of symbols ($V^*$).

Let $Y_i \in V$, then

```swift
First*(Y1 ... Yn)
	result = null
	for i = 1 ... n
		if Yi is a non-terminal
			result U= First[Yi]
			if not Nullable[Yi]
				break
		else // Yi is a terminal
			result U= {Yi}
			break
	return result
```

---

### Computing Follow

**Initialize**: $\text{Follow}[A] = \{\},\; \forall A \not= S'$.

```swift
repeat
	for each rule A -> B1 ... Bn
		for i = 1 ... n
			if Bi is a non-terminal
				Follow[Bi] U= First*(Bi+1 ... Bn)
				if all of Bi+1 ... Bn are nullable
					Follow[Bi] U= Follow[A] 
```

