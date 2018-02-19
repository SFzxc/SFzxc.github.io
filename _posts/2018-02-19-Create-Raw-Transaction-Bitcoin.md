---
layout: post
title:  "Create Raw Transaction Bitcoin"
date:   2018-02-19 15:00:00 +0700
categories: [bitcoin, blockchain, transaction, ruby]
comment: true
---

Đầu năm nổi siêng viết một bài vậy

## Mục tiêu

Tạo ra một bitcoin transaction gửi bitcoin từ 1 địa chỉ A sang địa chỉ B.

Hiểu được cấu tạo của raw transaction

## Yêu cầu

Nếu bạn chưa nắm các khái niệm căn bản về về private key, public key, address, wallet và transaction.
Mình recommend mọi người đọc qua nội dung cuốn [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook) của` O'Reilly Media` từ chương 4 đến chương 7. Còn nếu mọi người đã nắm chắc thì có thể bỏ qua bước này.

## Chuẩn bị

Chúng ta cần 1 địa chỉ bitcoin, và một ít bitcoin trong đó, 1 private key của địa chỉ trên để có thể sign raw transaction.



#### Tạo key/address



Có nhiều cách để tạo ra cặp bitcoin key/address

1. Chúng ta sẽ sử dụng gem [bitcoin-ruby](https://github.com/lian/bitcoin-ruby) (thư viện ruby để tương tác với bitcoin protocol/network) để tạo ra cặp key/address.
2. Có thể sử dụng [Bitcoin Explorer](https://github.com/libbitcoin/libbitcoin-explorer) (*Bitcoin Command Line Tool*) để generate
3. Sử dụng UI các wallets such as [CoinDaddy Wallet](https://counterwallet-testnet.coindaddy.io) (nhớ lưu lại 12 word passpharse). Trong phần dropbox trên thanh address có tính năng show private key. Sử dụng dụng cặp key/address để sử dụng.

Mình sẽ sử dụng cách 1

Install `gem 'bitcoin-ruby'`

Chúng ta sẽ sử dụng mạng testnet vì đơn giản là không có bitcoin

```ruby
require 'bitcoin-ruby'

include Bitcoin::Builder
Bitcoin.network = :testnet3

key      = Bitcoin::Key.generate
# <Bitcoin::Key:0x007fbda21f2220 @key=#<OpenSSL::PKey::EC:0x007fbda21f2108>, @pubkey_compressed=true>
priv_key = key.priv # 02a9f75bbbc0a876304ba6633fdf92879f9506ea0d0b572bce5e8afb93bd29e6
pub_key  = key.pub # 030894fe082bfcd3b15a4573ad15a83b43af8b2d9a6ac34863445b86e8206f49c8
addr     = key.addr # mwQe4KFo8s66BbtuFhGMpaC7mkGwi1idQo

# Nếu bạn đã có sẵn private_key rồi thì có thể import key từ định dạng base58 (WIF)
key = Bitcoin::Key.from_base58('cMfszV3Hx56bDxEYQEPCEHDwYVDnKkx2pLpgvdYfAUpb7jBoj2EQ')

# Hoặc xem định dạng base58 của key trên như thế nào
key.to_base58
```

Thường thì private key của testnet network (định dạng base58) sẽ bắt đầu bằng kí tự  `c` và địa chỉ thường là chữ `m`.

Nếu để lộ private key đồng nghĩa với việc người khác có thể lấy được toàn bộ bitcoin của bạn.

Có một bạn hỏi mình là có trường hợp nào mà attacker generate key liên tục rồi cái địa chỉ được sinh ra đã tồn tại và người ta có thể đánh cắp bitcoin của địa chỉ đó không. Nice question, đúng là theo lý thuyết thì vẫn có khả năng xảy ra. Nhưng thực sự là impossible.  Nếu bạn lấy toàn bộ hạt cát ở trái đất, rồi một hạt các đó tạo ra 1 hành tinh trái đất. Toàn bộ hạt cát trong toàn bộ các hành tinh mới tạo ra đó chính là số địa chỉ có thể được tạo ra. Thực sự là bruce force quá lớn, `Possible but improbable` Bạn có thể xem thêm [thảo luận ở đây](https://bitcointalk.org/index.php?topic=233503.0)



#### Get some btc



Nhiệm vụ tiếp theo của chúng ta nạp 1 ít testnet bitcoin vào địa chỉ `mwQe4KFo8s66BbtuFhGMpaC7mkGwi1idQo` để thực hiện giao dịch.

Testnet network có phương thức hoạt động không khác gì bitcoin mainnet, chỉ khác là bitcoin ở đây ko có giá trị. Một số trang cung cấp bitcoin testnet như `https://testnet.manu.backend.hamburg/faucet`

Bạn điền địa chỉ cần nhận bitcoin vào nhấn gửi bitcoin. Và đợi tầm 10 phút để có bitcoin sử dụng

Có nhiều trang web online cho phép bạn xem các trạng thái của các transaction hay thông tin các địa chỉ như

`https://live.blockcypher.com/btc-testnet/address/<địa chỉ bitcoin>`

`https://testnet.blockchain.info/tx/32f222ae2979497a56c44612994a7a5f547631d6fa99bf802e7b97f7033d93ae`

Tạo thêm 1 address nữa để nhận bitcoin. Mọi người có thể sử dụng các phương pháp ở trên để tạo.

Đến đây chúng ta cần phải hiểu thêm 1 khái niệm UTXOs (Unspent transactions ouput) . Các output của transaction trước đó sẽ được sử dụng làm input cho tx tiếp theo. Những output này chỉ được sử dụng 1 lần, nếu đã spent rồi thì không thể là input nữa.

1 tx có thể chứa nhiều utxo nên ta cần out_put để xác định xem utxo nào được sử dụng.

Thêm một chút nữa là cần import prev_tx đó để.  Gem `bitcoin-ruby` hỗ trợ nhiều cách để import previous tx như là `from_hash`, `from_json` hay là hex. Nếu là json thì bạn tạo 1 file json cùng cấp rồi import tên file vào `Bitcoin::P::Tx.from_json(json_path)`

Mình thì thích import từ hex format hơn, để hiểu rõ hơn về cấu trúc của tx hex format bạn có thể đọc lại chương Transaction trong cuốn sách trên.

Trong tx trên `https://testnet.blockchain.info/tx/32f222ae2979497a56c44612994a7a5f547631d6fa99bf802e7b97f7033d93ae?format=hex`  thêm param format = hex thì sẽ nhận được

```
01000000000101e97c87213558eacf9886f84045bfa1a894be50a3c6fe3b98443a953ba4089752010000001716001444148ec49aa12c4fef660d7118fe78ea75366319ffffffff0280a4bf07000000001976a914ae504b1c151b864cace83913eb1537383823f1f588ac0bee4ec30d00000017a914eda6073668307ed2851d849a41bd7badde77e5c0870247304402200e3ab54c5ec051d712101ddd9fdc6f91b77cade872d65047dcf8f0ad9f81390402205903a833a46a769c02b10895d8cc482e235a04971794cec588b0973280120f6e0121038e015bb2ecb9e674b5217b13c762b928e013d54d7994b9de586bcb098033e00800000000
```

 Đó là dùng UI, còn đối với code thì mình sữ dụng api của blockcypher get tx infor có chứa hex value để sử dụng.



#### Create a raw transaction



Xong giờ chúng ta đã có đủ nguyên liệu để tạo raw transaction.

```ruby
require 'bitcoin-ruby'
require 'pry'

include Bitcoin::Builder
Bitcoin.network = :testnet3

sender_addr    = 'mwQe4KFo8s66BbtuFhGMpaC7mkGwi1idQo'
recipient_addr = 'mvgnDMHF1yj9uhyJSE9sSugZN1dYvasMtm'
change_addr    = 'mwQe4KFo8s66BbtuFhGMpaC7mkGwi1idQo'

# Import from wif format
key = Bitcoin::Key.from_base58('cMfszV3Hx56bDxEYQEPCEHDwYVDnKkx2pLpgvdYfAUpb7jBoj2EQ')

prev_tx_hex = '01000000000101e97c87213558eacf9886f84045bfa1a894be50a3c6fe3b98443a953ba4089752010000001716001444148ec49aa12c4fef660d7118fe78ea75366319ffffffff0280a4bf07000000001976a914ae504b1c151b864cace83913eb1537383823f1f588ac0bee4ec30d00000017a914eda6073668307ed2851d849a41bd7badde77e5c0870247304402200e3ab54c5ec051d712101ddd9fdc6f91b77cade872d65047dcf8f0ad9f81390402205903a833a46a769c02b10895d8cc482e235a04971794cec588b0973280120f6e0121038e015bb2ecb9e674b5217b13c762b928e013d54d7994b9de586bcb098033e00800000000'
prev_tx = Bitcoin::P::Tx.new(prev_tx_hex.htb)
prev_tx_output_index = 0

tx = Bitcoin::Protocol::Tx.new
tx.add_in Bitcoin::Protocol::TxIn.new(prev_tx.binary_hash, prev_tx_output_index, 0)
tx.add_out Bitcoin::Protocol::TxOut.value_to_address(10_000_000, recipient_addr)
tx.add_out Bitcoin::Protocol::TxOut.value_to_address(90_000_000, change_addr) # Carefully here
sig = key.sign(tx.signature_hash_for_input(0, prev_tx))
tx.in[0].script_sig = Bitcoin::Script.to_signature_pubkey_script(sig, [key.pub].pack("H*"))

tx = Bitcoin::Protocol::Tx.new( tx.to_payload )

p tx.hash
p tx.verify_input_signature(0, prev_tx) == true

puts "json:\n"
puts tx.to_json # json
puts "\nhex:\n"
puts tx.to_payload.unpack("H*")[0] # hex binary
```

Nếu verify_input_signature == true thì có nghĩa đó là đã được kí thành công, bạn chính là chủ của utxo đó.

Có một lưu ý khá quan trọng ở đây chính là giá trị được add vào change_address. change output chính là chuyển ngược số
bitcoin còn lại về lại chính địa chỉ đó, thường thì các exchange sẽ thay đổi địa chỉ mới khi 1 giao dịch được thực hiện
và địa chỉ mới chính là change address. Ở đây mình để nguyên.

Thứ nữa là phải tính toán giá trị này cẩn thận. Fee mining chính bằng hiệu của số tiền toàn bộ utxos trừ đi tổng số tiền recipient và change address nhận được

Phần thừa chính là số btc miner nhận được. Ở đây mình để bừa giá trị của change_addr

Việc để tx fee bao nhiêu cho hợp lý mình sẽ không bàn ở đây và hẹn ở một post khác.

Đây là kết quả khi bạn chạy file trên

```

"9d42f40aa83371a344b38750731b0aca18d81904cdb94c991475c520136da4fb"
true
json:
{
  "hash":"9d42f40aa83371a344b38750731b0aca18d81904cdb94c991475c520136da4fb",
  "ver":1,
  "vin_sz":1,
  "vout_sz":2,
  "lock_time":0,
  "size":225,
  "in":[
    {
      "prev_out":{
        "hash":"32f222ae2979497a56c44612994a7a5f547631d6fa99bf802e7b97f7033d93ae",
        "n":0
      },
      "scriptSig":"304402202bc6c6b738f7d7cac077070ef1f918c874a46bbc5a56b6339e150a4b2ac92e3502200716ab18ef7979c7dd8d7be5bd6c33193b1d054aae5425cdc8ea0972bb4cb6d201 030894fe082bfcd3b15a4573ad15a83b43af8b2d9a6ac34863445b86e8206f49c8"
    }
  ],
  "out":[
    {
      "value":"0.10000000",
      "scriptPubKey":"OP_DUP OP_HASH160 a665a18fbfb8502078bbeb44abc85dda504a2605 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value":"0.90000000",
      "scriptPubKey":"OP_DUP OP_HASH160 ae504b1c151b864cace83913eb1537383823f1f5 OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}

hex:
0100000001ae933d03f7977b2e80bf99fad63176545f7a4a991246c4567a497929ae22f232000000006a47304402202bc6c6b738f7d7cac077070ef1f918c874a46bbc5a56b6339e150a4b2ac92e3502200716ab18ef7979c7dd8d7be5bd6c33193b1d054aae5425cdc8ea0972bb4cb6d20121030894fe082bfcd3b15a4573ad15a83b43af8b2d9a6ac34863445b86e8206f49c8ffffffff0280969800000000001976a914a665a18fbfb8502078bbeb44abc85dda504a260588ac804a5d05000000001976a914ae504b1c151b864cace83913eb1537383823f1f588ac00000000
```

Đến đây chúng ta đã tạo thành công 1 btc raw transaction.



#### Broadcast



Lưu ý mọi người vui lòng không dùng đoạn hex trên của mình để test broadcast vì như thế utxo sẽ spent và người sau chạy ví dụ này 1 lần nữa sẽ bị lỗi.



Nhiệm vụ tiếp theo chính là broadcast raw tx (hex format) này lên bitcoin network (ở đây là testnet)

Có nhiều cách để broadcast:

- Sử dụng blockcypher api
- Sử dụng UI của blockcypher (https://live.blockcypher.com/btc/pushtx/) (Nhớ chọn testnet)
- Nếu bạn đã cài sẵn một node bạn có thể dụng `bitcoin-cli signrawtransaction` để broadcast



#### End



Trên đây là toàn bộ chia sẽ của mình về cách tạo bitcoin raw transaction. Feel free to contact if you get any problems. Bài tiếp theo có lẽ sẽ nói về cách viết test (rspec) khi làm việc với btc tx.

