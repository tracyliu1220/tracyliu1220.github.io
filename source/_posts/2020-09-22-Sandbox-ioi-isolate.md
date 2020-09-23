---
title: isolate Sandbox 使用
categories: [Projects, MyOJ]
toc: true
date: 2020-09-22 20:32:52
tags:
- sandbox
- isolate
---

## Sandbox

Sandbox 的中文名字就是直譯過來的沙箱
他能讓程式在一個隔離的環境中執行
隔離的意思就是有限的路徑、有限的記憶體甚至是有限的 process 數量等等
通常是拿來測試一些不受信任的程式
讓我們就算執行了惡意程式也不會影響到我們的作業系統或環境

今天依然是在寫 OJ 中 judgehost 的部份
因為 judgehost 需要跑別人 submission 送來的程式
如果有人送個惡意程式的話就糟糕了
所以就需要用到 sandbox
把大家的 submission 放在 sandbox 裡面跑

## isolate

isolate 是 ioi 的那群人開發的一個[開源 sandbox](https://github.com/ioi/isolate)
甚至還有蠻不錯的 [manual page](http://www.ucw.cz/moe/isolate.1.html)
前陣子發現還不錯的 judgehost 開源 API —— [Judge0](https://github.com/judge0) 也是使用 isolate 搭配 Ruby on Rails 實現的

<!-- more -->

### Installation

首先是安裝

```bash
$ git clone https://github.com/ioi/isolate.git
$ apt-get install -y libcap-dev
$ cd isolate && make install
```

其中 `libcap-dev` 是它需要的 dependency
另外他還需要 `make` 跟 `gcc` 能讓 `make install` 這個指令順利跑
沒有的話要裝一下

接下來是一點關於 docker 的部份

Installation 這部份其實在[前一篇的 docker 設定](../../21/2020-09-21-Use-docker-compose-to-Build-Development-Environment/)中有稍微出現過
所以可以在那篇 `Dockerfile` 的 configuration 中看到幾乎一模一樣的：

```dockerfile
# isolate
RUN git clone https://github.com/ioi/isolate.git
RUN apt-get install -y libcap-dev
RUN cd isolate && make install && cd ..
```

如果要在 docker container 中跑的話
`docker run` 這個指令要記得加 `--privileged` 的參數
或者在 `docker-compose.yml` 中把 `privileged` 設為 `true`
讓 docker root 擁有真正 root 的權限才能創建 cgroup

### Initialization

```bash
$ isolate --cg --init
```
這個指令下下去後會 stdout 給我們一個路徑，通常是 `/var/local/lib/isolate/0`
而 `/var/local/lib/isolate/0/box` （這裡多了一個 `box` 喔！）就會是 sandbox 能夠運用的路徑
白話一點的意思就是所有在 sandbox 裡的程式都只能跑在這個資料夾下面

這邊特別要提一下
在 [manual page](http://www.ucw.cz/moe/isolate.1.html) 的最前面 initialization 的指令只有 `isolate --init` 而已，這個指令一樣會回給我們路徑
但是 `isolate --init` 的 sandbox 只能跑單一 process 的程式
而因為我 sandbox 內會需要跑類似 `g++ test.cpp` 這樣子之類的指令
這些指令需要不只一個 process （會有 fork）
所以我們需要 cgroup 幫我們把資源切好出來給我們跑才行
這裡的 `--cg` 就是要使用到 cgroup 的意思

### Run

isolate 能直接下 `--run` 跑的檔案**只能是執行檔**
下面來舉一個例子

{% codeblock compile lang:bash %}
#!/bin/bash
g++ test.cpp
{% endcodeblock %}

`compile` 是一個可執行的 shell script （`chmod +x compile`）
裡面的動作是用 `g++` 編譯 `test.cpp` 這個檔案

{% codeblock test.cpp lang:cpp %}
#include <bits/stdc++.h>
using namespace std;

int main() {
    cout << "Hello C++" << endl;
}
{% endcodeblock %}

以上是 `test.cpp` 的檔案內容

```bash
$ isolate --run -p -E PATH=$PATH --meta=meta.txt -- compile
OK (0.831 sec real, 0.851 sec wall)
```

直接執行這個指令，成功執行的話就會得到 `OK` 並在 `/var/local/lib/isolate/0/box` 的路徑下出現編譯好的 `a.out`

參數
- `-p`: 允許多個 process
- `-E`: 指定 environment variable，我這邊把 `PATH` 直接送進去讓它可以直接使用 `g++`
- `--meta`: sandbox 程式執行的情況，裡面會有 `exitcode` 之類的資訊，如果有用 manual page 中一些時間或空間上的限制，還會出現以下的 `status` 資訊
    - `RE`: run-time error, i.e., exited with a non-zero exit code
    - `SG`: program died on a signal
    - `TO`: timed out
    - `XX`: internal error of the sandbox

以下是上面那個例子的 `meta.txt`，注意 `meta.txt` 會出現在**下指令的當前目錄**的相對位置而不是 sandbox 中

{% codeblock meta.txt %}
time:0.824
time-wall:0.841
max-rss:154216
csw-voluntary:19
csw-forced:17
exitcode:0
{% endcodeblock %}


### Clean Up

```bash
isolate --cg --cleanup
```
下這個就可以把 sandbox 移除了
注意這個指令會讓 `/var/local/lib/isolate/0` 整個不見

