---
title: "OverTheWire Bandit Level 6 ~ 10 문제 풀이"
date: 2025-03-19 00:10:00 +0800
categories: [Wargame, Bandit]
tags: [cyber security, writeup, overthewire, bandit, linux]
description: OverTheWire Bandit 레벨 6에서 10까지 문제 풀이
---

# 문제 리스트
## Level 6 -> 7
[https://overthewire.org/wargames/bandit/bandit7.html](https://overthewire.org/wargames/bandit/bandit7.html)
### 문제 설명
The password for the next level is stored **somewhere on the server** and has all of the following properties:
- owned by user bandit7
- owned by group bandit6
- 33 bytes in size

### 문제 풀이
비밀번호 파일은 서버 어딘가에 있다고 한다. <br />
홈 디렉토리에서 `ls`를 시도해보니 역시 아무것도 나오지 않는다.<br />

```
bandit6@bandit:/home$ ls
bandit0   bandit15  bandit21  bandit27-git  bandit30-git  bandit6    drifter12  drifter5     formulaone2  krypton4
bandit1   bandit16  bandit22  bandit28      bandit31      bandit7    drifter13  drifter6     formulaone3  krypton5
bandit10  bandit17  bandit23  bandit28-git  bandit31-git  bandit8    drifter14  drifter7     formulaone5  krypton6
bandit11  bandit18  bandit24  bandit29      bandit32      bandit9    drifter15  drifter8     formulaone6  krypton7
bandit12  bandit19  bandit25  bandit29-git  bandit33      drifter0   drifter2   drifter9     krypton1     ubuntu
bandit13  bandit2   bandit26  bandit3       bandit4       drifter1   drifter3   formulaone0  krypton2
bandit14  bandit20  bandit27  bandit30      bandit5       drifter10  drifter4   formulaone1  krypton3
```
바깥 디렉토리로 이동하여 같은 명령어를 시도하니 위와 같은 결과가 나온다.<br />
이곳에는 엄청나게 많은 디렉토리들이 있는데 우리가 탈출한 디렉토리가 바로 bandit6이다.<br />

문제 설명을 보면 힌트를 제공하고 있다.<br />
bandit7이라는 유저와 bandit6라는 그룹이 소유한 파일이라고 한다. 그리고 파일 크기는 33 바이트이다.<br />

```
# Example: find -user {username} -group {groupname}

find / -group bandit6 -user bandit7
```
이렇게 특정 유저나 그룹이 소유한 파일을 찾을 때 유용하게 쓰이는 명령어는 `find`이다.<br />

```
find: ‘/var/cache/apparmor/baad73a1.0’: Permission denied
find: ‘/var/lib/polkit-1’: Permission denied
find: ‘/var/lib/amazon’: Permission denied
/var/lib/dpkg/info/bandit7.password
find: ‘/var/lib/apt/lists/partial’: Permission denied
find: ‘/var/lib/chrony’: Permission denied
find: ‘/var/lib/snapd/void’: Permission denied
find: ‘/var/lib/snapd/cookie’: Permission denied
```
해당 명령어를 실행하면 수없이 많은 파일들이 리스팅된다. 하지만 출력되는 모든 파일들이 다 접근 권한이 없다는 에러가 발생한다.<br />
그 중 단 한 파일만 접근 권한이 있는데 바로 `/var/lib/dpkg/info/bandit7.password`이다.<br />

해당 파일을 출력하면 비밀번호를 얻을 수 있다.
## Level 7 -> 8
[https://overthewire.org/wargames/bandit/bandit8.html](https://overthewire.org/wargames/bandit/bandit8.html)
### 문제 설명
The password for the next level is stored in the file **data.txt** next to the word **millionth**
### 문제 풀이
문제 설명을 보면 비밀번호는 data.txt 파일에 적혀있다고 친절하게 알려준다. 하지만 "millionth"라는 단어 옆에 있다고 하는 것을 보면 data.txt 파일의 크기가 엄청나게 크다는 것을 짐작할 수 있다.<br />

출력을 해보면 정말로 길다는 것을 알 수 있다.<br />

해당 문제를 푸는데는 크게 2가지 방법이 있을 수 있다.
#### Vim
Vim 텍스트 에디터로 data.txt 파일을 열고 해당 문자열을 검색하는 방법이 있다.
#### grep
grep 명령어는 파일 내부의 특정한 패턴을 찾아낸다. grep이 받는 패턴은 정규식이어야 한다.<br />

```bash
grep "millionth" data.txt
```
명령어를 입력하면 "millionth"를 포함한 줄이 출력이 돼 비밀번호를 얻을 수 있다.
## Level 8 -> 9
[https://overthewire.org/wargames/bandit/bandit9.html](https://overthewire.org/wargames/bandit/bandit9.html)
### 문제 설명
The password for the next level is stored in the file **data.txt** and is the only line of text that occurs only once
### 문제 풀이
이번에도 data.txt 파일을 가지고 노는 문제이다.<br />
문제를 읽어보면 data.txt에 단 한 번만 나오는 문자열을 찾아내라고 한다.<br />

```
...
PLsGPuNgYzI8YNu2Y7h4D4vz1nHPSuNl
YZMapJFORxWg84gej4UzQvGYSqBmsPOo
xEkmXBLggW8r1alEgwNX6ZIM6GGCsfmF
PHE4soLmy3nZfNOlX3jB8LYKYZRXuTah
0eJPctF8gK96ykGBBaKydhJgxSpTlJtz
o44oO4jbyPqoQQYX16586yC7Os2uz3ks
YbfaJNckJrgh9TvEBScUaEUCRhDJcgIL
GW8cRcKbnz53MAPYECx99O0T8POlPIFk
UuNP4xguSOjcTHAzdtHBgm2eNz1Z5133
1VKPEkd0bCtIRwMFVQfY7InulwOFyDsn
BooZo7QXA1Tft7d6zbVkgJiGoJzuBTXS
5EmwMKZHwF6Lwq5jHUaDlfFJBeHbcX0b
PHE4soLmy3nZfNOlX3jB8LYKYZRXuTah
6Boy6esAjnIxCYn8uI6KZ7VD7zysDM8i
...
```
역시나 이번에도 엄청나게 긴 문자열이 출력이 되고 비밀번호로 보이는 여러 텍스트들이 라인으로 구분돼 저장돼있다.<br />

자세히 보면 반복되는 문자열들이 보인다. 이 중에 딱 한 문자열만 파일에서 단 한 번만 등장한다고 하는데 그것을찾아내는 것이 문제의 핵심이다.<br />
#### uniq
우리가 써야할 명령어는 `uniq`이다. 해당 명령어는 특정 텍스트에서 유니크(한 번만 등장)하거나 유니크하지 않은 것들을 다 지워내 출력해준다.<br />

예시는 다음과 같다.
```
apple
banana
apple
cherry
banana
cherry
date
```
이런 문자열이 있다고 가정하고 `uniq -u` 을 입력하면 해당 파일에서 중복되는 것들은 모두 지워내고 유니크한 "date"만 출력한다.<br />

하지만 중요한 점이 있다. `uniq` 명령어는 정렬이 돼있는 파일에 한해서만 작동한다.<br />

즉, "apple banana apple" 이렇게 있을 때에 banana를 출력하지 않는다는 의미이다. 반드시 "apple apple banana"처럼 중복되는 문자가 붙어있어야 한다.<br />

리눅스에서는 또 `sort`라는 아주 좋은 명령어가 있다. 말 그대로 sort에 넘겨주는 파일을 정렬한다.<br />
이 명령어들을 이용한 최종 명령어는 아래와 같다.

```bash
sort data.txt | uniq -u
```
이렇게 data.txt를 정렬한 결과를 `|` 파이프를 통해 `uniq` 명령어에 보내주어 비밀번호를 추출할 수 있다.<br />

참고로 `uniq`에 `-d` 옵션을 넘겨주면 중복되는 문자열들을 보여주고 `-c` 옵션을 주면 각 중복되는 문자열마다 몇 번 중복되는지 출력한다.
## Level 9 -> 10
[https://overthewire.org/wargames/bandit/bandit10.html](https://overthewire.org/wargames/bandit/bandit10.html)
### 문제 설명
The password for the next level is stored in the file **data.txt** in one of the few human-readable strings, preceded by several ‘=’ characters.
### 문제 풀이
이번엔 정규식에 대해서 좀 고민이 필요한 문제이다.<br />

data.txt 파일에 읽을 수 없는 문자들이 쭉 나열돼 있는데 이 중 여러 '=' 문자로 시작되는 문자열 중 읽을 수 있는 것이 있는데 그것이 비밀번호이다.<br />

전에도 그랬듯 이럴 때 쓸 수 있는 `grep` 명령어를 사용하면된다.<br />
#### 정규식
이때 grep에 전달할 정규식을 고민해봐야한다.<br />

우리가 원하는 결과는 이렇다. 
1. '='로 시작되는 문자열
2. 1개가 아닌 여러 개의 '='로 시작되는 문자열

여기서 좀 복잡하게 생각할 수도 있지만 간단하게 생각해보면 쉽게 생각해낼 수 있다.<br />
작동하는 정규식은 `==`이다. 

```bash
grep -a '==' data.txt
```
이런식으로 명령어를 작성하면 '='를 2개이상 포함한 여러가지 문자열이 출력되는데 이 중 유일하게 읽을 수 있는 문자열이 바로 비밀번호 이다.

`-a` 옵션은 바이너리 파일을 텍스트 파일로 처리하도록 한다.<br />
`file data.txt` 명령어를 입력하면 data.txt 파일은 텍스트 파일이 아니라 데이터 파일인 것을 알 수 있다. 이 경우 그냥 `grep` 명령어를 사용하면 작동하지 않기 때문에 `-a` 옵션이 필요한 것이다.
## Level 10 -> 11
[https://overthewire.org/wargames/bandit/bandit11.html](https://overthewire.org/wargames/bandit/bandit11.html)
### 문제 설명
The password for the next level is stored in the file **data.txt**, which contains base64 encoded data
### 문제 풀이
data.txt 파일에 비밀번호가 담겨있는데 base64 방식으로 인코딩 돼 있다고 설명한다.<br />
base64는 암호화가 아니라 인코딩 방식이라 손쉽게 디코딩이 가능하다.

```bash
bandit10@bandit:~$ cat data.txt
VGhlIHBhc3N3b3JkIGlzIGR0UjE3M2ZaS2IwUlJzREZTR3NnMlJXbnBOVmozcVJyCg==
```
해당 파일의 내용을 살펴보면 알아볼 수 없는 문자열이 담겨있다. 이것이 바로 base64로 인코딩된 문자열이다.<br />
#### 디코딩
```
cat data.txt | base64 --decode
```
리눅스에서 기본으로 제공하는 `base64` 명령어를 사용하면 된다.<br />

혹은 그냥 위에서 출력된 문자열을 인터넷의 base64 디코더를 이용해서 얻어낼 수도 있다.
# 배운 것
리눅스의 `uniq`, `base64` 명령어를 배울 수 있는 기회가 됐다.<br />
그리고 `find` 명령어를 통해서 특정 파일을 찾아낼 수도 있다는 것을 알게됐다. 