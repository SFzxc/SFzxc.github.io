---
layout: post
title:  "Get month name"
date:   2016-06-11 03:43:45 +0700
categories: [ruby, codefights]
---

Map the given integer to a month.

**Example:**

* For `mo = 1`, the output should be `get_month_name(mo) = "Jan"`,
* For `mo = 0`, the output should be `get_month_name(mo) = "invalid month"`.

**Input/Output**

* [time limit] 4000ms (py)
* [input] integer mo (A non-negative integer).
* **Constraints:** `0 ≤ mo ≤ 15`.
* **[output] string**

A `3`-letter abbreviation of month number `mo` or `"invalid month"` if the month doesn't exist.

Here are abbreviations of all months:

**My Solution:**

```ruby
def get_month_name(mo)
  months = {
    1 => "Jan", 2 => "Feb", 3 => "Mar", 4 => "Apr",
    5 => "May", 6 => "Jun", 7 => "Jul", 8 => "Aug",
    9 => "Sep", 10 => "Oct", 11 => "Nov", 12 => "Dec"
  }
  months.key?(mo) ? months[mo] : "invalid month"
end
```

**Result Tests**:

```ruby
>>> get_month_name(1)
"Jan"
>>> get_month_name(0)
"invalid month"
>>>
```