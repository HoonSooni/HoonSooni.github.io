---
title: "OverTheWire Bandit Level 11 ~ 12 문제 풀이"
date: 2025-03-20 00:00:00 +0800
categories: [Wargame, Bandit]
tags: [cyber security, writeup, overthewire, bandit, linux]
description: OverTheWire Bandit 레벨 11에서 12까지 문제 풀이
---

# 문제 리스트
## Level 11 -> 12
### 문제 설명
The password for the next level is stored in the file **data.txt**, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions
### 문제 풀이
문제를 읽어보면 비밀번호는 data.txt 파일에 들어있지만 모든 알파벳들이 13칸 만큼 밀려있다고 한다.<br />
예를 들어 'a'를 1이라고 가정하고 13만큼 밀어내면 14가 돼 14번째 알파벳인 'n'이 되는 방식이다. 해당 파일의 대소문자 상관없이 모든 알파벳마다 해당 방식이 적용돼있다.

이 치환 방식은 암호학에서 카이사르 암호라고 불리우는 단순한 암호화 도구이고 ROT13이라고 불린다.<br />
이에 대해 더 자세히 알고싶다면 [이 곳](https://en.wikipedia.org/wiki/ROT13)을 살펴보면 된다.<br />

이를 해결하는 특별한 명령어는 없지만 `tr` 명령어를 이용하면 쉽게 역치환이 가능해진다.<br />
[tr 명령어 사용법](https://www.geeksforgeeks.org/tr-command-in-unix-linux-with-examples/)은 여기서 찾아볼 수 있다.<br />

---
```bash
bandit11@bandit:~$ cat data.txt
Gur cnffjbeq vf 7k16JArUVv5LxVuJfsSVdbbtaHGlw9D4
```
원래의 data.txt의 내용은 위와 같다.
"Gur cnffjbeq vf"를 보면 아마 원래 문자열은 "The password is"였구나 라는 것을 짐작할 수 있다. 즉, 정말 문제에서 설명했듯 ROT13 방식으로 인코딩이 돼있는 것이다.<br />

```bash
bandit11@bandit:~$ cat data.txt | tr "[n-za-m][N-ZA-M]" "[a-mn-z][A-MN-Z]"
The password is REDACTED
```
이런식으로 `tr` 명령어를 이용해서 n ~ z까지의 문자를 a ~ m으로, a ~ m까지의 문자를 n ~ z로 역치환을 할 수 있다. 대문자의 경우에도 같은 방식으로 처리하면 된다.
## Level 12 -> 13
### 문제 설명
The password for the next level is stored in the file **data.txt**, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use mkdir with a hard to guess directory name. Or better, use the command “mktemp -d”. Then copy the datafile using cp, and rename it using mv (read the manpages!)
### 문제 풀이
이번엔 문제 시작 전에 특별히 해야할 것이 있다.<br />
제출자가 말하기를 이번엔 일시적인 디렉토리를 하나 만들어서 진행하는 것을 권한다고 한다.<br />

해당 과정은 이러하다.
1. /tmp 디레토리 안에 또 다른 임시 디렉토리 생성. `mkdir`혹은 `mktemp`을 사용하면 된다.
2. `cp` 명령어를 이용해서 data.txt 파일을 새로 생성한 디렉토리에 옮긴다.
3. `mv` 명령어를 이용해서 파일 이름을 변경한다.

이 모든 과정은 사실 문제 풀이하는데에 반드시 필요한 과정은 아니지만 그래도 언급을 해두었으니 따르도록 하겠다.
#### 디렉토리 생성과 파일 옮기는 과정
```bash
bandit12@bandit:~$ cd /tmp

bandit12@bandit:/tmp$ mktemp -d
/tmp/tmp.sgYc9CBXr5

bandit12@bandit:/tmp$ cd tmp.sgYc9CBXr5

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ cp ~/data.txt .

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ ls
data.txt
```
코드를 차근차근 살펴보면 무슨 행동을 하고 있는지 이해가 갈 것이다.<br />

결론 적으로는 /tmp 디렉토리 안에 또 다른 임시 디렉토리를 만들고 data.txt 파일을 해당 디렉토리로 옮겨놓은 것이다. 출제자는 `mv`를 이용한 이름 변경도 하라고 했는데 나는 굳이 하지 않았다.
#### 비밀번호 알아내기
##### 원본 파일
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ cat data.txt
00000000: 1f8b 0808 dfcd eb66 0203 6461 7461 322e  .......f..data2.
00000010: 6269 6e00 013e 02c1 fd42 5a68 3931 4159  bin..>...BZh91AY
00000020: 2653 59ca 83b2 c100 0017 7fff dff3 f4a7  &SY.............
00000030: fc9f fefe f2f3 cffe f5ff ffdd bf7e 5bfe  .............~[.
00000040: faff dfbe 97aa 6fff f0de edf7 b001 3b56  ......o.......;V
00000050: 0400 0034 d000 0000 0069 a1a1 a000 0343  ...4.....i.....C
00000060: 4686 4341 a680 068d 1a69 a0d0 0068 d1a0  F.CA.....i...h..
00000070: 1906 1193 0433 5193 d4c6 5103 4646 9a34  .....3Q...Q.FF.4
00000080: 0000 d320 0680 0003 264d 0346 8683 d21a  ... ....&M.F....
00000090: 0686 8064 3400 0189 a683 4fd5 0190 001e  ...d4.....O.....
000000a0: 9034 d188 0343 0e9a 0c40 69a0 0626 4686  .4...C...@i..&F.
000000b0: 8340 0310 d340 3469 a680 6800 0006 8d0d  .@...@4i..h.....
000000c0: 0068 0608 0d1a 64d3 469a 1a68 c9a6 8030  .h....d.F..h...0
```
문제에서 설명했듯이 data.txt는 [hexdump](https://en.wikipedia.org/wiki/Hex_dump) 형식의 파일이다. <br />
여러번 압축이 된 데이터라고 하니 당연하게도 온전히 읽히는 문자열은 거의 없다.<br />
##### hexdump 되돌리기
리눅스의 `hexdump` 명령어는 아무 파일이나 다 16진수 데이터로 변환한다. <br />
이를 되돌리기 위해서는 `xxd`라는 16진수 데이터를 변환할 때 사용하는 명령어의 도움이 필요하다.<br />

`xxd` 명령어에는 `-r`이라는 옵션이 있는데 해당 옵션은 16진수 데이터를 2진수 데이터로 전환한다.<br />

```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ xxd -r data.txt > binary

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file binary

binary: gzip compressed data, was "data2.bin", last modified: Thu Sep 19 07:08:15 2024, max compression, from Unix, original size modulo 2^32 574
```
이렇게 data.txt 파일을 바이너리 파일로 변환하여 해당 파일의 정보를 출력하는 과정을 거쳤다.<br />
`file binary`의 출력 결과를 보면 gzip으로 압축된 데이터라고 나온다. 그리고 원래의 파일 이름은 "data2.bin"이라고 한다.
##### data2.bin gzip으로 압축 풀기
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ mv binary data2.bin.gz

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ gunzip data2.bin.gz

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ ls
data2.bin  data.txt

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data2.bin
data2.bin: bzip2 compressed data, block size = 900k
```
압축을 풀기 위해서 binary 파일을 원래 이름이었던 data2.bin.gz로 변환해주었다. 이름이 반드시 같을 필요는 없지만 그냥 그렇게 했다.<br />

그리고 `gunzip` 명령어로 gzip 파일의 압축을 풀어주어 data2.bin 파일을 얻어냈다.<br />
이때 또 파일 정보를 살펴보면 여전히 압축된 데이터라고 나오는데 이번엔 bzip2를 이용했다고 한다.
##### data2.bin bzip2으로 압축 풀기
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ mv data2.bin data2.bin.bz2

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ bunzip2 data2.bin.bz2

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data2.bin
data2.bin: gzip compressed data, was "data4.bin", last modified: Thu Sep 19 07:08:15 2024, max compression, from Unix, original size modulo 2^32 20480
```
위와 비슷한 과정을 거치면 된다.<br />
주의해야 할 점은 이번엔 `gzip`이 아니라 `bzip2` 방식을 이용해야 한다는 점이다.<br />

위 같이 명령어들을 입력하면 성공적으로 bzip2 방식을 이용한 압축 해제가 가능하다. 압축이 풀린 파일은 또 다시 gzip으로 압축된 파일이고 원래 이름은 data4.bin이라고 한다.
##### data4.bin gzip으로 압축 풀기
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ mv data2.bin data4.bin.gz

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ gunzip data4.bin.gz

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data4.bin
data4.bin: POSIX tar archive (GNU)
```
압축을 풀어주면 이번엔 tar 압축 파일이라고 한다. 또 다시 압축 해제를 해줘야한다.
##### data4.bin tar로 압축 풀기
tar은 다수의 파일을 디렉토리 구조, 파일 속성 등을 보존하면서 하나로 묶는데 사용된다.<br />
압축이라고 불리기는 하지만 용량이 거의 줄어들지 않기 때문에 일종의 "파일 합치기"에 가깝다.<br />

```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ tar -xf data4.bin

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ ls
data4.bin  data5.bin  data.txt

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data5.bin
data5.bin: POSIX tar archive (GNU)
```
`tar` 명령어에 `-x` 옵션을 주게되면 압축을 해제하고 `-f` 옵션은 파일 이름을 전달할 때 쓰인다.<br />
해제하고 나면 `ls`로 확인해 보았을 때 data5.bin이라는 파일이 하나 생긴 것을 볼 수 있다.<br />
원래 data4.bin이 tar로 압축되기 이전의 파일 이름이 data5.bin이었기 때문이다.
##### data5.bin tar로 압축 풀기
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ tar -xf data5.bin

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ ls
data4.bin  data5.bin  data6.bin  data.txt

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
```
이 전의 결과를 보면 data5.bin은 또 tar로 압축된 파일이라 같은 방식으로 해제해주면 data6.bin이 등장한다.<br /> 해당 파일의 정보를 보면 또 bzip2으로 압축됐다고 하니 해제해주자.
##### data6.bin bzip2로 압축 풀기
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ mv data6.bin data6.bin.bz2

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ bunzip2 data6.bin.bz2

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ ls
data4.bin  data5.bin  data6.bin  data.txt

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data6.bin
data6.bin: POSIX tar archive (GNU)
```
bzip2로 해제하니 또 tar 파일이 등장한다.
##### data6.bin tar로 압축 풀기
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ tar -xf data6.bin

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ ls
data4.bin  data5.bin  data6.bin  data8.bin  data.txt

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", last modified: Thu Sep 19 07:08:15 2024, max compression, from Unix, original size modulo 2^32 49
```
이번에 해제를 하니 또 data8.bin이라는 새로운 파일이 생겨났다. tar 압축할 때의 파일 이름인 것이다.<br />
해제된 녀석을 보니 또 gzip으로 압축된 파일이라고 하니 다시 해제해줘야 한다. <br />
참고로 원래 파일 이름은 "data9.bin"이라고 한다.
##### data8.bin gzip으로 압축 해제
```bash
bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ mv data8.bin data9.bin.gz

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ gunzip data9.bin.gz

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ file data9.bin
data9.bin: ASCII text

bandit12@bandit:/tmp/tmp.sgYc9CBXr5$ cat data9.bin
The password is REDACTED
```
data8.bin을 압축을 해제하고 나서야 드디어 ASCII text 파일이 등장했다.<br />
출력을 해보니 역시나 올바른 비밀번호가 등장했다. 이런식으로 노가다를 시키는 문제가 있을 줄이야.
# 배운 것
리눅스의 각종 압축, 해제 명령어들을 익힐 수 있었다. <br />
절대 잊지 않을 것 같다...<br /><br />
그리고 `mktemp`와 `tr`이라는 명령어에 대해서도 배울 수 있었다.