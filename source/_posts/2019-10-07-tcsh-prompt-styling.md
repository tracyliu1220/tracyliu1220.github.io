---
title: Tcsh Prompt 個人化
date: 2019-10-07 21:11:44
categories: Uncategorized
tags:
- linux
toc: true
---

最近學校好多作業都要使用系上的工作站
系上有提供能夠 ssh 進去的 Linux 跟 FreeBSD 空間
預設的 shell 是 tcsh
而且是很原生的 tcsh

多麼的原生呢？

![](https://i.imgur.com/KQnAHWk.png)

<!-- more -->

這麼的原生QQ，prompt 真的好醜QAQ
最致命的是它還是白色的，如果遇到 output 比較多的時候往上找 output 頭還會找到跟丟QAQ

覺得好看的 prompt 真的有必要，所以就去找了要怎麼改

## Default Prompt

跟 bash 不一樣的是，bash 的 prompt 設定是塞在一個叫 `PS1` 的環境變數裡，
而 tcsh 的環境變數就叫作 `prompt`，害我一開始搞錯方向ˊˋ

`echo $prompt`

如果是預設的話就會顯示下面這個

```
[%n@%m %c]%#
```

名詞解釋時間：
- `%n`: User name
- `%m`: The hostname up to the first '.'
- `%c`: The trailing component of the current working directory
- `%#`:
    - The first character of the promptchars shell variable for normal users.
    - '\#' for the superuser.

更多的可以到[這裡](https://stackoverflow.com/questions/33030442/how-can-i-change-my-tcsh-prompt-to-show-my-current-working-directory)看

## My New Prompt

為了永久設定，要在 home 目錄下建立一個 rc 檔案

`vim .cshrc`

裡面放入以下的內容：

```
set _green="%{\033[38;5;120m%}"
set _cyan="%{\033[38;5;123m%}"
set _white="%{\033[0m%}"
set prompt="${_green}%n@%m${_white}:${_cyan}%~${_white}%# "
```

前兩行是設定顏色的意思
`\033[38;5;${color_no}m` 是設定 front color 的意思，把 `${color_no}` 換成自己想要的顏色就好
`${color_no}` 要去 xterm-color 翻

如果電腦有 ruby 的話，可以下這個指令看 table ([reference](https://github.com/gawin/bash-colors-256))
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/gawin/bash-colors-256/master/colors)"
```

跑出來就會像以下這樣
![](https://i.imgur.com/OyLf4S8.png)

第三行的 `%{\033[0m%}` 是將顏色重設回預設的意思

而第四行就是設定 prompt 本體了
我把原來的 `%c` 換成 `%~`，會顯示比較完整的路徑，我覺得這樣比較能幫助我理解檔案們的相對位置關係

至於格式的部份我設成跟 Ubuntu 一樣，因為看習慣了ˊˇˋ
不過顏色我沒有完全照抄，Ubuntu 還有 Bold 的樣式而我只設定了 front color
覺得可以跟桌機有點不一樣，可以比較直覺的知道現在在哪裡做事

最後放上我的新 prompt 的截圖
![](https://i.imgur.com/qwtSsvi.png)

感覺有空的話可以再調一下資料夾跟執行檔的顏色
查到的[參考連結](https://blogs.aalto.fi/marijn/2016/07/05/add-a-splash-of-color-to-your-command-line-environment/)裡面就有提到作法，還有更詳細的 prompt 個人化的說明

