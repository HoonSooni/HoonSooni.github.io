---
title: "드림핵 lv.1 - KTX Notes"
date: 2025-02-23 00:21:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking] 
description: 드림핵 KTX Notes 웹해킹 워게임 풀이
---

# 문제 설명
**이 문제는 제 30회 해킹캠프 `박기범 - 웹 해킹의 첫 발걸음` 실습 플랫폼에 출제된 문제입니다.**

This Note Is Fast And Convenient as KTX... rlly...

플래그 형식은 `HACKCAMP{...}` 입니다.

---
# 문제 풀이
## 웹사이트 분석
![KTX Notes Main Page](https://1drv.ms/i/c/5cb37aa515b56a00/IQT5P_LNxYppTY6JJYhYPi2YAZWT6MWFBpXP_K5FU-2IXqI?width=660)
<br />
드림핵에서 제공하는 VM 서버를 실행하고 접속하면 해당 화면이 뜨게된다.<br />
아마 겉으로만 봐서는 *Save a Note* 부분에 파일 이름과 내용을 적고 Save를 넣으면 유저가 적은 내용 그대로 파일을 생성하는 것으로 보인다. <br />
*Download a Note*는 내가 만든 파일의 이름을 적으면 해당 파일을 다운로드 받을 수 있는 기능인 것 같다.

## 코드 분석
### app.py
```python
NOTES_DIR = "notes"
if not os.path.exists(NOTES_DIR):
    os.makedirs(NOTES_DIR)

# ...

@app.route('/save', methods=['POST'])
def save():
    filename = request.form.get('filename')
    content = request.form.get('content')
    
    if filename and content:
        filepath = os.path.join(NOTES_DIR, filename)
        if not os.path.exists(filepath):
            with open(filepath, 'w') as f:
                f.write(content)
        return send_file(filepath, as_attachment=True)
    
    flash("Error: Filename and content are required.", "danger")
    return redirect(url_for('index'))

@app.route('/download', methods=['GET'])
def download():
    filename = request.args.get('filename')
    
    if filename:
        filepath = os.path.join(NOTES_DIR, filename)
        try:
            return send_file(filepath, as_attachment=True)
        except FileNotFoundError:
            flash("Error: File not found.", "danger")
            return redirect(url_for('index'))
    
    flash("Error: Filename is required.", "danger")
    return redirect(url_for('index'))
```
파일의 길이가 이거보단 길지만 중요해 보이는 부분만 우선 가져와보았다.<br />
여기서 주목해야 할 점은 서버의 유저 인풋(filename)의 처리 방식이다. `save()`나 `download()` 둘 다 유저로부터 파일 이름을 받으면 이것을 `NOTES_DIR`과 결합하여 file path를 만드는 방식이다.

즉, 만약 유저가 filename으로 `"name"`을 적고 save를 했다면 해당 파일은 서버의 `"/notes/name"` 위치에 저장된다. <br />
download도 마찬가지이다. filename을 인풋으로 받고 해당 인풋을 `"/notes/{input}"`으로 치환하여 해당 파일을 다운로드 할 수 있게 제공해주는 것이다.

여기서 **취약점**이 발생한다.<br />
유저의 filename input을 필터링하고 있지 않기 때문에 유저가 해당 filename으로 서버의 각종 파일을 돌아다니며 정보를 탈취할 수 있다.
### Dockerfile

```python
# 1. Python 베이스 이미지 사용
FROM python:3.9-slim

# 2. 작업 디렉터리 설정
WORKDIR /app

# 4. 의존성 설치
RUN pip install flask

# 5. Flask 애플리케이션 소스 파일 복사
COPY . .

# 6. Flask 앱 실행
CMD ["python", "app.py"]
```
서버의 파일들 중 Docker 설정 파일을 보면 작업 디렉토리가 `/app`이란 것을 알 수 있다.<br />
그렇다는 말은 `flag`의 위치가 `/app/flag`라는 것을 간접적으로 알 수 있게된다.
## 최종 풀이
Download a Note 부분에 "/app/flag"를 입력하고 버튼을 눌러 플래그 파일을 다운로드 받는다.<br/>
그러고 나서 다운로드 받은 flag 파일을 출력해보면 그 안에 들어있는 플래그를 찾아낼 수 있다.
```shell
cat flag
HACKCAMP{REDACTED}
```