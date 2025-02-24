---
title: "드림핵 Beginner - phpreg"
date: 2025-02-23 00:11:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, regex] 
description: 드림핵 phpreg 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/873](https://dreamhack.io/wargame/challenges/873)
# 문제 설명

php로 작성된 페이지입니다.

알맞은 Nickname과 Password를 입력하면 Step 2로 넘어갈 수 있습니다.
Step 2에서 `system()` 함수를 이용하여 플래그를 획득하세요.
플래그는 `../dream/flag.txt`에 위치합니다.

플래그의 형식은 DH{...} 입니다.

---
# 문제 풀이
## 웹사이트 분석
### main page (step1)
![phpreg main](https://1drv.ms/i/c/5cb37aa515b56a00/IQQTDbQokrQ7Rb5WUNX3V0wZAVLuqYZssnQxaKl73mYeMJA?width=660)
<br />
가장 먼저 볼 수 있는 화면이다. Nickname과 Password를 user input으로 받는다. <br />
문제 설명을 읽어보면 올바른 값을 입력해야 다음(step2)으로 넘어갈 수 있다고 한다.<br />
우선 아무거나 입력해볼 수 있겠지만 해당 문제의 제목이 phpreg인걸 보면 정규식 문제일것이 분명하니 우선 코드먼저 봐야 할 것 같다.
## 코드 분석
### index.php
```html
<form method="post" action="/step2.php">
    <input type="text" placeholder="Nickname" name="input1">
    <input type="text" placeholder="Password" name="input2">
    <input type="submit" value="제출">
</form>
```
form을 확인해보면 메인 페이지에서 닉네임과 비밀번호가 제출하는 순간에 step2.php로 값이 전달되는 걸 볼 수 있다
### step2.php
```php
<?php
  // POST request
  if ($_SERVER["REQUEST_METHOD"] == "POST") {
	$input_name = $_POST["input1"] ? $_POST["input1"] : "";
	$input_pw = $_POST["input2"] ? $_POST["input2"] : "";

	// pw filtering
	if (preg_match("/[a-zA-Z]/", $input_pw)) {
	  echo "alphabet in the pw :(";
	}
	else{
	  $name = preg_replace("/nyang/i", "", $input_name);
	  $pw = preg_replace("/\d*\@\d{2,3}(31)+[^0-8\"]\!/", "d4y0r50ng", $input_pw);
	  
	  if ($name === "dnyang0310" && $pw === "d4y0r50ng+1+13") {
		echo '<h4>Step 2 : Almost done...</h4><div class="door_box"><div class="door_black"></div><div class="door"><div class="door_cir"></div></div></div>';

		$cmd = $_POST["cmd"] ? $_POST["cmd"] : "";

		if ($cmd === "") {
		  echo '
                <p><form method="post" action="/step2.php">
                    <input type="hidden" name="input1" value="'.$input_name.'">
                    <input type="hidden" name="input2" value="'.$input_pw.'">
                    <input type="text" placeholder="Command" name="cmd">
                    <input type="submit" value="제출"><br/><br/>
                </form></p>
		  ';
		}
		// cmd filtering
		else if (preg_match("/flag/i", $cmd)) {
		  echo "<pre>Error!</pre>";
		}
		else{
		  echo "<pre>--Output--\n";
		  system($cmd);
		  echo "</pre>";
		}
	  }
	  else{
		echo "Wrong nickname or pw";
	  }
	}
  }
  // GET request
  else{
	echo "Not GET request";
  }
?>
```
코드가 꽤나 긴데 핵심적인 부분만 살펴보면 된다.

해당 코드가 하는 일을 크게보면 다음과 같다.
1. `preg_match("/[a-zA-Z]/", $input_pw)`로  비밀번호에 알파벳이 담겨있는지 체크하고 그렇다면 잘못됨을 알린다.
2. `if ($name === "dnyang0310" && $pw === "d4y0r50ng+1+13")`를 체크하고 이 조건식이 만족될 때 그 다음 코드로 넘어간다. 만족하지 않는다면 "Wrong nickname or pw"를 출력한다. 즉, 닉네임과 비밀번호를 알아낸 셈이다.
3. 다음으로 `$_POST["cmd"]`를 통해서 `system()` 함수에 전달할 명령어를 받는다. 문제를 보면 플래그는 `../dream/flag.txt`에 있다고 한다. 

#### 진짜 닉네임 알아내기
```php
$name = preg_replace("/nyang/i", "", $input_name);
```
닉네임이 `dnyang0310`이 돼야하지만, 이를 직접적으로 입력해주면 위의 코드에 의해 "nyang"이 ""로 대체돼 문제가 결국 "d0310"이 돼버린다. <br />

그렇기 때문에 "dnyangnyang0310"을 해주면 될 것 같은데 작동하지 않는다. "/nyang/i"의 의미는 문자열에서 nyang을 찾는다는 의미이고 i는 대소문자를 상관하지 않는다는 의미이다.<br />

보통의 정규식에서는 해당 매치를 딱 한 번만 찾아내기 때문에 "dnyangnyang0310" 이렇게 하면 첫 번째로 등장하는 nyang이 제거되고 "dnyang0310"만 남을테지만 php의 preg_replace() 함수는 특이하게도 모든 매치를 찾아내기 때문에 "dny*nyang*ang0310" 이런식으로 중간에 끼워주어야 nyang이 ""로 대체되면서 최종 결과로 올바른 닉네임이 나오게 된다.

즉, 최종 닉네임은 `dnynyangang0310`이다.
#### 진짜 비밀번호 알아내기
비밀번호가 `d4y0r50ng+1+13`라는 것을 알아냈다. 하지만 이상한 점이 있다. <br />
분명 이 전에 조건식을 보면 비밀번호에는 알파벳이 존재하면 안된다. 

```php
`$pw = preg_replace("/\d*\@\d{2,3}(31)+[^0-8\"]\!/", "d4y0r50ng", $input_pw);`
```
그래서 php 코드 안에 있는 이 부분을 이용해야 한다.<br />
`preg_replace()`는 첫번째 인자로 들어가는 정규식을 만족하면 세번째 인자로 전달한 문자열을 두번째 인자인 d4y0r50ng로 변환해주는 함수이다.<br />

 `\d*\@\d{2,3}(31)+[^0-8\"]\!` 분석하기
1. `\d*`에서 d는 숫자(0 ~ 9)를 의미하고 `*`는 0개 이상 반복을 의미한다. 즉, 숫자가 있어도 되고 없어도 된다.
2. `\@\d{2, 3}`는 숫자(혹은 아무것도 없는 체로) 뒤에 @와 2 ~ 3자리의 숫자가 와야 한다는 의미이다.
3. `(31)+`은 "31"이 1번 이상 반복돼야 한다는 의미이다.
4. `[^0-8\"]\!` ^는 정규식에서 부정을 의미한다. 그래서 "31" 다음으로 올 수 있는 문자는 숫자 0 ~ 8 혹은 `"` 이 아닌 문자여야 하고 느낌표(!)로 끝나야 한다.

이를 모두 만족하는 문자열 중 하나는 `@13319!`이다. <br />
하지만 `$pw === "d4y0r50ng+1+13"` 비밀번호 판별 조건식을 보면 뒤에 "+1+13"가 붙기 때문에 이를 더한 값인 `@13319!+1+13`이 최종 비밀번호가 된다.
## 최종 풀이
```
Nickname: dnynyangang0310
Password: @13319!+1+13
```
해당 값을 입력하면 step2 페이지로 넘어가면서 "Almost done...."이라는 메세지가 출력된다.

문제에서 요구하는 것은 system() 함수를 이용하여 "../dream/flag.txt"의 값을 읽어오는 것이다.

```php
$cmd = $_POST["cmd"] ? $_POST["cmd"] : "";

if ($cmd === "") {
	echo '
		<p><form method="post" action="/step2.php">
			<input type="hidden" name="input1" value="'.$input_name.'">
			<input type="hidden" name="input2" value="'.$input_pw.'">
			<input type="text" placeholder="Command" name="cmd">
			<input type="submit" value="제출"><br/><br/>
		</form></p>
  ';
}
// cmd filtering
else if (preg_match("/flag/i", $cmd)) {
    echo "<pre>Error!</pre>";
}
else{
	echo "<pre>--Output--\n";
	system($cmd);
	echo "</pre>";
}
```
닉네임과 비밀번호를 알맞게 입력했을 때 실행되는 코드이다.<br />
여기서 `$_POST`에서 "cmd"값을 읽어와 이것을 `system()` 함수의 인자로 넘겨주면서 shell 명령어를 실행할 수 있는 방식이다.

여기서 주의해야 할 점은, `preg_match("/flag/i")` 함수 때문에 만약 cmd로 `cat ../dream/flag.txt` 같은 명령어를 입력하면 해당 필터링에 걸리게 돼 값을 읽어올 수가 없다.

여기서도 정규식을 이용해야 한다. 아마 대충 `cat ../dream/*.txt` 정도로 대체하면 해결 될 것 같다. <br />
해당 명령어를 command input에 넘겨주면 플래그를 얻을 수 있다.


```
--Output--
DH{Redacted}
```
---
# 배운 것
php의 `preg_replace()` 함수는 가장 첫 번째의 매치 만을 찾는 것이 아니라 모든 매치를 찾아낸다.