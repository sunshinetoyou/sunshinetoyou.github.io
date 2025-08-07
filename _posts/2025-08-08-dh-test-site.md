---
layout: post
title: "[DH]Test site"
date: 2025-08-07 23:31 +0900
category: Write-Up
tag: [dreamhack, web]
---
### **INFO**
![chall]
_Fig 1. Challenge_

쿠키를 변조하여 인증 우회를 하고, url 변조를 통해 블랙 리스트 필터링을 우회하는 문제이다.
### **RECON**
웹 서버의 전반적인 경로는 다음과 같다.

```
 /
 ├─ logintest
 ├─ login 
 ├─ admin
 └─ flag
```

`setcookie(userid)`와 `readcookie(setted_cookie)` 두 함수로 쿠키의 암복호화를 수행한다. 

경로 `/logintest`에서는 login 창과 다르게, 사용자 인증에 실패 시, 입력받은 사용자명에 따른 암호화된 쿠키값을 출력한다.

경로 `/admin` 에서는 관리자 인증과 블랙리스트 기반 url 필터링 기능이 존재한다.

### **SOLUTION**
쿠키를 설정하는데, 동일한 입력에 대해 동일한 출력이 나오는 취약점이 있다. 이는 중간에 쿠키를 탈취할 수 있다면 재사용 공격이 가능함을 의미한다. 

그렇다면 admin 계정의 쿠키값을 어떻게 획득할 수 있을까? 답은 `/logintest`에 있다. ID에 admin을 입력하면(PW는 아무거나 집어넣는다.), error 뒤에 붙은 `setcookie(userid)` 값을 확인할 수 있다.

![logintest]
_Fig 2. logintest 화면_

![logintest_error]
_Fig 3. logintest error_

그럼 이제 `/admin` 경로에 올바른 url를 입력해서 플래그를 획득할 차례이다. url이 최종적으로 실행되는 지점은 서버 내에서 실행되며, 획득해야 되는 파일을 /flag이다. 이는 localhost/flag라는 주소를 서버에 보내야 한다는 결론으로 도달한다.

하지만 블랙리스트 필터링을 담당하는 banlist를 보면, h나 . 같은 문자들이 포함되어있다. 이는 127.0.0.1이나 localhost와 같은 입력을 걸러낸다. 하지만 IPv4 주소 체계에서 주소는 32비트 숫자로 치환 가능하다.
따라서 `127.0.0.1/flag`을 `2130706433/flag`로 변환해서 넣어주면 플래그를 획득할 수 있다.

### **POC**

파이썬의 requests 모듈을 사용하여 poc를 만들었다.

```py
import requests

url = 'http://host8.dreamhack.games:12886/'

cookie = {'id':'8+fQVWU='}

res = requests.get(url+'/admin?url=2130706433/flag', cookies=cookie)
print(res.text)
```

[chall]: /assets/DreamHack/testsite/challenge.png
[logintest]: /assets/DreamHack/testsite/solution_logintest.png
[logintest_error]: /assets/DreamHack/testsite/solution_logintest_error.png
