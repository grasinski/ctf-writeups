# PicoCTF
## ARMssembly0
---
## Overview

Given a simple arm program, I was tasked to find the output of the program (in hex) with inputs `4134207980` and `950176538`.
Here is the program:
```
	.arch armv8-a
	.file	"chall.c"
	.text
	.align	2
	.global	func1
	.type	func1, %function
func1:
	sub	sp, sp, #16
	str	w0, [sp, 12]
	str	w1, [sp, 8]
	ldr	w1, [sp, 12]
	ldr	w0, [sp, 8]
	cmp	w1, w0
	bls	.L2
	ldr	w0, [sp, 12]
	b	.L3
.L2:
	ldr	w0, [sp, 8]
.L3:
	add	sp, sp, 16
	ret
	.size	func1, .-func1
	.section	.rodata
	.align	3
.LC0:
	.string	"Result: %ld\n"
	.text
	.align	2
	.global	main
	.type	main, %function
main:
	stp	x29, x30, [sp, -48]!
	add	x29, sp, 0
	str	x19, [sp, 16]
	str	w0, [x29, 44]
	str	x1, [x29, 32]
	ldr	x0, [x29, 32]
	add	x0, x0, 8
	ldr	x0, [x0]
	bl	atoi
	mov	w19, w0
	ldr	x0, [x29, 32]
	add	x0, x0, 16
	ldr	x0, [x0]
	bl	atoi
	mov	w1, w0
	mov	w0, w19
	bl	func1
	mov	w1, w0
	adrp	x0, .LC0
	add	x0, x0, :lo12:.LC0
	bl	printf
	mov	w0, 0
	ldr	x19, [sp, 16]
	ldp	x29, x30, [sp], 48
	ret
	.size	main, .-main
	.ident	"GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
	.section	.note.GNU-stack,"",@progbits
```
---
## Process

I first tried to figure out what was happening in main.

The first chunk of main is setting up the program, making sure all the important values used later are saved on the stack.
```
	stp	x29, x30, [sp, -48]!
	add	x29, sp, 0
	str	x19, [sp, 16]
	str	w0, [x29, 44]
	str	x1, [x29, 32]
```
We actually start to see what the program is doing at the first `ldr`:
```
    ldr	x0, [x29, 32]
	add	x0, x0, 8
	ldr	x0, [x0]
	bl	atoi
	mov	w19, w0
```
The above chunk loads the first integer `4134207980` into `x0` and converts it from ascii, our input, to an integer. Note that the result, a 32-bit integer is saved into w19 as a placeholder.

The next chunk similarly loads the second integer `950176538` into `x0`, converts it into an integer, and saves it into `w1`.
```
    ldr	x0, [x29, 32]
	add	x0, x0, 16
	ldr	x0, [x0]
	bl	atoi
	mov	w1, w0
	mov	w0, w19
```
Note that the value in `w0` is again the first integer `4134207980` because we had it saved in `w19`.

The next instruction is a function call.
```
bl	func1
```
Which corresponds to:
```
func1:
	sub	sp, sp, #16
	str	w0, [sp, 12]
	str	w1, [sp, 8]
	ldr	w1, [sp, 12]
	ldr	w0, [sp, 8]
	cmp	w1, w0
	bls	.L2
	ldr	w0, [sp, 12]
	b	.L3
```
In this function, the program saves the first integer to `[sp, 12]` and the second integer to `[sp, 8]`. Then, it loads the first integer, `4134207980`, into `w1` and the second integer, `950176538`, into `w0`.
It then compares `w1` to `w0` and we see there is a `bls` command which tells the program to branch if less than or equal (`ls` means lower or same).

Since `4134207980` is not less than or equal to `950176538`, we do not branch and set `w0` to be `[sp, 12]` which is the larger integer `4134207980` in this case.

After this, we return to main.
I assumed the last portion just printed the value we saved into `w0` since I only saw:
```
	mov	w1, w0
	adrp	x0, .LC0
	add	x0, x0, :lo12:.LC0
	bl	printf
```

So the flag (the integer this program printed in hex) ended up being `picoCTF{f66b01ec}`.