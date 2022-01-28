# CS:APP Architecture Lab

### Part A

```assembly
# sum.ys
# Execution begins at address 0
    .pos 0
    irmovq stack, %rsp	# Set up stack pointer
    call main			# Execute main program
    halt				# Terminate program

# Sample linked list
.align 8
ele1:
    .quad 0x00a
    .quad ele2
ele2:
    .quad 0x0b0
    .quad ele3
ele3:
    .quad 0xc00
    .quad 0

main:
	irmovq ele1, %rdi
    call sum_list
    ret

sum_list:
    pushq %r12
    irmovq $0, %rax
    jmp tloop
loop:
	mrmovq 0(%rdi), %r12
    addq %r12, %rax
    mrmovq 8(%rdi), %rdi
tloop:
	andq %rdi, %rdi
    jne loop
    popq %r12
    ret

.pos 0x100
stack:

```

```
Stopped in 28 steps at PC = 0x13.  Status 'HLT', CC Z=1 S=0 O=0
Changes to registers:
%rax:	0x0000000000000000	0x0000000000000cba
%rsp:	0x0000000000000000	0x0000000000000100

Changes to memory:
0x00f0:	0x0000000000000000	0x000000000000005b
0x00f8:	0x0000000000000000	0x0000000000000013
```

```assembly
# rsum.ys
# Execution begins at address 0
    .pos 0
    irmovq stack, %rsp	# Set up stack pointer
    call main			# Execute main program
    halt				# Terminate program

# Sample linked list
.align 8
ele1:
    .quad 0x00a
    .quad ele2
ele2:
    .quad 0x0b0
    .quad ele3
ele3:
    .quad 0xc00
    .quad 0

main:
    irmovq ele1, %rdi
    call rsum_list
    ret

rsum_list:
    pushq %r12
    irmovq $0, %rax
    andq %rdi, %rdi
    je finish
    mrmovq 0(%rdi), %r12
    mrmovq 8(%rdi), %rdi
    call rsum_list
    addq %r12, %rax
finish:
    popq %r12
    ret
  
.pos 0x100
stack:

```

```
Stopped in 42 steps at PC = 0x13.  Status 'HLT', CC Z=0 S=0 O=0
Changes to registers:
%rax:	0x0000000000000000	0x0000000000000cba
%rsp:	0x0000000000000000	0x0000000000000100

Changes to memory:
0x00b8:	0x0000000000000000	0x0000000000000c00
0x00c0:	0x0000000000000000	0x0000000000000090
0x00c8:	0x0000000000000000	0x00000000000000b0
0x00d0:	0x0000000000000000	0x0000000000000090
0x00d8:	0x0000000000000000	0x000000000000000a
0x00e0:	0x0000000000000000	0x0000000000000090
0x00f0:	0x0000000000000000	0x000000000000005b
0x00f8:	0x0000000000000000	0x0000000000000013
```

```assembly
# copy.ys
# Execution begins at address 0
    .pos 0
    irmovq stack, %rsp	# Set up stack pointer
    call main			# Execute main program
    halt				# Terminate program

.align 8
# Source block
src:
    .quad 0x00a
    .quad 0x0b0
    .quad 0xc00
# Destination block
dest:
    .quad 0x111
    .quad 0x222
    .quad 0x333

main:
	irmovq src, %rdi
	irmovq dest, %rsi
    irmovq $3, %rdx
	call copy_block
	ret
	
copy_block:
    pushq %r12
    pushq %r13
    pushq %r14
	irmovq $0, %rax
    irmovq $1, %r13
    irmovq $8, %r14
    jmp tloop
loop:
    mrmovq 0(%rdi), %r12
    addq %r14, %rdi
    rmmovq %r12, (%rsi)
    addq %r14, %rsi
    xorq %r12, %rax
    subq %r13, %rdx
tloop:
    andq %rdx, %rdx
    jne loop
    popq %r14
    popq %r13
    popq %r12
    ret

.pos 0x100
stack:

```

```
Stopped in 45 steps at PC = 0x13.  Status 'HLT', CC Z=1 S=0 O=0
Changes to registers:
%rax:	0x0000000000000000	0x0000000000000cba
%rsp:	0x0000000000000000	0x0000000000000100
%rsi:	0x0000000000000000	0x0000000000000030
%rdi:	0x0000000000000000	0x0000000000000048

Changes to memory:
0x0030:	0x0000000000000111	0x000000000000000a
0x0038:	0x0000000000000222	0x00000000000000b0
0x0040:	0x0000000000000333	0x0000000000000c00
0x00f0:	0x0000000000000000	0x000000000000006f
0x00f8:	0x0000000000000000	0x0000000000000013
```



### Part B

```
################ Fetch Stage     ###################################
...
# Is instrcution valid?
bool instr_valid = icode in { ..., IIADDQ };
# Does fetched instruction require a regid byte?
bool need_regids = icode in { ..., IIADDQ };
# Does fetched instruction require a constant word?
bool need_valC = icode in { ..., IIADDQ };
...
################ Decode Stage    ###################################
...
## What register should be used as the B source?
word srcB = [
	...
	icode in { ..., IIADDQ  } : rB;
	...
];
## What register should be used as the E destination?
word dstE = [
	...
	icode in { ..., IIADDQ } : rB;
	...
];
...
################ Execute Stage   ###################################
...
## Select input A to ALU
word aluA = [
	...
	icode in { ..., IIADDQ } : valC;
	...
];
## Select input B to ALU
word aluB = [
	...
	icode in { ..., IIADDQ } : valB;
	...
];
## Should the condition codes be updated?
bool set_cc = icode in { ..., IIADDQ };
...
```

```
ISA Check Succeeds
```



### Part C

```assembly
#/* $begin ncopy-ys */
##################################################################
# ncopy.ys - Copy a src block of len words to dst.
# Return the number of positive words (>0) contained in src.
#
# Include your name and ID here.
#
# Describe how and why you modified the baseline code.
#
##################################################################
# Do not modify this portion
# Function prologue.
# %rdi = src, %rsi = dst, %rdx = len
ncopy:

##################################################################
# You can modify this portion
	# Loop header
	xorq %rax, %rax
	iaddq $-7, %rdx
	jg loop_8
	iaddq $7, %rdx
	jg loop_1
	jmp Done

loop_1:
	mrmovq (%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, (%rsi)
	iaddq $8, %rdi
	iaddq $8, %rsi
	iaddq $-1, %rdx
	jg loop_1
	jmp Done

loop_8:
	mrmovq (%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, (%rsi)
	mrmovq 8(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, 8(%rsi)
	mrmovq 16(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, 16(%rsi)
	mrmovq 24(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, 24(%rsi)
	mrmovq 32(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, 32(%rsi)
	mrmovq 40(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, 40(%rsi)
	mrmovq 48(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, 48(%rsi)
	mrmovq 56(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $1, %r11
	andq %r10, %r10
	cmovg %r11, %rax
	rmmovq %r10, 56(%rsi)
	iaddq $64, %rdi
	iaddq $64, %rsi
	iaddq $-8, %rdx
	jg loop_8
	iaddq $7, %rdx
	jg loop_1
##################################################################
# Do not modify the following section of code
# Function epilogue.
Done:
	ret
##################################################################
# Keep the following label at the end of your function
End:
#/* $end ncopy-ys */

```

```
# correctness.pl
Simulating with instruction set simulator yis
	ncopy
0	OK
1	OK
2	OK
3	OK
4	OK
5	OK
6	OK
7	OK
8	OK
9	OK
10	OK
11	OK
12	OK
13	OK
14	OK
15	OK
16	OK
17	OK
18	OK
19	OK
20	OK
21	OK
22	OK
23	OK
24	OK
25	OK
26	OK
27	OK
28	OK
29	OK
30	OK
31	OK
32	OK
33	OK
34	OK
35	OK
36	OK
37	OK
38	OK
39	OK
40	OK
41	OK
42	OK
43	OK
44	OK
45	OK
46	OK
47	OK
48	OK
49	OK
50	OK
51	OK
52	OK
53	OK
54	OK
55	OK
56	OK
57	OK
58	OK
59	OK
60	OK
61	OK
62	OK
63	OK
64	OK
128	OK
192	OK
256	OK
68/68 pass correctness test
```

```
	ncopy
0	7
1	7	7.00
2	7	3.50
3	7	2.33
4	7	1.75
5	7	1.40
6	7	1.17
7	7	1.00
8	7	0.88
9	7	0.78
10	7	0.70
11	7	0.64
12	7	0.58
13	7	0.54
14	7	0.50
15	7	0.47
16	7	0.44
17	7	0.41
18	7	0.39
19	7	0.37
20	7	0.35
21	7	0.33
22	7	0.32
23	7	0.30
24	7	0.29
25	7	0.28
26	7	0.27
27	7	0.26
28	7	0.25
29	7	0.24
30	7	0.23
31	7	0.23
32	7	0.22
33	7	0.21
34	7	0.21
35	7	0.20
36	7	0.19
37	7	0.19
38	7	0.18
39	7	0.18
40	7	0.17
41	7	0.17
42	7	0.17
43	7	0.16
44	7	0.16
45	7	0.16
46	7	0.15
47	7	0.15
48	7	0.15
49	7	0.14
50	7	0.14
51	7	0.14
52	7	0.13
53	7	0.13
54	7	0.13
55	7	0.13
56	7	0.12
57	7	0.12
58	7	0.12
59	7	0.12
60	7	0.12
61	7	0.11
62	7	0.11
63	7	0.11
64	7	0.11
Average CPE	0.52
Score	60.0/60.0
```

