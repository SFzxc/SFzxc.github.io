---
layout: post
title:  "Cách giảm bớt tấn công Dos thông qua Http Attack"
date:   2017-05-07 00:00:23 +0700
categories: [DevOps]
comment: true
---

*CentOs 7: Cách giảm bớt tấn công Dos thông qua Http Attack*

Thật khó khăn để ngăn chặn hoàn toàn tấn công http, nhưng bạn có thể xoa dịu tấn công đó
Bài viết này sẽ hướng đến việc ngăn chặn nếu có request hơn `30 lần` trong vòng `60 giây`
Bạn có thể dựa trên đây để điều chỉnh hai thông số đó phù hợp với yêu cầu của mình

Chúng ta sẽ bàn về cách xác định ước lượng thông số này sao cho phù hợp ở một bài khác.

Attacker phải đợi 60 giây nữa trước khi có thể request lần server lần nữa. Và nếu vẫn tiếp tục
cố tình request, nó toàn bộ request sẽ bị chặn.

Bài hướng dẫn này sử dụng option `-direct` của `firewall-cmd` và không yêu cầu phải reboot

Chúng ta sẽ tạo một file có tên là `xt.conf` trong folder `/etc/modprobe.d/` và sau đó chèn vào
file đó config như sau:

```
options xt_recent ip_pkt_list_tot=30
```

Vì mặc định việc hệ thống chi đếm 20 hit mới nhất nên ta cần customize một chút, tăng nó lên 30

Sau đó load new configuration chúng ta vừa mới tạo bằng câu lệnh:

```
modprobe xt_recent
```

Bước tiếp theo, chúng ta add thêm 2 rules cho firewall như sau:

```
$ firewall-cmd --permanent --direct --add-rule ipv4 filter \
INPUT_direct 0 -p tcp --dport 80 -m state --state NEW -m recent --set

$ firewall-cmd --permanent --direct --add-rule ipv4 filter \
INPUT_direct 1 -p tcp --dport 80 -m state --state NEW -m recent --update \
--seconds 60 --hitcount 30 -j REJECT --reject-with tcp-reset

$ firewall-cmd --reload
```

Trả về `success` nếu add rule thành công.

Notes:
  - `INPUT_direct` là nhận tất các packets trước khi đi qua bất kì bộ lọc nào.
  - 0 và 1 là cấp độ ưu tiền của rule trong INPUT_direct

Để kiểm trả chắc chắn xem các rule đã được đăng kí đúng chưa, Ta sử dụng câu lệnh sau:

```
$ firewall-cmd --permanent --direct --get-all-rules

ipv4 filter INPUT_direct 0 -p tcp --dport 80 -m state --state NEW -m recent --set
ipv4 filter INPUT_direct 1 -p tcp --dport 80 -m state --state NEW -m recent --update --seconds 60 --hitcount 30 -j REJECT --reject-with tcp-reset
```

Và chúng ta cần test xem thực tế nó có hoạt động như vậy không :D

Tạo một file shell như sau `dos.sh` ở một server khác để hit vào.
Có thể giảm đối số 60 nhỏ lui một chút để việc testing diễn ra nhanh hơn.

```bash
#!/bin/bash

while true
do
/usr/bin/wget "http://server.example.com"
end
```

Biên dịch từ nguồn: https://www.certdepot.net/rhel7-mitigate-http-attacks/


