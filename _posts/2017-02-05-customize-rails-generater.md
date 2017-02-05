---
layout: post
title:  "Customize rails generate"
date:   2016-06-11 03:43:45 +0700
categories: [ruby, rails]
---
Với dự án lớn, việc phân chia cấu trúc thư mục rõ ràng thì sẽ dễ dàng cho việc maintain sau này. Ngoài các thư mục `MVC` còn phát sinh
thêm các thư mục như `decorators`, `services object`. Việc tạo tay các file trong các thư mục này sau mỗi lần generate rất phiền hà.
Thay vào đó ta có thể config để nó sinh ra kèm theo khi ta `rails g`

```
rails new new_app
```
```
cd new_app 
```
Ta chạy lệnh `rails g generator rails/my_customize`

```
Running via Spring preloader in process 13177
Expected string default value for '--jbuilder'; got true (boolean)
      create  lib/generators/rails/my_customize
      create  lib/generators/rails/my_customize/my_service_generator.rb
      create  lib/generators/rails/my_customize/USAGE
      create  lib/generators/rails/my_customize/templates
      invoke  test_unit
      create    test/lib/generators/rails/my_customize_generator_test.rb
```

Trong file `lib/generators/rails/my_customize/my_service_generator.rb`

```ruby
class Rails::MyCustomizeGenerator < Rails::Generators::NamedBase
  def create_service_file
    create_file "app/services/#{name}_service.rb", <<-FILE
class #{class_name}Service
  def initialize(current_user, params)
  end
end
    FILE
  end
end
```

Ở đây ta muốn với mỗi lần generate nó sẽ tạo thêm một file trong thư mục `services`
và có nội dung là 

```ruby
class #{class_name}Service
  def initialize(current_user, params)
  end
end
```

Sau đó, trong file `config/application.rb`
Ta thêm vào:

```ruby
    config.generators do |g|
      g.helper :my_customize
    end
```

Có rất nhiều tùy chọn config khác, ta có thêm tham khảo thêm ở 
[http://guides.rubyonrails.org/generators.html]

Như thế là ta đã config xong.
Có thể test ngay bằng

```
rails g controller user/information index 
```

```
      create  app/controllers/user/information_controller.rb
      route  namespace :user do
        get 'information/index'
      end
      invoke  erb
      create    app/views/user/information
      create    app/views/user/information/index.html.erb
      invoke  test_unit
      create    test/controllers/user/information_controller_test.rb
      invoke  my_service
      create    app/services/user/information_service.rb
      invoke  assets
      invoke    coffee
      invoke    scss

```
