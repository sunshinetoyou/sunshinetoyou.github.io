---
layout: post
title: "[DH] Typing Game Goes Hard"
category: Wargame
tag:
- dreamhack
- reversing
date: 2025-09-30 11:53 +0900
---
## **INFO**
![chall]
_Fig 1. challenge_

![file]
_Fig 2. given files_

`./deploy` 내부에는 배포된 [`chall`, `dictionary.txt`, `flag`] 파일 존재함.

## **RECON**
<!-- ![recon_1]
_Fig 3. file info_ -->

![recon_2]
_Fig 4. strings info_

당연하게도, `/dev/urandom` 문자열이 들어있는 것을 보아, 주어진 dictionary.txt에서 무작위로 단어를 고르는 로직이 존재함을 유추할 수 있다.
그리고 게임 모드는 Easy와 Hard로 나뉘는 것을 확인할 수 있다.

![recon_3]
_Fig 5. strace result(1)_

![recon_4]
_Fig 6. strace result(2)_

strace를 통해 프로그램이 호출하는 시스템콜을 봤다.</br>
```
Easy Mode
    (1) 입력할 단어가 주어짐 
    (2) 입력에 대해 별도의 처리 없이 다이렉트로 비교
    (3) 정답이라면 10번 반복 후 난이도 상향
Hard Mode
    (1) 입력할 단어 어디감?
```

아마 Hard Mode에서 입력할 단어를 유추하는 것이 이 문제의 핵심일 것 같다.

## **SOLVE**

단어 선택 알고리즘은 총 3개의 함수로 구성된다.<br>
랜덤값을 통해 2바이트 8개 offset 테이블을 구성하고, 이를 global_idx를 통해 주기를 가지고 전체 재배치하며, 정해진 로직에 따라 입력할 단어의 offset을 생성한다.

``` c
// (1) 초기 offset 테이블 구성 & global_idx 설정
void init_504060(uint16_t rand_value_2B)
{
  int idx;
  
  offset_table = rand_value_2B;
  for (idx = 1; idx < 8; idx = idx + 1) {
    (&offset_table)[idx] =
         ((&offset_table)[idx + -1] >> 14 ^ (&offset_table)[idx + -1]) * 0x6c07 + (short)idx;
  }
  global_idx = 8;
  return;
}

// (2) 주기마다 offset_table 재배치[FUN_00101507()] & offset 생성 로직
uint FUN_001015d4(void)
{
  ushort uVar1;
  byte bVar2;
  byte bVar3;
  byte bVar4;
  byte bVar5;
  byte bVar6;
  byte bVar7;
  byte bVar8;
  int iVar9;
  int iVar10;
  int iVar11;
  
  if (7 < global_idx) {
    FUN_00101507();
  }
  iVar9 = global_idx;
  global_idx = global_idx + 1;
  uVar1 = (&offset_table)[iVar9];
  bVar5 = (byte)uVar1 & 0xf;
  bVar6 = (byte)((int)(uint)uVar1 >> 4) & 0xf;
  bVar8 = (byte)(uVar1 >> 8);
  bVar7 = bVar8 & 0xf;
  bVar8 = bVar8 >> 4;
  iVar9 = (uint)bVar8 * 2 +
          (uint)bVar5 * 3 + (uint)bVar6 * 4 + (uint)bVar6 + ((uint)bVar7 * 8 - (uint)bVar7);
  bVar2 = (byte)(iVar9 >> 0x1f);
  iVar10 = (uint)bVar8 * 2 + (uint)bVar8 +
           (uint)bVar5 * 4 + ((uint)bVar6 * 8 - (uint)bVar6) + ((uint)bVar7 * 2 + (uint)bVar7) * 2 ;
  bVar3 = (byte)(iVar10 >> 0x1f);
  iVar11 = ((uint)bVar8 * 8 - (uint)bVar8) +
           (uint)bVar5 * 5 + ((uint)bVar6 * 2 + (uint)bVar6) * 2 + (uint)bVar7 * 4;
  bVar4 = (byte)(iVar11 >> 0x1f);
  return (uint)(byte)(((char)iVar11 + (bVar4 >> 4) & 0xf) - (bVar4 >> 4)) << 0xc |
         (uint)(byte)(((char)iVar9 + (bVar2 >> 4) & 0xf) - (bVar2 >> 4)) |
         (uint)(byte)(((char)iVar10 + (bVar3 >> 4) & 0xf) - (bVar3 >> 4)) << 4 |
         (uint)(bVar8 * 4 + bVar5 * 2 + bVar6 * 3 + bVar7 * 5 & 0xf) << 8;
}

// (3) offset_table 전체 재배열
void FUN_00101507(void)
{
  ushort local_10;
  int idx;
  
  for (idx = 0; idx < 8; idx = idx + 1) {
    local_10 = ((&offset_table)[(idx + 1) % 8] & 0x7fff | (&offset_table)[idx] & 0x8000) >> 1;
    if (((&offset_table)[(idx + 1) % 8] & 1) != 0) {
      local_10 = local_10 ^ 0x9908;
    }
    (&offset_table)[idx] = (&offset_table)[(idx + 4) % 8] ^ local_10;
  }
  global_idx = 0;
  return;
}
```

이렇게 보면 이해가 잘 안되니깐 그림으로 봐보자.

### init_504060()

![solve_1]
_Fig 7.offset-table_

우리가 사전에서 단어를 무작위로 선택하기 위해서 사용하는 offset_table이다.
2바이트로 8개가 묶여있는 배열이며, .data 섹션의 0x504060 위치에 존재한다.

![solve_2]
*Fig 8. init_504060()*

맨 처음 랜덤값을 offset_table의 0번째 인덱스 위치에 넣어준다. 이후 offset_table[0] 값을 통해 offset_table[7]까지 채워나간다.<br>
자세한 계산 로직은 첨부한 코드의 `(1) init_504060()` 함수에서 확인 가능하다.

### FUN_001015d4()
![solve_3]
_Fig 9. FUN_001015d4()의 offset 생성 로직_

`(2) FUN_001015d4()`에서 하나의 offset_table[idx] 값을 가지고 와서, 이를 4비트씩 나누고 정해진 계산을 한다. 계산한 결과를 재조합하여 offset을 최종적으로 설정한다.

그리고 `(3) FUN_00101507()`은 global_idx를 기준으로 특정 주기마다 offset_table을 전체 재배열해준다. 

### 풀이 방법!!

그렇다면 대망의 풀이는 어떻게 가능할까.<br>
첫 번째 접근은 EASY Mode에서 식별할 수 있는 10개의 단어의 offset을 구해서 초기 random 값을 역산하는 것이었다. 이는 위에서 설명한 3개의 로직을 구현하고, 2^16 만큼의 무작위 접근을 통해 해결 가능할 것으로 보인다. 하지만 구현 상의 오류인지 구현 과정에서 걸리는 시간이 늘어나서 다른 방법으로 풀이를 완성했다.

두 번째 풀이는 초기에 생성되는 random 값을 암호화키로 보고, Easy Mode에서 선택되는 단어 배열을 평문, Hard Mode에서 선택되는 단어 배열을 암호문으로 보면서 시작한다.<br>
이때, 암호문과 평문은 상관관계를 갖기 때문에, 확산이 부족한 상태이다.

따라서, 패치를 통해 로컬에서 암호문을 레인보우 테이블로 구성한다면, 플래그를 획득할 수 있다.
```py
def worker(rainbowTable, count):
    for _ in range(count):
        p = process('./TypingGameGoesHard') # Hard Mode 단어 보이게 패치된 프로그램
        wordlist = []
        answer = []
        
        # Easy Mode 단어 수집 -> wordlist
        for _ in range(10):
            p.recvuntil(b'Type this word as soon as possible: ')
            word = p.recvuntil(b'\n')
            wordlist.append(word[:-1])
            p.send(word)
        
        # Hard Mode 단어 수집 -> answer
        for _ in range(10):
            p.recvuntil(b'Type this word as soon as possible: ')
            fword = p.recvuntil(b'\n')
            answer.append(fword[:-1])
            p.send(fword)
        
        # !! 레인보우 테이블에 데이터 추가 !!
        # {
        #   (Easy Mode 단어 배열) : (Hard Mode 단어 배열),
        # }
        rainbowTable[tuple(wordlist)] = tuple(answer)
        p.close()
```
레인보우 테이블을 구성하는데 멀티스레딩을 사용하고도 시간이 꽤 들었지만(4시간 이내), 그동안 푹 잤기 때문에.. 이득이다!

첫 번째 풀이에 대한 해설은 추후에 업데이트 하겠다.

## **PoC**

### 첫 번째 풀이(풀이 실패)
```py
from pwn import *

def read_dictionary():
    with open('dictionary.txt', 'r') as f:
        return [line.strip() for line in f]

def main():
    p = remote('host8.dreamhack.games', 23434)
    
    wordlist = []
    for _ in range(10):
        p.recvuntil(b'Type this word as soon as possible: ')
        word = p.recvuntil(b'\n')
        wordlist.append(word[:-1])
        p.send(word)
    
    send_word = predict_word(wordlist) + '\n'
    p.sendafter(b"Type this word as soon as possible: [REDACTED]\n> ", send_word.encode())
    
    p.interactive()

def predict_word(wordlist: list):
    # 1) word -> offset 변환
    offsets = [dictionary.index(word.decode()) for word in wordlist]
    
    # 2) 시드 brute-force (0~65535)
    for seed in range(0, 0x10000):
        global_idx, arr = set_504060(seed)
        match = True
        for want in offsets:
            global_idx, out = FUN_1015d4(global_idx, arr)
            if out != want:
                match = False
                break
        if match:
            global_idx, next_offset = FUN_1015d4(global_idx, arr)
            print('Predicted offset:', next_offset)
            print('Predicted word:', dictionary[next_offset])
            return dictionary[next_offset]
        
    print('No matching seed found!')
    return None

def FUN_1015d4(global_idx: int, arr: list):
    if global_idx > 7:
        global_idx, updated_arr = update_504060(arr)
        arr = updated_arr
    
    u1 = arr[global_idx]
    global_idx += 1
    
    b5 = u1 & 0xF
    b6 = (u1 >> 4) & 0xF
    b7 = (u1 >> 8) & 0xF
    b8 = (u1 >> 12)
    
    i9 = b8*2 + b5*3 + b6*5 + b7*7
    b2 = (i9 >> 0x1F) & 0xFF
    
    i10 = b8*3 + b5*4 + b6*7 + b7*6
    b3 = (i10 >> 0x1F) & 0xFF
    
    i11 = b8*7 + b5*5 + b6*6 + b7*4
    b4 = (i11 >> 0x1F) & 0xFF
    
    return global_idx, ((i11 & 0xF) + (b4 >> 4) & 0xF) - (b4 >> 4) << 0xc | \
                        ((i9 & 0xF) + (b2 >> 4) & 0xF) - (b2 >> 4) | \
                        ((i10 & 0xF) + (b3 >> 4) & 0xF) - (b3 >> 4) << 0x4 | \
                        (b8*4 + b5*2 + b6*3 + b7*5 & 0xf) << 0x8

def set_504060(rand_value_2B: int) -> list:
    arr = [0] * 8
    arr[0] = rand_value_2B & 0xFFFF
    for idx in range(1,8):
        prev = arr[idx-1]
        val = ((prev >> 14) ^ prev) * 0x6c07 + idx
        arr[idx] = val & 0xFFFF
    return 8, arr

def update_504060(arr: list):
    for idx in range(8):
        next_idx = (idx + 1) % 8
        local_10 = ((arr[next_idx]&0x7FFF) | (arr[idx]&0x8000)) >> 1
        if (arr[next_idx] & 1) != 0:
            local_10 ^= 0x9908
        arr[idx] = arr[(idx + 4)%8] ^ local_10
    return 0, arr

if __name__ == "__main__":
    dictionary = read_dictionary()
    main()
```
### 두 번째 풀이
``` py
import pickle
from pwn import *

def main(rtb: dict):
    p = remote('host8.dreamhack.games', 13000)
    
    wordlist = []
    for _ in range(10):
        p.recvuntil(b'Type this word as soon as possible: ')
        word = p.recvuntil(b'\n')
        wordlist.append(word[:-1])
        p.send(word)
    
    idx = 0
    while idx < 10:
        try:
            print(rtb[tuple(wordlist)])
            send_word = rtb[tuple(wordlist)][idx] + b'\n'
            p.sendafter(b"Type this word as soon as possible: [REDACTED]\n> ", send_word)
            idx += 1
        except KeyError:
            print("NOT FOUND")
            break
    
    if idx == 10:
        p.interactive()
    else:
        main(rtb)

if __name__ == '__main__':
    with open('rainbow_table.pkl', 'rb') as f:
        loaded_dict = pickle.load(f)
    main(loaded_dict)
```
![flag]
_Fig 11. Get Flag!_

[chall]: /assets/DreamHack/typinggamegoeshard/chall.png
[file]: /assets/DreamHack/typinggamegoeshard/file.png
[recon_1]: /assets/DreamHack/typinggamegoeshard/recon_1.png
[recon_2]: /assets/DreamHack/typinggamegoeshard/recon_2.png
[recon_3]: /assets/DreamHack/typinggamegoeshard/recon_3.png
[recon_4]: /assets/DreamHack/typinggamegoeshard/recon_4.png
[solve_1]: /assets/DreamHack/typinggamegoeshard/solve_1.png
[solve_2]: /assets/DreamHack/typinggamegoeshard/solve_2.png
[solve_3]: /assets/DreamHack/typinggamegoeshard/solve_3.png
[make_rbt]: /assets/DreamHack/typinggamegoeshard/make_rbt.png
[flag]: /assets/DreamHack/typinggamegoeshard/flag.png