---
layout: post
title:  "Goroutines Worker Pools"
date:   2018-09-18 11:33:00 +0700
categories: [Goroutines, Golang, concurrency]
comment: true
---

Thông qua ví dụ dưới đây, chúng ta sẽ nhìn thấy được cách implement worker pool sử dụng goroutines và channels để giải quyết bài toàn n workers phải hoàn thành m jobs. Ai hoàn thành xong sớm sẽ tìm kiếm task còn lại để làm cho để khi tất cả hoàn thành.

Những công nhân sẽ nhận việc (jobs channel) và báo cáo kết quả.

Chúng ta sẽ sleep một giây để mô phỏng thời gian xử lý task.

Việc chúng ta cần làm là cần gửi công việc cho worker và thu thập kết quả (Việc này sẽ đơn giản hoá chỉ bằng việc in ra kết quả). Chúng ta sẽ tạo ra 1 channel cho việc gửi công việc cho worker.

Idea:

- Mỗi worker sẽ là 1 goroutine. for 1 -> 3
- Tạo một channel jobs make(chan int) sau đó đẩy job vào.
- Close channel để mọi người biết rằng đó là toàn bộ công việc cần hoàn thành.

Ước lượng thời gian hoàn thành sẽ là ~2 giây hơn. Vì giây thứ nhất sẽ có 3 job được hoàn thành bởi 3 công nhân. Và 2 job còn lại sẽ được xử lý ở giây thứ 2 bởi bất kì 2 trong 3 worker.


```go
package main

import (
    "fmt"
    "time"
    "sync"
)

var wg sync.WaitGroup

func worker(w int, jobs <-chan int) {
    defer wg.Done()
    for j := range jobs {
        fmt.Println("worker", w, "started job", j)
        time.Sleep(time.Second)
        fmt.Println("worker", w, "finished job", j)
    }
}

func main() {
    start := time.Now()

    jobs := make(chan int, 100)

    for w := 1; w <= 3; w++ {
        wg.Add(1)
        go worker(w, jobs)
    }

    for j := 1; j <= 5; j++ {
        jobs <- j
    }

    close(jobs)

    wg.Wait()

    fmt.Println(time.Since(start))
}
```

Kết quả:

```
worker 3 started job 1
worker 2 started job 3
worker 1 started job 2
worker 2 finished job 3
worker 2 started job 4
worker 1 finished job 2
worker 1 started job 5
worker 3 finished job 1
worker 2 finished job 4
worker 1 finished job 5
2.002697084s
```

Lược dịch và hiệu chỉnh lại code cho dễ hiểu từ ví dụ: https://gobyexample.com/worker-pools