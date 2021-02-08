---
layout: single
title: "프로그래밍 #2"
categories: 프로그래밍
tags: [C언어, 이론, 어셈블리어]
toc: true
toc_sticky: true
---

***이 포스팅은  온전히 직접 공부한 것을 기반으로*** ***기록한 페이지 입니다.***

## [어셈블리 언어]

인텔 문법 어셈블리 언어에 대해 설명하겠습니다.

여기서 사용하는 툴은 인텔 문법에 맞게 설정돼 있습니다.

GDB를 사용할 때 **set disassembly intel** 이나 줄여서 **set dis intel** 이라고 입력해 **역어셈블( Disassembly )**표기를 인텔로 설정할 수 있습니다.

홈 디렉터리의 .gdbinit 파일에 앞의 명령을 입력해 GDB를 실행할 때마다 이런 환경 설정이 되게 할 수 있습니다.

```sh
reader@hacking:~/booksrc $ gdb -q
(gdb) set dis intel
(gdb) quit
reader@hacking:~/booksrc $ echo "set dis intel" > ~/.gdbinit
reader@hacking:~/booksrc $ cat ~/.gdbinit
set dis intel
reader@hacking:~/booksrc $
```

<br>

이제 GDB에서 인텔 문법을 사용할 수 있게 설정했습니다.

인텔 문법 어셈블리 명령은 보통 다음과 같은 형식입니다.

```
명령 <목적지>, <근원지>
```

<br>

**목적지(Destination)**와 **근원지(Source)**는 레지스터나 메모리 주소, 값이 될 수 있습니다.

명령은 보통 직관적 **연상 기호(Mnemonic)**입니다.

**mov (Move)**명령은 근원지에서 목적지로 값을 이동시키고, **sub (Subtraction)**는 빼고, **inc (Increase)**는 증가시킵니다.

예를 들어 다음 명령은 값을 ESP에서 EBP로 이동시킨 후 ESP에서 8을 빼 결과를 ESP에 저장하라는 명령입니다.

```
8048375:	89 e5       mov ebp,esp
8048377:	83 ec 08    sub esp,0x8
```

<br>

실행 흐름을 제어하는 데 사용되는 명령도 있습니다.

**cmp (Compare)** 명령은 값을 비교하는 데  사용되고, 기본적으로 **j (Jump)**로 시작하는 명령은 코드의 다른 부분으로 점프하는 데 사용됩니다.

다음 예제의 **첫 번째 줄은 EBP의 4바이트값에서 4를 뺀 값과 9를 비교**합니다.

**두 번째 줄은 앞의 비교 결과를 보고 작거나같으면 점프하는 명령**입니다.

**값이 9보다 작거나 같으면 실행은 0x8048393으로 점프**합니다.

아니면 실행 흐름은 무조건 다음 명령으로 넘어갑니다.

**값이 9보다 작거나 같지 않으면** 실행은 **세 번째 줄에 의해 0x80483a6으로 점프**합니다.

```
804838b:	83 7d fc 09   cmp DWORD PTR [ebp-4],0x9
804838f:	7e 02         jle 8048393 <mani+0x1f>
8048391:	eb 13         jmp 80483a6 <main+0x32>
```

<br>

이 예제는 앞의 역어셈블에서 나왔습니다. 인텔 문법을 사용하도록 디버거를 설정으니 디버거로 어셈블리 명령 단계에서 첫 번째 프로그램을 살펴보겠습니다.

GDB 컴파일러 사용 시 GDB에서 소스코드를 볼 수 있는 추가 디버그 정보를 포함시키려면 -g 플래그를 사용합니다.

```sh
reader@hacking:~/booksrc $ gcc -g firstprog.c
reader@hacking:~/booksrc $ ls -l a.out
-rwxr-xr-x 1 matrix users 11977 Jul 4 17:29 a.out
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/libthread_db.so.1".
(gdb) list
1   #include <stdio.h>
2   
3   int main()
4   {
5     int i;
6     for(i=0; i < 10; i++)
7     {
8       printf("Hello, world!\n");
9     }
10  }
(gdb) disassemble main
Dump of assembler code for function main():
0x08048384 <main+0>:	push  ebp
0x08048385 <main+1>:	mov   ebp,esp
0x08048387 <main+3>:	sub   esp,0x8
0x0804838a <main+6>:	and   esp,0xfffffff0
0x0804838d <main+9>:	mov   eax,0x0
0x08048392 <main+14>:	sub   esp,eax
0x08048394 <main+16>:	mov   DWORD PTR [ebp-4],0x0
0x0804839b <main+23>:	cmp   DWORD PTR [ebp-4],0x9
0x0804839f <main+27>:	jle   0x80483a3 <main+31>
0x080483a1 <main+29>:	jmp   0x80483b6 <main+50>
0x080483a3 <main+31>:	mov   DWORD PTR [esp],0x80484d4
0x080483aa <main+38>:	call  0x80482a8 <_init+56>
0x080483af <main+43>:	lea   eax,[ebp-4]
0x080483b2 <main+46>:	inc   DWORD PTR [eax]
0x080483b4 <main+48>:	jmp   0x804839b <main+23>
0x080483b6 <main+50>:	leave
0x080483b7 <main+51>:	ret
End of assembler dump.
(gdb) break main
Breakpoint 1 at 0x8048394: file firstprog.c, line 6.
(gdb) run
Starting program: /hacking/a.out

Breakpoint 1, main() at firstprog.c:6
6	    for(i=0; i < 10; i++)
(gdb) info register eip
eip			0x8048394		0x8048394
(gdb)
```

<br>

먼저 소스코드가 출력되고 main() 함수의 역어셈블이 출력됩니다.

그리고 main()의 시작점에 **중지점(Breakpoint)**을 설정하고 프로그램을 실행시킵니다.

- **중지점 - 디버거가 해당 지점에 가면 프로그램 실행을 중지하도록 합니다.** 

중지점이 main() 함수 시작점에 설정됐으므로 프로그램은 

main() 함수의 어떤 명령도 실행하기 전에  중지점에 다다르고 멈추게됩니다. 

그 후 **EIP(명령 포인터)**의 값이 출력됩니다.

EIP가 main() 함수의 역어셈블 명령을 가리키는 메모리 주소를 갖고 있는 것을 보면,

<main+1> ~ <main+14>의 명령들은 **함수 프롤로그(Function Prologue)**라고 불립니다.
