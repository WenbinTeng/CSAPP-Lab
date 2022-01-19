# CS:APP Bomb Lab

### Main

According to the partial source code shown in `bomb.c`, we can learn that pattern bomb check input and explode.

```c
...
input = read_line();	/* Get input                   */
phase_1(input);    		/* Run the phase               */
phase_defused();		/* Drat!  They figured it out! */
...
```



### Phase 1

```
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq   
```

We can learn that `phase_1 ` will call `strings_not_equal` to make string comparation while `%esi` is the input argument and `%eax` stores the comparation result.

```
(gdb) p (char*)0x402400
$1 = 0x402400 "Border relations with Canada have never been better."
```

So "**<u>Border relations with Canada have never been better.</u>**" is the goal string.

```
Phase 1 defused. How about the next one?
```



### Phase 2

```
0000000000400efc <phase_2>:
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq   
```

We can learn that `phase_2` will call `read_six_numbers` then execute `cmp` to check the first number is 1(references to `0x400f0a`). According to `0x400f17~0x400f1c`, it will take a number from `%rbx-4` and add itself, then compare with the number in `%rbx`. We can derive the six input numbers are "**<u>1 2 4 8 16 32</u>**".

```
That's number 2.  Keep going!
```



### Phase 3

```
0000000000400f43 <phase_3>:
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
  400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
  400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax
  400f81:	eb 3b                	jmp    400fbe <phase_3+0x7b>
  400f83:	b8 c3 02 00 00       	mov    $0x2c3,%eax
  400f88:	eb 34                	jmp    400fbe <phase_3+0x7b>
  400f8a:	b8 00 01 00 00       	mov    $0x100,%eax
  400f8f:	eb 2d                	jmp    400fbe <phase_3+0x7b>
  400f91:	b8 85 01 00 00       	mov    $0x185,%eax
  400f96:	eb 26                	jmp    400fbe <phase_3+0x7b>
  400f98:	b8 ce 00 00 00       	mov    $0xce,%eax
  400f9d:	eb 1f                	jmp    400fbe <phase_3+0x7b>
  400f9f:	b8 aa 02 00 00       	mov    $0x2aa,%eax
  400fa4:	eb 18                	jmp    400fbe <phase_3+0x7b>
  400fa6:	b8 47 01 00 00       	mov    $0x147,%eax
  400fab:	eb 11                	jmp    400fbe <phase_3+0x7b>
  400fad:	e8 88 04 00 00       	callq  40143a <explode_bomb>
  400fb2:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb7:	eb 05                	jmp    400fbe <phase_3+0x7b>
  400fb9:	b8 37 01 00 00       	mov    $0x137,%eax
  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq   
```

We can learn that `phase_3` will call `sscanf()` to get input strings, so that we can get input format as shown below. Instructions in `0x400f60` and `0x400f6a` demand number of inputs in `%eax` be greater than or equal to 1 and argument in `0x8(%rsp)` be below or equal to 7.

```
(gdb) p (char*)0x4025cf
$1 = 0x4025cf "%d %d"
```

According to `0x400f75~0x400fab`, this is a classic switch-style code. For every item in switch table, it compares a constant number with `0xc(%rsp)`, then explode if they are not equal. So we can get all valid inputs.

"**<u>0 207</u>**"
"**<u>2 707</u>**"
"**<u>3 256</u>**"
"**<u>4 389</u>**"
"**<u>5 206</u>**"
"**<u>6 682</u>**"
"**<u>7 327</u>**"
"**<u>1 311</u>**" (default)

```
Halfway there!
```



### Phase 4

```
000000000040100c <phase_4>:
  40100c:	48 83 ec 18          	sub    $0x18,%rsp
  401010:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  401015:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  40101a:	be cf 25 40 00       	mov    $0x4025cf,%esi
  40101f:	b8 00 00 00 00       	mov    $0x0,%eax
  401024:	e8 c7 fb ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  401029:	83 f8 02             	cmp    $0x2,%eax
  40102c:	75 07                	jne    401035 <phase_4+0x29>
  40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
  401033:	76 05                	jbe    40103a <phase_4+0x2e>
  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
  40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
  40103f:	be 00 00 00 00       	mov    $0x0,%esi
  401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
  401048:	e8 81 ff ff ff       	callq  400fce <func4>
  40104d:	85 c0                	test   %eax,%eax
  40104f:	75 07                	jne    401058 <phase_4+0x4c>
  401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  401056:	74 05                	je     40105d <phase_4+0x51>
  401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
  40105d:	48 83 c4 18          	add    $0x18,%rsp
  401061:	c3                   	retq   
```

Similarly, `phase_4` will call `sscanf` and demands number of inputs in `%eax` be equal to `0x2` and argument in `0x8(%rsp)` be below or equals to `0xe` and argument in `0xc(%rsp)` equals to `0x0`. The input format is shown below.

```
(gdb) p (char*)0x4025cf
$1 = 0x4025cf "%d %d"
```

Furthermore, `phase_4` will call `func4` and check the return value in `%eax`, if it returns non-zero value, explode.

```
0000000000400fce <func4>:
  400fce:	48 83 ec 08          	sub    $0x8,%rsp
  400fd2:	89 d0                	mov    %edx,%eax
  400fd4:	29 f0                	sub    %esi,%eax
  400fd6:	89 c1                	mov    %eax,%ecx
  400fd8:	c1 e9 1f             	shr    $0x1f,%ecx
  400fdb:	01 c8                	add    %ecx,%eax
  400fdd:	d1 f8                	sar    %eax
  400fdf:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx
  400fe2:	39 f9                	cmp    %edi,%ecx
  400fe4:	7e 0c                	jle    400ff2 <func4+0x24>
  400fe6:	8d 51 ff             	lea    -0x1(%rcx),%edx
  400fe9:	e8 e0 ff ff ff       	callq  400fce <func4>
  400fee:	01 c0                	add    %eax,%eax
  400ff0:	eb 15                	jmp    401007 <func4+0x39>
  400ff2:	b8 00 00 00 00       	mov    $0x0,%eax
  400ff7:	39 f9                	cmp    %edi,%ecx
  400ff9:	7d 0c                	jge    401007 <func4+0x39>
  400ffb:	8d 71 01             	lea    0x1(%rcx),%esi
  400ffe:	e8 cb ff ff ff       	callq  400fce <func4>
  401003:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  401007:	48 83 c4 08          	add    $0x8,%rsp
  40100b:	c3                   	retq   
```

Through assembly code of `func4`, we can learn that this is recursive. Based on prior knowledge, we can assume that `%eax` stores the return value, `%edi, %esi, %ecx, %edx` stores the argument `a, b, c, d`. Then, we can draw the original code based on the analysis to the behavior of assembly code.

```c
int func4(a, b, d)
{
	r = d - b;
    c = (unsigned)r >> 31;
    r = (c + r) >> 1;
    c = r + b;
    
    if (c < a)
        return func4(a, c + 1, d) * 2 + 1;
    if (c > a)
        return func4(a, b, c - 1) * 2;
    
    return 0;
}
```

According to `0x40103a~0x401044`, we can get the initial value `a=input, b=0x0, d=0xe`. While the input is one of `0, 1, 3, 7` would satisfy `func4` return zero. Finally, we can get all valid inputs.

“<u>**0 0**</u>”

“<u>**1 0**</u>”

“<u>**3 0**</u>”

"**<u>7 0</u>**"

```
So you got that one.  Try this one.
```



### Phase 5

```
0000000000401062 <phase_5>:
  401062:	53                   	push   %rbx
  401063:	48 83 ec 20          	sub    $0x20,%rsp
  401067:	48 89 fb             	mov    %rdi,%rbx
  40106a:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  401071:	00 00 
  401073:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
  401078:	31 c0                	xor    %eax,%eax
  40107a:	e8 9c 02 00 00       	callq  40131b <string_length>
  40107f:	83 f8 06             	cmp    $0x6,%eax
  401082:	74 4e                	je     4010d2 <phase_5+0x70>
  401084:	e8 b1 03 00 00       	callq  40143a <explode_bomb>
  401089:	eb 47                	jmp    4010d2 <phase_5+0x70>
  40108b:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx
  40108f:	88 0c 24             	mov    %cl,(%rsp)
  401092:	48 8b 14 24          	mov    (%rsp),%rdx
  401096:	83 e2 0f             	and    $0xf,%edx
  401099:	0f b6 92 b0 24 40 00 	movzbl 0x4024b0(%rdx),%edx
  4010a0:	88 54 04 10          	mov    %dl,0x10(%rsp,%rax,1)
  4010a4:	48 83 c0 01          	add    $0x1,%rax
  4010a8:	48 83 f8 06          	cmp    $0x6,%rax
  4010ac:	75 dd                	jne    40108b <phase_5+0x29>
  4010ae:	c6 44 24 16 00       	movb   $0x0,0x16(%rsp)
  4010b3:	be 5e 24 40 00       	mov    $0x40245e,%esi
  4010b8:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  4010bd:	e8 76 02 00 00       	callq  401338 <strings_not_equal>
  4010c2:	85 c0                	test   %eax,%eax
  4010c4:	74 13                	je     4010d9 <phase_5+0x77>
  4010c6:	e8 6f 03 00 00       	callq  40143a <explode_bomb>
  4010cb:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
  4010d0:	eb 07                	jmp    4010d9 <phase_5+0x77>
  4010d2:	b8 00 00 00 00       	mov    $0x0,%eax
  4010d7:	eb b2                	jmp    40108b <phase_5+0x29>
  4010d9:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
  4010de:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  4010e5:	00 00 
  4010e7:	74 05                	je     4010ee <phase_5+0x8c>
  4010e9:	e8 42 fa ff ff       	callq  400b30 <__stack_chk_fail@plt>
  4010ee:	48 83 c4 20          	add    $0x20,%rsp
  4010f2:	5b                   	pop    %rbx
  4010f3:	c3                   	retq   
```

According to `0x40107a~0x401082`, `phase_5` will call `string_length` to check whether the length of the input string is `0x6`. At `0x4010b3~0x4010c4`, it will call `strings_not_equal` to make comparation, which goal string can be printed.

```
(gdb) p (char*)0x40245e
$1 = 0x40245e "flyers"
```

Before the comparation is called, `phase_5` execute a loop to push characters in stack, according to `0x401099~0x4010ac`. We can learn that `%rax` is a loop iterator as well as index to the stack, and `%rbx` stores our input. This section take a character in `0x4024b0(%rdx)` and move its lower 16 bits to `%rsp+10` with offset `%rax`, note that `%rdx` is the lower 16bits of input. This process will repeat six times, confirming the argument of the previous `string_length`. Now we can see what's inside the `0x4024b0`.

```
(gdb) p (char*)0x4024b0
$1 = 0x4024b0 <array> "maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"
```

Thus, we can use indexes to build the goal string "flyers", which lower 16 bits are **"9, 15, 14, 5, 6, 7"**. Convert them to characters, e.x., "**<u>ionefg</u>**".

```
Good work!  On to the next...
```



### Phase 6

```
00000000004010f4 <phase_6>:
  4010f4:	41 56                	push   %r14
  4010f6:	41 55                	push   %r13
  4010f8:	41 54                	push   %r12
  4010fa:	55                   	push   %rbp
  4010fb:	53                   	push   %rbx
  4010fc:	48 83 ec 50          	sub    $0x50,%rsp
  401100:	49 89 e5             	mov    %rsp,%r13
  401103:	48 89 e6             	mov    %rsp,%rsi
  401106:	e8 51 03 00 00       	callq  40145c <read_six_numbers>
  40110b:	49 89 e6             	mov    %rsp,%r14
  40110e:	41 bc 00 00 00 00    	mov    $0x0,%r12d
  401114:	4c 89 ed             	mov    %r13,%rbp
  401117:	41 8b 45 00          	mov    0x0(%r13),%eax
  40111b:	83 e8 01             	sub    $0x1,%eax
  40111e:	83 f8 05             	cmp    $0x5,%eax
  401121:	76 05                	jbe    401128 <phase_6+0x34>
  401123:	e8 12 03 00 00       	callq  40143a <explode_bomb>
  401128:	41 83 c4 01          	add    $0x1,%r12d
  40112c:	41 83 fc 06          	cmp    $0x6,%r12d
  401130:	74 21                	je     401153 <phase_6+0x5f>
  401132:	44 89 e3             	mov    %r12d,%ebx
  401135:	48 63 c3             	movslq %ebx,%rax
  401138:	8b 04 84             	mov    (%rsp,%rax,4),%eax
  40113b:	39 45 00             	cmp    %eax,0x0(%rbp)
  40113e:	75 05                	jne    401145 <phase_6+0x51>
  401140:	e8 f5 02 00 00       	callq  40143a <explode_bomb>
  401145:	83 c3 01             	add    $0x1,%ebx
  401148:	83 fb 05             	cmp    $0x5,%ebx
  40114b:	7e e8                	jle    401135 <phase_6+0x41>
  40114d:	49 83 c5 04          	add    $0x4,%r13
  401151:	eb c1                	jmp    401114 <phase_6+0x20>
  401153:	48 8d 74 24 18       	lea    0x18(%rsp),%rsi
  401158:	4c 89 f0             	mov    %r14,%rax
  40115b:	b9 07 00 00 00       	mov    $0x7,%ecx
  401160:	89 ca                	mov    %ecx,%edx
  401162:	2b 10                	sub    (%rax),%edx
  401164:	89 10                	mov    %edx,(%rax)
  401166:	48 83 c0 04          	add    $0x4,%rax
  40116a:	48 39 f0             	cmp    %rsi,%rax
  40116d:	75 f1                	jne    401160 <phase_6+0x6c>
  40116f:	be 00 00 00 00       	mov    $0x0,%esi
  401174:	eb 21                	jmp    401197 <phase_6+0xa3>
  401176:	48 8b 52 08          	mov    0x8(%rdx),%rdx
  40117a:	83 c0 01             	add    $0x1,%eax
  40117d:	39 c8                	cmp    %ecx,%eax
  40117f:	75 f5                	jne    401176 <phase_6+0x82>
  401181:	eb 05                	jmp    401188 <phase_6+0x94>
  401183:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  401188:	48 89 54 74 20       	mov    %rdx,0x20(%rsp,%rsi,2)
  40118d:	48 83 c6 04          	add    $0x4,%rsi
  401191:	48 83 fe 18          	cmp    $0x18,%rsi
  401195:	74 14                	je     4011ab <phase_6+0xb7>
  401197:	8b 0c 34             	mov    (%rsp,%rsi,1),%ecx
  40119a:	83 f9 01             	cmp    $0x1,%ecx
  40119d:	7e e4                	jle    401183 <phase_6+0x8f>
  40119f:	b8 01 00 00 00       	mov    $0x1,%eax
  4011a4:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  4011a9:	eb cb                	jmp    401176 <phase_6+0x82>
  4011ab:	48 8b 5c 24 20       	mov    0x20(%rsp),%rbx
  4011b0:	48 8d 44 24 28       	lea    0x28(%rsp),%rax
  4011b5:	48 8d 74 24 50       	lea    0x50(%rsp),%rsi
  4011ba:	48 89 d9             	mov    %rbx,%rcx
  4011bd:	48 8b 10             	mov    (%rax),%rdx
  4011c0:	48 89 51 08          	mov    %rdx,0x8(%rcx)
  4011c4:	48 83 c0 08          	add    $0x8,%rax
  4011c8:	48 39 f0             	cmp    %rsi,%rax
  4011cb:	74 05                	je     4011d2 <phase_6+0xde>
  4011cd:	48 89 d1             	mov    %rdx,%rcx
  4011d0:	eb eb                	jmp    4011bd <phase_6+0xc9>
  4011d2:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx)
  4011d9:	00 
  4011da:	bd 05 00 00 00       	mov    $0x5,%ebp
  4011df:	48 8b 43 08          	mov    0x8(%rbx),%rax
  4011e3:	8b 00                	mov    (%rax),%eax
  4011e5:	39 03                	cmp    %eax,(%rbx)
  4011e7:	7d 05                	jge    4011ee <phase_6+0xfa>
  4011e9:	e8 4c 02 00 00       	callq  40143a <explode_bomb>
  4011ee:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
  4011f2:	83 ed 01             	sub    $0x1,%ebp
  4011f5:	75 e8                	jne    4011df <phase_6+0xeb>
  4011f7:	48 83 c4 50          	add    $0x50,%rsp
  4011fb:	5b                   	pop    %rbx
  4011fc:	5d                   	pop    %rbp
  4011fd:	41 5c                	pop    %r12
  4011ff:	41 5d                	pop    %r13
  401201:	41 5e                	pop    %r14
  401203:	c3                   	retq   
```

At the beginning, `phase_6` will call `read_six_numbers` to get inputs. According to `0x40111e`, the value in `%eax` must be below or equal to 6, and `%rsp` equals to `%eax` at the same time, which means the element at the top of stack must be below or equal to 6. 

Through analyzing the behavior of  section `0x401128~0x401151`, we can learn that it contains two layers of loop. In the inner loop, it will take input values in the stack, and make comparation. The `r12` will initialize to zero, after adding one and comparing to six(iteration), will be assigned to `%ebx`, then assign to `%rax` as index to take values in `(%rsp,%rax,4)`to `%eax`. If the values in `%eax` and `0x0(%rbp)` are equal, explode. After executing increasing of `%ebx` and comparing to five, `%rax` will be updated to `%ebx` and take next loop. In the outer loop, `%r13` will be updated to point out the next value in stack. In conclusion, this section limits the value of six inputs must be below or equal to six, and be various to each other as well.

In section `0x401153~0x40116d`, there is another loop to convert every element to seven subtract itself.

In section `0x40116f~0x4011a9`, there is another loop to convert element to indexes to something we can know from the assembly code. So we can print memory at `0x6032d0` to see what happens here.

```
(gdb) x /16xg 0x6032d0
0x6032d0 <node1>:	0x000000010000014c	0x00000000006032e0
0x6032e0 <node2>:	0x00000002000000a8	0x00000000006032f0
0x6032f0 <node3>:	0x000000030000039c	0x0000000000603300
0x603300 <node4>:	0x00000004000002b3	0x0000000000603310
0x603310 <node5>:	0x00000005000001dd	0x0000000000603320
0x603320 <node6>:	0x00000006000001bb	0x0000000000000000
0x603330:			0x0000000000000000	0x0000000000000000
...
```

We can learn that `0x6032d0` stores the header of a list. The first (long) word stores a value, and the second (long) word stores a pointer to next node.

The actual function of the code above is  to sort the list nodes in descending order by list node values in lower 4 bytes. According to memory information, the correct order is "**3 4 5 6 1 2**", after executing `7-x`, that is "**<u>4 3 2 1 6 5</u>**".

### Secret Phase

```
00000000004015c4 <phase_defused>:
  4015c4:	48 83 ec 78          	sub    $0x78,%rsp
  4015c8:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  4015cf:	00 00 
  4015d1:	48 89 44 24 68       	mov    %rax,0x68(%rsp)
  4015d6:	31 c0                	xor    %eax,%eax
  4015d8:	83 3d 81 21 20 00 06 	cmpl   $0x6,0x202181(%rip)        # 603760 <num_input_strings>
  4015df:	75 5e                	jne    40163f <phase_defused+0x7b>
  4015e1:	4c 8d 44 24 10       	lea    0x10(%rsp),%r8
  4015e6:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  4015eb:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  4015f0:	be 19 26 40 00       	mov    $0x402619,%esi
  4015f5:	bf 70 38 60 00       	mov    $0x603870,%edi
  4015fa:	e8 f1 f5 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  4015ff:	83 f8 03             	cmp    $0x3,%eax
  401602:	75 31                	jne    401635 <phase_defused+0x71>
  401604:	be 22 26 40 00       	mov    $0x402622,%esi
  401609:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  40160e:	e8 25 fd ff ff       	callq  401338 <strings_not_equal>
  401613:	85 c0                	test   %eax,%eax
  401615:	75 1e                	jne    401635 <phase_defused+0x71>
  401617:	bf f8 24 40 00       	mov    $0x4024f8,%edi
  40161c:	e8 ef f4 ff ff       	callq  400b10 <puts@plt>
  401621:	bf 20 25 40 00       	mov    $0x402520,%edi
  401626:	e8 e5 f4 ff ff       	callq  400b10 <puts@plt>
  40162b:	b8 00 00 00 00       	mov    $0x0,%eax
  401630:	e8 0d fc ff ff       	callq  401242 <secret_phase>
  401635:	bf 58 25 40 00       	mov    $0x402558,%edi
  40163a:	e8 d1 f4 ff ff       	callq  400b10 <puts@plt>
  40163f:	48 8b 44 24 68       	mov    0x68(%rsp),%rax
  401644:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  40164b:	00 00 
  40164d:	74 05                	je     401654 <phase_defused+0x90>
  40164f:	e8 dc f4 ff ff       	callq  400b30 <__stack_chk_fail@plt>
  401654:	48 83 c4 78          	add    $0x78,%rsp
  401658:	c3                   	retq   
  401659:	90                   	nop
  40165a:	90                   	nop
  40165b:	90                   	nop
  40165c:	90                   	nop
  40165d:	90                   	nop
  40165e:	90                   	nop
  40165f:	90                   	nop
```

Note that the `secret_phase` will be called in `phase_defused`, which will be called after executing every phase. We need to find out how to entry this phase.

At the beginning of `phase_defused`, it will call `num_input_strings` to check out whether the number of input strings equals to `0x6`, if not, `secret_phase` will not be called.

Then, `phase_defused` will call the `sscanf()` to get input string.

```
(gdb) p (char*)0x402619
$1 = 0x402619 "%d %d %s"
(gdb) p (char*)0x603870
$2 = 0x603870 <input_strings+240> "0 0"
```

We can learn that string in `0x603870` is the input of `phase_4`, and the `%s` mentioned in `0x402619` should be the entrance password. In `0x40160e`, `strings_not_equal` will be call to make string comparation, the goal string can be printed in `0x402622`.

```
(gdb) p (char*)0x402622
$1 = 0x402622 "DrEvil"
```

Then, we can entry the `secret_phase`.

```
Curses, you've found the secret phase!
But finding it and solving it are quite different...
```

The assembly code of `secret_phase` is shown below.

```
0000000000401242 <secret_phase>:
  401242:	53                   	push   %rbx
  401243:	e8 56 02 00 00       	callq  40149e <read_line>
  401248:	ba 0a 00 00 00       	mov    $0xa,%edx
  40124d:	be 00 00 00 00       	mov    $0x0,%esi
  401252:	48 89 c7             	mov    %rax,%rdi
  401255:	e8 76 f9 ff ff       	callq  400bd0 <strtol@plt>
  40125a:	48 89 c3             	mov    %rax,%rbx
  40125d:	8d 40 ff             	lea    -0x1(%rax),%eax
  401260:	3d e8 03 00 00       	cmp    $0x3e8,%eax
  401265:	76 05                	jbe    40126c <secret_phase+0x2a>
  401267:	e8 ce 01 00 00       	callq  40143a <explode_bomb>
  40126c:	89 de                	mov    %ebx,%esi
  40126e:	bf f0 30 60 00       	mov    $0x6030f0,%edi
  401273:	e8 8c ff ff ff       	callq  401204 <fun7>
  401278:	83 f8 02             	cmp    $0x2,%eax
  40127b:	74 05                	je     401282 <secret_phase+0x40>
  40127d:	e8 b8 01 00 00       	callq  40143a <explode_bomb>
  401282:	bf 38 24 40 00       	mov    $0x402438,%edi
  401287:	e8 84 f8 ff ff       	callq  400b10 <puts@plt>
  40128c:	e8 33 03 00 00       	callq  4015c4 <phase_defused>
  401291:	5b                   	pop    %rbx
  401292:	c3                   	retq   
  401293:	90                   	nop
  401294:	90                   	nop
  401295:	90                   	nop
  401296:	90                   	nop
  401297:	90                   	nop
  401298:	90                   	nop
  401299:	90                   	nop
  40129a:	90                   	nop
  40129b:	90                   	nop
  40129c:	90                   	nop
  40129d:	90                   	nop
  40129e:	90                   	nop
  40129f:	90                   	nop
```

The `secret_phase` will call the `read_line()` to get input, then call the `strtol()` to convert string to a long word. The long word will be stored in `%rax`, then will be assigned to `%rbx`. The value of `-0x1(%rax)` should be below or equal to `0x3e8`. Then it comes to `fun7`, `%rbx` and `0x6030f0` will be the input argument. After this call, it will compare the return value in `%eax` with `0x2`, if not equal, explode. 

```
0000000000401204 <fun7>:
  401204:	48 83 ec 08          	sub    $0x8,%rsp
  401208:	48 85 ff             	test   %rdi,%rdi
  40120b:	74 2b                	je     401238 <fun7+0x34>
  40120d:	8b 17                	mov    (%rdi),%edx
  40120f:	39 f2                	cmp    %esi,%edx
  401211:	7e 0d                	jle    401220 <fun7+0x1c>
  401213:	48 8b 7f 08          	mov    0x8(%rdi),%rdi
  401217:	e8 e8 ff ff ff       	callq  401204 <fun7>
  40121c:	01 c0                	add    %eax,%eax
  40121e:	eb 1d                	jmp    40123d <fun7+0x39>
  401220:	b8 00 00 00 00       	mov    $0x0,%eax
  401225:	39 f2                	cmp    %esi,%edx
  401227:	74 14                	je     40123d <fun7+0x39>
  401229:	48 8b 7f 10          	mov    0x10(%rdi),%rdi
  40122d:	e8 d2 ff ff ff       	callq  401204 <fun7>
  401232:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  401236:	eb 05                	jmp    40123d <fun7+0x39>
  401238:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  40123d:	48 83 c4 08          	add    $0x8,%rsp
  401241:	c3                   	retq   
```

We can learn that if the input value equals zero, this function will return `0xffffffff`, which is what we don't expect. Then it will make comparation between `%rdi`(equals to `0x6030f0`) and `%esi`(our input), if less than or equal to it, this function will return `0x0`,  which is what we don't expect as well. So we can see what's inside the `0x6030f0`.

```
(gdb) x /64xg 0x6030f0
0x6030f0 <n1>:		0x0000000000000024	0x0000000000603110
0x603100 <n1+16>:	0x0000000000603130	0x0000000000000000
0x603110 <n21>:		0x0000000000000008	0x0000000000603190
0x603120 <n21+16>:	0x0000000000603150	0x0000000000000000
0x603130 <n22>:		0x0000000000000032	0x0000000000603170
0x603140 <n22+16>:	0x00000000006031b0	0x0000000000000000
0x603150 <n32>:		0x0000000000000016	0x0000000000603270
0x603160 <n32+16>:	0x0000000000603230	0x0000000000000000
0x603170 <n33>:		0x000000000000002d	0x00000000006031d0
0x603180 <n33+16>:	0x0000000000603290	0x0000000000000000
0x603190 <n31>:		0x0000000000000006	0x00000000006031f0
0x6031a0 <n31+16>:	0x0000000000603250	0x0000000000000000
0x6031b0 <n34>:		0x000000000000006b	0x0000000000603210
0x6031c0 <n34+16>:	0x00000000006032b0	0x0000000000000000
0x6031d0 <n45>:		0x0000000000000028	0x0000000000000000
0x6031e0 <n45+16>:	0x0000000000000000	0x0000000000000000
0x6031f0 <n41>:		0x0000000000000001	0x0000000000000000
0x603200 <n41+16>:	0x0000000000000000	0x0000000000000000
0x603210 <n47>:		0x0000000000000063	0x0000000000000000
0x603220 <n47+16>:	0x0000000000000000	0x0000000000000000
0x603230 <n44>:		0x0000000000000023	0x0000000000000000
0x603240 <n44+16>:	0x0000000000000000	0x0000000000000000
0x603250 <n42>:		0x0000000000000007	0x0000000000000000
0x603260 <n42+16>:	0x0000000000000000	0x0000000000000000
0x603270 <n43>:		0x0000000000000014	0x0000000000000000
0x603280 <n43+16>:	0x0000000000000000	0x0000000000000000
0x603290 <n46>:		0x000000000000002f	0x0000000000000000
0x6032a0 <n46+16>:	0x0000000000000000	0x0000000000000000
0x6032b0 <n48>:		0x00000000000003e9	0x0000000000000000
0x6032c0 <n48+16>:	0x0000000000000000	0x0000000000000000
...
```

We can learn that `0x6030f0` stores the header of a binary tree. The first (long) word stores a value, and the second and third (long) word stores the pointer to left and right child node.

Note that `fun7` is a recursive function, according to `0x401213` and `0x401229`, `%rid` will be assigned to its left child or right child address, and the corresponding return values are `2 * %rax` or `2 * %rax + 1`. What we need is a return value of `0x2`, and `0x16` is the only value that satisfies this condition. So our input string is "**<u>22</u>**".

```
Wow! You've defused the secret stage!
```

