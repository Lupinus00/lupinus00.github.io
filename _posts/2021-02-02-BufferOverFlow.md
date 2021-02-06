---
layout: single
title: "버퍼 오버플로우 공격"
categories: 보안
tags: [C언어, bufferOverFlow]
toc: true
toc_sticky: true
---

***이 포스팅은 책 "해킹 공격의 예술 개정 2판"기반으로 기록한 페이지 입니다.***


## Ch1. 버퍼오버플로우란 ?

[**버퍼 오버플로우(오버런)**](https://ko.wikipedia.org/wiki/%EB%B2%84%ED%8D%BC_%EC%98%A4%EB%B2%84%ED%94%8C%EB%A1%9C)은 메모리를 다루는 데에 오류가 발생하여 잘못된 동작을 하는 프로그램 취약점이다.

버퍼 오버플로우(Buffer overflow) 취약점은 컴퓨터 초기부터 지금까지 계속 발생하고 있습니다. 

대부분의 **인터넷 웜**은 **전파**를 위해 **버퍼 오버플로우 취약점을 이용**합니다.

C는 하이레벨 PL 이지만 자체적으로 데이터 무결성을 검사하는 기능은 없어서 프로그래머가 직접 데이터 무겨성을 검사해야 합니다. 

만약 C 컴파일러에서 직접 데이터 무결성을 검사했다면 실행 파일의 속도가 엄청나게 느려졌을 것입니다.

C는 단순한 언어이므로 프로그래머가 프로그램을 원하는 대로 제어하고 효율적으로 만들 수 있다는 장점이 있지만

프로그래머가 신경을 쓰지 않을 경우 버퍼 오버플로우와  메모리 누수 현상이 발생할 수 있는 단점이 있습니다.

이는 C 언어에서 어떤 변수가 메모리에 할당돼 있을 경우 

그 변수의 내용이 메모리 경계를 벗어나는지 벗어나지 않는지 검사하는 내부 안전장치가 없다는 의미입니다.

이것을 예를 들면 8바이트 크기의 버퍼에 10바이트를 넣는 것이 가능하다는 뜻 입니다.

해당 변수의 정해진 크기보다 넘어서는 크기의 값을 넣는 것을 버퍼 오버플로우(오버런)이라고 하고 

10바이트를 8바이트에 저장하려할 때 나머지 2바이트는 다른 메모리 영역을 덮어쓰게 됩니다.

이러한 버퍼 오버런이 가져오는 피해는 프로그램이 잘못된 연산으로 의도치 않게 종료되는 상황이 생기게 됩니다.



다음은 오버플로우의 예시를 코드로 보겠습니다 .



### Ch1.1 overflow_example.c

```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[])
{
    int value = 5;
    char buffer_one[8], buffer_two[8];
    
    strcpy(buffer_one, "one");
    strcpy(buffer_two, "two");
    
    printf("[BEFORE] 버퍼2| 주소: %p, 데이터: \'%s\'\n", buffer_two, buffer_two);
    printf("[BEFORE] 버퍼1| 주소: %p, 데이터: \'%s\'\n", buffer_one, buffer_one);
    printf("[BEFORE] value| 주소: %p, 데이터: %d\n", &value, value);
    
    // ===============================================================
    
    printf("\n[STRCPY] %d바이트를 버퍼2에 복사\n\n", strlen(argv[1]));
    strcpy(buffer_two, argv[1]); // 첫 인자를 버퍼2에 복사
    
    printf("[AFTER] 버퍼2| 주소: %p, 데이터: \'%s\'\n", buffer_two, buffer_two);
    printf("[AFTER] 버퍼1| 주소: %p, 데이터: \'%s\'\n", buffer_one, buffer_one);
    printf("[AFTER] value| 주소: %p, 데이터: %d\n", &value, value);
}
```



### Ch1.2 컴파일 후 실행결과

```shell
reader@hacking:~/booksrc $ gcc -o overflow.out overflow_example.c
reader@hacking:~/booksrc $ ./overflow.out 1234567890
[BEFORE] 버퍼2| 주소: 0xbffff7f0, 데이터: 'two'
[BEFORE] 버퍼1| 주소: 0xbffff7f8, 데이터: 'one'
[BEFORE] value| 주소: 0xbffff804, 데이터: 5

[STRCPY] 10바이트를 버퍼2에 복사

[BEFORE] 버퍼2| 주소: 0xbffff7f0, 데이터: '1234567890'
[BEFORE] 버퍼1| 주소: 0xbffff7f8, 데이터: '90'
[BEFORE] value| 주소: 0xbffff804, 데이터: 5
```

코드에선 이전값을 보여주고 10바이트를 8바이트 공간에 복사하는 과정을 거친 후 이후 값을 보여주는 과정을 거칩니다.

우선 실행결과에서 버퍼2와 버퍼1의 주소를 보면 8바이트 차이가 나는걸 볼 수 있습니다.  

(메모리상 버퍼1은 버퍼2 바로 다음에 있다는 의미)

그리고 실행결과를 보면 인자값 '1234567890' 을 버퍼2에 저장하니 버퍼2의 출력결과가 '1234567890' 가 출력되고, 

10바이트중  뒤에 2바이트인 '90'은 메모리 구조상 버퍼2의 다음에 있는 버퍼1에 저장되기 때문에 버퍼1의 출력결과가 '90'이 되는 것입니다.

이렇게 버퍼1의 값이 'one'에서 '90'으로 덮어씌여진 것이 버퍼 오버플로우를 이용한 데이터를 바꾸어 프로그램을 망치는 방법입니다. 

큰 버퍼를 이용하면 자연히 다른 변수까지도 오버플로우하고, 충분히 큰 버퍼를 사용하면 프로그램을 죽이기까지도 합니다.





## Ch2. 스택 기반 버퍼 오버플로우



### Ch2.1 auth_overflow.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_authentication(char *password) {
	int auth_flag = 0;
	char password_buffer[16];
	
	strcpy(password_buffer, password);
	
	if(strcmp(password_buffer, "Hello") == 0)
		auth_flag = 1;
	if(strcmp(password_buffer, "World") == 0)
		auth_flag = 1;
		
	return auth_flag;
}

int main(int argc, char *argv[]) {
    if(argv < 2) {
        printf("Usage: %s <password> \n", argv[0]);
    	exit(0);
    }
    
    if(check_authentication(argv[1])) {
        printf("\n=-=-=-=-=-=-=-=-=-=\n");
        printf("\t접근 허용.\n");
        printf("\n=-=-=-=-=-=-=-=-=-=\n");
    } else {
        printf("\n 접근 불가.\n");
    }
}
```

이 프로그램은 패스워드를 인자로 받아 check_authentication() 함수를 호출해 패스워드를 체크한 후

올바르게 입력 시 접근 허용이 출력되고, 아닐 시 접근 불가가 출력되는 프로그램 입니다.

check_authentication() 함수의 코드를 보면 Hello와 World라는 패스워드만 허용이 되는것을 볼 수 있는데 올바르게 작동하는지를 보겠습니다.



### Ch2.2 실행 결과 

```shell
reader@hacking:~/booksrc $ gcc -o auth_overflow.out auth_overflow.c
reader@hacking:~/booksrc $ ./auth_overflow.out
Usage: ./auth_overflow.out <password>
reader@hacking:~/booksrc $ ./auth_overflow.out test

접근 불가.
reader@hacking:~/booksrc $ ./auth_overflow.out Hello

=-=-=-=-=-=-=-=-=-=
    접근 허용.
=-=-=-=-=-=-=-=-=-=
reader@hacking:~/booksrc $ ./auth_overflow.out World

=-=-=-=-=-=-=-=-=-=
    접근 허용.
=-=-=-=-=-=-=-=-=-=
reader@hacking:~/booksrc $ 
```

위 실행 결과를 보면 패스워드를 올바르게 입력 시 접근 허용, 

올바르지 않게 입력할 시 접근 불가가 출력되는 것을 볼 수 있습니다.

즉, 코드가 잘 작동하는 것을 볼 수 있습니다.

하지만, 이 코드엔 문제가 있는데 바로 **버퍼 오버런**을 이용해 올바르지 않는 패스워드를 허용하는 모순된 결과를 보여줍니다.



### Ch2.2.1 버퍼 오버런 이용 결과

```shell
reader@hacking:~/booksrc $ ./auth_overflow.out AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

=-=-=-=-=-=-=-=-=-=
    접근 허용.
=-=-=-=-=-=-=-=-=-=
reader@hacking:~/booksrc $
```

위 실행 결과 처럼 올바른 패스워드(Hello, World)가 아닌 문자를 길게 입력하자 접근 허용이 된 것을 볼 수 있습니다.

다음은 디버거를 사용해 좀 더 자세히 살펴보겠습니다.



### Ch2.3 C 파일 디버깅

```
reader@hacking:~/booksrc $ gdb -q ./auth_overflow.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1		#include <stdio.h>
2		#include <stdlib.h>
3		#include <string.h>
4
5		int check_authentication(char *password) {
6			int auth_flag = 0;
7			char password_buffer[16];
8	
9			strcpy(password_buffer, password);
10
(gdb)
11			if(strcmp(password_buffer, "Hello") == 0)
12				auth_flag = 1;
13			if(strcmp(password_buffer, "World") == 0)
14				auth_flag = 1;
15		
16			return auth_flag;
17		}
18
19		int main(int argc, char *argv[]) {
20    		if(argv < 2) {
(gdb) break 9
Breakpoint 1 at 0x8048421: file auth_overflow.c, line 9.
(gdb) break 16
Breakpoint 2 at 0x804846f: file auth_overflow.c, line 16.
(gdb)
```

우선,  -q 옵션은 gdb 디버거를 실행한 후 인사말이 나타나지 않게 해주고, 

9번, 16번 줄에 중지점을 설정했습니다. 중지점에서 멈췄으니 이제 메모리를 조사해 보겠습니다. 



```
(gdb) run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Starting program: /home/reader/booksrc/auth_overflow AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, check_authentication (password=0xbffff9af 'A' <repeats 30 times>) at auth_overflow.c:9
9			strcpy(password_buffer, password);
(gdb) x/s password_buffer
0xbffff7a0:	  ")????o??????)\205\004\b?o??p???????"
(gdb) x/x &auth_flag
0xbffff7bc:	 0x00000000
(gdb) print 0xbffff7bc - 0xbffff7a0
$1 = 28
(gdb) x/16xw password_buffer
0xbffff7a0:		0xb7f9f729		0xb7fd6ff4		0xbffff7d8		0x08048529
0xbffff7b0:		0xb7fd6ff4		0xbffff870		0xbffff7d8		0x00000000
0xbffff7c0:		0xb7ff47b0		0x08048510		0xbffff7d8		0x080484bb
0xbffff7d0:		0xbffff9af		0x08048510		0xbffff838		0xb7eafebc
(gdb)
```

첫 번째 중지점은 strcpy() 함수 바로 전에 있습니다.

password_buffer 포인터를 조사해보니 0xbffff7a0 주소에 위치하고 초기화되지 않은 임의의 값(쓰레기 값)이 들어있습니다.

auth_flag 변수의 포인터를 조사해보니 0xbffff7bc에 위치하고 값은 0으로 세팅되어 있습니다.

print 명령으로 계산해보니 auth_flag는 password_buffer의 시작점에서부터 28바이트 뒤에 있음을 알 수 있습니다.

또한 auth_flag는 password_buffer의 메모리 블록 안에서 찾을 수 있습니다.

auth_flag의 위치는 바로 password_buffer의 메모리 블록의 우측에 있는 0x00000000 입니다.



``` 
(gdb) continue
Continuing.

Breakpoint 2, check_authentication(password=0xbffff9af 'A' <repeats 30 times>) at auth_overflow.c:16
16			return auth_flag;
(gdb) x/s password_buffer
0xbffff7a0:	  'A' <repeat 30 times>
(gdb) x/x &auth_flag
0xbffff7bc:	 0x00004141
(gdb) x/16xw password_buffer
0xbffff7a0:		0x41414141		0x41414141		0x41414141		0x41414141
0xbffff7b0:		0x41414141		0x41414141		0x41414141		0x00004141
0xbffff7c0:		0xb7ff47b0		0x08048510		0xbffff7d8		0x080484bb
0xbffff7d0:		0xbffff9af		0x08048510		0xbffff838		0xb7eafebc
(gdb) x/4cb &auth_flag
0xbffff7bc:		65 'A'  65 'A'  0 '\0'  0 '\0'
(gdb) x/dw &auth_flag
0xbffff7bc:		16705
(gdb)
```

strcpy() 함수 이후 디버깅을 계속해 다음 중지점에 도달했다.

여기서도 메모리를 조사해보면, password_buffer가 오버플로우돼 auth_flag의 첫 두 바이트가 0x41로 바뀐 것을 볼 수 있습니다.

x86은 리틀 엔디언 구조여서 0x00004141로 나와 있습니다.

4바이트를 각기 조사해보면 메모리에 실제 어떤 값이 있는지 볼 수 있습니다.

궁극적으로 프로그램은 이 값을 16705라는 정수로 처리합니다.

```
(gdb) continue
Continuing.

=-=-=-=-=-=-=-=-=-=
    접근 허용.
=-=-=-=-=-=-=-=-=-=
Program exited with code 034.
(gdb)
```

오버플로우 후에는 check_authentication 함수가 0 대신 16705를 리턴합니다.

if 구문에서 0이 아닌 값은 모두 인증(참)이 되는 것으로 처리했으므로 프로그램은 접근 허용 쪽 구문을 실행하게 됩니다.

이 예제에선 덮어써진 auth_flag 변수가 **실행 제어 포인트** (execution control point) 입니다.

하지만 이 프로그램은 변수의 메모리 위치에 영향을 받게 조작되 만들어진 것이기 때문에

아래 auth_overflow2.c 코드에서는 변수 순서를 바꿔 선언했습니다.



#### ch2.4 auth_overflow2.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_authentication(char *password) {
	char password_buffer[16]; 	// auth_overflow.c에선 auth_flag가 먼저 선언되었음.
    int auth_flag = 0;			// 여기선 password_buffer가 먼저 선언됨
	
	strcpy(password_buffer, password);
	
	if(strcmp(password_buffer, "Hello") == 0)
		auth_flag = 1;
	if(strcmp(password_buffer, "World") == 0)
		auth_flag = 1;
		
	return auth_flag;
}

int main(int argc, char *argv[]) {
    if(argv < 2) {
        printf("Usage: %s <password> \n", argv[0]);
    	exit(0);
    }
    
    if(check_authentication(argv[1])) {
        printf("\n=-=-=-=-=-=-=-=-=-=\n");
        printf("\t접근 허용.\n");
        printf("\n=-=-=-=-=-=-=-=-=-=\n");
    } else {
        printf("\n 접근 불가.\n");
    }
}
```



위 코드는 단순하게 메모리상에서 auth_flag 변수를 password_buffer 앞에 오게 바꾼 것입니다.

오버플로우 메모리 오류가 발생하지 않으므로 이제 더 이상 auth_flag변수가 **실행 제어 포인트**가 아닙니다.

이렇게 하게 되면, 메모리상 auth_flag 변수가 password_buffer 앞에 있기 때문에

**password_buffer가 오버플로우가 발생해도 auth_flag가 덮어쓰이지 않으므로** 

해당 프로그램에서 당장의 버퍼 오버플로우 문제를 해결할 수 있게 됩니다.   

**마치며**

​	처음에 오버플로우에 대해서 "아, 그냥 이런게 있구나" 하고 넘어갔는데 막상 책을 통해 공부해보니 내부 시스템적으로도 큰 공부가 되었고

​	코드를 보고 깊게 해석해보며, 버퍼 오버플로우 약점이 있는 코드를 조금은 분간할 수 있는 수준이 된 것 같아서 뜻 깊은 시간이었습니다.

​	버퍼 오버런 공격을 이용해 데이터가 변조, 프로그래머가 짠 의도대로 흐르지 않게 방해 할 수 있다는 것을 배우고나니

​	금전적인 요소가 걸린 프로그램에 이러한 문제점이 있다면, 그런 프로젝트가 널리 사용되고 있다면,

​	이 공격기법으로 인해 발생되는 피해가 엄청날 수도 있다는 것을 느꼈습니다. 

​	*(물론 그런 프로젝트는 여러 대단하신 프로그래머분들이 잘 처리를 하셨겠지만...)*
