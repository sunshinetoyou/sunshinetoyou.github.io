---
layout: post
title: "[CTF] WHY2025 : LAZY CODE 1.0"
date: 2025-08-10 13:06 +0900
category: Write-Up
tag: [CTF, WHY2025CTF, reversing]
---
### **INFO**
![chall]
_Fig 1. Challenge_

해당 문제는 `WHY2025 : lazycode2` 문제와 비슷합니다.

### **RECON**
주어진 파일(lazy-code-1.exe)을 실행시켜 보면, 600초마다 카운트가 1씩 오르면 카운트 1000까지 기다려야 되는 것을 확인할 수 있다.

![recon_1]
_Fig 2. Program Execution_

### **SOLUTION**
그렇다면 기드라를 통한 패치를 통해 600초를 0초로 줄이면 즉시 플래그를 획득할 수 있을 것이다.

메인로직은 Fig 3과 같다.
sleep 함수 안에 들어가는 인자값을 패치하면 된다. (Fig 3는 이미 패치된 상태임)

![main]
_Fig 3. Main logic_

패치하고 기드라에서 [File] -> [Export Program ..] 을 눌러 패치 프로그램을 획득한다.

![patch]
_Fig 4. How to patch_

### **POC**
패치된 프로그램을 동작시켜보면, 플래그가 잘 나오는 것을 확인할 수 있다.

![result]
_Fig 5. Patched program Execution_

[chall]: /assets/CTF/WHY2025/lazycode/challenge.png
[recon_1]: /assets/CTF/WHY2025/lazycode/recon_1.png
[main]: /assets/CTF/WHY2025/lazycode/solution_main.png
[patch]: /assets/CTF/WHY2025/lazycode/solution_patch.png
[result]: /assets/CTF/WHY2025/lazycode/poc_result.png