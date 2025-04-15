---
title: "webhacking.kr old 30 문제 풀이"
date: 2025-04-15 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, database, environment configuration]
description: webhacking.kr old 30 문제 풀이
---

# 문제 풀이
## 분석
페이지에 접속하면 "Your personal directory is ./upload/WNICKBrFNwCe/"라고 친절히 파일이 업로드되는 디렉토리를 알려주고 그 아래에는 소스코드 페이지로 이동되는 a 태그까지 있다.
### 소스코드
```php
<?php
  if($_GET['view_source']) highlight_file(__FILE__);
  $db = mysqli_connect() or die();
  mysqli_select_db($db,"chall30") or die();
  $result = mysqli_fetch_array(mysqli_query($db,"select flag from chall30_answer")) or die();
  if($result[0]){
    include "/flag";
  }
?>
```
데이터베이스에서 플래그 값을 읽어오고 만약 결과값이 존재한다면 "/flag"라는 파일을 가져오는 것을 볼 수 있다.<br />
## 시도
### flag 파일 업로드
flag라는 이름을 가지는 파일을 하나 만들고 그 안에 `<?php echo $result; ?>` 코드를 집어넣었다.<br />
해당 파일을 업로드해서 "/upload/WNICKBrFNwCe/index.php"에 접속해보니 아무런 값도 뜨지 않는다.<br />

"/upload/WNICKBrFNwCe/flag.php"로 제대로 업로드가 됐는지 확인을 해보니까 WAF에 의해서 열린 홑화살괄호가('<') 필터링에 의해 지워진 것이 확인된다.<br />
### index.php 파일 업로드
index.php라는 파일을 하나 만들어서 내부에 원래 소스코드와 동일하게 하되 `echo $result;`를 중간에 추가해보았다.<br />
그러고 업로드를 해보니 "no hack"이라는 문구로 index.php 파일을 직접 업로드하는 것은 불가능하다고 알려준다.<br />
### flag 파일 변경 후 재업로드
이번에는 flag 파일에 php 코드를 입력하는 대신 "something"이라는 아무 문자열을 넘겨주었다.<br />
이제는 이 문자열을 "/upload/WNICKBrFNwCe/"에서 확인이 가능해야하는데 여전히 보이지 않는다. HTML의 주석에도 보이지 않는다.<br />

그렇다는 것은 해당 쿼리가 제대로 작동하지 않는다는 추론을 할 수 있다.<br />
## 최종 풀이
여러가지 방법들을 시도하는 과정에서 쿼리가 제대로 작동하지 않을 수 있다는 결론이 섰다.<br />
왜 작동을 제대로 안하는지 알아봐야할 차례이다.<br />
### mysqli_connect() 함수의 문제
아무리 생각해봐도 모르겠어서 다른 풀이들을 보며 힌트를 얻었다.<br />
소스코드의 `$db = mysqli_connect() or die();` 데이터베이스 연결 부분이 잘못됐기 때문이었다.<br />
`mysqli_connect()` 함수는 여러 인자들을 입력받는다. 데이터베이스에 연결을 시도하는데 다양한 값을 입력받는 것이 당연한데 아무 값도 전달되지 않는 점을 의심하지 않은 점이 나의 불찰이다.<br />

어찌됐든 우리에게는 index.php에 변화를 줄 수 있는 방법이 없다. 원래라면 해당 파일을 조작하여 제대로된 값들을 인자로 넣어주어 해결할 수 있을 것 같은데 그것이 안된다는 뜻이다.<br />

[mysqli_connect 공식 문서](https://www.php.net/manual/en/function.mysqli-connect.php)를 살펴보면 connect 함수는 `mysqli::construct()`와 같다고 하고 이 생성자는 아래와 같이 선언돼 있다.<br />
```php
public mysqli::connect(
    ?string $hostname = null,
    ?string $username = null,
    #[\SensitiveParameter] ?string $password = null,
    ?string $database = null,
    ?int $port = null,
    ?string $socket = null
): bool
```
총 6개의 인자를 받는데 기본값으로 null이 들어가게끔 돼있다.<br />
그리고 문서의 아래쪽에는 다음과 같은 각 인자들의 추가 정보가 적혀있다.<br />

- hostname이 null이라면 php.ini 파일의 `mysqli.default_host`값으로 대체한다.
- username이 null이라면 php.ini 파일의 `mysqli.default_user`값으로 대체한다.
- password가 null이라면 php.ini 파일의 `mysqli.default_pw`값으로 대체한다.
- port가 null이라면 php.ini 파일의 `mysqli.default_port`값으로 대체한다.
- socket가 null이라면 php.ini 파일의 `mysqli.default_socket`값으로 대체한다.

아마 서버의 php.ini 파일 내에 제대로된 정보가 설정돼있지 않아서 쿼리가 값을 받아오지 못하므로 플래그를 읽어오지 못하는 것으로 예상된다.<br />
### .htaccess 설정 파일
우리가 업로드하는 파일은 /upload/WNICKBrFNwCe 디렉토리로 업로드된다. 이 경로를 벗어나게 하는 방법은 없는 것 같다.<br />
즉, 우리에게는 php.ini를 루트 디렉토리로 업로드 할 수 있는 방법이 딱히 없다.<br />

```
php_value mysqli.default_host "HOST"
php_value mysqli.default_user "USER"
php_value mysqli.default_pw "PASSWORD"
```
하지만 위와 같은 방식으로 각 디렉토리의 설정을 담당하는 .htaccess 파일을 이용해 php.ini의 설정값을 건드릴 수 있다고 한다.<br />
### SQL 서버 구축
이제 우리가 해야할 것은 직접 SQL 서버를 구축하여 문제 페이지에서 내가 구축한 서버에 접근하도록 해야한다.<br />
접근하여 `select flag from chall30_answer` 쿼리가 작동할 수 있도록 유도하면 되는 것이다. 왜냐하면 php 코드를 다시 살펴보면 우리는 쿼리의 결과 어떻든 상관없이 어떠한 데이터가 반환되게만 하면 되는 것이기 때문이다.

```sql
mysql> create database chall30;
Query OK, 1 row affected (0.00 sec)

mysql> use chall30;
Database changed

mysql> create table chall30_answer( flag int );
Query OK, 0 rows affected (0.04 sec)

mysql> insert into chall30_answer values (1);
Query OK, 1 row affected (0.04 sec)

mysql> select flag from chall30_answer;
+------+
| flag |
+------+
|    1 |
+------+
1 row in set (0.02 sec)
```
위 쿼리들을 수행하여 chall30이라는 데이터베이스를 만들고 그 안에 chall30_answer이라는 이름을 가진 테이블을 하나 만들어주었다. 그리고 그 안에 아무 flag 값을 하나 넣어준 상태이다.<br />
#### 다른 컴퓨터에서 접근할 수 있도록 설정
##### my.cnf
이제 이 SQL 서버를 외부에서 접근할 수 있도록 설정해야한다.<br />

```
bind-address = 0.0.0.0
mysqlx-bind-address = 0.0.0.0
```
본인 컴퓨터 혹은 가상 컴퓨터에서 mysql 설정 파일인 `my.cnf`파일에 접근하여 위에 명시된 두 값을 로컬호스트에서 모든 주소를 허용한다는 의미의 `0.0.0.0`으로 변환해주었다.<br />

```
❯ brew services restart mysql

❯ netstat -ant | grep 3306
tcp4       0      0  *.3306                 *.*                    LISTEN
tcp4       0      0  *.33060                *.*                    LISTEN
```
mysql 서버를 재실행해주면 해당 설정이 적용이 된다.<br />
`netstat` 명령어를 이용해서 지금 내 컴퓨터에 3306 포트가 어떤식으로 열려있는지 확인해주면 모든 주소를 제대로 허용하고 있는 것이 보인다.<br />
##### 새로운 유저 생성
```sql
mysql> create user 'test'@'%' identified by 'testpw123';
Query OK, 0 rows affected (0.09 sec)

mysql> grant all privileges on *.* to 'test'@'%' with grant option;
Query OK, 0 rows affected (0.04 sec)

mysql> flush privileges;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
##### 맥북의 방어벽 해제
```
❯ sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/local/mysql/bin/mysqld

❯ sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /usr/local/mysql/bin/mysqld
```
맥북을 사용하고 있는 경우라면 해당 방어벽을 꺼야 외부에서의 접근을 허용할 수 있다.
##### 포트포워딩
모든 설정을 다 해주고 심지어 내가 사용하고 있는 인터넷 라우터에 접근해 포트포워딩까지 변경해주었다.<br />
즉, 내 컴퓨터를 다른 서버들과 같이 정말로 외부에서 누구나 접근할 수 있도록 설정한 셈이다.<br />
#### mysql_native_password
이렇게 모든 작업이 다 끝났음에도 불구하고 .htaccess를 조작하여 시도해보아도 딱히 작동하는 것이 보이지 않는다.<br />

몇가지를 더 알아보니 mysql의 버전 업그레이드에 따라 비밀번호 인증에 사용되는 플러그인에 변화가 생긴 것에서 문제가 발생했을지도 모른다는 사실을 알게됐다.<br />

mysql은 유저의 비밀번호를 처리할 때 원래는 mysql_native_password라는 플러그인을 사용했다.<br />
하지만 이제는 보안 상의 이유로 mysql_sha2_password 플러그인을 사용하는데 webhackingkr이 오래된 워게임 사이트인 만큼 mysql과 php의 버전이 다른 탓에 원래의 플러그인을 이용해야만 접근이 가능하다.<br />

```sql
mysql> ALTER USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'testpw123';
ERROR 1524 (HY000): Plugin 'mysql_native_password' is not loaded
```
하지만 mysql 8.0 버전 이후로는 변경하는 것 자체가 막혀있다.<br />
알아본 결과 원래라면 플러그인 변경이 가능했는데 이제는 완전히 막혀있다는 모양이다.<br />
### 다른 환경으로 다시 서버 구축
내 컴퓨터로 환경 설정 하다가 화병 날 것 같아서 무료 호스팅 서비스를 찾아서 다시 해보기로 했다.<br />
한국인들을 대상으로 제공하는 호스팅 서비스인 [cloudtype](https://cloudtype.io/ko/home)이다.<br />
#### cloudtype MariaDB 서버
![cloudtype_mariadb](https://1drv.ms/i/c/5cb37aa515b56a00/IQT0ZivJrBrgSYlOo_VWfMGWAabOuqvIKJveF9xhSS15Bks?width=660)<br />
일단 우리가 필요한 것은 mysql 서버라서 새로운 프로젝트를 열어 mysql을 검색해봤다.<br />
그래보니 mysql이 아니라 mariadb만 지원하는 것으로 확인된다. 두 데이터베이스는 사실상 같은 거라고 봐도 무방하다.<br />

하지만 우리가 봉착한 문제가 mysql_native_password 버전 문제였기 때문에 mariadb에서도 동일한지 검색을 해보았다.<br /> 
사람들 말로는 mariadb에서는 sha2 방식을 사용하지 않고 여전히 native를 쓴다고 하니 일단 최신 버전의 mariadb로 서버를 만들어보았다.<br />
#### DB 구축
```sql
MariaDB [mysql]> select user, plugin from user;
+-------------+-----------------------+
| User        | plugin                |
+-------------+-----------------------+
| mariadb.sys | mysql_native_password |
| root        | mysql_native_password |
| root        | mysql_native_password |
| healthcheck | mysql_native_password |
| healthcheck | mysql_native_password |
| healthcheck | mysql_native_password |
| test        | mysql_native_password |
+-------------+-----------------------+
```
위와 동일한 방법으로 데이터베이스, 테이블, 가짜 플래그 값, 새로운 유저 생성을 마치니 위와 같은 결과가 출력된다.<br />
맨 아래에 test라는 유저가 있고 플러그인을 보면 우리가 원하는데로 mysql_native_password로 설정돼있다.<br />
#### 연결
![cloudtype_connection](https://1drv.ms/i/c/5cb37aa515b56a00/IQQYTW91nTbxSJNvA-X_vfrKAYExKOHAJCjl6uC1E2tAaEQ?width=660)<br />
이제 cloudtype의 서버 정보란에서 외부에서 접근할 수 있는 주소를 복사하여 .htaccess 파일의 host 부분에 넣어주고 문제 파일에 업로드 한 후 /upload/randomstring/ 에 접속하면 플래그가 출력되는 것을 확인할 수 있다.