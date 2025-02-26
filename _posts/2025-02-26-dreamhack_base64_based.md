---
title: "드림핵 lv.1 - Base64 Based"
date: 2025-02-26 00:22:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking] 
description: 드림핵 Hack The Elon 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/1785](https://dreamhack.io/wargame/challenges/1785)
# 문제 설명
Read `flag.php` to get the flag.

---
# 문제 풀이
## 웹사이트 분석
사이트에 접속하면 다음과 같은 글이 뜬다.
```
File Content Viewer

No file parameter provided.
```
File Content Viewer이라는 서비스를 제공하는 듯 하다. 그 아래에 file parameter이 제공되지 않았다고 하니 아마 url로 파일 이름을 받아서 해당 파일의 내용을 출력해주는 듯 하다.<br />

`http://host1.dreamhack.games:22766/?file=asd`처럼 아무 파일 이름이나 던져보니까 Invalid file name이라는 경고 문자가 뜬다. <br />

이쯤 돼서 코드를 살펴보자.
## 코드 분석
**코드 구조**
```
DockerFile
deploy/
	- src/
		- flag.php
		- hello.php
		- index.php
	- flag
	- run.sh
```
해당 코드의 파일 구조를 보면 이런 식이다. 
### index.php
```php
if (isset($_GET['file'])) {
$encodedFileName = $_GET['file'];
	if (stripos($encodedFileName, "Li4v") !== false){
		echo "<p class='error'>Error: Not allowed ../.</p>";
		exit(0);
	}
	if ((stripos($encodedFileName, "ZmxhZ") !== false) || (stripos($encodedFileName, "aHA=") !== false)){
		echo "<p class='error'>Error: Not allowed flag.</p>";
		exit(0);
	}
	$decodedFileName = base64_decode($encodedFileName);

	$filePath = __DIR__ . DIRECTORY_SEPARATOR . $decodedFileName;

	if ($decodedFileName && file_exists($filePath) && strpos(realpath($filePath),__DIR__) == 0) {
		echo "<p>Including file: <strong>$decodedFileName</strong></p>";
		echo "<div>";
		require_once($decodedFileName);
		echo "</div>";
	} else {
		echo "<p class='error'>Error: Invalid file or file does not exist.</p>";
	}
} else {
	echo "<p class='error'>No file parameter provided.</p>";
}
```
쓸데없는 부분들은 지우고 중요한 부분만 가져왔다.<br />

예상대로 url에서 file이라는 이름을 가진 param을 받는다. 그런데 특이한 점이 있다면 param으로 받은 파일 이름을 한 번 base64_decode() 함수를 이용해서 디코딩을 해주고 있다.<br />
그렇다는 것은 우리가 넘겨줘야 할 문자열은 파일 주소의 base64로 인코딩 된 문자열이라는 의미다. 그래야 encoding된 것이 서버에서 decode되면서 올바른 파일을 읽어오는 동작을 할 수 있기 때문이다.<br />
## 최종 풀이
파일 구조를 보면 src 디렉토리 바깥에 flag 파일이 들어있는 것을 알 수 있다. "../flag"를 base64로 인코딩 된 문자열을 ?file= 부분에 넣어줘야 한다.

인코딩 된 문자열은 [https://www.base64encode.org/](https://www.base64encode.org/) 해당 웹사이트에서 얻을 수 있다. 물론 직접 코드를 짜서 실행해도 된다.

### 인코딩된 문자열
`../flag`를 인코딩 해보면 `Li4vZmxhZw==` 이러한 문자열이 나온다. <br />
하지만 해당 문자열을 file 이름으로 전달하면 에러가 발생하는 것을 알 수 있다.

```php
if (stripos($encodedFileName, "Li4v") !== false){
	echo "<p class='error'>Error: Not allowed ../.</p>";
	exit(0);
}
if ((stripos($encodedFileName, "ZmxhZ") !== false) || (stripos($encodedFileName, "aHA=") !== false)){
	echo "<p class='error'>Error: Not allowed flag.</p>";
	exit(0);
}
```
이렇게 코드상에서 "../"과 "flag"라는 문자열일 필터링 하고있다. 그래서 "../flag"로는 읽어올 수 없다.

처음에는 이걸 어떻게 우회할 수 있을까 고민해봤는데 파일 구조를 다시 보니 고민할 필요가 없었다. <br />
#### flag.php의 역할
```php
<?php 
if (!defined('ALLOW_INCLUDE')) {
    http_response_code(403);
    exit('Direct access is not allowed.');
} else {
    $file = file_get_contents('/flag');
    echo trim($file); 
}
?>
```
코드를 보면 url + /flag 이런식으로의 직접적인 접근은 불가능하다고 적혀있다. <br />
그 대신 직접적으로 접근하지만 않는다면 친절하게도 flag 값을 읽어서 화면에 echo 해준다. 

결론적으로 우리는 "../flag"로 접근하는 것을 포기하고 "./flag.php"로 접근해야 한다. 이를 인코딩하면 `Li9mbGFnLnBocA==`가 결과로 나오고 이것을 살펴보면 필터링에 걸리지 않는다는 것이 보인다.

```
File Content Viewer

Including file: ./flag.php

DH{REDACTED}
```
그렇게 플래그를 얻어 문제를 해결할 수 있다.
# 배운 것
Base64 인코딩 디코딩을 해주는 웹사이트와 php에서 특정 php 파일의 직접적인 접근을 막는 `defined('ALLOW_INCLUDE')` 함수도 알게되는 계기가 됐다.