---
title: "OverTheWire Bandit Level 20 ~ 22 문제 풀이"
date: 2025-03-27 00:00:00 +0800
categories: [Wargame, Bandit]
tags: [cyber security, writeup, overthewire, bandit, linux]
description: OverTheWire Bandit 레벨 20에서 22까지 문제 풀이
---

# 문제 리스트
## Level 20 -> 21
[https://overthewire.org/wargames/bandit/bandit21.html](https://overthewire.org/wargames/bandit/bandit21.html)
### 문제 설명
There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

**NOTE:** Try connecting to your own network daemon to see if it works as you think
### 문제 풀이
이번 문제에서도 setuid binary 파일을 이용하는 문제이다.<br />
bandit20에 접속해서 `ls`를 입력해 해당 파일의 이름이 `suconnect`라는 것을 쉽게 알아낼 수 있다.<br />

```bash
bandit20@bandit:~$ ./suconnect
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
```
그 다음에 파일을 실행해보면 어떻게 사용해야하는지 설명서가 출력된다.<br />
포트 넘버를 인자로 넘겨주면 localhost의 해당 포트로 TCP 연결을 해주는 프로그램이다. 그리고 만일 올바른 비밀번호가 상대쪽에서 넘어오면 다음 비밀번호가 응답으로 전달될 거라는 조언도 해준다.<br />
#### 포트 열기
처음에는 이미 열려있는 포트가 있고 해당 포트를 찾아내어 접속하는 문제라고 생각했다.<br />
하지만 상대쪽에서 현재의 비밀번호를 보내주어야 다음 비밀번호가 나온다고 하는데 그렇다면 내가 직접 해야하는게 아닌가 라는 결론에 도달하였다.<br />

```bash
bandit20@bandit:~$ nc -lvp 12345
Listening on 0.0.0.0 12345
```
그래서 새로운 터미널 창을 열어서 똑같이 bandit20에 접속하고 `nc -lvp 12345` 명령어로 포트 localhost:12345 서버를 오픈했다.<br />
#### 포트 접속하기
```bash
bandit20@bandit:~$ ./suconnect 12345
```
서버를 하나 오픈했으니 이제 오픈한 터미널 말고 다른 터미널에서 suconnect 파일을 이용해 접속해주면 된다.<br />
#### 포트 연 쪽에서 현재 비밀번호 보내기
```bash
Connection received on localhost 58394
```
접속이 완료됐으면 포트를 연 쪽에서 localhost 포트 58394에서 연결을 했다는 메세지가 출력된다.<br />

현재 bandit20의 비밀번호를 그대로 터미널을 통해 입력하면 자동으로 58394 측에서 bandit21의 비밀번호를 응답해주면서 비밀번호를 알아낼 수 있다.
## Level 21 -> 22
[https://overthewire.org/wargames/bandit/bandit22.html](https://overthewire.org/wargames/bandit/bandit22.html)
### 문제 설명
A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.
### 문제 풀이
이번에는 cron이라는 개념에 대해서 배울 수 있는 문제이다.<br />
cron은 특정 기간을 설정하여 주기적으로 어떠한 코드를 자동으로 실행해주는 프로그램이라고 한다. 그리고 `/etc/cron.d/` 디렉토리를 살펴보라고 권하고있다.<br />
#### /etc/cron.d/ 살펴보기
```bash
bandit21@bandit:~$ cd /etc/cron.d
bandit21@bandit:/etc/cron.d$ ls
cronjob_bandit22  cronjob_bandit23  cronjob_bandit24  e2scrub_all  otw-tmp-dir  sysstat
```
해당 디렉토리로 이동하여 파일 리스트를 출력해보았다.<br />
이중에서 우리가 필요한 정보를 얻어내야한다. 우리가 필요한 정보는 당연하게도 bandit22로 넘어갈 때 쓰이는 비밀번호이다.<br />

```bash
bandit21@bandit:/etc/cron.d$ cat cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```
bandit22와 관련된 파일 이름인 `cronjob_bandit22`를 출력해보면 이렇다.<br />
bandit22 유저의 권한으로 실행되는 명령어가 보인다.<br />
`@reboot`는 아마도 bandit22로 서버에 접속했을 때 실행되는 코드인 듯 하고 그 아래에 있는 `* * * * *`  부분은 cron format에 의하면 매 분마다 실행되는 코드를 의미한다고 한다.<br />

즉, `cronjob_bandit22` 파일은 bandit22 서버가 실행될 때 그리고 매 분마다 `/usr/bin/cronjob_bandit22.sh &> /dev/null`라는 명령어를 실행한다는 것이다.<br />

참고로 위의 코드는 `/usr/bin/cronjob_bandit22.sh`라는 쉘 파일을 실행하고 결과를 `/dev/null` 디렉토리로 옮긴다. /dev/null은 리눅스에서 블랙홀 파일으로 불리우고 무언가를 지울 때 사용하는 파일이다.
#### 비밀번호 알아내기
```bash
bandit21@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv

bandit21@bandit:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
BANDIT22_PASSWORD
```
cronjob_bandit22.sh이 어떤 동작을 하는지 확인해보면 `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv`라는 임시 파일의 권한을 644로 부여하고 `/etc/bandit_pass/bandit22` 파일의 내용을 임시 파일에 넣고있다.<br />

즉, 임시 파일 안에는 bandit22의 비밀번호가 담겨있고 임시 파일의 권한이 cron에 의해 매번 접근 가능하도록 허용되기 때문에 해당 임시 파일을 읽어내면 비밀번호를 알아낼 수 있다.
## Level 22 -> 23
[https://overthewire.org/wargames/bandit/bandit23.html](https://overthewire.org/wargames/bandit/bandit23.html)
### 문제 설명
A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

**NOTE:** Looking at shell scripts written by other people is a very useful skill. The script for this level is intentionally made easy to read. If you are having problems understanding what it does, try executing it to see the debug information it prints.
### 문제 풀이
이번에도 비슷한 유형의 문제이다.<br />
#### /etc/cron.d
```bash
bandit22@bandit:~$ cd /etc/cron.d

bandit22@bandit:/etc/cron.d$ ls
cronjob_bandit22  cronjob_bandit23  cronjob_bandit24  e2scrub_all  otw-tmp-dir  sysstat

bandit22@bandit:/etc/cron.d$ cat cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
```
또 다시 `/etc/cron.d/` 디렉토리에 접근해서 cronjob_bandit23을 읽어보았다.<br />
해당 파일 내부에는 이 전과 동일하게 cronjob_bandit23.sh 라는 파일을 실행하여 결과를 지우는 행동을 한다.<br />
#### /usr/bin/cronjob_bandit23.sh
```bash
bandit22@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```
`/usr/bin/cronjob_bandit23.sh` 파일을 읽어보면 위와 같은 코드가 나온다. 해당 코드는 `$(whoami)`값을 읽어내어(이 상황에서는 bandit22이다) 해당 유저의 비밀번호를 임시 파일에 저장하는 행위를 한다.<br />

여기서 주목해야할 점은 임시 파일의 이름 생성 부분이다.<br />
`mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)` 이렇게 초기화되는 변수 `mytarget`이 바로 /tmp 디렉토리에 생기는 임시 파일의 이름이 된다.<br />
결국 `/tmp/mytarget` 형식으로 생성이 된다는 것이다.<br />

우리는 현재 bandit22로 접속해있기 때문에 변수 값은 항상 같을 수밖에 없다.<br />
우리가 알아내야하는 비밀번호는 bandit23 유저의 것이니 whoami 값을 "bandit23"으로 바꿔치면 된다.
#### 임시 파일 이름
```bash
bandit22@bandit:/etc/cron.d$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349

bandit22@bandit:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
BANDIT23_PASSWORD
```
변수 `mytarget`이 생성되는 과정을 "bandit23" 문자열을 이용하여 똑같이 진행하니 어떠한 랜덤 문자열이 등장했다.<br />
해당 값을 임시 파일 이름이라고 가정하고 출력을 해보니 예상대로 bandit23의 비밀번호가 출력이 된다.