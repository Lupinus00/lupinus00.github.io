---
layout: single
title: "프로그래밍 #2"
categories: 프로그래밍
tags: [C언어, 이론, 어셈블리어]
toc: true
toc_sticky: true
---

***이 포스팅은 책 "해킹 공격의 예술 개정2판"을 온전히 직접 공부한 것을 기반으로*** ***기록한 페이지 입니다.***

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

**두 번째 줄은 앞의 비교 결과를 보고 작거나 같으면 점프하는 명령**입니다.

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
그러나 우선 GDB를 알아보겠습니다.

GDB 디버거는 examine을 줄인 명령 x를 사용해 메모리를 조사하는 방법을 제공합니다.

GDB 디버거는 프로그램 실행의 매 순간을 확실하게 조사하고, 멈추고, 한 단계 들어가고, 필요한 만큼 반복합니다.

대부분 실행 중인 프로그램은 단지 프로세서와 메모리 세그먼트들이므로 

메모리 조사는 무엇이 어떻게 진행되는지 보는 첫 번째 방법입니다.

GDB 의 조사 명령은 다양한 방법으로 메모리 주소를 보는 데 사용됩니다.

이 명령에는 조사할 메모리의 위치와 메모리를 어떻게 보여줄지에 대한 두 개의 인자가 필요합니다.

표현 형식도 한 글자의 축양형을 사용합니다. 또 옵션 중 문자 앞에 숫자로 얼마나 많은 아이템을 조사할 것인지 쓸 수 있습니다.

보통의 형식은 다음과 같습니다.

- o - 8진법으로 보여준다.
- x - 16진법으로 보여준다.
- u - 부호가 없는(Unsigned) 표준 10진법으로 보여준다.
- t - 2진법으로 보여준다.

이 표현 형식은 메모리 주소를 조사할 때 examine 명령과 함께 사용됩니다.

<br>

다음 예제를 보면 EIP 레지스터의 현재 메모리가 예시로 사용됩니다.

GDB에선 축약형 명령이 자주 사용되기때문에 **info register eip**를 **i r eip**로 줄여서 쓰겠습니다.

```
(gdb) i r eip
eip		0x8048384		0x8048384 <main+16>
(gdb) x/o 0x8048384
0x8048384 <main+16>:	077042707
(gdb) x/x $eip
0x8048384 <main+16>:	0x00fc45c7
(gdb) x/u $eip
0x8048384 <main+16>:	16532935
(gdb) x/t $eip
0x8048384 <main+16>:	0000000000111111000100010111000111
(gdb) 
```

EIP에 저장된 주소를 이용해 EIP 레지스터가 가리키는 메모리를 조사합니다.

디버거로 해당 레지스터를 직접 살펴볼 수 있고, $eip는 그 순간에 EIP가 갖고 있는 값을 의미합니다.

**8진수 077042707**은 **16진수 0x00fc45c7**과 같고, **10진수 16532935**와 같고, **2진수 00000000111111000100010111000111**과 같은 값입니다.

examine 명령 중 숫자는 목표 주소에서 여러 개를 조사하기 위한 형식으로 사용됩니다.

```
(gdb) x/2x $eip
0x8048384 <main+16>: 0x00fc45c7	0x83000000
(gdb) x/12x $eip
0x8048384 <main+16>:	0x00fc45c7	0x83000000	0x7e09fc7d	0xc713eb02
0x8048394 <main+32>:	0x84842404	0x01e80804	0x8dffffff	0x00fffc45
0x80483a4 <main+48>:	0xc3c9e5eb	0x90909090	0x90909090	0x5de58955
(gdb) 
```

메모리 단일 유닛의 기본 크기는 워드(word)라 불리는 4바이트 크기입니다.

examine 명령에서 표시 유닛 크기를 형식 문자 끝에 한 문자를 추가해 바꿀 수 있고, 유요한 크기 문자는 다음과 같습니다.

- b - 단일 바이트
- h - 2바이트의 하프워드
- w - 4바이트의 워드
- g - 8바이트의 자이언트(2워드)

가끔 워드가 2바이트여서 헷갈릴 때가 있는데, 이런 경우 **더블워드나 DWORD가 4바이트 값**입니다.

또 **다른경우 word나 DWORD 모두 4바이트인 경우**도 있습니다.

**보통 2바이트 값을 얘기할 때는 쇼트(Short)나 하프워드를 사용**합니다.

다음은 GDB 결과에서 다양한 크기의 메모리를 보겠습니다.

```
(gdb) x/8xb $eip
0x8048384 <main+16>:	0xc7	0x45	0xfc	0x00	0x00	0x00	0x00	0x83
(gdb) x/8xh $eip
0x8048384 <main+16>:	0x45c7	0x00fc	0x0000	0x8300	0xfc7d	0x7e09	0xeb02	0xc713
(gdb) x/8xw $eip
0x8048384 <main+16>:	0x00fc45c7	0x83000000	0x7e09fc7d	0xc713eb02
0x8048394 <main+32>:	0x84842404	0x01e80804	0x8dffffff	0x00fffc45
(gdb) 
```

위 결과를 자세히 보면 뭔가 이상한 부분을 발견할 수 있습니다.

첫 번째 examine 명령은 당연히 여덟 바이트를 보여주고, 더 큰 유닛을 사용하는 examine 명령은 전체로서 데이터를 더 보여줍니다.

하지만 첫 번째 examine은 0xc7과 0x45 두 바이트를 보여주는데, 하프워드로 같은 메모리 주소를 조사하니 바이트가 거꾸로 된 **0x45c7**을 보여줍니다.

첫 번째 명령의 네 바이트를 순서대로 써보면 0xc7, 0x45, 0xfc, 0x00인데, 4바이트 워드로도 **0x00fc45c7** 같이 **역바이트 현상**을 볼 수 있습니다.

이런 현상은 x86 프로세서가 값을 **리틀 엔디언 바이트 순서로 저장**하기 때문입니다.

즉, **맨 하위 바이트가 처음에 저장된다는 의미**입니다.

```sh
(gdb) x/4xb $eip
0x8048384 <main+16>:	0xc7	0x45	0xfc	0x00
(gdb) x/4ub $eip
0x8048384 <main+16>:	199		69		252		0
(gdb) x/1xw $eip
0x8048384 <main+16>:	0x00fc45c7
(gdb) x/1uw $eip
0x8048384 <main+16>:	16532935
(gdb) quit
The program is running. Exit anyway? (y or n) y
reader@hacking:~/booksrc $ bc -ql
199*(256^3) + 69*(256^2) + 252*(256^1) + 0*(256^0)
3343252480
0*(256^3) + 252*(256^2) + 69*(256^1) + 199*(256^0)
quit
reader@hacking:~/booksrc $ 
```

bc는 커맨드라인 계산기 프로그램인데, 올바르지 않은 순서로 바이트를 조합했을 때 말도 안되게 틀린 3343252480이라는 결과를 보여줍니다.

이 결과를 보면 아키텍처의 바이트 순서는 꼭 알아야 할 중요한 요소임을 알 수 있습니다.

대부분의 디버깅 툴과 컴파일러가 바이트 순서를 자동으로 관리해주지만 결국 자기 스스로 메모리를 직접 관리할 수 있어야 합니다.

바이터 순서 변환에 대해 덧붙이면 GDB는 examine 명령을 사용해 또 다른 변환을 할 수 있습니다.

앞에선 GDB가 기계어 명령을 사람이 읽을 수 있는 어셈블리 명령으로 역어셈블할 수 있음을 살펴 봤고, 

이번엔 examine 명령에서 instruction을 줄인 **i** 문자를 사용해 역어셈블된 어셈블리 언어의 명령 메모리를 볼 수 있었습니다.

```sh
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) break main
Breakpoint 1 at 0x8048384: file firstprog.c, line 6.
(gdb) run
Starting program: /home/reader/booksrc/a.out

Breakpoint 1, main () at firstprog.c:6
6	for(i=0; i < 10; i++)
(gdb) i r $eip
eip		0x8048384		0x8048384 <main+16>
(gdb) x/i $eip
0x8048384 <main+16>:	mov		DWORD PTR [ebp-4],0x0
(gdb) x/3i $eip
0x8048384 <main+16>:	mov		DWORD PTR [ebp-4],0x0
0x804838b <main+23>:	cmp		DWORD PTR [ebp-4],0x9
0x804838f <main+27>:	jle		0x8048393 <main+31>
(gdb) x/7xb $eip
0x8048384 <main+16>:	0xc7	0x45	0xfc	0x00	0x00	0x00	0x00
(gdb) x/i $eip
0x8048384 <main+16>:	mov		DWORD PTR [ebp-4],0x0
(gdb)
```

위 결과를 보면 a.out 프로그램은 GDB에서 main()에 중지점이 설정돼 실행됐고, EIP 레지스터는 기계어 명령이 있는 메모리를 가리키고 있으므로 훌륭하게 역어셈블할 수 있습니다.

앞에서 살펴본 objdump 역어셈블로 EIP가 실제 가리키는 일곱 바이트는 해당 어셈블리 명령을 위한 기계어임을 확인할 수 있습니다.

```
8048384:	c7 45 fc 00 00 00 00	mov		DWORD PTR [ebp-4],0x0
```

어셈블리 언어에 대한 내용이 너무 긴거 같지만, 사실상 포스팅 하나로 끝낼 수 있는 내용은 절대적으로 아니라고 봅니다. 

**정리**

- 어셈블리어는 기계가 이해하는 기계어를 좀 더 쉽게(사람이 이해하기 쉽게)만들어 주는 언어입니다.
- GDB 디버거를 이용해 소스코드에 분기를 지정한 후 해당 지점의 주소와 그 주소가 가진 값을 어셈블리 코드로 볼 수 있었다.
- 어셈블리어는 사람이 보기에도 어렵다.



## [문자열]

> Update: 2021-02-10



- char_array.c

```c
#include <stdio.h>
int main(void)
{
    char str_a[20] = "Hello, world!\n\0";
    printf(str_a);
}
```

<br>

gcc 컴파일러에서 -o 옵션을 사용해 컴파일한 결과 파일을 정할 수 있습니다.

다음과 같이 프로그램을 **char_array.out 이라는 파일명**의 **실행 가능한 바이너리**로 컴파일하는 데 사용됩니다.

```sh
reader@hacking:~/booksrc $ gcc -o char_array.out char_array.c
reader@hacking:~/booksrc $ ./char_array.out
Hello, world!
reader@hacking:~/booksrc $
```

앞에 c파일 소스코드에서 **크기가 20바이트 이고 str_a라는 이름의 문자형 배열이 선언**되었고,

**초기값으로 "Hello, world!\n\0"이 저장**되었는데 **배열의 시작은 0부터 이고, 맨마지막에 \0이 저장되었음**을 꼭 기억하세요.

마지막에 \0은 널 바이트라고 부릅니다. 크기가 20인 배열인데 사실상 13바이트만 실제로 사용이 되었습니다.

끝의 널 바이트는 어떤 함수가 문자열을 처리할 때 여기서 중단하라고 알리기 위한 구획 문자로 사용되기 때문입니다.

남은 여분의 바이트들은 쓰레기 값이 들어있고, 널 바이트 문자 배열의 다섯 번째 요소에 들어가면 printf() 함수는 Hello만 출력하게 됩니다.

문자 배열에 각 문자를 하나씩 입력하는 것은 매우 고생스러운 일이고 문자열은 꽤 자주 사용되므로 문자열을 다룰 수 있는 표준 함수 세트가 만들어졌습니다.

예를 들어 strcpy() 함수는 근원지 문자열의 각 바이트를 목적지로 복사하는 작업을 반복해(널 종격 바이트를 복사한 후에는 멈춤) 근원지에서 목적지로 문자열을 복사합니다.

- strcpy(목적지, 근원지)

<br>

char_array.c 프로그램이 문자열 라이브러리를 사용해 같은 일을 하게 strcpy()를 사용해 다시 쓰인 것을 보겠습니다.

- 문자열 라이브러리 - string.h

<br>

- char_array2.c

```c
#include <stdio.h>
#include <string.h>

int main()
{
	char str_a[20];
	
	strcpy(str_a,"Hello, world!\n");
	printf(str_a);
}
```

char_array2.c 를 디버거로 살펴보면

```sh
reader@hacking:~/booksrc $ gcc -g -o char_array2.out char_array2.c
reader@hacking:~/booksrc $ gdb -q ./char_array2.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list
1	#include <stdio.h>
2	#include <string.h>
3
4	int main()
5	{
6		char str_a[20];
7	
8		strcpy(str_a,"Hello, world!\n");
9		printf(str_a);
10	}
(gdb) break 7

Breakpoint 1 at 0x80483c4: file char_array2.c, line 7.
(gdb) break strcpy
Function "strcpy" not defined.
Make breakpoint pending on future shared library load? (y or [n]) y
Breakpoint 2 at (strcpy) pending.
(gdb) break 9
Breakpoint 3 at 0x80483d7: file char_array2.c, line 9.
(gdb) 
```

프로그램이 실행되면 strcpy() 중지점이 결정됩니다.

각 중지점에서 EIP와 EIP가 가리키는 명령을 살펴볼텐데, 가운데 중지점에서 EIP를 위한 메모리 공간이 다름에 주목하세요.

```
(gdb) run
Starting program: /home/reader/booksrc/char_array2.out
Breakpoint 4 at 0xb7f076f4
Pending breakpoint "strcpy" resolved
Breakpoint 1, main () at char_array2.c:7
7		strcpy(str_a, "Hello, world!\n");
(gdb) i r eip
eip		0x80483c4		0x80483c4 <main+16>
(gdb) x/5i $eip
0x80483c4 <main+16>: mov	DWORD PTR [esp+4],0x80484c4
0x80483cc <main+24>: lea	eax,[ebp-40]
0x80483cf <main+27>: mov	DWORD PTR [esp],eax
0x80483d2 <main+30>: call	0x80482c4 <strcpy@plt>
0x80483d7 <main+35>: lea	eax,[ebp-40]
(gdb) continue
Continuing.

Breakpoint 4, 0xb7f076f4 in strcpy () from /lib/tls/i686/cmov/libc.so.6
(gdb) i r eip
eip		0x80483d7		0x80483d7 <main+35>
(gdb) x/5i $eip
0x80483d7 <main+35>:	lea		eax,[ebp-40]
0x80483da <main+38>:	mov		DWORD PTR [esp],eax
0x80483dd <main+41>:	call	0x80482d4 <printf@plt>
0x80483e2 <main+46>:	leave
0x80483e3 <main+47>:	ret
(gdb)
```


