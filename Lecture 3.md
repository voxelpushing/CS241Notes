# Lecture 3

###RAM

- Big array of **n** bytes (**n** is large), away from the CPU 
- Each cell has an address 0, 1, 2, ..., **n**-1 
- Each 4-byte block 4*k*, . . ., 4*k*+3 (for *k*=0,1,. . .) is a word 
- Words have addresses 0x0, 0x4, 0x8, 0xc, 0x10, 0x14, 0x18, 0x1c, 0x20, ... 
- Known as *word aligned*; i.e. divisible by 4

> Data in RAM must be loaded into registers before the CPU can use it! 

### Communicating with RAM

We have two operations, load and store. 

> Load transfers a word from a source address in RAM into a target register.
>
> `lw $t, i($s)`, where `$t`is the target register, and `$s`is the base address, with `i` as the offset, an integer in two's complement. 
>
> Binary: `1000 11ss ssst tttt iiii iiii iiii iiii`. In this course, the size between each index in the memory is 4 bytes, hence `i` must always be a multiple of 4.

`i` in this case is known as an *immediate value*, something that is directly encoded in the MIPS instructions. Register numbers are encoded in unsigned binary, but immediate values should be in two's complement.

> Save transfers a word from a register to a target location in the RAM.
>
> `sw $t, i($s)`, where `$t`is the register, and `$s`is the base address of RAM to save data to, with `i` as the offset, an integer in two's complement. 
>
> Binary: `1010 11ss ssst tttt iiii iiii iiii iiii`. In this course, the size between each index in the memory is 4 bytes, hence `i` must always be a multiple of 4.

#### Example

**Given `$1` stores the base address of an array and `$2` stores the length, load the value at index 7 into `$3`.**

`lw $3, 28($1)`, the offset is 28 since the base address starts at 0, then we have to move 7 indexes down to get to index 7. Convert to binary later lulululululul.

### **Load Immediate and Skip**

> Load immediate and skip loads the next word in memory, the block after this instruction, to a target register. It then skips the next word.
>
> `lis $d`, where `$d` is the target register.

How `lis` skips the next word, is that it increments the `PC` by 4. So the next instruction (currently the word) is incremented to the following.

#### Consider the following program:

| Address (in RAM) | Assembly |  Hexadecimal  |
| :--------------: | :------: | :-----------: |
|     `0x0000`     | `lis $1` | `0x0000 0814` |
|     `0x0004`     | `lis $2` | `0x0000 1014` |
|     `0x0008`     | `jr $0`  | `0x0000 0008` |
|     `0x000c`     | `jr $31` | `0x03e0 0008` |

What happens when we run this program?

The program loads the value of the second instruction as bits, it does not know if it is an instruction or some word. Then it skips to the third instruction, which tells the program to run the instruction in address `0x0000` (since `$0` is always 0). So the program just infinite loops.

### **Branching**

We can use two options, branch and equal and branch on not equal.

> Branch on equal: `beq $s, $t, i`
>
> Branch on not equal: `bne $s, $t, i`
>
> Where `$s` and `$t` is are the registers whose values you want to compare. `i` is the number of instructions to go forward or back. This just needs to be an integer, since the MIPS multiplies this number by 4.

#### Example

Write a program to compute the absolute value of `$1` and store result in `$1`.

```assembly
slt $2, $1, $0 ; compare $1 to 0. $2 is 1 if $1 < 0, 0 other wise
beq $2, $0, 1  ; greater than 0, jump directly to last
sub $1, $0, $1 ; less than 0 subtract from 0 to make positive
jr $31				 ; return
```

#### Example

Write a program that adds all the even numbers from 2 to 20 (inclusive) and stores the sum in register \$3.

```assembly
lis $1					; load 20 to $1
.word 20
lis $2					; load 2 to $2
.word 2
sub $3, $3, $3	; initialize $3 to 0 by subtracting itself
add $3, $3, $1	; $3 = $3 + $1 (start of loop)
sub $1, $1, $2	; $1 = $1 - $2
bne $1, $0, -3	; check if $1 is 0. i.e. all even numbers have been added, otherwise loop
jr $31					; return
```

### Multiplication and Division

When we multiply two 32 bit numbers, we need 64 bits to store them. However, the registers are only 32 bits wide. Then, we use two special registers: `hi` and `lo`. 

$\times$: `hi` stores the 32 most significant bits while `lo` stores the 32 least significant bits.

$\div$:`lo` stores the quotient and `hi` stores the remainder.

We cannot access these registers directly, so there are special instructions to move values from these registers: `mfhi` and `mflo`.

#### Example

Given `$1` stores the base address of an array and `$2` stores a valid index, write a program that loads the value into `$3`.

```assembly
lis $4					; put 4 into $4
.word 4
mult $2, $4			; $2 x 4
mflo $4					; save $2 x 4 to $4
add $4, $4, $1	; $4 = $4 + $1
lw $3, 0($4)		; $3 = MEM[$4], cannot do lw $4, $4($1), since $4 cannot be the i (bad format)
jr $31					; return
```

### Assembly Language

Assembly language replaces the binary encoding of machine language instructions with easier to use mnemonics; i.e. its more English-like code.

Nice features:

- Readability, less chance of errors, etc 
- Can make an Assembler to automatically translate ASM to ML 
- 1 line of assembly translates to 1 line of machine code 
- Has extra features to simplify coding (*directives*: e.g. .word) 
- Allows for comments and extra whitespace (stripped out at pre-processing) 

Assemblers allow programmers to **label instructions** and to use the labels within the assembly language instruction so programmers do not have to manually calculate jump addresses or branch offsets.

Example:

```assembly
; ABS program									; Label in beq instr
slt $2, $1, $0								     slt $2, $1, $0
beq $2, $0, 1										   beq $2, $0, foo
sub $1, $0, $1								     sub $1, $0, $1
jr $31												foo: jr $31
```

