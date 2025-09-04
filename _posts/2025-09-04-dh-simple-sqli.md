---
layout: post
title: "[DH] simple_sqli"
category: Write-Up
tag:
- dreamhack
- web
date: 2025-09-04 17:22 +0900
---
### INFO
![chall]
_Fig 1. challenge_

웹 페이지와 소스코드가 주어진다.

### RECON
![recon]
_Fig 2. recon_

로그인할 때 id와 pw를 입력받는데 이 부분에 취약점이 있다.

![vul]
_Fig 3. ??: 짜쨘~_

필터링도 안하고, 다이렉트로 SQL 쿼리에 집어넣는다.

이럼 땡큐지, 바로 문제 풀이로 들어가보자.

### SOLVE
id에 admin을 넣고 뒤를 주석처리하면 될 것 같다.

그렇게 하기 위해서 admin"--를 입력하면..

![ans]
_Fig 4. 플래그!_

플래그를 획득할 수 있다!

[chall]: /assets/DreamHack/simple_sqli/chall.png
[recon]: /assets/DreamHack/simple_sqli/recon.png
[vul]: /assets/DreamHack/simple_sqli/vul.png
[ans]: /assets/DreamHack/simple_sqli/ans.png