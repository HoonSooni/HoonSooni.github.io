---
title: "드림핵 Beginner - ex-reg-ex"
date: 2025-02-23 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, regex] 
description: 드림핵 ex-reg-ex 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/834](https://dreamhack.io/wargame/challenges/834)
# 문제 설명

문제에서 요구하는 형식의 문자열을 입력하여 플래그를 획득하세요. 플래그는 `flag.txt` 파일과 `FLAG` 변수에 있습니다.

플래그 형식은 DH{...} 입니다.

---
# 문제 풀이
## 파일 분석
### app.py
```python
#!/usr/bin/python3
from flask import Flask, request, render_template
import re

app = Flask(__name__)

try:
    FLAG = open("./flag.txt", "r").read()       # flag is here!
except:
    FLAG = "[**FLAG**]"

@app.route("/", methods = ["GET", "POST"])
def index():
    input_val = ""
    if request.method == "POST":
        input_val = request.form.get("input_val", "")
        m = re.match(r'dr\w{5,7}e\d+am@[a-z]{3,7}\.\w+', input_val)
        if m:
            return render_template("index.html", pre_txt=input_val, flag=FLAG)
    return render_template("index.html", pre_txt=input_val, flag='?')

app.run(host="0.0.0.0", port=8000)

```
app.py 하나만 주어지는 문제이다.<br />
Flask를 이용해서 서버를 구동하고 있고 `/`를 제외한 다른 엔드포인트도 존재하지 않는 간단한 웹 서비스다.

라우터 함수를 보면 POST 요청이 들어오면 들어온 문자열을 정규식으로 한 번 필터 해준 다음 통과된다면 FLAG를 출력하는 방식이다.

즉, 우리는 `r'dr\w{5,7}e\d+am@[a-z]{3,7}\.\w+'` <- 이 조건식을 만족하는 값을 입력해주면 플래그를 얻을 수 있다는 의미이다.
## 정규식 분석
```
dr\w{5,7}e\d+am@[a-z]{3,7}\.\w+
```
정규식은 항상 최대한 분해해서 생각해봐야 쉬워진다.

1. `dr\w{5, 7}e`
	이는 'dr'로 시작해야하고 그 뒤로 5 ~ 7글자는 숫자, 알파벳, 그리고 밑줄(`_`) 중 하나여야 한다는 의미다. 그리고 마지막으로 오는 문자는 반드시 e로 끝나야한다.
2. `\d+am@[a-z]{3, 7}`
	맨 앞의 `\d+`는 해당 문자열이 반드시 숫자(0 ~ 9) 하나 이상으로 시작해야 한다는 의미이다. 그 다음 `am@`은 하나 이상의 숫자 다음으로 와야할 문자들을 그대로 나타낸다.<br />
	그 다음 `[a-z]{3, 7}`은 영어 소문자가 3개에서 7개는 있어야 한다는 의미이다.
3. `\.\w+`
	`\.`은 위의 조건 그 다음으로 마침표가 와야한다는 의미이다. 그냥 온점이 아니라 `\.`인 이유는 Dot(.)가 정규식에서 특수문자이기 때문에 `\`로 escape가 필요하기 때문이다.<br/>
	\w+는 위에서도 보았듯이 숫자, 알파벳, 그리고 밑줄(`_`)로 이루어진 문자가 반드시 하나 이상은 와야 한다는 의미다.

이를 모두 종합한 정규식을 만족하는 문자열은 굉장히 다양하지만 하나만 꼽아보자면 다음과 같다. <br />
`drrrrrre3am@abc.com`.

```
 Input: drrrrrre3am@abc.com
 Flag: DH{Redacted}
```

즉, 해당 정규식은 완벽하지는 않지만 어느정도 이메일 형식을 지키도록 하는 정규식인 것이다.

--- 
# 배운 것
정규식 문법에 대해 잘 몰랐는데 어느정도 기본적인 것을 익힐 수 있는 기회가 됐다.