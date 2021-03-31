---
title: 實作 Flask 後端中的 Hash ID 需求
categories: Uncategorized
toc: true
date: 2021-03-30 23:02:16
tags:
- flask
- crypto
---

## Hash ID

上個學期有個小組的學期作業，要寫一個雲端服務，
我們組的主題是實作出一個功能較多又比較兼顧 UI 的 [when2meet](https://www.when2meet.com/)

在實作分享功能的時候，
我們希望有一個頁面的資訊（一個叫做 meeting 的 model）可以用網址分享，
讓沒登入的人也可以看得到，
但是這樣的分享方式如果直接用 restful api 的標準的話，
直接使用 `api/meetings/<meeting_id>` （這裡 `<meeting_id>` 是整數）當網址來呼叫需要的 api，
會讓不相干的人直接在 `<meeting_id>` 打上數字戳進來

所以要使用 hash\_id 來實作，
最簡單的方式是直接隨機 random 出一串字串當作 model 的 id，
但我總覺得這樣做很不舒服，
所以我選擇直接拿原來的 id 做 encrypt

<!-- more -->

流程會變成像下面這樣：

後端 -> id -(encrypt)-> hash\_id -> user

至於 encrypt 的方式我選擇使用最簡單的 DES，
多虧大二時的密碼學有稍微認真一下才知道這個名詞XD

## DES Encrypt/Decrypt using Python

這個 section 要來記錄一下如何用 python 從 integer id 轉換成 hash\_id 的字串（16 進位表示），
又要如何從 hash\_id 轉回 id

### Installation

```
pip3 install pycryptodome
```

### Import

```python
import binascii
from Crypto.Cipher import DES
```

### Encrypt (id -> hash\_id)

```python
id = 1

secret_key = '12345678'                       # 設定一個 secret_key，長度只能為 8
secret_key = secret_key.encode()              # secret_key = b'12345678'，將 secret_key 從 string encrypt 成 bytes

des = DES.new(secret_key, DES.MODE_ECB)       # 建立一個 DES 物件

plain = str(id)                               # 將 id 轉成 string
plain = (16 - len(plain)) * '0' + plain       # 前面補零補到 16 個位數
enc = des.encrypt(plain.encode())             # 一樣將 plain 轉乘 bytes 之後再做 DES encrypt（des 只吃 bytes）
print(enc)                                    # b'_j}R\x8e9O9<\x11H\xc3U\x0b8\x1f'

enc = binascii.hexlify(enc)                   # 因為 des 回傳的東西是 bytes 編碼非常不好看所以我們再使用 hexlify 的編碼讓它看起來只有數字 0~9 還有字母 a~f
print(enc)                                    # b'5f6a7d528e394f393c1148c3550b381f'

enc = enc.decode()                            # 需要用 bytes 編碼的部分結束了，將它轉回 string
print(enc)                                    # 5f6a7d528e394f393c1148c3550b381f

enc = enc.upper()                             # 個人喜好喜歡把他們都轉成大寫，不轉也可以
print(enc)                                    # 5F6A7D528E394F393C1148C3550B381F
```

注意 `des.encrypt()` 跟 `binascii.hexlify()` 傳入的東西都需要是 bytes 編碼

### Decrypt (hash\_id -> id)

知道 encrypt 的步驟後 decrypt 就相對簡單了，
倒著做回來就好

```python
hash_id = '5F6A7D528E394F393C1148C3550B381F'

secret_key = '12345678'                       # 設定一個 secret_key，要跟 encrypt 時相同
secret_key = secret_key.encode()              # secret_key = b'12345678'，將 secret_key 從 string encrypt 成 bytes

des = DES.new(secret_key, DES.MODE_ECB)       # 建立一個 DES 物件

dec = binascii.unhexlify(hash_id.lower().encode())
dec = des.decrypt(dec).decode()

id = int(dec)                                 # id = 1
```

## Flask Model

再來以當時那個 project 當例子，
來看看在 Flask 中是怎麼實作的，
完整版請點[這裡](https://github.com/tracyliu1220/MeetingElf-backend/blob/main/app/mod_meeting/models.py)

```python
class Meeting(db.Model):
  id = db.Column(db.Integer, primary_key=True)

  # 其他 attributes 在這裡先略過

  @property
  def hash_id(self):
    des = DES.new(app.config['MEETING_HASH_KEY'], DES.MODE_ECB)
    plain = str(self.id)
    plain = (16 - len(plain)) * '0' + plain
    enc = des.encrypt(plain.encode())
    enc = binascii.hexlify(enc).decode().upper()
    return enc

  @staticmethod
  def get_id(hash_id):
    des = DES.new(app.config['MEETING_HASH_KEY'], DES.MODE_ECB)
    try:
      dec = binascii.unhexlify(hash_id.lower().encode())
      dec = des.decrypt(dec).decode()
      id = int(dec)
    except:
      return None
    return id
```

基本上跟前一個 section 一樣，
只是 `get_id` decrypt 的時候用 `try` 多檢查一下有沒有 decrypt 成功

在 `hash_id` 那個 function 我加上了 decorator `@property`，
將 `hash_id` 設為只能讀取不能修改，
因為雖然它的意義很像一個真的存在的 attribute，
因為實際上並沒有一個叫 `hash_id` 的 attribute，
它的值完全是由 id 轉過來的

另外 `@property` 還有一個好處，
在使用 `Meeting` 這個 class 時要像下面這樣

```python
meeting = Meeting()
print(meeting.hash_id)
```

注意 `hash_id` 後面沒有括號，
意義上更像一個 attribute 而不是一個動作 (function) 了，
真棒/

更多有關 `@property` 的說明可以在[這裡](https://www.maxlist.xyz/2019/12/25/python-property/)找到

而 `get_id` 我加上了 `@staticmethod` 的 decorator，
用法就可以像下面這樣

```python
id = Meeting.get_id(hash_id)
```

差別在於加上 `@staticmethod` 就不用每次 `get_id` 都創一個物件出來，
純呼叫 `get_id` 的 function 就好
