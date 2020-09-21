---
title: 使用 docker-compose 建立開發環境
categories: [Projects, MyOJ]
toc: true
date: 2020-09-21 07:21:46
tags:
- docker
---

## 動機

最近覺得自己的開發技能還有很多欠缺的地方
尤其是完整的開發流程
以往都是直接在自己的本機開發
也沒有使用 CI/CD

然後又突然想寫寫看 OJ
於是我打算直接做中學
並做以下幾個改變

- 使用 docker-compose 建立開發環境
- 善用 environment variables
- 做 testing 以及 CI/CD
- 認真寫 git commit messages
- 將學到的東西跟踩到的坑寫成 blog 文章

<!-- more -->

大概就是一些提升開發 sense 的東西XD
當然這些技能很多都是要從頭學的
例如今天的 docker 就是以前使用過別人的但沒有自己寫過

## 架構

暑假的時候大概看過資料
一般 OJ 除了傳統上的的前後端還多了一個 judgehost
雖然最陌生，但想了想好像 judgehost 比較不複雜
所以我決定先從 judgehost 開始寫
又因為 judgehost 通常是計算最繁重的
所以我選了相對輕量的 Flask 作 server
ioi 開源的 isolate 套件作我的 sandbox

這篇主要是記錄我的 docker 環境如何架設

## File Structure

```
.
├── app
│   └── our application codes here
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```

在 judgehost 目錄下有一個子資料夾 `app` 和建環境要用的檔案
`app` 下就放了跟 judgehost application 相關的 code（跟 Flask 有關的 code 都在這裡）

## `Dockerfile`

`Dockerfile` 是用來 build docker image 的

```dockerfile
FROM ubuntu:20.04
WORKDIR /app

ADD requirements.txt /app

RUN apt-get update
# pip packages
RUN apt-get install -y python3-pip
RUN pip3 install -r requirements.txt
# utils
RUN apt-get install -y git
RUN apt-get install -y make
RUN apt-get install -y gcc
RUN apt-get install -y --no-install-recommends vim
RUN apt-get install -y --no-install-recommends redis-server
# isolate
RUN git clone https://github.com/ioi/isolate.git
RUN apt-get install -y libcap-dev
RUN cd isolate && make install && cd ..

EXPOSE 5000
```

- `FROM`: 初始 image，這邊使用跟我桌機一樣的 ubuntu:20.04 當開發環境的 os
- `WORKDIR`: 建立一個資料夾並將這個資料夾作為主要工作區（docker 會自動 cd 進去）
- `ADD`: 這裡是將 host 中的 `requirements.txt` 檔案加到 docker image 的 `/app`
- `RUN`: 一些 build image 時才會跑的初始 command，通常都是一些裝東西的用途
- `EXPOSE`: 告訴使用這個 image 的人我們的 application 會使用哪些 port

## `docker-compose.yml`

`docker-compose` 這個 command 是用來將 images 建立成可以跑的 docker containers
也就是代替了 `docker run` 這個指令

```yaml
version: '3'

services:
  judgehost:
    build:
      context: .
    environment:
      - FLASK_APP=judgehost.py
      - FLASK_RUN_HOST=0.0.0.0
      - FLASK_ENV=development
      - REDIS_DB_HOST=redis
      - SANDBOX_ROOT=/var/local/lib/isolate/
    command: bash -c 'isolate --cg --init && flask run'
    ports:
      - '5000:5000'
    volumes:
      - './app:/app'
    tty: true
    privileged: true
    depends_on:
      - 'redis'
  redis:
    image: 'redis:alpine'
```

這個 `docker-compose.yml` 檔案定義了
- `judgehost`、`redis`: 兩個服務，這兩個服務分別會用一個 container 去 run
- `judgehost`
    - `build`: `judgehost` 這個服務的 image 要用剛剛提到的 `Dockerfile` 去 build，而 `context` 的 `.` 代表 `Dockerfile` 所在的目錄位置，也就是當前目錄
    - `environment`: 一些環境變數
    - `command`: 每次跑 `docker-compose up` 時會在 container 中跑的指令
    - `ports`: Port forwarding，`host_port:container_port`，其中 host 是本機
    - `volumes`: 將本機的 `./app` 資料夾 mount 進 container 的 `/app` 資料夾，也就是說這兩邊會同步，改了其中一邊另外一邊也會跟著變動
    - `tty`: 這邊設為 `true`，它的意義我還不是很清楚，但知道它可以讓 container 在執行完 command 後不立即結束
    - `previleged`: 讓 container 中的 root 擁有真正的 root 權利
    - `depends_on`: 這個服務會需要用到其他哪些服務，這裡因為 `judgehost` 會需要使用 `redis` 的服務，`docker-compose` 在建立的時候就會先建立 `redis`
- `redis`
    - `image`: 因為是直接使用別人已經建立好的 image，所以就直接設定 image 是什麼而不用 build 了

### `docker-compose` 相關指令

```
docker-compose build
```
build 會將 `docker-compose.yml` 中需要用 `Dockerfile` build 的 images 建立起來，建立好後用 `docker images` 的指令可以看到

```
docker-compose up
```
images 建立好後下這行，會將 container 們建立起來並用虛擬網路連接

```
docker-compose up -d
```
這個功能同上，但是加了 `-d` 的選項就會在背景執行，也不會有 log 跑出來

```
docker-compose down
```
移除 container 們以及虛擬網路

## 一些踩到的坑

### `ADD requirements.txt`

我一開始以為只要透過 `volumes` 那邊將本機的檔案 mount 過去
再在 container 那邊 `RUN pip3 install -r requirements.txt` 就可以把 image build 起來
但實際上在 build 時只跟 `Dockerfile` 有關而與 `docker-compose.yml` 無關
所以在 build 階段是看不到即將要被 mount 上去的檔案的
因此需要 `ADD requirements.txt` 將 `requirements.txt` 加到 container 的 `WORKDIR` 中再去執行 `pip3`

### `FLASK_RUN_HOST=0.0.0.0`

docker 在做 port forwarding 的時候預設是將 `0.0.0.0:container_port` 拉到 `host_port`
但 Flask 預設是跑在 127.0.0.1
所以要多設定 `FLASK_RUN_HOST` 的環境變數讓它可以跑在 `0.0.0.0` 才 forward 得出來

## 碎碎唸

其實 docker 這個東西之前在參加社團時已經用過了
但是完全都是社團學長們寫好我們只要把用他們的東西把環境架起來而已
真的不知道裡面在幹嘛XD
自己寫一遍真的有恍然大悟的感覺

下一篇應該會來寫寫 sandbox 怎麼用
希望這個禮拜有時間寫
