---
layout: post
title:  Boostcamp ngày chủ nhật
date:   2016-06-11 03:43:45 +0700
categories: [ruby, codefights]
---

**Bitcoin JSON RPC**

Setup một bitcoin server local để viêt exchange bitcoin web app.
```
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install bitcoind -y
```


```
cd ~/.bitcoin
```
Tạo một file config có tên sau : bincoin.conf
```
prune=600
maxconnections=12
maxuploadtarget=20
rpcuser=btnuser
rpcpassword=btnpassword
daemon=1
keypool=10000
rpcport=8392

````
```
reboot
bitcoind
```

https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list
```
bitcoin-cli <method>
```
Sau khi đã dùng được cli, ta tiến hành chọn một ruby gem để hỗ trợ call api từ server
```
bitcoin-client
```
Tạo một rails app, install vào và bắt đầu sữ dụng
https://github.com/sinisterchipmunk/bitcoin-client
