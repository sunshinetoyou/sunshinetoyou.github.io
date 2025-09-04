---
layout: post
title: "[DH] session-basic"
date: 2025-09-03 16:50 +0900
category: Write-Up
tag: [dreamhack, web]
---
### **INFO**
![chall]
_Fig 1. challenge(Level 1)_

웹페이지의 URL과 서버에서 구동되고 있는 소스코드(app.py)가 주어진다.

### **RECON**
주어진 app.py를 천천히 살펴보자.

![flag_trigger]
_Fig 2. 플래그 획득 방법_

Fig 2에서 확인할 수 있듯이, 사용자명이 admin이면 Flag를 노출시킬 수 있다!
그렇다면 사용자명은 어디서 설정하고 있을까?

사용자명은 서버의 세션 저장소에 딕셔너리로 저장되어 있으며, 쿠키에 저장된 세션값을 통해 설정된다.

최종적으로 우리는 admin의 세션 ID를 탈취하면 플래그를 획득할 수 있다.

### **SOLUTION**
목표를 세웠으면 방법을 찾아보자.

app.py를 좀 더 살펴보면, 아주 쉽게 세션 저장소를 탈취할 수 있는 방법을 알 수 있다.

![how]
_Fig 3. What the ..?_

아.. 주석의 TODO를 보면 개발 중인 코드가 검토없이 올라온 상황인 것 같다.

![시말서]
_Fig 4. 분명.._

실제로는 어떻게 되려나 궁금하다 하핳

어쨌든 우리는 다음과 같은 방법으로 FLAG를 획득하면 될 것이다.

    1. `/admin` 경로에서 세션 저장소 탈취
    2. admin의 세션 ID를 획득
    3. sessionid의 값으로 admin의 세션 ID를 주입
    4. `/` 경로에서 FLAG 획득

### **POC**
실제로 플래그 획득을 진행하는 과정은 다음 사진들과 같다.

![poc_1]
_Fig 5. STEP 1,2_

![poc_2]
_Fig 6. STEP 3_

![poc_3]
_Fig 7. STEP 4_

[chall]: /assets/DreamHack/session_basic/chall.png
[flag_trigger]: /assets/DreamHack/session_basic/flag_trigger.png
[how]: /assets/DreamHack/session_basic/howto.png
[시말서]: /assets/DreamHack/session_basic/시말서.jpg
[poc_1]: /assets/DreamHack/session_basic/poc_1.png
[poc_2]: /assets/DreamHack/session_basic/poc_2.png
[poc_3]: /assets/DreamHack/session_basic/poc_3.png