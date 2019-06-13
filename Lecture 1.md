# Lecture 1
## Sequential Programs
The starting point will be the bare hardware, at the very bottom. Binary (00110100) -> Assembly -> Higher level languages (C-like). In CS241, a simulated MIPS machine will be used.
## Binary and Hexadecimal
- A *bit* is a single 0 or 1 (b)
- A *nibble* is 4 bits
- A *byte* is 8 bits (B), there are 2⁸ = 256 different patterns for a byte
- A *word* is a group of bits or digits/characters processed as a unit. 32-bit for this course

1010 in binary can mean many different things:
	* 10 - unsigned binary
	* -2 - signed binary where first 1 is sign indicator
	* -6 - complement of 2
	* `\n`in ASCII
	* grey - if 0000 is black and 1111 is white

Thus, the meaning is different in different situations, and depends on interpretation. For example, in files, things like headers and file extensions help the computer interpret the bits. In programming, things like type declarations give information on how to interpret the bits. You can change how they are interpreted through casting lul. 

---

#### Decimal (base 10)
The digits are from 0-9. A number like `123 = 1×10² + 2×10 + 3`.
#### Binary (base 2)
Only 0s and 1s as digits. A number like `0b11001 = 1×2⁴+1×2³+1 = 25`.
When covering from decimal to binary, start off with the largest power of 2 that is smaller than the number, and progressively subtract smaller powers of 2 to create the binary.
##### Negatives with Sign Bit
For representing negative numbers, use the first bit as the sign. `1 := 負` and `0 := 正`. With this approach, the addition and subtract is difficult. In addition, there is a positive and negative 0: `0b00` and `0b10`. Using this convention, `0b11001 = -9`.
##### Negatives with Two’s Complement
	1. Interpret the n-bit number as an unsigned integer
	2. If the first bit is 0, then we are done
	3. Otherwise, subtract 2ⁿ from the number.
If n = 3, then 2ⁿ = 8. We have the bit patterns of:
`000 001 010 011    100     101 110 111`
` 0   1   2   3   4-8 = -4   -3  -2  -1`
Now, the range of numbers for a n-bit number is $-2^{n-1}$ to $-2^{n-1}-1$. We have only 1s and 0s, the left bit still gives the sign but addition is just $\mod{2^n}$.
Alternatively, positive numbers are as is, and negative numbers are derived using a bit flip for the magnitude and adding 1.
*-73 to 8-bit binary (two’s complement) would be:*

	* Magnitude: 01001001
	* Bit flip: 10110110
	* Add 1: 10110111
Going from 2’s complement to decimal has two ways of doing it (if negative, flip, add 1, convert the magnitude):
	* Reverse the process
	* Do the process again                                                   *there is a proof if you wanna do it*