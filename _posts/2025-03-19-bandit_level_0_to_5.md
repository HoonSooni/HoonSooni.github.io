---
title: "OverTheWire Bandit Level 0 ~ 5 문제 풀이"
date: 2025-03-19 00:00:00 +0800
categories: [Wargame, Bandit]
tags: [cyber security, writeup, overthewire, bandit, linux]
description: OverTheWire Bandit 레벨 0에서 5까지 문제 풀이
---

# 문제 풀이
## Level 0
[https://overthewire.org/wargames/bandit/bandit0.html](https://overthewire.org/wargames/bandit/bandit0.html)
### 문제 설명
The goal of this level is for you to log into the game using SSH. The host to which you need to connect is **bandit.labs.overthewire.org**, on port 2220. The username is **bandit0** and the password is **bandit0**. Once logged in, go to the [Level 1](https://overthewire.org/wargames/bandit/bandit1.html) page to find out how to beat Level 1.<br />
### 문제 풀이
overthewire의 가장 첫 번째 문제이다.<br />
터미널로 SSH를 이용한 로그인을 하는 것이 목표이다. 주소는 "bandit.labs.verthewire.org"이고 포트는 2220, username과 password는 bandit0이라고 한다.<br />

터미널에서 `ssh bandit0@bandit.labs.overthewire.org -p 2220`라는 명령어를 입력하면 서버에 접근이 되고 비밀번호를 입력해주면 서버에 접속된다.<br />
`ssh` 명령어는 IP 주소 혹은 URL을 받아서 접속을 시도하는 명령어이고 주소 앞 '@' 문자 앞에 username을 명시할 수 있다. 그리고 `-p` 옵션을 통해서 포트 번호를 명시하는 것도 가능하다.
## Level 0 -> 1
[https://overthewire.org/wargames/bandit/bandit1.html](https://overthewire.org/wargames/bandit/bandit1.html)
### 문제 설명
The password for the next level is stored in a file called **readme** located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port 2220) to log into that level and continue the game.
### 문제 풀이
level 0에서 접속했던 서버 내의 홈 디렉토리에 보면 readme라는 파일이 있다고 한다.<br />
해당 파일의 내용을 읽어내어 다음 레벨로 넘어갈 수 있는 비밀번호를 얻어내야한다.

```bash
cat readme
```
파일 내용을 출력해주는 `cat` 명령어를 이용하여 readme의 내용을 알아낼 수 있다.<br />

해당 비밀번호로 level 0과 같은 방식으로 bandit1 유저로 접근하면 된다.
## Level 1 -> 2
[https://overthewire.org/wargames/bandit/bandit2.html](https://overthewire.org/wargames/bandit/bandit1.html)
### 문제 설명
The password for the next level is stored in a file called **-** located in the home directory
### 문제 풀이
이번에는 비밀번호가 readme가 아니라 -라는 이름을 가진 파일에 있다고 한다.<br />
단순하게 `cat -`을 입력하면 비밀번호 출력은 전혀 안되고 터미널이 대기 상태에 들어가고 내가 작성한 키보드 인풋을 그대로 출력하는 것을 알 수 있다.<br />

이는 원래 `cat` 명령어의 작동 방식이 그러하기 때문이다.<br />
`cat`은 표준 입력(키보드 입력)을 받고 표준 출력(화면)을 하는 명령어이다. 인자로 파일 이름이 들어왔을 때에만 파일을 읽는 행동을 한다.<br />

`-`은 표준 입/출력을 의미하는 특수 문자이다. 그렇기 때문에 `cat`은 `-`을 파일 이름으로 인지하지 못하고 특수 문자로 받아들여 원래의 `cat`만 입력했을 때 처럼 작동하는 것이다.<br />
#### 다양한 풀이
이를 해결하기 위해서는 다양한 방법이 존재한다.

```bash
cat ./-
cat ~/-
```
파일의 완전한 위치를 건내주어 읽어내는 방식

```bash
cat < -
```
리디렉션을 이용하는 방식

```bash
more -
```
`more` 명령어를 이용하는 방식
## Level 2 -> 3
[https://overthewire.org/wargames/bandit/bandit3.html](https://overthewire.org/wargames/bandit/bandit3.html)
### 문제 설명
The password for the next level is stored in a file called **spaces in this filename** located in the home directory
### 문제 풀이
이번에도 파일을 읽어내는 문제이다.<br />
파일 이름은 "spaces in this filename"으로 파일 이름에 " " 스페이스 문자가 섞여있는 상황이다.<br />

GUI가 있는 OS의 경우에는 파일 이름에 스페이스가 들어가는 것이 문제가 되지 않지만 터미널에서 접근할 때에 문제가 발생한다.<br />
터미널에서는 명령어와 옵션 그리고 인자들 사이를 구분하기 위해 스페이스를 사용하는데 이렇게 파일 이름에 섞여있으면 명령어들이 파일 이름을 올바르게 받아들이지 못하게 된다.<br />

```bash
cat "spaces in this filename"

cat 'spaces in this filename'
```
이럴 때는 위 명령어처럼 파일 이름을 큰 혹은 작은 따옴표로 감싸면 된다.<br />

```bash
cat spaces*
```
와일드카드를 사용하는 다른 방식도 있다.
## Level 3 -> 4
[https://overthewire.org/wargames/bandit/bandit4.html](https://overthewire.org/wargames/bandit/bandit4.html)
### 문제 설명
The password for the next level is stored in a hidden file in the **inhere** directory.
### 문제 풀이
이번 비밀번호는 inhere이라는 디렉토리의 숨겨진 파일에 있다고 한다.
```bash
bandit3@bandit:~$ ls
inhere

bandit3@bandit:~$ cd inhere

bandit3@bandit:~/inhere$ ls -a
total 12
drwxr-xr-x 2 root    root    4096 Sep 19 07:08 .
drwxr-xr-x 3 root    root    4096 Sep 19 07:08 ..
-rw-r----- 1 bandit4 bandit3   33 Sep 19 07:08 ...Hiding-From-You

bandit3@bandit:~/inhere$ cat ...Hiding-From-You
```
처음에 inhere 디렉토리의 위치를 파악하기 위해 `ls` 명령어를 이용했다. 특정 디렉토리의 파일 리스트를 리턴해주는 명령어이다.<br />

결과를 보니 홈 디렉토리에 있다. 그래서 `cd` 명령어로 해당 디렉토리로 옮겨준다. `cd`는 change directory의 줄임말로 말 그대로 디렉토리를 변경해주는 명령어다.<br />

inhere 디렉토리 안에서 또 다시 `ls` 명령어를 하게 되면 아무것도 보이지 않게 된다. 비밀번호 파일이 숨겨져있기 때문이다.<br />
이럴 땐 `ls` 명령어에 `-a` 옵션을 주어 숨겨진 파일까지 볼 수 있다.<br />

파일 이름 중 "...Hiding-From-You"라는 파일이 눈에 띄여 출력을 해보면 비밀번호를 얻을 수 있다.
## Level 4 -> 5
[https://overthewire.org/wargames/bandit/bandit5.html](https://overthewire.org/wargames/bandit/bandit5.html)
### 문제 설명
The password for the next level is stored in the only human-readable file in the **inhere** directory. Tip: if your terminal is messed up, try the “reset” command.
### 문제 풀이
위와 비슷한 문제이다. <br />
inhere 디렉토리에 접근하여 파일을 읽어내야한다. 해당 디렉토리에 가서 `ls` 명령어로 파일 목록을 확인해보면 "-file00" ~ "-file09"까지 쭉 나열돼있다.<br />

```
cat ./-file00
```
그 중 하나의 내용을 출력해보면 이해할 수 없는 문자로 이루어진 문자열이 출력된다.<br />

처음에는 이것을 특정 방식으로 복호화를 하거나 디코딩을 해야하나 싶었지만 파일을 하나하나씩 출력해보니 비밀번호를 알 수 있게 된다.<br />

파일 중 하나에 읽을 수 있는 올바른 비밀번호가 있다.
## level 5 -> 6
[https://overthewire.org/wargames/bandit/bandit6.html](https://overthewire.org/wargames/bandit/bandit6.html)
### 문제 설명
The password for the next level is stored in a file somewhere under the **inhere** directory and has all of the following properties:
- human-readable
- 1033 bytes in size
- not executable
### 문제 풀이
inhere 디렉토리에 들어가면 "maybehere"이라는 이름을 가진 디렉토리들이 00부터 19까지 존재한다.<br /> 각 디렉토리에는 여러 파일들이 존재하는데 단 하나의 파일을 제외하고 나머지는 쓸데없는 텍스트를 가지고 있다.<br />

각 디렉토리마다 9개의 파일이 존재하기 때문에 20 * 9, 총 180개의 파일을 확인 해야해서 shell 스크립트로 자동화를 해야하나 고민해봤지만 문제를 한 번 더 읽어보고 생각을 고쳐먹었다.<br />

문제를 보면 비밀번호가 담긴 파일은 사람이 읽을 수 있는 텍스트이고 사이즈가 1033 바이트 그리고 실행 불가능한 파일이라고 한다.<br /> 

```
bandit5@bandit:~/inhere$ ls -l
total 80
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere00
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere01
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere02
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere03
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere04
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere05
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere06
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere07
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere08
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere09
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere10
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere11
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere12
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere13
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere14
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere15
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere16
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere17
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere18
drwxr-x--- 2 root bandit5 4096 Sep 19 07:08 maybehere19
```
`ls` 명령어에 `-l` 옵션을 주게되면 각 파일의 사이즈도 함께 출력된다.<br />
위 결과에서 보이는 username과 날짜 사이에 나오는 숫자(여기에선 4096)가 해당 파일 혹은 디렉토리의 바이트 사이즈이다.<br />

이런식으로 `ls -l maybehere00` 포멧으로 모든 디렉토리의 파일들을 다 검사하다보면 사이즈가 1033인  파일이 하나 나온다. 해당 파일을 출력해보면 비밀번호를 얻어낼 수 있다.
# 배운 것
리눅스에서 `-`로만 이루어졌거나 시작하는 파일들을 처리하는 법에 대해서 익힐 수 있었다.<br />
그리고 왜 Dashed Filename은 문제가 발생하는지 원리에 대해 배울 수 있었다.