---
layout: post
title: "[DH]My Favorit Fruit"
date: 2025-08-08 22:45 +0900
category: Write-Up
tag: [dreamhack, rev]
---
### **INFO**
![chall]
_Fig 1. Challenge_

### **RECON**
정해진 과일의 이름을 전부 입력하면 플래그가 나오는 구조임을 쉽게 확인할 수 있다.

(출제자가)좋아하는 과일은 다음과 같다.
```c
char planets[][8] = {"banana", "strawberry", "erwin", "mandarin", "melon"};
```

하지만 입력은 `scanf("%9s", &input)`으로 총 9개의 문자만 한번에 입력받는다.
간단하게 주어진 실행파일에 과일들을 집어넣으면 풀리지 않는다는 의미이다.

### **SOLUTION**
그렇다면 9개 이상의 데이터를 입력받게끔만 수정해주면 된다.

과일을 입력받아서 플래그를 조합하는 로직을 가져와서 poc를 작성한다.

### **POC**
```c
#include <stdio.h>
#include <string.h>

unsigned char DAT_00104020[] = { 0x30, (중략) , 0x7b };

void makeFlag(char *param_1)
{
  size_t len;
  int i;
  
  len = strlen(param_1);
  for (i = 0; i < 69; i = i + 1) {
    DAT_00104020[i] = DAT_00104020[i] ^ param_1[i % len];
  }
  return;
}

int main() {
    char input[256];

    for (int i = 0; i < 5; i++) {
        scanf("%s", &input);
        makeFlag(input);
    }

    printf("%s", DAT_00104020);
    return 0;
}
```

[chall]: /assets/DreamHack/mff/challenge.png