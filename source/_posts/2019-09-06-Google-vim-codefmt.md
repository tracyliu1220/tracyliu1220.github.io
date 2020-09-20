---
title: Google vim-codefmt
categories: Uncategorized
tags:
  - vim
date: 2019-09-06 14:46:17
toc: true
---

[Google vim-codefmt](https://github.com/google/vim-codefmt)

## Commands

`:FormatCode <TAB>`

## Installation

### With [vim-plug](https://github.com/junegunn/vim-plug)

`vim ~/.vimrc`

<!-- more -->

```
call plug#begin('~/.vim/plugged')

Plug 'google/vim-maktaba'
Plug 'google/vim-codefmt'
Plug 'google/vim-glaive'

call plug#end()
```

`:PlugInstall`

### Autoformatting

`vim ~/.vimrc`

```
augroup autoformat_settings
  autocmd FileType bzl AutoFormatBuffer buildifier
  autocmd FileType c,cpp,proto,javascript AutoFormatBuffer clang-format
  autocmd FileType dart AutoFormatBuffer dartfmt
  autocmd FileType go AutoFormatBuffer gofmt
  autocmd FileType gn AutoFormatBuffer gn
  autocmd FileType html,css,sass,scss,less,json AutoFormatBuffer js-beautify
  autocmd FileType java AutoFormatBuffer google-java-format
  autocmd FileType python AutoFormatBuffer yapf
augroup END
```

### Installing Formatters

#### clang-format

`sudo apt install clang-format`

`vim ~/.vimrc`

```
call glaive#Install()
Glaive codefmt clang_format_executable='/usr/bin/clang-format'
```
