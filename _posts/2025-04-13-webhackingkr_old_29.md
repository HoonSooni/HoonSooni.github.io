---
title: "webhacking.kr old 29 문제 풀이"
date: 2025-04-13 00:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, file upload, sql injection]
description: webhacking.kr old 29 문제 풀이
---

[https://webhacking.kr/challenge/web-14/index.php](https://webhacking.kr/challenge/web-14/index.php)
# 문제 풀이
## 분석
문제 페이지에 접속해보니 "Flag is in another table"이라는 문구와 함께 파일을 업로드할 수 있는 form이 주어진다.<br />

그 아래에는 HTML Table이 존재하는데 time, ip, file 컬럼이 있고 내가 파일을 업로드 할 때마다 값이 하나씩 들어가게 된다.<br />
언제 업로드했는지, ip는 무엇인지, 파일 이름이 무엇인지 출력된다.<br />
## 시도
### 아무 파일이나 업로드
내 컴퓨터 내에 있는 아무 python file을 업로드 해보니 테이블에 row 하나가 추가됐을 뿐 달리 특별한 현상은 일어나지 않았다.<br />
### 웹쉘 업로드
웹쉘을 업로드 해보았지만 URL 상으로 웹쉘에 접근할 수 있는 방법이 없어서 실패했다.
## 최종 풀이
여러가지 시도를 하던 중, "Flag is in another table"이라는 문구가 자꾸만 신경쓰였다.<br />
table이라는 것이 HTML 테이블을 의미하는 것인지 아니면 데이터베이스의 테이블을 의미하는 것인지 헷갈렸는데 아무리 생각해도 HTML 테이블은 아닌 것 같아 SQLI 취약점으로 해당 문제에 접근해야 한다는 느낌이 들었다.<br />

우리가 쿼리를 심을 수 있는 유일한 방법은 파일을 업로드하는 것인데, 파일 내부의 내용을 가져가서 쿼리를 작성하는지 아니면 파일 이름을 가지고 하는지 모르기 때문에 우선 시도해보기 간단한 파일 이름을 이용한 SQL 인젝션에 도전했다.<br />
### 파일 이름으로 SQL 인젝션
우리가 파일을 업로드할 때 파일 이름, ip 주소, 업로드 시간이 뜬다.<br />
이때 우리는 아마 이 정보들이 데이터베이스에 저장될 것이라는 예상이 가능하다.<br />

```sql
INSERT INTO tablename VALUES (column1_value, column2_value, column3_value)
```
데이터베이스에 정보를 저장할 때에는 보통 INSERT 구문을 이용한다.<br />

```
Content-Disposition: form-data; name="upfile"; filename="random_file_name.html"
```
우리가 파일을 업로드할 때(POST 요청을 보낼 때) 개발자 도구의 네트워크 탭을 통해서 어떤 데이터가 전송되는지 확인이 가능하다.<br /> 
해당 페이지에서는 단순하게 파일 이름만 전달되는 것이 확인된다.<br />

아마 서버에서는 해당 filename 값을 얻어와서 `VALUES ($_POST["filename"])`와 같은 방식으로 쿼리에서 유저 인풋을 사용할 것으로 예상된다.<br />
#### 컬럼 순서
SQL에서 INSERT를 할 때에 우리가 알아야하는 것은 컬럼의 순서이다.<br />
지금 우리가 값을 넣고있는 테이블은 컬럼이 각각 time, ip, filename으로 총 3개로 구성됐을 것이다.<br />
이때 괄호에 들어가는 값들이 순서대로 들어가는데 이때 순서가 맞지 않으면 각 컬럼마다 데이터 형식이 다르기 때문에 제대로된 값이 들어가지 않을 수 있다.<br /> 

`filename', 'my_ip_address', 0); --`로 파일 이름을 설정하고 우선 업로드를 시도해보면 "upload error!"이라는 문구와 함께 작동하지 않는다.<br />
세 컬럼의 순서를 요리조리 바꿔가면서 시도해보면 filename, time, ip 순서로 값을 입력해야 작동된다는 것을 확인할 수 있다.<br />

이때 추가적으로 알아야할 사실은 ip 주소를 내 맘대로 임의의 값으로 설정하면 제대로 값이 테이블에 들어가지 않는다.<br />
아마 실제 ip와 해당 값을 체크하는 어떠한 서버의 처리과정이 있는 것으로 예상된다.<br />
#### 데이터베이스 이름
```sql
(select database()), 0, 'ip_address'); -- 
```
이제는 데이터베이스의 이름을 알아낼 차례이다.<br />
하지만 이렇게 파일 이름을 설정하니 에러가 발생한다. 안되는 이유를 예상해보면 아마 `VALUES ('$_GET[filename]')` 이런식으로 값이 들어가는데 작은 따옴표를 탈출시키지 않아서 생기는 문제인 것 같다.<br />

```sql
filename'), (select database()), 0, 'ip_address'); -- 
```
이번에는 또 이렇게 작성해서 시도해보니 여전히 작동하지 않는다. 모종의 이유로 단순하게 파일 이름만 전송했을 때에 문제가 발생한다. <br />

```sql
temp', 0, 'ip_address'), (select database()), 0, 'ip_address'); -- 
```

```
1970-01-01 09:00:00|119.192.89.197|temp|
1970-01-01 09:00:00|119.192.89.197|chall29|
```
이번에는 임시 데이터를 하나 넣어주고 그 다음의 데이터를 우리가 원하는 값을 넣어주는 방식으로 변경했더니 드디어 데이터베이스의 이름을 알아냈다.<br />
#### 테이블 이름
```sql
temp', 0, 'ip_address'), (select table_name from information_schema.tables where table_schema='chall29'), 0, 'ip_address'); --
```
이렇게 파일 이름을 조작하여 테이블 이름을 얻어내려 시도해보았다.<br />
하지만 분명 쿼리 문법에 다 맞게 한 것 같은데 자꾸만 에러가 나서 `table_name` 부분을 `group_concat(table_name)`으로 변환했더니 이제서야 작동을 한다.<br />

아마 쿼리의 결과가 2개 이상 반환되면 문제가 발생하는 모양이다.<br />

테이블 이름으로 값이 2개가 나왔다. files와 flag_congratz.<br />
당연히 flag_congratz 내부에 플래그가 있을 것으로 예상되니 해당 테이블의 컬럼 이름을 알아낼 차례이다.
#### 컬럼 이름
```sql
temp', 0, 'ip_address'), (select column_name from information_schema.columns where table_name='flag_congratz'), 0, 'ip_address'); --
```
해당 쿼리를 작성하면 'flag'라는 컬럼 이름을 얻게 된다.
#### 컬럼 내부의 값
```sql
temp', 0, 'ip_address'), (select flag from flag_congratz), 0, 'ip_address'); --
```
이제 테이블과 컬럼 이름을 모두 알아냈으니 값을 읽어오기만 하면 플래그가 출력된다.