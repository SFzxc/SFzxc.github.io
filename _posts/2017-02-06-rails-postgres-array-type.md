---
layout: post
title:  "Rails postgres array type"
date:   2017-02-06 03:43:45 +0700
categories: [rails, postgres]
---

Rails hỗ trợ rất tốt kiểu type array của Postgres. Đây kiểu dữ liệu rất ích với những vấn đề liên quan đến tags,..
Giả dụ vấn đề đặt ra là search like tìm những bài báo có chứa bất kì tag nào like với chuối string đưa vào.

Tạo một bài báo có column là tags

`rails g model Article name:text tags:text`

Thay đổi một chút trong migration

```ruby
class CreateArticles < ActiveRecord::Migration[5.0]
  def change
    create_table :articles do |t|
      t.text :name
      t.text :tags, array: true, default: []

      t.timestamps
    end
  end
end
```

Thêm dữ liệu vào bảng Articles

```
Article.create!(name: 'Traitors Of Time', tags: ['news', 'research', 'scence'])
Article.create!(name: 'Traitors Of Time', tags: {technical, biology, scence})
Article.create!(name: 'Traitors Of Time', tags: ARRAY['comic', 'research', 'scence'])
```
Nếu muốn tìm chính bài báo có chính xác tên của tag. Ta có hàm `ANY`

```ruby
Article.where(":name = ANY(tags)", name: "news")
```


`ANY` không thể search với `ilike`

Nếu muốn search ilike với tag bất kì,ta có thể sử dụng `array_to_string`:

```ruby
Article.where("array_to_string(tags, '||') ILIKE :name", name: "%string%")
```

Tuy nhiên cách này chỉ thích hợp nếu số lượng record không quá lớn, khoảng vài trăm trở lại.
