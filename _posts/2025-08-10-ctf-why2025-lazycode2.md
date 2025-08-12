---
layout: post
title: "[CTF] WHY2025 : LAZY CODE 2.0"
date: 2025-08-10 13:15 +0900
category: Write-Up
tag: [CTF, WHY2025CTF, reversing]
---
### **INFO**
![chall]
_Fig 1. Challenge_

[WHY2025 : lazycode] [lazycode]와 실행 파일 형식만 바뀐 문제이다.

### **RECON**
리눅스 환경에서 실행시켜주면 lazycode.exe와 동일하게 출력된다.

![recon_1]
_Fig 2. Program Execution_

### **SOLUTION**
동일하게 패치를 통해서 문제 풀이를 진행한다. (참고: [WHY2025 : lazycode] [lazycode])
### **POC**
실행 결과는 Fig 3과 같다.

![result]
_Fig 3. Patched program Execution_

[chall]: /assets/CTF/WHY2025/lazycode2/challenge.png
[recon_1]: /assets/CTF/WHY2025/lazycode2/recon_1.png
[result]: /assets/CTF/WHY2025/lazycode2/poc_result.png

[lazycode]: https://sunshinetoyou.github.io/posts/2025-08-10-ctf-why2025-lazycode.md