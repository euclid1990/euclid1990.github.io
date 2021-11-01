---
layout: post
title:  "Hướng dẫn debug code Go trên Visual Studio Code / Terminal"
date:   2021-11-01 00:49:04 +0700
categories: Go
tags:
  - Design patterns
hide_thumbnail: true
image: /assets/img/posts/2021-11-01-how-to-debug-go-code-with-vscode-terminal/thumbnail.png
---

Trong bài viết này mình sẽ chia sẻ về cách debug Go code trên Visual Studio Code. Trước hết, chúng ta hãy tạo 1 đoạn code sample về tính số Fibonacci từ một danh sách các số cho trước trong Go như sau:

`fibonacci.go`
```go
package main

import "fmt"

var m = make(map[int]int, 0)

func fib(n int) int {
	if n < 2 {
		return n
	}

	var f int
	if v, ok := m[n]; ok {
		f = v
	} else {
		f = fib(n-2) + fib(n-1)
		m[n] = f
	}
	return f
}

func main() {
	args := []int{5, 1, 9, 98, 6}
	for _, n := range args {
		x := fib(n)
		fmt.Printf("fib(%d) = %d\n", n, x)
	}
}
```

Để chạy chương trình ta dùng lệnh đơn giản sau:

```bash
$ go run fibbonaci.go
```

## Debug trên Terminal

### Cài đặt

- Delve Debugger: Xem hướng dẫn cài đặt tại [đây](https://github.com/go-delve/delve/tree/master/Documentation/installation)

### Sử dụng

- [1]. Compile và debugging main package

  ```bash
  $ dlv debug fibonnaci.go
  ```

- [2]. Show source code

  ```bash
  (dlv) list fibonnaci.go:26
  22: func main() {
  23:         args := []int{5, 1, 9, 98, 6}
  24:         for _, n := range args {
  25:                 x := fib(n)
  26:                 fmt.Printf("fib(%d) = %d\n", n, x)
  27:         }
  28: }
  ```

- [3]. Set breakpoint: Có thể set theo Line number hoặc Function name

  ```bash
  (dlv) break fibonacci.go:26
  Breakpoint 1 (enabled) set at 0x10cdff8 for main.main() ./fibonacci.go:26

  (dlv) break main.fib
  Breakpoint 2 (enabled) set at 0x10cdd53 for main.fib() ./fibonacci.go:7

  (dlv) break fibonacci.go:25
  Breakpoint 3 (enabled) set at 0x10cdfe5 for main.main() ./fibonacci.go:25
  ```

- [4]. Lấy danh sách breakpoints hiện tại

  ```bash
  (dlv) breakpoints
  Breakpoint runtime-fatal-throw (enabled) at 0x1039440 for runtime.fatalthrow() /usr/local/go/src/runtime/panic.go:1163 (0)
  Breakpoint unrecovered-panic (enabled) at 0x10394c0 for runtime.fatalpanic() /usr/local/go/src/runtime/panic.go:1190 (0)
          print runtime.curg._panic.arg
  Breakpoint 1 (enabled) at 0x10cdff8 for main.main() ./fibonacci.go:26 (0)
  Breakpoint 2 (enabled) at 0x10cdd53 for main.fib() ./fibonacci.go:7 (0)
  Breakpoint 3 (enabled) at 0x10cdfe5 for main.main() ./fibonacci.go:25 (0)
  ```

- [5]. Để xoá Breakpoint số (1)

  ```bash
  (dlv) clear 1
  ```

- [6]. Run/Execute program cho tới khi gặp breakpoint / dùng lệnh `continue` hoặc `c`

  ```bash
  (dlv) continue
  > main.main() ./fibonacci.go:25 (hits goroutine(1):1 total:1) (PC: 0x10cdfe5)
      20: }
      21:
      22: func main() {
      23:         args := []int{5, 1, 9, 98, 6}
      24:         for _, n := range args {
  =>  25:                 x := fib(n)
      26:                 fmt.Printf("fib(%d) = %d\n", n, x)
      27:         }
      28: }

  (dlv) print n
  5

  (dlv) print x
  Command failed: could not find symbol value for x
  ```

  Ở trên ta thấy giá trị của `x` chưa defined vì theo logic chương trình cần tính toán ở hàm `fib(n)`

- [7]. Nhảy bước đơn vào từng `block` với `step` hoặc `s` / Muốn nhảy khỏi `block` với `stepout` hoặc `so`

  ```bash
  (dlv) step
  > main.fib() ./fibonacci.go:7 (hits goroutine(1):1 total:1) (PC: 0x10cdd53)
      2:
      3: import "fmt"
      4:
      5: var m = make(map[int]int, 0)
      6:
  =>   7: func fib(n int) int {
      8:         if n < 2 {
      9:                 return n
      10:         }
      11:
      12:         var f int

  (dlv) so
  > main.fib() ./fibonacci.go:7 (hits goroutine(1):2 total:2) (PC: 0x10cdd53)
      2:
      3: import "fmt"
      4:
      5: var m = make(map[int]int, 0)
      6:
  =>   7: func fib(n int) int {
      8:         if n < 2 {
      9:                 return n
      10:         }
      11:
      12:         var f int
  ```

- [8]. Nhảy bước đơn từng câu lệnh với `next` hoặc `n`

  ```bash
  (dlv) next
  > main.fib() ./fibonacci.go:8 (PC: 0x10cdd6a)
      3: import "fmt"
      4:
      5: var m = make(map[int]int, 0)
      6:
      7: func fib(n int) int {
  =>   8:         if n < 2 {
      9:                 return n
      10:         }
      11:
      12:         var f int
      13:         if v, ok := m[n]; ok {
  (dlv) next
  > main.fib() ./fibonacci.go:12 (PC: 0x10cdd88)
      7: func fib(n int) int {
      8:         if n < 2 {
      9:                 return n
      10:         }
      11:
  =>  12:         var f int
      13:         if v, ok := m[n]; ok {
      14:                 f = v
      15:         } else {
      16:                 f = fib(n-2) + fib(n-1)
      17:                 m[n] = f
  ```

- [9]. Trace stack với lệnh `stack`

  ```bash
  (dlv) stack
  0  0x00000000010cde25 in main.fib
    at ./fibonacci.go:16
  1  0x00000000010cde25 in main.fib
    at ./fibonacci.go:16
  2  0x00000000010cdfee in main.main
    at ./fibonacci.go:25
  3  0x000000000103bb63 in runtime.main
    at /usr/local/go/src/runtime/proc.go:225
  4  0x000000000106f021 in runtime.goexit
    at /usr/local/go/src/runtime/asm_amd64.s:1371
  ```

- [10]. Khi cần khởi động lại dùng lệnh `restart`

  ```bash
  (dlv) restart
  ```

## Debug trên Visual Studio Code

### Cài đặt

- [Go Plugin](https://marketplace.visualstudio.com/items?itemName=golang.Go)

![](/assets/img/posts/2021-11-01-how-to-debug-go-code-with-vscode-terminal/marketplace-go.png){: .align-center}

Sau khi cài đặt `Go Plugin` thì VSCode sẽ tự động cài đặt [Delve](https://github.com/go-delve/delve/tree/master/Documentation/installation) một open-source sẽ dùng cho việc debug code Go.

### Sử dụng

- [1]. Để thực hiện debug bạn vào `[Run and Debug]` hoặc nhấn tổ hợp phím `Command+Shift+D`
  ![](/assets/img/posts/2021-11-01-how-to-debug-go-code-with-vscode-terminal/start-debug.png){: .align-center}

- [2]. Để tiến hành debug bạn click vào button `[Run and Debug]` như trong ảnh
  ![](/assets/img/posts/2021-11-01-how-to-debug-go-code-with-vscode-terminal/debugging.png){: .align-center}

- Vậy là OK.
