---
title: "webhacking.kr old 36 문제 풀이"
date: 2025-04-19 00:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking]
description: webhacking.kr old 36 문제 풀이
---

[https://webhacking.kr/challenge/bonus-8/](https://webhacking.kr/challenge/bonus-8/)
# 문제 풀이
```
While editing index.php file using vi editor in the current directory, a power outage caused the source code to disappear.  
Please help me recover.
```
문제 페이지에 접속해보면 위와 같은 문장이 출력된다.<br />
index.php를 vi 에디터로 수정을 하다가 전기가 나가서 작업물들이 날아갔다고 하는데 이를 복구하는 것을 도와달라고 요청하는 문구이다.<br />
## 최종 풀이
평소 리눅스나 vi, vim 에디터를 사용해보지 않았다면 잘 모를 수 있는데, 해당 텍스트 에디터로 작업을 하면 자동으로 swap 파일이 생성된다.<br />
이 파일은 갑자기 vim 에디터가 저장없이 종료됐을 때 자동으로 생성되는 일종의 백업 파일이다.<br />

다시 말해서 해당 문제는 우리가 vim 에디터의 성질을 알고 있냐 아니냐를 판별한다.<br />
index.php를 수정하다가 갑자기 종료됐으니 index.php.swp라는 swap 파일이 생성됐을 것이다.<br />

swap 파일의 확장자는 .swp이고 파일 이름으로는 본인이 에디터로 조작하고 있던 파일 이름을 그대로 가져온다.<br />

참고로 리눅스나 맥에서 swp 파일이 생성되면 숨김파일로 자동으로 설정돼서 숨김파일을 의미하는 '.' 온점을 파일 이름 맨 앞에 붙여야 한다.<br />

`https://webhacking.kr/challenge/bonus-8/.index.php.swp` 그래서 이런식으로 swap 파일에 접근해보면 해당 파일이 다운로드되고 이를 읽어보면 맨 마지막에 플래그가 있는 것을 볼 수 있다.