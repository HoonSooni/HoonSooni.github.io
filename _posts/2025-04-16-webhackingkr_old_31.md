---
title: "webhacking.kr old 31 문제 풀이"
date: 2025-04-16 08:22:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking]
description: webhacking.kr old 31 문제 풀이
---

[https://webhacking.kr/challenge/web-16/?server=119.192.89.197](https://webhacking.kr/challenge/web-16/?server=119.192.89.197)
# 문제 풀이
```
$port = rand(10000,10100);
$socket = fsockopen($_GET['server'],$port,$errno,$errstr,3) or die("error : {$errstr}");

  
**Warning**: fsockopen(): unable to connect to 119.192.89.197:10078 (No route to host) in **/var/www/html/challenge/web-16/index.php** on line **23**  
error : No route to host
```
문제 페이지에 접속을 해보면 위 같은 문구가 출력된다.<br />
php 코드의 짧은 부분을 보여주고 그 아래에는 에러메세지이다.<br />

코드 먼저 보면, 10000에서 10100 사이의 값을 랜덤으로 가져와 포트 값으로 사용하고 URL 파라미터로 받은 `server` 값에 `fsockopen()` 함수를 이용하여 접속하는 코드이다.<br />
아래에 에러 메세지를 살펴보면 `fsockopen()` 함수가 연결을 실패했다고 한다. 10078 포트에 연결을 시도했는데 실패했다.<br />

계속해서 새로고침을 하여 혹시나 알맞는 포트를 찾을 수 있을지 테스트 해보았는데 아무리해도 모르겠다.<br />
## 시도
### 여러번 새로고침하여 에러메세지가 안뜨는 포트 필터링하기
```python
from multiprocessing import Process, Manager, Lock
import requests
import bisect
import re

# Preventing from inserting already existing value into the list
def insert_no_duplicates(lst, value, lock):
    with lock:
        idx = bisect.bisect_left(lst, value)
        if idx == len(lst) or lst[idx] != value:
            lst.insert(idx, value)

# Multithreading Function
def process(url, ports, lock):
    for _ in range(20):
        resp = requests.get(url)

        if "unable to connect" in resp.text:
            match = re.search(r':(\d+)', resp.text)
            if match:
                port = int(match.group(1)) % 10000
                insert_no_duplicates(ports, port, lock)
        else:
            print(resp.text)

if __name__ == '__main__':
    url = "https://webhacking.kr/challenge/web-16/?server=119.192.89.197"

    manager = Manager()
    ports = manager.list()
    lock = Lock()

    processes = []
    for _ in range(10):
        p = Process(target=process, args=(url, ports, lock))
        processes.append(p)
        p.start()

    for p in processes:
        p.join()

    # Printing out suspicious ports
    suspicious_ports = []
    for i in range(1, 101):
        if not i in ports:
            suspicious_ports.append(i)
    
    print(suspicious_ports)
```
이 때문에 참 괴랄한 코드를 짜보았다.<br />
스레드 하나로 진행하면 10번 접속시도하는 것 조차 은근 오래걸려서 멀티스레딩을 시도했다.<br />

코드를 대충 요약하면 스레드를 10개로 나누고 각 스레드마다 총 20번의 접속을 시도한다.<br />
그리고 에러가 발생했을 시에 접속이 불가한 포트들을 따로 모아둔다. 이때 한 리스트에 모으는데 이 리스트는 항상 정렬돼있고 중복되는 수를 제거한다.<br />

맨 마지막에 1부터 100까지 반복을 돌면서 그 리스트에 어떤 값이 없는지 체크한다.<br />
값이 리스트에 없다는 것은 그 중 작동하는 포트가 있을 수 있다는 의미이기 때문이다.<br />

그렇게 해당 코드를 한 5번 정도 돌려보고 결과에 중복되는 포트 값이 있는지 확인해보았다.<br /> 계속해서 중복되는 포트가 바로 접속이 가능한 포트이기 때문이다.<br />
하지만 아무리 찾아봐도 결과 리스트 중 중복되는 값을 찾기가 힘들었다. 물론 있긴 하지만 매번 중복되는 것이 아니라서 이상함을 감지할 수 있었다.
### 소켓 연결 시도
```python
import socket

host = "119.192.89.197"

for port in range(10000, 10101):
    try:
        # Create a socket object (IPv4, TCP)
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(3)  # optional timeout in seconds

        # Connect to the server
        s.connect((host, port))
        print(f"Connected to {host}:{port}")

        s.close()

        print(port)

    except socket.error as err:
        print(f"Connection error: {err}")
```
이번에는 내가 직접 `fsockopen()` 함수와 동일한 코드를 만들어보았다.<br />
10000부터 10100까지 모든 포트에 접속을 시도해보았는데 연결에 성공한 포트가 단 하나도 발견되지 않았다.<br />

역시나 이상하다.<br />
## 최종 풀이
생각해보니까 내가 직접 server 값을 조작할 수가 있다.<br />
한 번 내 컴퓨터의 포트를 하나 열어서 연결을 기다려볼까 싶어서 그렇게 해보니 플래그를 얻어낼 수 있었다.<br />