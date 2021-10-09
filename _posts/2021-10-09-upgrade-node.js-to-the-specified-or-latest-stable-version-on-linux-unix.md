---
layout: post
title:  "Nâng cấp Node.js lên phiên bản chỉ định hoặc mới nhất trên Unix/Linux"
date:   2021-10-09 08:00:04 +0700
categories: TIL
hide_thumbnail: true
image:
---

Nay có việc phải code React.js mà Node trong máy hơi cũ, không xài được Typescript do dính lỗi:\
```bash
The engine "node" is incompatible with this module. Expected version "^10.13.0 || ^12.13.0 || ^14.15.0 || >=15.0.0".
```
Check phiên bản node hiện tại trong máy thì thấy đang chạy `v14.3.0`. Lên ngay phiên bản lastest stable version bằng cách chạy lần lượt **3** câu lệnh đơn giản sau:
  - Clear npm cache trên máy

    `$ sudo npm cache clean -f (force)`
  - Cài đặt package `n` – Interactively Manage Your Node.js Versions ([Xem thêm tại đây](https://www.npmjs.com/package/n))

    `$ sudo npm install -g n`
  - Tiến hành nâng cấp lên phiên bản chạy ổn định (stable)

    `$ sudo n stable`

    Bạn có thể thay thế `stable` bằng `latest`, `lts` (long term support) hoặc bất kỳ phiên bản nào, ví dụ `14.18.0`.

Sau khi nâng cấp xong hãy kiểm tra lại phiên bản node trên máy bằng lệnh:
```bash
$ node -v
```
