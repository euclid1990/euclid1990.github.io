---
layout: post
title:  "[Go] Tránh start một program đang chạy với Flock"
date:   2018-07-19 09:49:04 +0700
categories: Go
tags:
  - 多重起動
  - Flock
hide_thumbnail: true
image: /assets/img/posts/2018-07-19-golang-file-lock-using-flock-system-call/thumbnail.png
---

Hiện tại mình đang làm một dự án thực hiện xử lý bên dưới server (background processing). Hệ thống được viết toàn bộ bằng [Go](https://golang.org/) để tận dụng hết tốc độ cũng như sức mạnh của xử lý bất đồng bộ Goroutines. Cách đây gần một năm mình cũng từng code một dự án [Go](https://golang.org/) cho khách để xử lý cỡ `~100Gb` dữ liệu, nhưng mà thời gian trôi qua `“Nước đổ lá khoai”` kiến thức trôi về nơi xa vắng hết.

Hôm nay gặp một vấn đề không hề mới, đấy là Khách hàng đang chạy `cron jobs` một action trong Go Program cách nhau cứ mỗi 30 phút 1 nháy (Mục đích là để cập nhật lại cơ sở dữ liệu). Tuy nhiên do thời gian cần để hoàn thành mỗi action biến thiên, đôi lúc hơn 30 phút. Nên dễ xảy ra tình trạng chạy chống chéo, mệnh ai nấy làm, tự tay ... :joy: (:jp: プログラムの多重実行、多重起動)

Ví dụ cho dễ hiểu, mình có một chương trình `main.go` như sau:

```go
package main

import (
    "log"
    "time"
)

func main() {
    log.Println("--- [Start] main program ---")
    log.Println("...")
    time.Sleep(30 * time.Second)
    log.Println("--- [Finish] main program ---")
}
```

Bạn thử chạy 2 lần câu lệnh `$ go run main.go` ở 2 cửa sổ **Terminal** và đây là kết quả:

```terminal
$ ps -elf | grep main.go
0 S root        52     1  0  80   0 - 38913 -      05:59 pts/0    00:00:00 go run main.go
0 S root        89    46  0  80   0 - 18696 -      05:59 pts/1    00:00:00 go run main.go
```

Việc một chương trình bị chạy nhiều lần một thời điểm gây lãng phí tài nguyên và khả năng xảy ra sai lệch dữ liệu là rất lớn, và đó cũng là bài toán mà chúng ta cần giải quyết.

![](/assets/img/posts/2018-07-19-golang-file-lock-using-flock-system-call/bury-it.jpg){: .align-center}

Để ngăn ngừa `main program` chạy chồng chéo, có 2 phương án giải quyết được đưa ra:

1. **Lock file**

    Khi chạy chương trình sẽ kiếm tra sự tồn tại của một file đã được ghi ra trước đó, nếu tồn tại, sẽ dừng chương trình lại. Nếu không tồn tại, sẽ tạo nó và tiếp tục thực hiện chương trình.

1. **PID file**

    PID file hay còn gọi là Process ID file. Phương pháp này tương tự như **Lockfile** kể trên, ngoại trừ việc nội dung bên trong file chính là process ID của tiến trình đang đảm nhiệm việc chạy chương trình.

    Phương pháp này hẳn là tốt hơn **Lockfile**, vì với thông tin process ID, có thể cho chúng ta thông tin về chương trình nhiều hơn, kiểm tra xem chương trình có đang chạy không. Đơn cử với case process bị killed hoặc bị lỗi gây terminated mà **lock file** không được xoá bỏ.

Tuy nhiên trong bài viết này mình sẽ mô tả cách số [1] **Lock file**, vì đơn giản trong lúc thử nhiều cách mình với KH có tranh luận khá nhiều về code phần này (một phần do rơi rớt kiến thức về `Go` nên bị KH chỉnh :rofl:).

Còn cách số [2] thì bạn chỉ cần dùng thêm `os.Getpid()` để lấy processID để ghi vào file. Vụ check `isAlive` thì thực hiện bằng cách tìm Process qua [func FindProcess(pid int)](https://golang.org/pkg/os/#FindProcess) → Gửi blank signal (0) đến pId thông qua [func (*Process) Signal](https://golang.org/pkg/os/#Process.Signal).

## Talk is cheap

### 1. Flock Command

Đầu tiên để đáp ứng bài toán mình gợi ý Khách cmn lệnh Linux [`flock`](https://www.systutorials.com/docs/linux/man/1-flock/), và bảo họ thêm vào `Crontab` :sunglasses: nhanh vkl :satisfied:

```terminal
flock -xn /usr/src/myapp/main.lck -c 'go run main.go'
```
Lock là cơ chế đồng bộ và giới hạn truy cập đến tài nguyên đươc chia sẻ trong một môi trường có nhiều luồng xử lý cùng truy cập.

`flock` command sẽ thực hiện open `/usr/src/myapp/main.lck` file và thực thi câu lệnh trong option `-c`. `-n` sẽ chấm dứt (terminate) việc chạy chương trình nếu như không thể thực hiện `lock` ngay lập tức.

Option `-x` ở trên ám chỉ `exclusive lock` đôi khi gọi là `write lock`, hay còn gọi là `read-write lock`, là lock mà một luồng xử lý phải chiếm hữu khi muốn cập nhật một vùng nhớ được chia sẻ. (`shared lock` thì chỉ đơn thuần là `read-only lock`).

Test thử:

```terminal
$ flock -xn /usr/src/myapp/main.lck -c 'go run main.go' > /dev/null 2>&1 &
[1] 1008
$ flock -xn /usr/src/myapp/main.lck -c 'go run main.go'
```

Rõ ràng sau khi chạy `main.go` thì trong khi mà chương trình còn đang thực hiện, ta không thể thực thi song song.

Trên có đoạn `> /dev/null 2>&1 &` nếu bạn không hiểu thì xem giải thích sau nhé :expressionless: thỉnh thoảng mình cũng quên :rofl:

```
> Để chỉ thị redirect
/dev/null Hố đen của vũ trụ *nix, chỉ có vào không có ra
2 Chính là STDERR (2)
> Để chỉ thị redirect
& Là kí hiệu cho file descriptor, nếu không có nó thì '1' sẽ bị hiểu là file name
1 Là file descriptor đại diện cho STDOUT (1)
& Detach process từ current shell
```

Lưu ý: Lệnh `flock` được cài sẵn ở Linux distributions, nếu bạn dùng Mac thì có thể cài thêm.

### 2. syscall.Flock inside Golang

Tuy nhiên sau khi gợi ý cách trên thì KH phàn nàn phải update lại `Crontab`, rồi mỗi action trong main program cần đặt tên một file lock riêng, blah ... Và muốn một cái gì đó flexible hơn =))

Tư tưởng vẫn như **Flock Command**

```go
package main

import (
    "io/ioutil"
    "log"
    "os"
    "syscall"
    "time"
)

const LOCKFILE = "main.lck"

func main() {
    // Make LockFile
    if _, err := os.Stat(LOCKFILE); err != nil {
        ioutil.WriteFile(LOCKFILE, []byte(""), 0644)
    }

    // Open LockFile for reading only
    fd, _ := syscall.Open(LOCKFILE, syscall.O_RDONLY, 0000)
    defer syscall.Close(fd)

    // Attempts to lock the LockFile. Exit immediately if the lock cannot be acquired
    if err := syscall.Flock(fd, syscall.LOCK_EX|syscall.LOCK_NB); err != nil {
        log.Println("Cannot acquire lock. Terminate program now !")
        os.Exit(1)
    }

    ...
}
```

:up:

```terminal
$ go run main.go  > /dev/null 2>&1 &
[1] 1539
$ go run main.go
2018/07/19 08:23:00 Cannot acquire lock. Terminate program now !
exit status 1
```

Đột nhiên tới đây mình nghĩ thế này mà phải lock nhiều chỗ thì mệt chết, nên tách ra function riêng để tái sử dụng đại loại kiểu này :upside_down_face:

```go
package main

import (
    "io/ioutil"
    "log"
    "os"
    "syscall"
    "time"
)

const LOCKFILE = "main.lck"

func FLock() {
    // Make LockFile
    if _, err := os.Stat(LOCKFILE); err != nil {
        ioutil.WriteFile(LOCKFILE, []byte(""), 0644)
    }

    // Open LockFile for reading only
    fd, _ := syscall.Open(LOCKFILE, syscall.O_RDONLY, 0000)
    defer syscall.Close(fd)

    // Attempts to lock the LockFile. Exit immediately if the lock cannot be acquired
    if err := syscall.Flock(fd, syscall.LOCK_EX|syscall.LOCK_NB); err != nil {
        log.Println("Cannot acquire lock. Terminate program now !")
        os.Exit(1)
    }
}

func main() {
    FLock()
    ...
}
```

Thôi xong, ko lock được ! Ez, do đoạn `defer syscall.Close(fd)`. Vì trong Go, các function defer được push vào và pop ra theo cơ chế của Stack, nên khi `Flock()` kết thúc, `defer` được tự động pop ra và thực thi. [More detail](https://blog.golang.org/defer-panic-and-recover).

> A defer statement pushes a function call onto a list. The list of saved calls is executed after the surrounding function returns.

Làm thế nào để 2 concurent chờ nhau, :thumbsup: [`sync.WaitGroup`](https://golang.org/pkg/sync/#example_WaitGroup) chút kiến thức còn rơi rớt =))

```go
package main

import (
    "io/ioutil"
    "log"
    "os"
    "sync"
    "syscall"
    "time"
)

const LOCKFILE = "main.lck"

// <-chan bool: recieve-only channel
// chan<- bool: send-only channel
func FLock(cLock <-chan bool, wg *sync.WaitGroup) {
    // Make LockFile
    if _, err := os.Stat(LOCKFILE); err != nil {
        ioutil.WriteFile(LOCKFILE, []byte(""), 0644)
    }

    // Open LockFile for reading only
    fd, _ := syscall.Open(LOCKFILE, syscall.O_RDONLY, 0000)
    defer func() {
        // Wait for MAIN action finish
        <-cLock
        syscall.Close(fd)
    }()

    // Attempts to lock the LockFile. Exit immediately if the lock cannot be acquired
    if err := syscall.Flock(fd, syscall.LOCK_EX|syscall.LOCK_NB); err != nil {
        log.Println("Cannot acquire lock. Terminate program now !")
        os.Exit(1)
    }

    // Notice to Main action that lock can be acquired
    wg.Done()
}

func main() {
    var wg sync.WaitGroup
    cLock := make(chan bool)
    wg.Add(1)
    go FLock(cLock, &wg)
    wg.Wait()
    log.Println("--- [Start] main program ---")
    log.Println("...")
    time.Sleep(15 * time.Second)
    log.Println("--- [Finish] main program ---")
    cLock <- true
    close(cLock)
}
```

Code rất trong sáng và KH bảo, tao nghĩ chỉ cần dùng mỗi `chan` là đủ, không gần `sync.WaitGroup`.

![](/assets/img/posts/2018-07-19-golang-file-lock-using-flock-system-call/confused.png){: .align-center width=12px}

```go
package main

import (
    "io/ioutil"
    "log"
    "os"
    "syscall"
    "time"
)

const LOCKFILE = "main.lck"

func FLock(cLock chan bool) {
    // Make LockFile
    if _, err := os.Stat(LOCKFILE); err != nil {
        ioutil.WriteFile(LOCKFILE, []byte(""), 0644)
    }

    // Open LockFile for reading only
    fd, _ := syscall.Open(LOCKFILE, syscall.O_RDONLY, 0000)
    defer func() {
        // Wait for MAIN action finish
        <-cLock
        syscall.Close(fd)
    }()

    // Attempts to lock the LockFile. Exit immediately if the lock cannot be acquired
    if err := syscall.Flock(fd, syscall.LOCK_EX|syscall.LOCK_NB); err != nil {
        log.Println("Cannot acquire lock. Terminate program now !")
        os.Exit(1)
    }

    // Notice to Main action that lock can be acquired
    cLock <- true
}

func main() {
    cLock := make(chan bool)
    go FLock(cLock)
    // Wait Lockfile complete to acquire
    <-cLock
    log.Println("--- [Start] main program ---")
    log.Println("...")
    time.Sleep(15 * time.Second)
    log.Println("--- [Finish] main program ---")
    cLock <- true
    close(cLock)
}
```

![](/assets/img/posts/2018-07-19-golang-file-lock-using-flock-system-call/okay.jpg){: .align-center}

Đúng là chỉ cần channel là đủ :rofl: Cuối cùng mình sửa lại thành thế này theo ý mình. Code dài hơn nhưng nó làm mình gợi nhớ về thuở xa xưa khi mới tập tành tìm cách xác định khi nào một chan bị closed :joy:

```go
package main

import (
    "io/ioutil"
    "log"
    "os"
    "syscall"
    "time"
)

const LOCKFILE = "main.lck"

func FLock(cLock chan bool) {
    // Make LockFile
    if _, err := os.Stat(LOCKFILE); err != nil {
        ioutil.WriteFile(LOCKFILE, []byte(""), 0644)
    }

    // Open LockFile for reading only
    fd, _ := syscall.Open(LOCKFILE, syscall.O_RDONLY, 0000)
    defer func() {
        syscall.Close(fd)
        // Notify to MAIN, we have been unlocked LockFile
        close(cLock)
    }()

    // Attempts to lock the LockFile. Exit immediately if the lock cannot be acquired
    if err := syscall.Flock(fd, syscall.LOCK_EX|syscall.LOCK_NB); err != nil {
        log.Println("Cannot acquire lock. Terminate program now !")
        os.Exit(1)
    }

    // Notice to MAIN action that lock can be acquired
    cLock <- true
    // Wait to MAIN action finish all processing
    cLock <- true
}

func main() {
    cLock := make(chan bool)
    go FLock(cLock)
    // Wait Lockfile complete to acquire
    <-cLock
    log.Println("--- [Start] main program ---")
    log.Println("...")
    time.Sleep(15 * time.Second)
    log.Println("--- [Finish] main program ---")
    for {
        select {
        // Waiting FLock close cLock chan
        case _, ok := <-cLock:
            // cLock is closed
            if !ok {
                cLock = nil
            }
        }
        // Finish MAIN action after LockFile is unlocked
        if cLock == nil {
            break
        }
    }
}
```

Tổng quát hơn ta có thể declare một `struct Flock`, cùng với các method `Lock()`, `Wait()` & `Unlock()` để dễ sử dụng trong nhiều action. Công việc này xin dành cho bạn đọc =))

Cùng một vấn đề có rất nhiều cách Implement để giải quyết. Dù là cách nào thì cũng học được kiến thức mới, nếu ko phải học thì cứ cho là ôn lại kiến thức cũ cũng được :expressionless:

## References

- [The ps Command](http://www.linfo.org/ps.html)
- [Preventing duplicate cron job executions](http://bencane.com/2015/09/22/preventing-duplicate-cron-job-executions/)
- [多重起動を禁止する](https://qiita.com/ymko/items/522493e2b188aa8f267a)
