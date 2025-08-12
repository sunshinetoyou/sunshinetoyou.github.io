---
layout: post
title: "[CTF] WHY2025 : 3-BALL MARK"
date: 2025-08-09 22:37 +0900
category: Write-Up
tag: [CTF, WHY2025CTF, reversing]
---
### **INFO**
![chall]
_Fig 1. Challenge_

노란색 공 1개, 파란색 공 2개 중에 10번 연속으로 노란색 공을 뽑으면 플래그를 획득할 수 있다.

### **RECON**
일단 주어진 프로그램을 실행시켜보면 Fig 2와 같이 1~3 사이의 입력을 요구한다.

![recon_1]
_Fig 2. Program execution_

반복해서 동작해보면, 동일한 입력을 통과시키는 것을 확인할 수 있다. 이는 랜덤값의 시드를 설정하는 `srand` 함수가 Fig 3처럼 잘못 사용되었기 때문이다.

![recon_2]
_Fig 3. srand(0)이 최종 시드로 사용됨_

그리고 노란색 공의 위치를 맞춰야 하는 횟수는 10번으로, 상대적으로 손으로 시도할만한 횟수이다.

![recon_3]
_Fig 4. main logic_

이는 코인 무제한으로 즐기는 유리 다리 건너기를 하면 된다는 것을 알 수 있다.

![유리다리]
_Fig 5. ???: 내가 구별하는 방법을 알아요!!_

### **SOLUTION**
**될 때까지 시도해본다..**

매우 무식한 방법이지만 효율적이다.

번외로 코드를 작성하여, srand(0)일 때의 공이 섞이는 과정을 추적하는 것도 가능하다.

하지만 해당 문제의 경우에는 손으로 입력하면서 푸는 것이 매우 효율적이다. 

(총 요구 정답 횟수가 10회인 것을 확인하고, 1분만에 풀었다.)

### **POC**
(생략)

[chall]: /assets/CTF/WHY2025/3ballmark/challenge.png
[recon_1]: /assets/CTF/WHY2025/3ballmark/recon_1.png
[recon_2]: /assets/CTF/WHY2025/3ballmark/recon_2.png
[recon_3]: /assets/CTF/WHY2025/3ballmark/recon_main.png
[유리다리]: /assets/CTF/WHY2025/3ballmark/유리다리.png
