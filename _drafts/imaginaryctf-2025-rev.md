---
layout: post
title: "[ImaginaryCTF-2025] REV writeup"
category: Write-Up
tag: [dreamhack, reversing]
---

## 


## weired-app
### INFO
![chall1]
_Fig 1. weired-app challenge_

apk 파일이 주어졌다.
### RECON
apk 파일은 안드로이드 실행파일이며, Fig 2와 같이 컴파일된다.

![apk 컴파일 과정]
_Fig 2. APK 컴파일 과정 (출처: https://naro-security.tistory.com/41)_

jadx와 jd-gui와 같은 도구를 사용하면 apk, dex 파일을 java 수준으로 디컴파일하여 정적 분석이 가능하다.

정적 분석의 시작은 두 가지 방법으로 나뉜다.

진입점부터 차근차근 접근하거나, 동적 분석(프로그램 실행)을 통해 식별한 주요 문자열의 참조 위치부터 접근한다.

필자는 사용하는 노트북에 apk 실행하기 위한 환경이 구축되어 있지 않아서, 진입점부터 분석을 시작했다.

### SOLVE
APK의 진입점은 manifest 파일에서 식별할 수 있다.

![ep]
_Fig 3. EP(Entry Point)_

해당 APK 파일의 진입점은 `com.example.test2.MainActivity` 이다.

![oncreate]
_Fig 4. onCreate()_

진입 이후 가장 먼저 실행되는 함수는 `onCreate()` 이다.

![transformedFlag]
_Fig 5. transformedFlag_

`onCreate()`의 시작 부분을 보면, `transformedFlag` 변수에  `MainActivityKt.transformFlag()`로 플래그를 변환하여 저장한다.

![transformedFlag_text]
_Fig 6. transformed Flag Value_

이 변수가 사용되는 최종 함수는 `MainActivityKt.Greeting()`이다.
변환된 플래그 값(`idvi+1{s6e3{)arg2zv[moqa905+`)을 알 수 있다.


### POC
`transformedFlag()`의 역연산 로직을 작성하여 획득한 변환된 플래그 값을 올바른 플래그 값으로 만들었다.

```py
t = "idvi+1{s6e3{)arg2zv[moqa905+"
letters = "abcdefghijklmnopqrstuvwxyz"
digits  = "0123456789"
special = "!@#$%^&*()_+{}[]|"

orig = []
for i, ch in enumerate(t):
    if ch in letters:
        orig.append(letters[(letters.index(ch) - i) % len(letters)])
    elif ch in digits:
        orig.append(digits[(digits.index(ch) - 2*i) % len(digits)])
    elif ch in special:
        orig.append(special[(special.index(ch) - (i*i)) % len(special)])
    else:
        orig.append('?')

print(''.join(orig))  # ictf{FLAG}
```

[chall1]: /assets/CTF/imaginaryCTF2025/wired-app/chall.png
[apk 컴파일 과정]: /assets/CTF/imaginaryCTF2025/wired-app/APK%20컴파일%20과정.png
[apk 컴파일 과정]: /assets/CTF/imaginaryCTF2025/wired-app/ep.png
[apk 컴파일 과정]: /assets/CTF/imaginaryCTF2025/wired-app/main_oncreate.png
[apk 컴파일 과정]: /assets/CTF/imaginaryCTF2025/wired-app/transformedFlag.png
[apk 컴파일 과정]: /assets/CTF/imaginaryCTF2025/wired-app/transformedFlag_text.png
