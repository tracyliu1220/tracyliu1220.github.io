---
title: 使用 travis-ci 自動部署 Hexo Blog 到 GitHub Page
categories: Uncategorized
toc: true
date: 2021-03-22 16:57:32
tags:
- hexo
- travis-ci
---

## Hexo 部署到 GitHub Page

Hexo 部署到 GitHub Page 應該算是最常見的方式了
我之前都是照[這篇](https://medium.com/@bebebobohaha/%E4%BD%BF%E7%94%A8-hexo-gitpage-%E6%90%AD%E5%BB%BA%E5%80%8B%E4%BA%BA-blog-5c6ed52f23db)最後面的方式部署
但我在去年決定改成使用 travis-ci 來幫我自動化的將部落格丟到伺服器上

## 我的 Hexo Blog git 架構

我延續了使用叫做 \<username\>.github.io 的 repository，
Branch 名稱與規劃如下：
- Branch `site`: source，markdown 稿子
- Branch `master`: 由 `hexo generate` 產生的靜態檔案（伺服器位置

<!-- more -->

所以我的 travis-ci 需要做的事有
- 在 `site` 這個 branch 將 npm 的 dependencies 安裝好
- 在 `site` 這個 branch 使用 `hexo generate` 來產生靜態檔案
- 將 `hexo generate` 產生的資料夾 `public` 整包內容丟到 branch `master`

## travis-ci 設定

### 取得 GitHub Personal Access Token

Settings \> Developer settings \> Personal access tokens 點選 Generate new token，
勾選 public\_repo 之後產生 token

### 將 Token 當作環境變數放入 travis-ci 環境

找到相對應的 repository 點選 settings

![](https://i.imgur.com/19OaghG.png)

接著找到 Environment Variables 的區塊，
將剛剛產生的 token 內容複製進去並命名為 `GITHUB_TOKEN`

![](https://i.imgur.com/tTInsPD.png)

## .travis.yml 內容

接下來我們要在 branch `site` 這邊的根目錄下新增一個叫 `.travis.yml` 的檔案，
之後 travis-ci 在我們每次 push 之後都會自己去看這個檔案，
執行我們 deploy 想做的動作

```
sudo: false
language: node_js
node_js:
  - 12
cache:
  - npm
branches:
  only:
    - site

script:
  - npm install
  - hexo generate

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN
  keep-history: false
  committer_from_gh: true
  target_branch: master
  on:
    branch: site
  local-dir: public
```

## Results

完成之後我們在每次 push 之後來 https://travis-ci.com/github/\<username\>/\<username\>.github.io
這個網址看我們的 deploy 有沒有成功還有 deploy 的進度
如果出現下圖綠色的樣子就代表成功了！

![](https://i.imgur.com/LBejbmj.png)
