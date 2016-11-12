### [Rack](https://github.com/rack/rack)
  Rack là một module web server interface. hay nói cách khác là một [middleware](https://vi.wikipedia.org/wiki/Middleware). Cho phép webserver và web app giao tiếp với nhau.Nó chia nhỏ những request HTTP được gửi đến rồi đưa nó vào những đường ống khá nhau và xử lý chúng thành từng mảnh cho đến khi nó gửi lại phản hồi từ web app (controller).
Nó gồm 2 compoments Handler và Adapter. được sử dụng để giao tiếp với web server và application(framwork).

### Sự so sánh
Trong bài này. chúng ta sẽ nói về những web app server thông dụng. Đi sâu từng cái đề thấy được sự khác nhau và có thể là một số đối lập của chúng. Mục đích là để bạn có cách hiểu tốt hơn trong việc lựa chọn web server phù hợp trong từng trường hợp nhất định.

### [Passenger](https://github.com/phusion/passenger) 
  Passenger là một server được khuyến khích sử dụng cho rails app.
Giàu tính năng, set-up đơn giản. Nó loại bỏ các middleman server truyền thống trong kiến trúc setup server bằng cách trực tiếp intergration với Apache and NGinx webserver.

  Passenger cung cấp tính sẵn sàng làm việc với đa app trên cùng 1 server. Có khả năng xử lý những client chậm chạp. Request và responses được lưu vào bộ đệm đầy đủ nên khiến nó khó có khả năng bị tấn công tắt nghẽn tài nguyên hệ thống.
Bởi vì nó khá thông dụng và được sử dụng trong nhiều trường hợp nên không khó để tìm ra lời giải đáp hoặc chuyên gia khi bạn gặp phải một vấn đề nào đó trên các diễn đàn. Cũng có các công ty đang phát triển và đưa ra các hỗ trợ thương mại.
Open Source ver xử lý đa nhiệm trên 1 luồng trong khi bản enterprise ver có thể config để làm việc được cả đơn và đa luồng và đương nhiên là đi kèm một số tính nâng cao: xử lý song song, deploy hàng loạt, kiểm soát và giới hạn tài nguyên.

### [Puma](https://github.com/puma/puma)
  Puma là một web server sinh ra để dành riêng cho Ruby web server.  Lấy cảm hứng từ WS Mongrel.
Đã có rất nhiều sự thay đổi trong quá trình phát triển.Puma's developer, Evan Phoenix đã dựa trên kiến trúc của Mongrel loại bỏ những phức tạp gây ra bởi hiệu suất kém. và thiết kế để hỗ trợ xử lý song song.

  Nó cho phép bạn thiết lập tối đa và tối thiểu số luồng để xử lý jobs và cũng làm việc trên cluster mode.
Mặc dù không trực tiếp hỗ trợ sử lý đa app. Bạn vẫn có thể sử dụng thông qua [Jungle](https://github.com/puma/puma/tree/master/tools/jungle)

### [Thin](https://github.com/macournoyer/thin): Mạnh mẽ và là một HTTP Server khá thú vị.
  "Bảo mật, ổn định, nhanh và có khả năng mở rộng". Đó là những điều khi ta nhắc đến Thin. Và nó vẫn đang được tính cực phát triển. Nó được thiết kế để hoạt động được trên tất cả framwork implement Rack Spec
Là một app server dựa trên Event/Machine. Thin có khả năng xử lý yêu cầu kéo dài mà không cần sự giúp đỡ của giải pháp proxy ngược giống như các loại khác.

### [Unicorn](https://github.com/defunkt/unicorn)
  Là một app server khá nghiêm chỉnh. Nó cũng có thể được sử dụng tốt cho Python. Khá đầy đủ tính năng. Tuy nhiên nó không được thiết kế để cố gắng làm được tất cả mọi thứ. Unicorn chỉ làm những gì cần phải thực hiện(việc của một web app server) và ủy thác những việc còn lại cho thứ có thể làm tốt hơn.
Giống như NGINX, Unicorn có thể thực thi và deploy ứng dụng mà ko cần kết nối với client.

*Một số tính năng nâng cao:
-Tất cả worker chạy trong một không gian riêng biêt, lưu trữ một request mỗi lần.
-Sẵn sàng lắng nghe nhiều interface


 
































