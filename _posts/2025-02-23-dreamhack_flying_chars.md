---
title: "드림핵 Beginner - Flying Chars"
date: 2025-02-23 00:20:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking] 
description: 드림핵 Flying Chars 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/850](https://dreamhack.io/wargame/challenges/850)

# 문제 설명

날아다니는 글자들을 멈춰서 전체 문자열을 알아내세요! 플래그 형식은 DH{전체 문자열} 입니다.

❗첨부파일을 제공하지 않는 문제입니다.  
❗플래그에 포함된 알파벳 중 `x`, `s`, `o`는 모두 소문자입니다.  
❗플래그에 포함된 알파벳 중 `C`는 모두 대문자입니다.

---
# 문제 풀이
해당 문제는 간단해도 너무 간단하다. 문제에서 기본으로 제공하는 파일은 없고 그냥 웹사이트 하나만 주어진다. <br />

웹사이트에 방문하면 글자들이 미친듯이 화면의 좌에서 우로 각기 다른 속도를 가지고 날아다닌다. 너무나도 빨라서 맨눈으로 다 포착하기는 무리가 있다. <br />

이 문제를 푸는 방법에는 여러가지가 있을 수 있겠지만 나는 한 번 프론트 단의 코드를 조작하여 풀어보고 싶어서 console을 켜보았다. <br />

해당 사이트의 HTML을 보던 중 수상한 이미지들을 발견했다. 문자들이 화면에 날아다니는 줄 알았는데 그것이 아니라 사진이 날아다니는 것이었다. 각 사진에는 1 ~ 3개의 문자를 보여준다.

![flying chars image](https://1drv.ms/i/c/5cb37aa515b56a00/IQR06HGsbKncTrqSJz6Mt7f1ATixo7mKQecL2Lo5cVNPPdA?width=660)
<br />

이렇게 문자들을 담은 사진들을 모두 다 살펴보면 최종 플래그를 얻을 수 있다.