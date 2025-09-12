---
layout: post
title: "[DH] Small Counter"
tag:
- dreamhack
- reversing
category:
- writeup
date: 2025-09-12 14:04 +0900
---
### INFO
![chall]
_Fig 1. challenge_

실행파일이 하나 주어진다.

### RECON

![da_1]
_Fig 2. Dynamic Analysis 1_

실행 파일을 동작시키면, 10부터 1까지 출력하고 종료된다.

실행파일을 기드라로 분석하면 아래와 같은 로직을 분석할 수 있다.

```
FOR idx = 10, idx > 0, idx -= 1
    PRINT idx
    IF idx == 3
        SET encoded_flag
    END
END

IF idx == 5
    FLAG_GEN encoded_flag -> flag
    PRINT flag
END
```

그렇다면 메모리 상에 디코딩된 플래그가 존재할 것이며, GDB를 통해서 값을 획득하는 것을 노려볼 수 있다.

### SOLVE
먼저, gdb <실행파일>과 start 명령을 통해 동적 분석을 위한 베이스를 준비한다.

![sol_1]
_Fig 2. _

그리고 기드라에서 BP를 설정할 정적 주소를 확인한다.<br>
단, 기드라에서 디폴트로 설정한 0x100000 를 빼고 오프셋을 구한다.

![sol_2]
_Fig 3. _

pwndbg를 설치하여, 해당 도구에서 사용할 수 있는 rebase 함수를 통해 동적 주소에 오프셋을 가지고 접근했다.

continue를 통해 설정한 bp 위치로 이동한다.

![sol_3]
_Fig 4. _

이동한 위치를 살펴보면 rbp - 0x4 위치에 있는 값을 8 Byte만큼 가져왔을 때 값이 0임을 확인할 수 있다. 

이대로 실행하면 플래그를 디코딩하여 프린트하는 명령줄은 실행되지 않는다.

![sol_4]
_Fig 5. _

dword ptr [rbp - 0x4] 값을 5로 변조하고 동작시키면 플래그를 획득할 수 있다.

### POC

사용한 명령어 모음이다. GDB에 익숙하지 않다면 한번 따라 쳐보면 도움이 될 것이다.

``` cmd
# 특정 주소에 bp 걸기
break *$rebase(0x????)

# 특정 주소(메모리)를 어셈블리, 16진수 값으로 보여줌
x/i $rebase(0x????)
x/wx $rebase(0x????)

# 다음 bp가 나올때까지 프로그램 실행
c
countinue
```

기드라로 주소 확인하여 넣어주는 것 대신에 더 효율적인 방법이 있나 찾아봐야겠다.


[chall]: /assets/DreamHack/smallcounter/chall.png
[sol_1]: /assets/DreamHack/smallcounter/sol_1.png
[sol_2]: /assets/DreamHack/smallcounter/sol_2.png
[sol_3]: /assets/DreamHack/smallcounter/sol_3.png
[sol_4]: /assets/DreamHack/smallcounter/sol_4.png