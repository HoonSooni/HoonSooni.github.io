---
title: "드림핵 Beginner - web-misconf-1"
date: 2025-02-23 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, configuration] 
description: 드림핵 web-misconf-1 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/45](https://dreamhack.io/wargame/challenges/45)
# 문제 설명

기본 설정을 사용한 서비스입니다.  
로그인한 후 Organization에 플래그를 설정해 놓았습니다.

--- 
# 문제 풀이
## 파일 분석
문제에서 제공하는 파일을 살펴보면 `deploy`라는 디렉토리에 `defaults.ini` 파일이 보인다. 문제에서 이 서비스는 기본 설정을 사용한 서비스라고 하니 해당 파일은 설정 파일이라는 것을 짐작해볼 수 있다. *참고로 `.ini` 확장자 자체가 설정 파일을 의미한다.*
### defaults.ini
파일을 살펴보면 700줄에 달하는 설정 값이 담겨져 있기 때문에 하나하나 살펴볼 수는 없다. <br />
우선 문제에서 언급했던 "organization"이라는 문자열과 플래그와 연관이 있을 듯 싶으니 `ctrl + f`로 찾아보면 된다. <br />

```python
# specify organization name that should be used for unauthenticated users
org_name = DH{THIS_IS_FAKE_FLAG}
```
계속 찾다보면 수상한 부분이 등장한다. 가짜 플래그를 담고있는 `org_name`이라는 설정값이다.<br />
## 웹사이트 분석
설정 파일에선 가짜 플래그를 담고있으니 이젠 진짜를 알아내야 하는데 해당 파일에선 도저히 찾을 수가 없다. 문제의 VM 서버를 열어 접속해본다.
### login page
![web-misconf-1_login](https://1drv.ms/i/c/5cb37aa515b56a00/IQQteUylvcZQSoA_EEv5mFwHAc6QX7r_5AgcMhR9y9ns4M4?width=660)
<br />
사이트에 접속하면 곧바로 로그인 창이 뜨게 된다.<br />
어떤 값으로 로그인을 하면 될까, 아이디를 새로 만들어야 하나, 회원가입 버튼이 없는데 등등 생각하다가 다시 설정 파일을 둘러보았다.
```python
# default admin user, created on startup
admin_user = admin

# default admin password, can be changed before first start of grafana, or in profile settings
admin_password = admin
```
중간에 이러한 설정값들이 들어있다. 서버가 시작될 때 기본적으로 admin 유저를 만들고 비밀번호도 아이디와 동일하게 admin으로 설정된다.

`ID: admin, PW: admin`으로 로그인 하면 된다.

이때 갑자기 비밀번호를 바꾸는 창이 나오는데 본인이 원하는 적절한 번호로 변경하고 넘어가면 된다. 다시 쓸 일은 없으니 외우지 않아도 된다.
### main page
![web-misconf-1_main_page](https://1drv.ms/i/c/5cb37aa515b56a00/IQSDIHYdf9WxQrZa3WqDtTSaAdLqfuGfN-QGJbYdVepfKvE?width=660)
<br />
![web-misconf-1_panel](https://1drv.ms/i/c/5cb37aa515b56a00/IQR6D5zzAVi0SIgiNw5M9pl9ARix9-EPLSJJdtw9T6_UUMA?width=660)
<br />
이렇게 메인 페이지에 들어가면 어떠한 서비스를 전체적으로 컨트롤 할 수 있는 창이 나온다. 여기서 우리가 얻어야할 것은 진짜 플래그이다. <br />

처음에는 왼쪽 패널 맨 아래에 있는 `Server Admin` 카테고리의 `Orgs`에 들어가면 플래그를 찾을 수 있겠거니 싶었지만 아니었다. <br />
### setting page
![web-misconf-1_setting_flag](https://1drv.ms/i/c/5cb37aa515b56a00/IQSWNtavJMqlTqcx01427zIRAT7A85pamyhxdJmH0gjlXHo?width=660)
<br />
그래서 그 다음으로는 `Orgs` 아래에 있는 `Settings`에 들어가 보았다.<br />
들어가니 무언가 설정값으로 보이는 것들이 나열돼있다. 조금만 더 내려가면 바로 진짜 플래그를 발견할 수 있다.

--- 
# 배운것
`.ini`가 설정 파일의 확장자 중 하나라는 라는 것을 알게됐다.