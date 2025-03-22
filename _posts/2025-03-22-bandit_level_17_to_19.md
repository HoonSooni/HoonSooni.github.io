---
title: "OverTheWire Bandit Level 17 ~ 19 문제 풀이"
date: 2025-03-22 00:00:00 +0800
categories: [Wargame, Bandit]
tags: [cyber security, writeup, overthewire, bandit, linux]
description: OverTheWire Bandit 레벨 17에서 19까지 문제 풀이
---

# 문제 리스트
## Level 17 -> 18
[https://overthewire.org/wargames/bandit/bandit18.html](https://overthewire.org/wargames/bandit/bandit18.html)
### 문제 설명
There are 2 files in the homedirectory: **passwords.old and passwords.new**. The password for the next level is in **passwords.new** and is the only line that has been changed between **passwords.old and passwords.new**

**NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19**
### 문제 풀이
`passwords.old`와 `passwords.new` 파일에 수많은 랜덤 문자열들이 들어있고 그 중 단 한 라인만 다른데 그게 바로 비밀번호라고 한다.<br />
말도안되게 긴 파일이 아니라서 정말로 라인라인마다 직접 확인을 해도 되겠지만 지금은 연습 문제를 풀이하고 있으니 이에 맞는 명령어를 찾아야한다.<br />
#### grep을 이용한 풀이
```bash
grep -Fxvf passwords.old passwords.new
```
`grep` 명령어에 다양한 옵션을 넘겨주어 해결할 수 있다.<br />

* `-F`: Regex 패턴이 아니라 순수 문자열의 매칭을 찾아달라는 의미 
*  `-x`: 라인 전체가 매칭되게 하는 옵션 (일부 문자열 매칭을 방지)
* `-f`: 패턴을 파일에서 가져오는 옵션 (라인 하나하나)
* `-v`: 반대로 매칭하는 옵션. 즉, 매칭되지 않는 문자열을 출력

여기서 주의해야 할 것은 passwords.old 파일이 왼쪽에 위치해야 passwords.new 파일에서의 비밀번호를 출력해낸다. <br />
passwords.old 파일의 한 줄 한 줄이 grep 명령어의 패턴으로 동작하기 때문이다.<br />
만일 `grep -Fxvf passwords.new passwords.old`처럼 파일 이름을 뒤바꾸면 passwords.old 파일의 비밀번호가 나오기 때문에 실제 비밀번호가 아닌 값을 얻게된다.
#### diff를 이용한 풀이
```bash
diff -a --suppress-common-lines -y passwords.new passwords.old
```
`diff`라는 명령어를 이용해서도 해결할 수 있다.<br />

`-a` 옵션은 인자로 넘어가는 파일들을 ASCII Text 파일로 다뤄달라는 요청이다. `--suppress-common-lines` 중복되는 라인들은 출력하지 않는 옵션과 `-y`를 통해서 어느 라인이 다른지 양 옆으로 보여주는 옵션도 추가해줬다.<br />

이렇게하면 왼쪽에는 새로운 비밀번호, 오른쪽에는 이 전의 비밀번호(passwords.old 파일에 있는 유일하게 다른 라인)가 출력된다.
## Level 18 -> 19
[https://overthewire.org/wargames/bandit/bandit19.html](https://overthewire.org/wargames/bandit/bandit19.html)
### 문제 설명
The password for the next level is stored in a file **readme** in the homedirectory. Unfortunately, someone has modified **.bashrc** to log you out when you log in with SSH.
### 문제 풀이
문제에 나와있듯이 bandit18에 접속하면 "Byebye !"라는 문자열이 출력되면서 서버가 꺼진다.<br />
.bashrc 파일이 변형돼서 그렇다는데 그렇다면 bandit17에 접속해서 해당 파일을 손을 봐야할 것 같다.
#### bandit17에 접근해 .bashrc 확인
```bash
bandit17@bandit:~$ ls -al
total 36
drwxr-xr-x  3 root     root     4096 Sep 19  2024 .
drwxr-xr-x 70 root     root     4096 Sep 19  2024 ..
-rw-r-----  1 bandit17 bandit17   33 Sep 19  2024 .bandit16.password
-rw-r--r--  1 root     root      220 Mar 31  2024 .bash_logout
-rw-r--r--  1 root     root     3771 Mar 31  2024 .bashrc
-rw-r-----  1 bandit18 bandit17 3300 Sep 19  2024 passwords.new
-rw-r-----  1 bandit18 bandit17 3300 Sep 19  2024 passwords.old
-rw-r--r--  1 root     root      807 Mar 31  2024 .profile
drwxr-xr-x  2 root     root     4096 Sep 19  2024 .ssh
```
`ls -al` 명령어로 파일 목록을 확인해보면 .bashrc가 존재한다.<br />

vim으로 .bashrc 파일에 들어가 확인해보니 bandit18이나 "Byebye !" 문자열에 대해서 찾아볼 수가 없다.<br />
그래서 홈 디렉토리 위의 /home 디렉토리로 빠져나가 /bandit18/.bashrc에 접근해봤는데 역시나 접근이 막혀있다.
#### .bashrc가 로드되는 것을 피하면서 ssh 접속하기
리눅스에서 해당 컴퓨터(서버)가 켜질 때 .bashrc가 로드된다. 리눅스는 .bashrc는 일종의 시작 프로그램인 셈이다.<br />
그래서 .bashrc가 실행될 때 로드되는 것을 방지하는 코드가 있는지 검색해보니 ssh의 `-t` 옵션을 알게됐다.<br />

```
-t      Force pseudo-terminal allocation.  This can be used to execute arbitrary screen-based programs on a remote machine, which can be very useful, e.g. when implementing menu services.  Multiple -t options force tty allocation, even if ssh has no local tty.
```
`man ssh` 페이지를 살펴보면 `-t` 옵션은 pseudo-terminal allocation을 강제한다고 한다. 여기서 pseudo-terminal은 TTY를 의미하는데 TTY는 원래 TeleTYpewriter을 의미하지만 현재는 단순하게 터미널 세션을 뜻한다.<br />
터미널 세션은 특별한 것이 없고 그냥 컴퓨터 명령어를 실행할 수 있는 텍스트 기반의 인터페이스이다. 즉, shell이 TTY이자 pseudo-terminal이라는 것이다.<br />

리눅스에는 다양한 shell이 존재한다.<br />
대표적으로 sh(유닉스 초기 쉘)와 bash(가장 많이 쓰이는 쉘), zsh(다양한 플러그인을 지원하는 쉘) 등이 있다.<br />

쉘의 종류는 아주 많고 리눅스의 대부분이 sh와 bash는 기본적으로 가지고있다. <br />
.bashrc 파일은 bash 쉘이 실행될 때 자동으로 실행되는 코드 파일이다.<br />
다시 말해서, bash가 아닌 sh 쉘을 실행하면 .bashrc가 실행되지 않는다는 것이다.<br />

```bash
ssh -t bandit18@bandit.labs.overthewire.org -p 2220 /bin/sh
```
그래서 이런식으로 `-t` 옵션을 이용하여 이번에 사용할 쉘을 sh로(기본적으로 "/bin/sh" 경로에 있음) 명시해주면 bash가 실행되는 대신 sh가 실행되면서 .bashrc의 실행을 막을 수 있는 것이다.<br />

접속이 완료되면 이제 `cat readme` 명령어로 비밀번호를 읽어내면 된다.
## Level 19 -> 20
[https://overthewire.org/wargames/bandit/bandit20.html](https://overthewire.org/wargames/bandit/bandit20.html)
### 문제 설명
To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.
### 문제 풀이
이번에는 setuid 바이너리 파일이 홈 디렉토리에 있는데 해당 파일을 이용해서 비밀번호를 알아내라고 한다.<br />
우선 바이너리 파일을 어떻게 사용하면 되는지 알아내기 위해서 아무런 인자 없이 실행하라고 한다.<br />
```bash
bandit19@bandit:~$ ls
bandit20-do

bandit19@bandit:~$ ./bandit20-do
Run a command as another user.
  Example: ./bandit20-do id
```
실행해보니까 id 값을 인자로 넘겨주라고 한다.<br />

```bash
bandit19@bandit:~$ ./bandit20-do bandit20
env: ‘bandit20’: Permission denied

bandit19@bandit:~$ ./bandit20-do bandit19
env: ‘bandit19’: Permission denied

bandit19@bandit:~$ ./bandit20-do bandit18
env: ‘bandit18’: Permission denied
```
테스트용으로 여러가지 id값을 전달해주니 자꾸만 권한이 없다고 한다.

```bash
bandit19@bandit:~$ ./bandit20-do id
uid=11019(bandit19) gid=11019(bandit19) euid=11020(bandit20) groups=11019(bandit19)
```
알고보니 인자로 임의의 유저 이름을 넣는 것이 아니라 진짜 순수 문자열 "id"를 넣는것이었다.<br />
`id`의 결과를 보면 현재 사용자와 그룹 아이디 전부 bandit19이지만 euid(실행 중인 프로세스의 유효 ID)가 bandit20이라는 것을 알 수 있다.<br />

```bash
bandit19@bandit:~$ id
uid=11019(bandit19) gid=11019(bandit19) groups=11019(bandit19)
```
이번엔 `bandit20-do` 파일을 사용하지 않고 그냥 `id`를 입력하니 euid가 보이지 않는다.<br />
여기서 우리는 bandit20-do 파일을 통해서 실행되는 명령어(프로세스)만 bandit20의 권한을 가진다는 것을 알 수 있다.<br />

```bash
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
```
그래서 이렇게 해당 파일의 인자로 `cat /etc/bandit_pass/bandit20`을 넘겨주어 비밀번호를 얻어내면 된다.