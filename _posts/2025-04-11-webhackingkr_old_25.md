---
title: "webhacking.kr old 25 문제 풀이"
date: 2025-04-11 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, php] 
description: webhacking.kr old 25 문제 풀이
---

# 문제 풀이
```
total 20
drwxr-xr-x 2 root root 4096 Aug 24  2019 .
drwxr-xr-x 3 root root 4096 Aug 24  2019 ..
-rw-r--r-- 1 root root   82 Aug 24  2019 flag.php
-rw-r--r-- 1 root root   31 Aug 24  2019 hello.php
-rw-r--r-- 1 root root  605 Aug 24  2019 index.php
```
웹사이트에 접속하면 서버의 루트 디렉토리의 파일 목록들 보여주고 그 아래에는 커다란 textarea가 존재한다.<br />
그리고 URL을 살펴보면 file이라는 파라미터에 기본으로 "hello"라는 값이 주어져있고 textarea에는 "hello world"가 적혀있다.<br />

파일 목록 중 hello.php가 있는 것을 보면 아마 file로 넘어간 값의 이름을 가지는 php 파일을 textarea에 보여주는 식으로 작동하는 것 같다.<br />

?file=flag로 접근해보면 "FLAG is in the code"라는 문구만 나온다.<br />
역시나 php 코드 자체는 볼 수 없고 해당 php 파일의 실행 결과만 볼 수 있는 것 같다.<br />
## 최종 풀이
우리의 목표는 flag.php의 코드를 알아내는 것이다.<br />
하지만 일반적으로 알려져있기로 php 파일의 코드 내용을 서버의 컴퓨터가 아닌 유저의 입장에서는 읽을 수 없다.<br />

하지만 지금 문제는 php 코드의 결과를 textarea를 통해 볼 수 있기 때문에 이럴 때는 php wrapper 개념을 이용하면 된다.<br />
### PHP Wrapper
[PHP Wrapper 공식문서](https://www.php.net/manual/en/wrappers.php.php)를 참고하면 php:// 접두사를 이용해서 php의 다양한 I/O 스트림에 접근할 수 있다고 한다.<br />

그 중에서도 특히 filter 부분을 자세히 보면 특정 스트림(여기에선 파일)을 필터에 통과시켜 읽어낼 수 있는 방법이 있다고 한다.<br />

즉, `php://filter/read={filter}/resource={filename}` 형식으로 특정 php 파일의 내용을 가져오는 것이 가능하다는 의미이다.<br />
중간의 filter 부분에는 php의 다양한 필터들이 들어갈 수 있는데 대표적으로 String Filters와 Conversion Filters가 있다.<br />
String에는 toUpper, toLower 혹은 rot13같은 것들 그리고 Conversion에는 base64 같은 녀석들이 있다.<br />

이들을 사용하는 방법은 string.toUpper, string.rot13, convert.base64-encode처럼 사용하면 되기에 최종적인 페이로드는 다음과 같이 될 것이다.<br />
```
?file=php://filter/read=convert.base64-encode/resource=flag

?file=php://filter/read=string.rot13/resource=flag
```

두 페이로드 중 하나만 사용하면 되는데 인코딩 방식에 따라서 적절하게 복호화를 진행하면 아래와 같은 코드가 나온다.<br />
```php
<?php
  echo "FLAG is in the code";
  $flag = "REDACTED";
?>
```
이렇게 꽤나 쉽게 플래그 값을 읽어내는 것에 성공할 수 있었다.<br />

이제 이 값을 제출을 하면 되는데 어디다 제출을 해야하나 막막하던 와중 webhackingkr 페이지 nav 바 중 Auth라는 버튼을 누르면 플래그 제출하는 창이 뜨는 것이 떠올랐다.<br />
이 플래그 창은 어디다 쓰는건지 궁금했는데 이럴 때 사용하는 것이었다.<br />
# 배운 것 
PHP Wrapper이라는 것의 개념을 배울 수 있었다.