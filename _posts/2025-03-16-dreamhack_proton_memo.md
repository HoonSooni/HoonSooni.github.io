---
title: "드림핵 lv.4 - Proton Memo"
date: 2025-03-16 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, prototype pollution] 
description: 드림핵 Proton Memo 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/884](https://dreamhack.io/wargame/challenges/884)
# 문제 설명
기본적인 기능만 존재하는 메모 앱입니다. 플래그를 얻기 위해 비밀 메모를 읽어주세요!

---
# 문제 풀이
## 웹사이트 분석
### / 페이지
메인 페이지에 접속을 해보면 아무런 CSS 처리도 돼있지 않은 바닐라 HTML 페이지가 나온다.<br />

기본으로 secret이라는 제목을 가진 게시글이 하나 나오고 클릭을 해보면 /view/boardID 페이지로 넘어가게 되는데 비밀 글인 만큼 비밀번호를 입력하는 칸이 출력된다. <br />
문제 설명을 보면 플래그를 얻기 위해 해당 비밀번호 검사를 우회하거나 비밀번호를 알아내야 할 것 같다.
### /new 페이지
해당 페이지에서는 3개의 간단한 인풋 박스가 있고 각각 Title, Content, Password 입력을 받는다.<br />
Password를 입력하든 안하든 무조건 비밀 글로 설정이 된다. 입력을 하지 않았다면 그냥 비밀번호 칸을 공백으로 둔체 제출을 누르면 해당 게시글에 접근할 수 있다.
### /edit/boardID
메인 페이지에서 업로드된 게시글을 클릭하면 "Edit Memo"라는 버튼이 있는데 그걸 눌렀을 때 나오는 페이지이다.<br />

해당 페이지에서는 Title 혹은 Content 중 무엇을 변경하고 싶은지 선택하고 변경 내용을 입력한다. 마지막으로 해당 게시글의 비밀번호를 입력하여 정말 변경할 수 있는 권한이 있는지 체크한다.
## 코드 분석
### app.py
```python
import time
import os
from uuid import UUID
from flask import Flask, render_template, request, redirect, url_for, abort
from utils import set_attr
from models import Memo

def get_memo_with_auth_or_abort(memo_id: str, password: str) -> Memo:
    memo = Memo.get_memo_by_id(memo_id)

    if memo is None:
        abort(404)
    elif not memo.check_password(password):
        abort(403)

    return memo


secret = Memo("secret", open("/flag", "r").read(), os.urandom(20).hex())

Memo.add_memo_to_collection(secret)

app = Flask(__name__)

@app.route("/")
def index():
    memo_title_list = [
        (memo.id, memo.get_title()) for memo in Memo.collections.values()
    ]

    return render_template("index.html", memos=memo_title_list)

@app.route("/new", methods=["GET", "POST"])
def new_memo():
    if request.method == "POST":
        title = request.form["title"]
        content = request.form["content"]
        password = request.form["password"]

        Memo.add_memo_to_collection(Memo(title, content, password))

        return redirect(url_for("index"))
    return render_template("new_memo.html")

@app.route("/edit/<uuid:memo_id>", methods=["GET", "POST"])
def edit_memo(memo_id: UUID):
    memo_id = str(memo_id)

    if request.method == "GET":
        return render_template("edit_memo.html", memo_id=memo_id)
    elif request.method == "POST":
        selected_option = request.form["selected_option"]
        edit_data = request.form["edit_data"]
        password = request.form["password"]

        memo = get_memo_with_auth_or_abort(memo_id, password)

        set_attr(memo, selected_option + ".data", edit_data)
        set_attr(memo, selected_option + ".edit_time", time.time())

        return redirect(url_for("index"))

@app.route("/view/<uuid:memo_id>", methods=["GET", "POST"])
def view_memo(memo_id: UUID):
    memo_id = str(memo_id)

    if request.method == "GET":
        return render_template("enter_password.html", memo_id=memo_id)
    elif request.method == "POST":
        password = request.form["password"]

        memo = get_memo_with_auth_or_abort(memo_id, password)

        contents = (
            memo.get_title_with_edit_time() + "\n" + memo.get_content_with_edit_time()
        )

        return render_template("view_memo.html", memo=contents, memo_id=memo_id)

if __name__ == "__main__":
    app.run(debug=False)
```
코드를 보면 플래그는 secret이라는 게시글에 입력된 것을 알 수 있다. 비밀번호는 길이가 20인 랜덤 문자열이다.
#### /edit 라우터
공격을 시도할 만한 부분이 어디있을까 찾아보다가 /edit에서 호출하는 `set_attr()` 함수 부분이 좀 신경쓰인다.<br />

```python
def set_attr(obj, prop, value):
    prop_chain = prop.split('.')
    cur_prop = prop_chain[0]
    if len(prop_chain) == 1:
        if isinstance(obj, dict):
            obj[cur_prop] = value
        else:
            setattr(obj, cur_prop, value)
    else:
        if isinstance(obj, dict):
            if  cur_prop in obj:
                next_obj = obj[cur_prop]
            else:
                next_obj = {}
                obj[cur_prop] = next_obj
        else:
            if hasattr(obj, cur_prop):
                next_obj = getattr(obj, cur_prop)
            else:
                next_obj = {}
                setattr(obj, cur_prop, next_obj)
        set_attr(next_obj, '.'.join(prop_chain[1:]), value)
```
**utils.py**<br />
`set_attr()` 함수는 utils.py 파일에 정의돼있다.<br />
이 함수는 재귀함수인데 obj 인자로 들어온 객체를 재귀적으로 끝까지 파고들어 value 값을 넣어주는 함수이다.<br />

예를 들어 prop 인자로 "apple.banana.kiwi"가 들어온다면 `apple["banana"]`, `banana["kiwi"]` 이런 식으로 kiwi까지 도달한다면 value 인자 값으로 초기화 해준다.<br />

```python
memo = get_memo_with_auth_or_abort(memo_id, password)

set_attr(memo, selected_option + ".data", edit_data)
set_attr(memo, selected_option + ".edit_time", time.time())
```
`set_attr()`를 호출하는 /edit 부분을 다시 보면 우리가 접근하고자 하는 객체는 memo이다.<br />
그리고 두 번째 인자로 /edit 페이지에서 선택한 `option + '.data'`가 들어가고 마지막 인자로는 /edit 페이지에 입력한 Edit Data 부분이 들어간다.<br />

만약 `selected_option`이 `title`이라면 `memo.title = edit_data` 이런식으로 작동한다는 것이다.
### models.py
```python
class Memo:
    collections: Dict[str, Memo] = {}

    title: Title
    content: Content
    password: Password

    def __init__(self, title: str, content: str, password: str):
        self.id = str(uuid4())
        self.title = Title(title)
        self.content = Content(content)
        self.password = Password(password)

    def get_raw_title(self):
        return self.title

    def get_raw_content(self):
        return self.content

    def get_title(self):
        return self.title.get_title()

    def get_content(self):
        return self.content.get_content()

    def get_title_with_edit_time(self):
        return self.title.get_title_with_edit_time()

    def get_content_with_edit_time(self):
        return self.content.get_content_with_edit_time()

    def check_password(self, password):
        return self.password.check_password(password)

    @staticmethod
    def get_memo_by_id(memo_id: str) -> Memo | None:
        return Memo.collections[memo_id] if memo_id in Memo.collections.keys() else None

    @staticmethod
    def add_memo_to_collection(memo: Memo):
        Memo.collections[memo.id] = memo
```
models.py에서 다른 부분은 제쳐두고 Memo 객체의 생성을 정의하는 Memo 클래스를 살펴보자.<br />
해당 클래스를 잘 보면 각 메모가 가져야할 `title`, `content`, `password를` 제외한 또 다른 `collections`라는 dictionary가 함께 있는 것이 보인다.<br />

app.py에서 /new 라우터에서 새로운 메모가 생성될 때 마다 `add_memo_to_collection()` 함수도 함께 호출되기 때문에 모든 memo 인스턴스를 통해서 다른 memo 객체들의 접근이 가능하다.<br />
## 최종 풀이
memo 인스턴스로 다른 memo에 접근할 수 있다는 것을 이제 알아냈다.<br />
즉, `selected_option`을 `collections.uuid(메모id).password`로 설정하고 edit_data를 원하는 비밀번호로 설정하면 해당 메모의 비밀번호가 변경된다는 것이다.<br />
### 테스트하기
우선 해당 방식이 정말 먹히는지 시도해봐야한다.<br />
#### 순서도
1. 메인 페이지에서 secret 메모를 클릭하여 URL에 나오는 비밀 메모의 ID를 따로 저장한다.
2. dummy 메모를 하나 생성한다.
3. 방금 생성한 메모에 접근하여 Edit 페이지에 들어간다.
4. 개발자 도구를 이용해서 `selected_option` 옵션 중 하나의 값을 `"collections.secretID.title"`로 변경하고 선택한다.
5. Edit Data 칸에 원하는 타이틀을 입력한다.
6. dummy 메모의 비밀번호도 입력하고 저장한다.

이 방식으로 secret 메모의 title을 내가 원하는 문자열로 변경이 된 것을 확인할 수 있다.<br />
### 비밀번호 변경하기
이제 title 대신 진짜 password를 변경해서 접근할 수 있을 것 같다.<br />
한 번 시도해보면 여전히 접근할 수 없을 것이다. 인코딩 때문이다. <br />

```python
def check_password(self, password: str) -> bool:
	return self.data == hashlib.sha256(password.encode()).hexdigest()
```
비밀번호를 체크하는 함수는 항상 인자로 들어온 비밀번호를 `sha256` 해쉬 함수로 인코딩을 하고 검사를 한다. 애초에 Memo 객체의 Password 객체가 생성될 때부터 인코딩되기 때문이다.<br />
그래서 만약 비밀번호를 "123"으로 변경하고 "123"으로 접근한다면 "123"이 `sha256` 방식으로 변환돼 틀린 비밀번호라고 판정이 되는 것이다

```python
import hashlib

print(hashlib.sha256("pass".encode()).hexdigest())
```
그래서 이런식으로 내가 설정하고자 하는 비밀번호("pass")를 똑같이 인코딩한 값을 그대로 Edit Data에 넣어주어 해당 문제를 해결할 수 있다.
## 또 다른 풀이
사실 위에서 설명한 풀이는 Prototype Pollution이라고 하기에는 애매하고 그저 웹 앱 코드의 일종의 버그를 발생시키는 풀이이다. <br />

사실 파이썬에서는 Prototype Pollution이라기 보다는 Class Pollution이 더 맞는 표현이다.<br />
파이썬에는 `__class__`로 임의 객체의 속성 변조 혹은 `__globals__`를 이용한 전역변수 변조 등이 있다.<br />
### `__class__`
```python
__class__.collections.secretID.password
```
위에 언급했듯 `__class__`를 이용하면 임의 객체의 속성을 변조할 수 있다. 여기서 임의 객체는 /edit 라우터에서 `set_attr()`로 넘겨지는 memo 객체를 의미한다.
### `__globals__`
```python
__init__.__globals__.Memo.collections.secretID.password
```
단순 `__globals__`로 접근하면 app.py의 전역 변수에 접근하는 것이 아니라 models.py에 접근하게 된다. <br />
그래서 앞에 `__init__`을 추가해 비로소 app.py의 Memo 객체를 건드릴 수 있다.

---
이렇게 변경하고 나서 secret 메모에 들어가서 "pass"라는 비밀번호를 입력하면 올바르게 접속이 되고 플래그를 얻어낼 수 있다.
# 배운 것
Prototype Pollution이라는 것에 대해서 처음 알게됐고 자바스크립트 뿐만 아니라 파이썬에서도 비슷한 취약점을 이용할 수 있다는 것을 배웠다.