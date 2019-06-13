# Lecture 5

We want to automate the process of translating assembly into machine language. We take in the characters of the assembly as a stream of ASCII characters, and group them into meaningful **tokens**. Tokens include:

* Register numbers
* Hexadecimal numbers
* `.word`

A3 and A4 are going to be focused on checking if the tokens are valid. Tokenization is already done with `asm.rkt` and `asm.cc`. If any thing is an erorr, output a message cotaining the word `ERROR` to `stderr`.

The challenge with the assembler is that it does things like:

* Discards comments and whitespace
* Labels are used to compute memory addresses and branch offsets

Since labels can be defined after its being used, we need a way to property assemble it. This is where the 2-pass assembler comes into play.

On the first pass, we group the tokens into instructions, and verify that theyre valid. Then, we keep track of the memory address each instruction will be given when loaded into memory. Then, we build a **symbol table**, which is a map that maps the label to its address. On the second pass, we translate the instructions into machine code. If a label is encountered, we look up its address and compute its branch offset. The properly assembled machine instructions is then output to `stdout`.

#### Example:

Assemble this shit:

```assembly
Avengers: list $2
					.word Avengers
					
0000 0000 0000 0000 0001 0000 0001 0100
0000 0000 0000 0000 0000 0000 0000 0000 
```

When we print out the machine instruction, we obtain the pieces as a sequence of tokens, but we can't just do `printf("0000")` or something, since each `char` is 1 byte, printing the entire instruction would take 32 bytes, not 32 bits. We use an `int` since they're 32 bits.

We need the bits to be in the proper place, hence we basically interpret our pieces as an `int`, and shift it the proper amount of places to make it into proper machine code. After all the pieces are `int`s with the proper shift, we bitwise or all of them together, to make the final instruction. 

When storing things like offsets, the `int` is 32-bits, while the instructions only 16-bits. We need to mask the first 16-bits. We can do a bitwise-and between the int and `0x0000ffff`, since the `&` is going to turn all first 4 bytes into 0s. Then we can finally do `|` of this with the instruction made previously.

However, when we print this `int`, we will be printing out lots of ASCII chars, which is not what we want since we only want 4 bytes. So, we need to take each byte of the `int`, and make it into a `char`. We do this by shifting the `int` and casting it into a char.