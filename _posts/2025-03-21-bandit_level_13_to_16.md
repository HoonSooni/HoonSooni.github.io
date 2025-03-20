---
title: "OverTheWire Bandit Level 13 ~ 16 문제 풀이"
date: 2025-03-21 00:00:00 +0800
categories: [Wargame, Bandit]
tags: [cyber security, writeup, overthewire, bandit, linux]
description: OverTheWire Bandit 레벨 13에서 16까지 문제 풀이
---

# 문제 리스트
## Level 13 -> 14
[https://overthewire.org/wargames/bandit/bandit14.html](https://overthewire.org/wargames/bandit/bandit14.html)
### 문제 설명
The password for the next level is stored in **/etc/bandit_pass/bandit14 and can only be read by user bandit14**. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. **Note:** **localhost** is a hostname that refers to the machine you are working on.
### 문제 풀이
이번 문제는 비밀번호를 얻는 문제가 아니라 private SSH 키를 이용하는 문제이다.<br />
비밀번호는 `/etc/bandit_pass/bandit14`에 있지만 bandit14 유저만 접근할 수 있다.<br />

Private Key는 bandit13 유저의 홈 디렉토리에 있어서 찾을 필요가 없다.<br />
#### SSH 키를 이용한 서버 접속
```bash
bandit14@bandit:~$ ssh
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface] [-b bind_address]
           [-c cipher_spec] [-D [bind_address:]port] [-E log_file]
           [-e escape_char] [-F configfile] [-I pkcs11] [-i identity_file]
           [-J destination] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-P tag] [-p port] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           destination [command [argument ...]]
       ssh [-Q query_option]
```
우선 `ssh` 명령어가 어떤 옵션들을 받는지 알아 보기 위해 위와 같은 명령어를 입력한다.<br />
옵션중에 `-i` 옵션은 identity file을 의미한다고 한다. 즉, private ssh file의 위치를 입력받는 것이다.<br />

```bash
bandit13@bandit:~$ ssh -i sshkey.private bandit14@localhost -p 2220
```
이런식으로 `-i` 옵션 다음에 `sshkey.private` 파일의 위치를 넘겨주고 그 다음 접근하고자 하는 유저와 주소, 그리고 포트를 원래 하던 방식으로 넣어주어 입력하면 접속이 된다.<br />

위에서는 주소를 "localhost"로 명시하고 있지만 해당 주소는 "bandit.labs.overthewire.org"이 될 수도 있다.<br /> localhost는 지금 커멘드를 작성하고 있는 서버를 의미하고 있기 때문에 bandit.labs.overthewire.org와 같다.<br />

bandit14에 접속한 이후에 `/etc/bandit_pass/bandit14` 파일의 값을 출력하면 비밀번호를 얻어낼 수 있다.
## Level 14 -> 15
[https://overthewire.org/wargames/bandit/bandit15.html](https://overthewire.org/wargames/bandit/bandit15.html)
### 문제 설명
The password for the next level can be retrieved by submitting the password of the current level to **port 30000 on localhost**.
### 문제 풀이
localhost 주소의 포트 30000에 현재 레벨의 비밀번호를 제출하면 된다고 한다.<br />
접속하는 방법은 `telnet`, `nc`, `ssh` 등의 명령어를 사용하면 된다.<br />

```bash
bandit14@bandit:~$ nc localhost 30000
Level14 password
Correct!
Level15 password
```
이번에는 nc를 이용해서 localhost의 30000번 포트와 네트워크 연결을 시도했다.<br />
시도하니 아무런 문자도 나오지 않아서 그냥 해당 레벨의 비밀번호를 입력해봤더니 "Correct!"가 출력되며 다음으로 넘어가는 비밀번호를 얻을 수 있게됐다.
## Level 15 -> 16
[https://overthewire.org/wargames/bandit/bandit16.html](https://overthewire.org/wargames/bandit/bandit16.html)
### 문제 설명
The password for the next level can be retrieved by submitting the password of the current level to **port 30001 on localhost** using SSL/TLS encryption.

**Helpful note: Getting “DONE”, “RENEGOTIATING” or “KEYUPDATE”? Read the “CONNECTED COMMANDS” section in the manpage.**
### 문제 풀이
이번에는 저번 문제와 비슷한 유형인데 SSL 혹은 TLS 암호화를 이용한다고 한다.<br />

```bash
bandit15@bandit:~$ openssl s_client -connect localhost:30001
```
리눅스에서 SSL 암호화를 이용하면서 동시에 네트워크 연결을 하려면 `openssl` 명령어를 이용하면 된다.<br />
이렇게 연결을 시도하면 다양한 정보들이 출력되고 터미널은 인풋을 받기 위해 대기를 한다. 이 때 해당 레벨의 비밀번호를 입력하면 다음 비밀번호가 출력된다.
## Level 16 -> 17
[https://overthewire.org/wargames/bandit/bandit17.html](https://overthewire.org/wargames/bandit/bandit17.html)
### 문제 설명
The credentials for the next level can be retrieved by submitting the password of the current level to **a port on localhost in the range 31000 to 32000**. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

**Helpful note: Getting “DONE”, “RENEGOTIATING” or “KEYUPDATE”? Read the “CONNECTED COMMANDS” section in the manpage.**
### 문제 풀이
localhost의 31000 ~ 32000 포트 중 열려있는 것을 확인하고 SSL/TLS로 연결할 수 있는 포트가 있는지 알아내라고 한다. 해당 포트에 또 다시 해당 레벨의 비밀번호를 입력하면 해결할 수 있다.<br />
#### 접속 가능한  포트 알아내기
```bash
nmap -p31000-32000 localhost
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```
리눅스에서 포트 스캔을 할 때 가장 많이 쓰이는 명령어인 `nmap`을 이용해봤다. <br />
위의 명령어는 포트 31000 ~ 32000 중 열려있는 포트를 리턴한다. 결과로 총 5개의 포트가 열려있다고 나온다.<br />
#### 접속 시도하기
```bash
bandit16@bandit:~$ openssl s_client -connect localhost:31790
```
하나하나 접속을 시도해보던 중 포트 31790에서 SSL 커넥션을 받는 것을 알게됐다. <br />
이제 비밀번호를 입력하면 되는데 입력만 하면 "KEYUPDATE"라는 문자열만 뜨고 결과는 나오지 않는 상황에 봉착했다.<br />

문제에도 나와있듯이 "KEYUPDATE"가 출력된다면 manpage의 "CONNECTED COMMANDS" 부분을 살펴보라고 한다. 그런데 어떤 명령어의 manpage인지 전혀 모르겠고 검색해봐도 나오지 않고 GPT에게 물어봐도 알아낼 수 없었다.<br />
결국 그냥 KEYUPDATE가 출력될 때 해결하는 방법에 대해서만 알아냈다. 해결법은 위 명령어 맨 마지막에 `-ign_eof` 명령어를 붙이면 된다.<br />
보내는 문자열에 'k'가 포함됐다면 KEYUPDATE 기능이 작동한다고 한다. 이를 비활성화 시키는 옵션이다.<br />

```bash
bandit16@bandit:~$ openssl s_client -connect localhost:31790 -ign_eof
```
결과적으로 이러한 명령어를 작성하여 입력하면 전처럼 접속이 되고 비밀번호를 입력하면 "Correct!"라는 문자열과 함께 성공했다는 것을 알 수 있다.<br />

하지만 또 반전이 있는데 결과로 비밀번호가 나오는 것이 아니라 RSA Private Key가 출력됐다<br />
이 전 Level 13 -> 14 풀이를 했을 때 private ssh 파일이 RSA Private Key였던 것을 이용하여 그대로 풀이하면 된다.
#### 비밀번호 알아내기
```
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----
```
private ssh 키는 위와 같다.<br />
해당 값을 복사하여 어떠한 파일에 저장해야한다. 그대로 `ssh` 명령어에 이 긴 문자열을 입력하는 것이 가능한지는 모르겠지만 이 전 문제 풀이할 때 했던 방식 그대로 하는 것이 안전하기 때문에 보수적으로 나가기로 했다.<br />

bandit16의 홈 디렉토리에 파일을 만드려고 시도했지만 권한이 없어서 실패했다.<br />
그래서 `mktemp -d` 명령어를 이용해서 /tmp 디렉토리에 임시 디렉토리를 하나 만들고 그 안에 "sshkey.private"이라는 파일을 만들어 위 내용을 붙여넣어줬다.<br />
결국 `/tmp/tmp-randomnum/sshkey.private` 파일에 private ssh 키가 저장된 것이다.<br />

```bash
bandit16@bandit:~$ ssh -i /tmp/tmp.4xFvAqfQLY/sshkey.private bandit17@localhost -p 2220

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0664 for '/tmp/tmp.4xFvAqfQLY/sshkey.private' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/tmp/tmp.4xFvAqfQLY/sshkey.private": bad permissions
bandit17@localhost: Permission denied (publickey).
```
해당 키를 가지고 ssh 접속을 bandit17에 시도해보니 이번엔 ssh 키 파일의 접근 권한 관련 문제로 접속에 실패했다. <br />
권한이 너무 오픈돼 있다는 말과 "UNPROTECTED PRIVATE KEY FILE"이라며 private 파일인데 보호가 돼있지 않다 라는 결과만 뱉을 뿐이다.<br />
결과 중간에 보면 Permissions 0664가 문제라고 하는데 파일의 권한을 좀 살펴봐야 할 것 같다.

```bash
bandit16@bandit:~$ ls -al /tmp/tmp.4xFvAqfQLY
total 17016
drwx------ 2 bandit16 bandit16     4096 Mar 20 14:45 .
drwxrwx-wt 1 root     root     17412096 Mar 20 14:50 ..
-rw-rw-r-- 1 bandit16 bandit16     1675 Mar 20 14:45 sshkey.private
```
해당 파일의 접근 권한을 살펴보면(`ls -al` 명령어를 이용하면 파일 리스트를 출력함과 동시에 여러 정보도 함께 볼 수 있다) -rw-rw-r--으로 정말로 664 권한을 가지고 있다.<br />
파일 권한에는 총 세자리가 있는데 왼쪽에서 첫 번째는 파일의 소유자가 행할 수 있는 권한, 두 번째는 파일을 소유한 그룹에 속한 유저들이 행할 수 있는 권한, 세 번째는 기타 사용자들이 행할 수 있는 권한이다.<br />

```bash
bandit16@bandit:~$ chmod 600 /tmp/tmp.4xFvAqfQLY/sshkey.private
bandit16@bandit:~$ ls -al /tmp/tmp.4xFvAqfQLY
total 17016
drwx------ 2 bandit16 bandit16     4096 Mar 20 14:45 .
drwxrwx-wt 1 root     root     17412096 Mar 20 14:57 ..
-rw------- 1 bandit16 bandit16     1675 Mar 20 14:45 sshkey.private
```
private 파일이라면 소유자 이외에는 접근할 수 없이 막아야 하기 때문에 권한을 변경하는 `chmod` 명령어를 이용하여 -rw-------로 변경해주었다.<br />

이 상태로 아까 ssh 접속 시도할 때 썼던 같은 명령어를 사용하면 bandit17에 접근할 수 있다.<br />
그러고 나서 이 전 레벨들의 비밀번호를 저장하고 있는 `/etc/bandit_pass/banditOO` 파일에 접근하면 비밀번호를 알아낼 수 있다.<br />
레벨 16에서 17로 넘어가는 비밀번호는 bandit17에 저장돼있으니 `cat /etc/bandit_pass/bandit17` 명령어를 입력하면 된다.