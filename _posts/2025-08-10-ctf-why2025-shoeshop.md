---
layout: post
title: "[CTF] WHY2025 : SHOE SHOP 1.0"
date: 2025-08-10 13:50 +0900
category: Write-Up
tag: [CTF, WHY2025CTF, web]
---
### **INFO**
![chall]
_Fig 1. Challenge_

### **RECON**
신발을 판매하는 전형적인 온라인 쇼핑몰 웹사이트가 주어진다.

![homepage]
_Fig 2. Home Page_

계정 등록 이후 로그인을 하면 상단 네비게이션 바의 메뉴가 `Cart`로 변경된다.

![login]
_Fig 3. Home Page (After Login)_

세 종류의 신발을 판매하는데, 마지막 신발인 "Exclusive: Ultraboost Flagship"은 재고 없음 처리가 되어있다. 이 부분에서 플래그는 저 신발을 Cart에서 확인하면 나오겠다고 게싱을 할 수 있다.
![shop]
_Fig 4. Shop Page_

장바구니 페이지에서 그동안 추가한 신발들을 모아볼 수 있다. 
![cart]
_Fig 5. Cart Page_

### **SOLUTION**
장바구니 페이지의 URL은 `https://shoe-shop-1.ctf.zone/index.php?page=cart&id=2307` 이다.

index.php 파일에서 `page`와 `id` 인자를 받아서 화면을 동적으로 구성하고 있음을 알 수 있다.

여기서 수상하게 느낀 점은 다음 순서와 같다.
- 나는 cart를 처음 만들었다.
- 그런데 cart의 식별자 같은 id 값이 2307이다.
- 다른 사람과 cart를 공유한다?
- id에 대한 권한 관리가 제대로 이루어지지 않았다면 다른 사람의 cart에 접근이 가능하겠네?
- 재고가 없는 신발을 cart에서 볼 수 있겠네?

### **POC**

그래서 id를 임의로 조작하여 보니, 재고가 없는 신발을 장바구니 페이지에서 확인할 수 있었다. (플래그도 같이)

![result]
_Fig 6. Get Shoes & Flag!_


[chall]: /assets/CTF/WHY2025/shoeshop/challenge.png
[homepage]: /assets/CTF/WHY2025/shoeshop/recon_home.png
[login]: /assets/CTF/WHY2025/shoeshop/recon_login.png
[shop]: /assets/CTF/WHY2025/shoeshop/recon_shop.png
[cart]: /assets/CTF/WHY2025/shoeshop/recon_cart.png
[result]: /assets/CTF/WHY2025/shoeshop/poc_result.png