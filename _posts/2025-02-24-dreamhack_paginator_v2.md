---
title: "드림핵 lv.1 - Paginator v2"
date: 2025-02-24 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, sql injection] 
description: 드림핵 Paginator v2 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/1810](https://dreamhack.io/wargame/challenges/1810)
# 문제 설명
**이 문제는 제 30회 해킹캠프 `박기범 - 웹 해킹의 첫 발걸음` 실습 플랫폼에 출제된 문제입니다.**

페이지를 보여주는 시스템이에요.  
숨겨진 내용도 있다고요?  
찾아주세요... ;(

플래그 형식은 `HACKCAMP{...}` 입니다.

---
# 문제 풀이
## 코드 분석
해당 문제에서 제공하는 파일들은 총 4개이다. `app.py`, `docker-compose.yml`, `Dockerfile`, 그리고 `flag`.<br />
flag에는 당연히 진짜 플래그는 존재하지 않는다. app.py를 제외한 나머지 2개의 파일은 서버 설정 파일이라 중요하지 않다.
### app.py
```python
from flask import Flask, request, g
import sqlite3

app = Flask(__name__)
DATABASE = 'db.db'

FLAG = open("flag", "r", encoding="utf-8").read().strip()

def get_db():
    db = getattr(g, "_database", None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
    return db

def init_db():
    db = get_db()
    try:
        db.execute("""
            CREATE TABLE IF NOT EXISTS posts (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT,
                content TEXT
            )
        """)

        db.execute("""
            CREATE TABLE IF NOT EXISTS flag (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                secret_flag TEXT
            )
        """)

        db.executescript("""
            INSERT INTO posts (title, content) VALUES ('Welcome', 'This is the first post!');
            INSERT INTO posts (title, content) VALUES ('About', 'This is an about page.');
            INSERT INTO posts (title, content) VALUES ('Contact', 'Contact us at admin@example.com');
        """)

        db.execute("INSERT OR IGNORE INTO flag (id, secret_flag) VALUES (1, ?)", (FLAG,))
        db.commit()
    except Exception as e:
        print("[ERROR] Database initialization failed:", e)

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, "_database", None)
    if db is not None:
        db.close()


@app.route('/')
def index():
    return "<strong>/post</strong> pls!<br>pls like and subscribe my post lol xd"

@app.route('/post')
def post():
    post_id = request.args.get("id", "")

    if not post_id:
        return "Missing post ID."

    try:
        db = get_db()
        query = f"SELECT title, content FROM posts WHERE id = {post_id}"
        result = db.execute(query)
        row = result.fetchone()

        if row:
            return f"<h2>{row[0]}</h2><p>{row[1]}</p>"
        else:
            return "Post not found."
    except Exception as e:
        return f"Database Error: {e}"

if __name__ == '__main__':
    with app.app_context():
        init_db()
    app.run(host="0.0.0.0", port=31004)
```
서버의 주 축을 담당하는 `app.py` 코드이다. main 파트를 보면 `init_db()` 함수로 서버가 실행되면 데이터베이스 하나가 새로 만들어지고 기본 값들이 입력된다.

```python
        db.executescript("""
            INSERT INTO posts (title, content) VALUES ('Welcome', 'This is the first post!');
            INSERT INTO posts (title, content) VALUES ('About', 'This is an about page.');
            INSERT INTO posts (title, content) VALUES ('Contact', 'Contact us at admin@example.com');
        """)

        db.execute("INSERT OR IGNORE INTO flag (id, secret_flag) VALUES (1, ?)", (FLAG,))
```
총 만들어지는 테이블의 수는 2개이다. `posts`와 `flag`.<br />
그리고 각 테이블이 가지는 기본 값은 위와 같다. `flag`라는 테이블에 `FLAG` 값이 들어가기 때문에 우리의 목표는 `flag` 테이블에 접근하여 값을 가져오는 것이다.

**취약점**
```python
@app.route('/post')
def post():
    post_id = request.args.get("id", "")

    if not post_id:
        return "Missing post ID."

    try:
        db = get_db()
        query = f"SELECT title, content FROM posts WHERE id = {post_id}"
        result = db.execute(query)
        row = result.fetchone()

        if row:
            return f"<h2>{row[0]}</h2><p>{row[1]}</p>"
        else:
            return "Post not found."
    except Exception as e:
        return f"Database Error: {e}"
```
`app.py`를 살펴보면 아래쪽에 `/post` 엔드 포인트를 처리하는 코드가 보인다. url에서 id 값을 가져와서 해당 id와 매치하는 포스트를 출력하는 코드이다. <br />
하지만 여기서 유저로부터 받은 `post_id`가 필터링이 되지않고 곧바로 SQL 쿼리 문에 들어가기 때문에 **SQL injection** 공격이 발생할 수 있다.<br />
# 최종 풀이
우리는 여기서 `f"SELECT title, content FROM posts WHERE id = {post_id}"`가 flag 테이블 값을 출력하도록 유도해야 한다.<br/>

결론부터 이야기하자면 최종 쿼리는 다음과 같다.
```sql
SELECT title, content FROM posts WHERE id = 1 UNION SELECT null, (SELECT secret_flag FROM flag WHERE id = 1)
```
그러므로 `post_id`가 `UNION SELECT null, (SELECT secret_flag FROM flag WHERE id = 1)`가 된다는 말이다.

UNION을 이용하면 두 번째 쿼리의 값이 첫 번째로 덮어씌워진다. <br />
쿼리 `SELECT title, content FROM posts WHERE id = 1`는 title로 "Welcome", content로 "This is the first post!"를 가지는 데이터를 하나 가져온다는 의미이다.<br />

이때 `UNION`을 사용해서 `title`을 `null`로 만들고 content를 (SELECT secret_flag FROM flag WHERE id = 1)로 치환하여 플래그를 얻는 원리이다.<br />

컬럼 이름으로 secret_flag나 flag의 id가 1이라는 정보들은 `init_db()` 함수를 잘 살펴보면 알 수 있다.


```
None // title

HACKCAMP{redacted} // content
```
url 칸에 `/post?id="UNION QUERY"`를 입력하면 이런식으로 플래그가 출력되는 것을 알 수 있다.

# 배운 것
SQL 배운지 오래됐는데 오랜만에 UNION SELECT에 대해 다시 공부할 수있는 기회가 됐다.