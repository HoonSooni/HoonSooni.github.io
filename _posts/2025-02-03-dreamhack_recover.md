---
title: "드림핵 lv.1 - Recover"
date: 2025-02-03 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, reverse engineering, reversing] 
description: 드림핵 Recover 리버싱 워게임 풀이
---

## 문제 설명 (난이도 1)
**Description (en)**

This challenge provides chall binary, along with encrypted file.

The chall binary encrypts the flag.png file containing the flag, then stores it as an encrypted file.

Recover flag.png file to get the flag!

The flag format for this challenge is DH{...}.
<br /><br />
**Description (ko)**

본 문제에서는 chall 바이너리와 encrypted 파일이 제공됩니다.

chall 바이너리는 플래그가 담긴 flag.png 파일을 정해진 방식으로 암호화하여 encrypted 파일로 저장합니다.

flag.png 파일을 복구하여 플래그를 획득하세요.

플래그 형식은 DH{...} 입니다.

<hr />

## 문제 풀이
해당 문제에서 제공하는 zip파일을 압축 해제해보면 리눅스 환경에서 실행 가능한 chall 파일과 암호화된 바이너리 파일인 encrypted가 주어진다.

본인은 MacOS 환경을 사용하고 있어서 프로그램을 실행해보지는 못하고 Ghidra로 열어보는 방법 뿐이라 우선 열어봤다.
### Main 함수
```c
undefined8 FUN_00101209(void) {
    size_t sVar1;
    long in_FS_OFFSET;
    byte local_2d;
    int local_2c;
    undefined *local_28;
    FILE *local_20;
    FILE *local_18;
    long local_10;
    
    local_10 = *(long *)(in_FS_OFFSET + 0x28);
    local_28 = &DAT_00102004;
    local_20 = fopen("flag.png","rb");
    if (local_20 == (FILE *)0x0) {
        puts("fopen() error");
                        /* WARNING: Subroutine does not return */
        exit(1);
    }
    local_18 = fopen("encrypted","wb");
    if (local_18 == (FILE *)0x0) {
        puts("fopen() error");
        fclose(local_20);
                        /* WARNING: Subroutine does not return */
        exit(1);
    }
    local_2c = 0;
    while( true ) {
        sVar1 = fread(&local_2d,1,1,local_20);
        if (sVar1 != 1) break;
        local_2d = (local_2d ^ local_28[local_2c % 4]) + 0x13;
        fwrite(&local_2d,1,1,local_18);
        local_2c = local_2c + 1;
    }
    fclose(local_20);
    fclose(local_18);
    if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                        /* WARNING: Subroutine does not return */
        __stack_chk_fail();
    }
    return 0;
}
```
메인 함수처럼 보이는 녀석의 디컴파일 된 코드이다. 잡다한 코드들은 다 무시하고 가장 중요한 while루프를 살펴보면 될 것 같다.

```cpp
while( true ) {
    sVar1 = fread(&local_2d,1,1,local_20);
    if (sVar1 != 1) break;
    local_2d = (local_2d ^ local_28[local_2c % 4]) + 0x13;
    fwrite(&local_2d,1,1,local_18);
    local_2c = local_2c + 1;
}
```
반복문의 형태가 아주 간단하다. <br />

1. 바이트를 읽어오기 위한 파일을 하나 들여온다.
2. 모든 바이트마다 특정한 연산을 적용한다.
   `local_2d = (local_2d ^ local_28[local_2c % 4]) + 0x13;`
3. 연산이 적용된 바이트를 다른 파일에 작성한다.

여기서 신경쓸 부분은 연산하는 것 뿐이다. 그 전에 각 변수들의 역할을 추상해보면 다음과 같다.<br />
```
local_2d: 파일에서 받아오는 바이트
local_28: 바이트 배열, 접근할 때 항상 4의 나머지를 구하는 것을 보면 크기가 4라는 것이 짐작 가능.
local_2c: while문 맨 마지막에 1씩 더하는 것을 보면 일반적인 반복문의 i와 같다는 걸 알 수 있음.
```

**local_28이 무엇인지 알아내기** <br />
결론적으로 우리가 할 일은 해당 반복문의 연산을 그대로 거꾸로 역연산 해주면 된다. encrypted 파일의 바이트를 하나씩 읽어와서 0x13을 빼주고 어떠한 배열을 이용해 xor 연산하면 원본 바이트를 얻을 수 있다.

이때 그 어떠한 배열이 무엇인지 알아내보자.

![dat_main](https://1drv.ms/i/c/5cb37aa515b56a00/IQR5mFnUso63QYbtILcRKHJSAUKDftICK3oLLIfMQmLb8QY?width=628&height=114)
<br />
메인 함수에서 while문 위에 `local_28`을 초기화하는 부분을 보면 `&DAT_00102004`에서 값을 불러오고 있다. 더블 클릭을 해서 값을 보면 아래와 같은 결과를 Listing View에서 볼 수 있다.

![bytes](https://1drv.ms/i/c/5cb37aa515b56a00/IQTAWkMtgkBjTJwDPgK36qIUAbjme5KqcAAPVay9yF891Yw?width=1024)
<br />
크기가 4일 것으로 예상했는데 역시나였다. 배열 원소 하나당 바이트 하나를 가지는 char 타입의 배열 [0xde, 0xad, 0xbe, 0xef]를 이용해 xor 연산을 했던 것이다.

![gdb](https://1drv.ms/i/c/5cb37aa515b56a00/IQQxV7aI7gK1Qqq0Y8BN3rsyAehZokiGCeDfot1mI1h36CQ?width=1584&height=372)
<br />
*처음에 위 사진의 화면을 보고도 발견하지 못해서 동적으로만 확인할 수 있는 건가 싶어 gdb로 main 함수를 분석해 값을 얻었다가 이 글을 쓸 때 쯤 우연히 기드라로 바로 확인이 가능하다는 것을 알았다...*

### 복호화 코드 작성하기
```python
s = [0xde, 0xad, 0xbe, 0xef]

i = 0

decrypted_bytes = bytearray()

with open("./encrypted", "rb") as file:
    while byte := file.read(1):
        tempByte = int.from_bytes(byte, "big")
        tempByte -= 0x13
        tempByte &= 0xFF # to prevent overflow
        tempByte = tempByte ^ s[i % 4]
        i += 1
        decrypted_bytes.append(tempByte)

with open("./decrypted_output", "wb") as output_file:
    output_file.write(decrypted_bytes)
```

이 전에 메인 함수에서 봤던 연산을 그대로 역순으로 작성해보았다. 그리고 역연산 된 바이트들을 decrypted_output이라는 파일로 옮기는 코드가 아래에 작성됐다.

### 결과
![file_output](https://1drv.ms/i/c/5cb37aa515b56a00/IQQcrKkcMv8GQr6Hrrc6ywM3ASTWpOb9VDtcNbKM_Mgj-bw?width=1024)
<br />
디코딩 코드를 실행하면 descrypted_output이 생성된 것이 확인된다.

![converted_file](https://1drv.ms/i/c/5cb37aa515b56a00/IQQtKEsJ0Kb5SIc6Hd0CUxaFAUiEGWErqHJ2aCZCTsc7Fg4?width=1024)
<br />

문제 설명에 원래 파일이 png 형식이었다고 하니 변환을 해준다. 그러면 위 사진처럼 DH{} 태그가 담긴 사진이 나온다.