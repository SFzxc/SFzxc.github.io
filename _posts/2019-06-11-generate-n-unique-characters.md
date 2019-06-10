---
layout: post
title:  "Generate n unique characters"
date:   2019-06-11 23:54:00 +0700
categories: [ruby]
---

```ruby
# generator.rb

ASCII = ('!'..'~')
ALPHA = ASCII.grep(/[[:alpha:]]/)

class Generator
  def self.generate(length: 10)
    alpha = ALPHA.dup
    (1..length).map do
      alpha.delete_at(rand(alpha.size))
    end.join
  end
end


# generator_test.rb

require "minitest/autorun"
require "./generator"

class GeneratorTest < Minitest::Test
  def test_unique
    50.times do
      uniq_string = Generator.generate(length: 20)
      assert_match /^[a-zA-Z]+$/, uniq_string
      assert_equal uniq_string.size, uniq_string.chars.uniq.size
    end
  end
end
```
