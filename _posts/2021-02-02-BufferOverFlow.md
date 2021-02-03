---
layout: single
title: "버퍼 오버플로우 공격"
categories: 보안
tags: [C언어, bufferOverFlow, Attack]
toc: true
toc_sticky: true
---

***이 포스팅은 책 "해킹 공격의 예술 개정 2판"기반으로 기록한 페이지 입니다.***



## 버퍼 오버플로우

[버퍼 오버플로우(오버런)](https://ko.wikipedia.org/wiki/%EB%B2%84%ED%8D%BC_%EC%98%A4%EB%B2%84%ED%94%8C%EB%A1%9C)은 메모리를 다루는 데에 오류가 발생하여 잘못된 동작을 하는 프로그램 취약점이다.

버퍼 오버플로우(Buffer overflow) 취약점은 컴퓨터 초기부터 지금까지 계속 발생하고 있습니다. 대부분의 **인터넷 웜**은 **전파**를 위해 **버퍼 오버플로우 취약점을 이용**합니다.

C는 하이레벨 PL 이지만 자체적으로 데이터 무결성을 검사하는 기능은 없어서 프로그래머가 직접 데이터 무겨성을 검사해야 합니다. 

만약 C 컴파일러에서 직접 데이터 무결성을 검사했다면 실행 파일의 속도가 엄청나게 느려졌을 것입니다.

C는 단순한 언어이므로 프로그래머가 프로그램을 원하는 대로 제어하고 효율적으로 만들 수 있다는 장점이 있지만,

프로그래머가 신경을 쓰지 않을 경우 버퍼 오버플로우와  메모리 누수 현상이 발생할 수 있는 단점이 있습니다.

이는 C 언어에서 어떤 변수가 메모리에 할당돼 있을 경우 그 변수의 내용이 메모리 경계를 벗어나는지 벗어나지 않는지 검사하는 내부 안전장치가 없다는 의미입니다.

이것을 예를 들면 8바이트 크기의 버퍼에 10바이트를 넣는 것이 가능하다는 뜻 입니다.

해당 변수의 정해진 크기보다 넘어서는 크기의 값을 넣는 것을 버퍼 오버플로우(오버런)이라고 하고, 

10바이트를 8바이트에 저장하려할 때 나머지 2바이트는 다른 메모리 영역을 덮어쓰게 됩니다.

이러한 버퍼 오버런이 가져오는 피해는 프로그램이 잘못된 연산으로 의도치 않게 종료되는 상황이 생기게 됩니다.

다음은 오버플로우의 예시를 코드로 보겠습니다.

#### [코드 및 결과]
- #### overflow_example.c

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



- #### 컴파일 후 실행결과

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
