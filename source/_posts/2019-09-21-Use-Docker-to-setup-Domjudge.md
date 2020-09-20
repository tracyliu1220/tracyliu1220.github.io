---
title: 使用 Docker 架設 DOMjudge
date: 2019-09-21 11:07:41
categories: Uncategorized
tags:
- docker
- DOMjudge
toc: true
---

基於某些原因社團需要架設 DOMjudge，因為不想弄髒環境所以最後選擇建在 Docker 裏面
架設 DOMjudge 需要 3 個以上的 Container，[Domjudge 官網](https://hub.docker.com/r/domjudge/domserver/)上有幫我們整理好 Docker 指令要怎麼下

## MariaDB Container

```
docker run -it --name dj-mariadb -e MYSQL_ROOT_PASSWORD=rootpw -e MYSQL_USER=domjudge -e MYSQL_PASSWORD=djpw -e MYSQL_DATABASE=domjudge -e CONTAINER_TIMEZONE=Asia/Taipei -p 13306:3306 mariadb --max-connections=1000
```

<!-- more -->

用來放 DB 的 container 幾乎和官網上的一模一樣，只要多加 `-e CONTAINER_TIMEZONE=Asia/Taipei` 這項就好

## DOMserver Container

```
docker run -v /sys/fs/cgroup:/sys/fs/cgroup:ro --link dj-mariadb:mariadb -it -e MYSQL_HOST=mariadb -e MYSQL_USER=domjudge -e MYSQL_DATABASE=domjudge -e MYSQL_PASSWORD=djpw -e MYSQL_ROOT_PASSWORD=rootpw -e CONTAINER_TIMEZONE=Asia/Taipei -p 12345:80 --name domserver domjudge/domserver:latest
```

接著是 DOMserver 本體，也是只要加上 `-e CONTAINER_TIMEZONE=Asia/Taipei` 就好

### Find the Passwords

接著跑下面這行進入 DOMserver 虛擬機本體

```
docker container exec -it domserver bash
```

找到 `opt/domjudge/domserver/etc/initial_admin_password.secret` 和 `/opt/domjudge/domserver/etc/restapi.secret` 兩個檔案
裡面分別是 admin 和 judgehost 的密碼
（DOMjudge 7.0.0 之後 admin 密碼就不是預設 admin 了）

## Judgehost Container

```
docker run -it --privileged -v /sys/fs/cgroup:/sys/fs/cgroup:ro --name judgehost-0 --link domserver:domserver --hostname judgedaemon-0 -e DAEMON_ID=0 -e CONTAINER_TIMEZONE=Asia/Taipei -e JUDGEDAEMON_PASSWORD=${judgedaemon_password} domjudge/judgehost:latest
```
其中 `${judgedaemon_password}` 要換成剛剛在 DOMserver 本體內撈到的 judgehost 密碼
Judgehost 可以架好多個，把上面 `name`、`hostname`、`DAEMON_ID` 的參數調整一下就好

## Connect to the Server

在瀏覽器中打開 `http://localhost:12345` 就能連到 DOMjudge的網頁啦
要登入 admin 的話要在 user 打 admin，password 就是剛剛撈到的
