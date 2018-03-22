---
layout: post
title:  "Giai cuu transaction Bitcoin bi mac ket"
date:   2018-03-22 09:00:00 +0700
categories: [bitcoin, blockchain, transaction, ruby]
comment: true
---

Làm thể nào để giải cứu một giao dịch bitcoin đang bị mắc kẹt

Một giao dịch bị mắc kẹt là sao ta ?! Bạn vô tình hay cố ý trả công cho miner quá thấp(set phí mining quá thấp), transaction ngày qua ngày bị mắc kẹt trong mempool(1) vì không được miner đoái hoài đến. Ngày ngày trôi qua, bạn hối hận vì giao dịch mãi vẫn chưa được xác thực. Muốn nâng mining fee lên cho nhanh verify nhưng hông được vì tx một khi đã broadcast thì không thể thay đổi nội dung được nữa. Cùng đi tìm hiểu nguyên nhân tại sao set fee thấp lại bị mắc kẹt và cách giải quyết cho vấn đề này.

Definations

Transaction: Như ý nghĩa của nó, mình sẽ viết tắt tx.

Bitcoin mempool: là nơi để chứa những giao dịch chưa được verify (tức là những giao dịch chưa được xác nhận là hợp lệ bởi blockchain network).

Miner: là những người sẽ sử dụng khả năng tính toán của máy tính để giải các thuật toán, xác nhận các giao dịch có hợp lệ hay không và thêm nó vào blockchain.

Why Transactions Become Stuck

Bitcoin tx fees thực sự khó khăn để hiểu vì liên quan đến nhiều concept khác nhau. Một chỉ số quan trọng mà bạn cần phải nhớ đó là mật độ phí giao dịch của bạn. Mật độ phí (d) được tính theo công thức fee giao dịch (f, đơn vị satoshis) trên từng size (s, đơn vị là bytes).

	d=f/s

Trong những ngày đầu thuở sơ khai của Bitcoin, fee chỉ là một phần nhập rất nhỏ của miner. Nhưng ngày nay, nó thực sự khác biệt. Fee góp phần đáng kể vào doanh thu nên các miner cố gắng tối ưu hoá fees mỗi block đào được.

Pending txs là những giao dịch đang chờ để verify, được chọn lựa bằng cách sắp xếp chúng theo mật độ phí từ cao xuống thấp. Chúng được remove ra khỏi hàng đợi và add vào candidate block. Bước này được lặp lại liên tục cho đó khi block đó full tức là đạt đến ngưỡng maximum block size (hiện tại đang là 4 Mb).  Tại thời điểm đó, những giao dịch còn lại chưa được add vào candidate block đó sẽ phải tiếp tục đợi block kế tiếp để được đưa vào.

Hầu hết các block ngày nay đều full size, điều đó chứng tỏ rằng luôn có txs luôn đứng đợi trọng hàng đợi. Người ta gọi hàng đợi đó là mempool. Và điều thứ nữa, là một tx mới hơn cũng có thể nhảy vọt lên đứng đầu hàng đợi nếu tx đó được trả mật độ fees cao hơn tx của bạn đã tạo trước đó.

![mempool](/images/mempool.png)

			-> (1) https://bitcoinfees.earn.com/ Mempool hiện tại và  dự đoán mật độ BTC fee <-

Giải thích một chút về đồ thị phía trên, nhìn vào dòng đầu tiên, trong vòng 24 hours qua có 3392 unconfirmed transactions, đã đặt mức phí là 1 đến 10 satoshis/byte và được verify ngay ở block tiếp theo (delay 0).

Child-Pays-for-Parent

Lấy một ví dụ, tưởng tượng rằng Alice trả Bob 5 mBTC, với phí là 0.125 mBTC, (12,500 satoshis). Transaction size là 250 bytes suy ra mật độ phí là 50 satoshis/byte. Mật độ maket tx phí vào thời điểm đó đang là 150 satoshis/byte. Cho nên tại thời điểm Alice's fee chỉ bằng 1/3 so với mức cạnh tranh.

Đã 1 ngày trôi qua, tx của Alice vẫn chưa được verify. Có một cách giải quyết ở đây là:

Giả dụ Bob nợ tiền David 2mBTC, giờ Bob sẽ tạo ra một giao dịch để trả nợ cho David và input đầu vào là Output có chứa 5mBTC từ giao dịch chưa được xác nhận của Alice. Và Bob cố tính chịu phí giao dịch còn thiếu để giải cứu giao dịch của Alice bằng cách gộp chung với phí khi gửi tiền cho David. Công thức tính phí mới được tính như sau:

	fc = d×b - fp

Trong đó fc là tổng phí tx con sẽ trả trong tx gửi cho David, d sẽ là maket fee density(mật độ phí trung bình để 1 tx được add vào block gần nhất - đồng nghĩa với việc verify sẽ độ tầm dưới 10 phút). b là tổng size của cả 2 transaction. Giải dụ mức phí thông dụng hiện tại là 150 satoshis/byte,  tổng kích thước cả 2 tx cha con là 75000 bytes (2x250 bytes), tx cha (tx của Alice đã bỏ ra ) nên bạn sẽ cần trả là:

	150x500-12,500 = 62500, or 0.65200 mBTC.

Sau khi kí tx gửi David và broadcast lên bitcoin network, bạn có thể dùng các block explorer để theo dõi số lượng confirmation của tx của Alice (tx cha) và David (tx con).

Tóm váy

Bài viết không đi sâu vào các bước technical cụ thể mà bàn về phương pháp và hướng giải quyết, hy vọng được đi chi tiết hơn ở các bài khác. Các bước trên đã được tested và đã hoạt động như mô tả. Tuy nhiên cần lưu ý một số điểm sau, fee thông dụng có thể dao động theo thời gian. Có thể fee như mô tả sẽ không giống như lúc bạn đó bài này, đôi lúc mempool quá thấp thì 1 satoshis/byte cũng sẽ được verify ở block tiếp theo. Bạn có thể theo dõi biểu đồ phí ở trang này. https://bitcoinfees.earn.com/ hoặc https://estimatefee.com/. Một điều quan trọng nữa là Bitcoin Nodes cũng có giới hạn kích thước mempool, nếu nó vượt ngưỡng thì nó sẽ xoá những tx chờ comfirm có density thấp từ dưới lên. Nếu điều này xảy ra bạn phải re-publish lại tx cha để publish được tx con.

Việc giải cứu là không khó nếu dùng đúng công cụ và một ít thời gian để nghiên cứu.

TL;DR

Các bước giải cứu 1 transaction bị mắc kẹt:

- Xác định địa chỉ bạn muốn dùng để giải cứu (địa nằm trong tx bị mắc kẹt)
- Xác định market fee density sử dụng  https://bitcoinfees.earn.com/ hoặc https://estimatefee.com/.
- Tính toán lượng fee cần phải add để giải cứu bằng công thức fc = d×b - fp. Trong đó fc là phí phải add ở tx con, d là market fee density, b là tổng kích thước cả 2 txs, fp là lượng phí mà tx cha đã trả.
- Kí và broadcast new tx, sau đó là đợi.



References

How to Clear a Stuck Bitcoin Transaction by Rich Apodaca

Mastering Bitcoin

Transaction_fees
