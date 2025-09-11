---
layout: post
title: "[DH] Recover"
category: Write-Up
tag:
- dreamhack
- reversing
date: 2025-09-11 21:56 +0900
---
### INFO
![chall]
_Fig 1. challenge_

![files]
_Fig 2. files_

실행파일과 데이터 파일이 주어진다.

### RECON
기드라로 파일을 열어보면, 메인 로직이 바로 보인다.

![정적분석_1]
_Fig 3. Main Logic_

    1) 1바이트씩 `flag.png`를 읽는다.
    2) 암호화를 한다.
    3) 암호화된 값을 `encrypted` 파일에 저장한다.

그렇다면 해당 로직을 역연산하여 flag.png를 획득하면 될 것 같다.

### SOLVE

최대한 기드라의 Retype 기능을 이용해서 POC를 작성하였다.

전에 `Byte` 자료형을 `uchar`으로 매핑하여야 된다는 멍청한 소리를 하였는데,, 헷갈리셨을 분들에게 미리 심심한 사과의 말씀을 드린다..

`uintN_t` 자료형을 사용하는 것이 더 올바르다.<br>
해당 자료형은 N비트 만큼의 고정 크기를 가지는 정수형을 의미한다.<br>
`Byte`는 `uint8_t`로 변환하여 POC 코드를 쉽게 작성할 수 있다.

POC를 실행시키면, flag.png를 얻을 수 있고 이미지 안에 플래그가 존재한다.

### POC

```c
#include <stdio.h>
#include <stdbool.h>
#include <stdint.h>

const uint8_t data[4] = { 0xde, 0xad, 0xbe, 0xef };

int main() {

    FILE* encrypted_data;
    FILE* flag_png;
    int idx;
    int sVar1;
    uint8_t buf;

    encrypted_data = fopen("./encrypted","rb");
    flag_png = fopen("./flag.png","wb");

    idx = 0;
    while( true ) {
        sVar1 = fread(&buf,1,1,encrypted_data);
        if (sVar1 != 1) break;
        buf = (buf-19) ^ data[idx % 4];
        fwrite(&buf,1,1,flag_png);
        idx = idx + 1;
    }

    return 0;
}
```

### 이번 문제를 풀며 느낀점

기드라의 코드를 retype하여 C로 POC를 작성하는 것이 매우 편하다는 것을 알았다. 하지만 C의 자료형과 친숙하지 않아서 서칭하는 과정에서 시간 소요가 컸었다. 

POC 작성을 위한 C 자료형 정리를 해서 포스트를 해야겠다.

[chall]: /assets//DreamHack/recover/chall.png
[files]: /assets/DreamHack/recover/files.png
[정적분석_1]: /assets/DreamHack/recover/정적분석_1.png