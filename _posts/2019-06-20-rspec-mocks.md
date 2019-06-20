---
layout: post
title:  "Rspec Mocks"
date:   2019-06-20 23:54:00 +0700
categories: [ruby, rspec, mock]
---

### RSpec Mocks

https://relishapp.com/rspec/rspec-mocks/docs

Khi viết test thỉnh thoảng chúng ta muốn sử dụng mock object thay vì gọi real objects. Mục đích là giảm thời  gian gọi vào database/third party api để lấy giá trị cũng như kiểm soát được giá trị luôn luôn không đổi.

`User(id: integer, name: string, created_at: datetime, updated_at: datetime)`

**Method Stubs**

```ruby
allow(User).to receive(:name) { "Songoku" }
```

Như ở trên sẽ thông báo cho bất kì instance của class User trả về giá trị `Songoku` khi gọi phương thức `name`.

**Test Double**

RSpec 3 còn cung cấp 3 cách khác nhau để implement một mock object.

- `spy`chấp nhận tất cả các method name, và sẽ trả về chính nó nếu method đó nếu chưa được định nghĩa và tất nhiên sẽ không raise lỗi

  ```ruby
  require 'rspec/mocks/standalone'
  
  user_spy = spy(User)
  user_spy.whatever # #<Double User()>
  ```

- `double` sẽ raise lỗi nếu nó nhận một phương thức không có. 

  ```ruby
  require 'rspec/mocks/standalone'
  
  user_double = double(User)
  user_double.whatever # RSpec::Mocks::MockExpectationError (#<Double User() received unexpected message :name with (no args))
  
  user_double = double(User, whatever: nil)
  user_double.whatever # nil
  ```

  Bạn phải khai báo một cách tường mình giá trị trả về của `whatever` thì mới sử dụng được.

- `instance_double` Cái này sẽ kiểm soát gắt hơi `double` một chút là không cho phép khai báo tuỳ tiện một method nào đó không có

  ```ruby
  require 'rspec/mocks/standalone'
  
  user_inst_double = instance_double(User, whatever: nil) # RSpec::Mocks::MockExpectationError (the User class does not implement the instance method: whatever)
  
  user_inst_double = instance_double(User, name: nil)
  user_inst_double.name # nil
  user_inst_double.id # RSpec::Mocks::MockExpectationError (#<InstanceDouble(User) (anonymous)> received unexpected message :id with (no args))
  ```

  Nếu chúng ta chỉ muốn kiểm soát giá trị của `name` còn những giá trị như `id`, `created_at` đều trả về `nil` thì ta có thể khai báo như sau:

  ```ruby
  user_inst_double = instance_double(User, name: nil).as_null_object
  user_inst_double.id # nil
  ```

**Benchmark**

```ruby
require 'rspec/mocks/standalone'
require 'benchmark/ips'

Benchmark.ips do |bm|
  bm.report("spy") { spy(User, id: 1) }
  bm.report("double") { double(User, id: 1) }
  bm.report("instance double") { instance_double(User, id: 1) }
  bm.report("actual object") { User.new(id: 1) }
  bm.compare!
end

# Warming up --------------------------------------
#                  spy     4.271k i/100ms
#               double     3.929k i/100ms
#      instance double     2.051k i/100ms
#        actual object     3.612k i/100ms
# Calculating -------------------------------------
#                  spy     46.188k (±36.4%) i/s -    183.653k in   5.078702s
#               double     49.192k (±34.1%) i/s -    180.734k in   5.478936s
#      instance double     20.198k (±26.2%) i/s -     92.295k in   5.261195s
#        actual object     32.799k (±19.1%) i/s -    155.316k in   5.003408s

# Comparison:
#               double:    49192.3 i/s
#                  spy:    46187.7 i/s - same-ish: difference falls within error
#        actual object:    32799.1 i/s - same-ish: difference falls within error
#      instance double:    20198.1 i/s - 2.44x  slower
```

Hmm không hiểu tại sao instance double lại chậm hơn actual object vậy...
