---
layout: post
title: "[DH] flag printer"
category: Write-Up
tag:
- dreamhack
- reversing
date: 2025-09-09 20:13 +0900
---
### INFO
![chall]
_Fig 1. challenge_

주어진 파일은 실행파일과 텍스트파일 2개다.
(Dockerfile도 있다.)

### RECON

#### 동적 분석

![동적분석2]
_Fig 2. Dynamic Analysis 1_
strace를 통해 함수의 호출을 살펴본다. 사용자 입력으로 'test'를 입력하니, `&-17`와 어떤 값을 비교하고 올바른 명령이 아니라는 출력을 내뱉는다.

아마 입력을 논리식을 통해 암호화하여 비교하는 로직이 있겠거니 추측했다.

![동적분석1]
_Fig 3. Dynamic Analysis 2_
help, id, print 명령어가 있는 것을 확인했다. 

#### 정적 분석
![정적분석2]
_Fig 4. Static Analysis_

분석한 내용을 간단하게 순서도로 표현하면 Fig 4와 같다.(생략된 부분이 있음을 확인해야 한다.)

이중에서 빨간색 바운더리된 부분이 플래그를 얻기 위해 중요한 부분이다.

먼저, 사용자 입력을 인덱스로 변환하는 과정을 살펴보자.

사용자 입력을 표와 같이 인덱스로 치환한다.

|사용자 입력|인덱스(상위 32비트)|인덱스(하위 32비트)|
|:-:|:-:|:-:|
|print|인덱스(하위32비트)|0x00000000|
|id|인덱스(하위32비트)|0x00000001|
|help|인덱스(하위32비트)|0x00000002|
|dosu|0x00000001|0xFFFFFFFF|

여기서 dosu를 입력했을 때만 `strtok()`로 공백 뒤의 문자열을 가져와서 한 번 더 인덱스를 계산한다. 마치 sudo로 권한을 올려서 명령어를 실행시키는 것과 비슷하다.

dosu는 비교되는 문자열 `&-17`을 XOR 66하여 얻어냈다.

모든 사용자 입력에 대해 처리가 끝났을 때의 인덱스를 반환한다.

리턴된 인덱스의 하위 32비트를 가지고 분기를 처리한다.

두 번째로 '플래그 출력' 함수의 소스 코드를 봐보자.

```c
void print_flag(int param_1)

{
  if (param_1 == 0) __filename = "./art";
  else __filename = "./flag";

  __stream = fopen(__filename,"r");
  
  // (파일 내용 출력)
}
```

원래 `./art` 파일의 아스키 그림을 출력해내던 것을 DH{...} 형태로 출력하게 만들기 위해서는 인자의 값을 0이 아닌 값으로 만들어줘야 한다.

### SOLVE

그렇다면 정적분석을 통해 확인한 dosu의 위력(?)을 확인해보자

dosu <문자열> 이 유력한 페이로드로 보이는데..

dosu print가 동작한다.


|사용자 입력|인덱스(상위 32비트)|인덱스(하위 32비트)|
|:-:|:-:|:-:|
|dosu|0x00000001|0xFFFFFFFF|
|print|0xFFFFFFFF|0x00000000|

`dosu print`에 대한 최종 리턴값은 0xFFFFFFFF00000000이라 분기도 통과하고 플래그 출력 함수의 인자값에 0이 아닌 수를 넣을 수 있다.

이를 서버에 있는 함수에 명령하면 올바른 플래그를 획득할 수 있다.

### POC
(따로 없당..)

신기해서 남겨두는 기드라 호출 흐름 그래프 
![정적분석1]

[chall]: /assets/DreamHack/flag%20printer/chall.png
[동적분석1]: /assets/DreamHack/flag%20printer/동적분석_1.png
[동적분석2]: /assets/DreamHack/flag%20printer/동적분석_2.png
[정적분석1]: /assets/DreamHack/flag%20printer/정적분석_1.png
[정적분석2]: /assets/DreamHack/flag%20printer/정적분석_2.png