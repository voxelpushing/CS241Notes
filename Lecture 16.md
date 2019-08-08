# Lecture 16

### Context-Senstive Analysis

We needa do things like

* Type checking
* Func and var declarations
* Scope
* Other shit

#### Parse Tree

```c++
using namespace std;
// Parse Tree
class Tree{
  public:
  string rule; // e.g. "expr expr PLUS term"
  vector<string> tokens; // e.g. "expr", "expr", "PLUS", "term"
  vector<Tree> children;
};
// Tree traversal
void shit(const Tree &t){
  // do some shit with t
  for (const auto &t: t.children){
    shit(t);
  }
}
```

**To build symbol table,** we traverse the parse tree to collect variable declarations. For each node corresponding to the rule `dcl -> TYPE ID`, extract the `ID`'s name and type (`int` or `int*`). Then, check if the declaration is already in the symbol table (if it is fuck whoever wrote the program), finally, we add it to the symbol table. We checked for multiple declarations.

When we traverse a parse tree, for each node corresponding to the rule `factor -> ID` and `lvalue -> ID`, if the `ID`'s name is not in the symbol table, give the mandem and error. We checked for undeclared variables.

Both these things should be done in one pass.

---

**To implement symbol table,** the symbol table is gonna be like a `map<string, string> symbolTable`. However, this does not account for scope or procedures. You can have an `x` declared in different procedures scopes, and we need to account for that. Hence, each different prcedure has its own symbol table.

We are going to have a top-level symbol table that stores all procedure names, so a `map<string, map<string, string>> procSymbolTable`. How do we get all the procedure names though? We kind of do what we did with the vars. So when we traverse the parse tree, if we have these rules:

* `procedure -> INT ID LPAREN…`
* `main -> INT WAIN…`

Then, we take the name and check if it is in the top level table. If it is, then we have an error, otherwise we put it in. Then, we have a global var to store the "current procedure", and we update it each time we find a new procedure. So when we get our declarations, we put them in the proper table.

However, we also need to save the signature for the procedures as well, so we need extra information in the in the top-level symbol table. In `wlp4`, the signature for procedures are just the parameter list, so the table now looks like `map<string, pair<vector<string>, map<string,string>>> procSymbolTable;`.

To compute the signature, whenever we have the rule `INT ID LPAREN params`. `params` have the following rules:

* `paramlist -> dcl`
* `paramlist -> dcl COMMA paramlist`
* `params ->`

All of this information can be done in one pass. 

---

Languages need types to let us know how to interpret the bits. A good type system prevents us from reinterpreting the bits as something else. To check type correctness, we need to determine the type associated with each variable and expression. Also, ensure that all operators are applied to operands of the same correct type. Since we need to keep track of the types, more things need to be added in the symbol table.

If no rule can be applied, or if the expression type does not match the context, then we ERROR. So we want a function like this:

```cpp
string typeOf(Tree &t){
	t.children.forEach{ child ->
		compute typeOf(child);
	}
	use t.rule to decide what type rule is relevant
		- combine types of children to determine type
		- if not possible return ERROR
}
```

