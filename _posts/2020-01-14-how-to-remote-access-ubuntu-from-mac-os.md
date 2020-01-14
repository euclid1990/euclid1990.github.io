---
layout: post
title:  "Điều khiển từ xa máy tính Ubuntu qua máy Mac"
date:   2020-01-14 08:00:04 +0700
categories: Tips
tags:
  - RDP
  - VNC
  - Remote
hide_thumbnail: true
image: /assets/img/posts/2020-01-14-how-to-remote-access-ubuntu-from-mac-os/thumbnail.jpg
---

Hiện tại do nhu cầu công việc nên mình bắt buộc phải dùng 2 máy tính có 2 hệ điều hành khác nhau là Ubuntu 16.04 / Linux và MacOS High Sierra. Do máy Mac là máy làm việc chính nên khi phát sinh nhu cầu cần làm việc trên máy Ubuntu để tiết kiệm thời gian di chuyển chỗ ngồi 😏 mình thường xuyên sử dụng phần mềm remote giúp truy cập sang máy Ubuntu từ máy Mac một cách gọn lẹ 🤗

Bài viết sau sẽ hướng dẫn các bạn cách cài đặt truy cập từ xa và sử dụng.

### 1. Cài đặt máy Ubuntu

- Cài đặt `x11vnc`
```terminal
$ sudo apt-get install x11vnc
```
- Thiết lập Firewall
```terminal
$ sudo ufw allow 5900/tcp
```
- Thiết lập mật khẩu truy cập máy Ubuntu từ xa
```terminal
$ x11vnc -storepasswd
```
Sau khi gõ lệnh trên và enter, bạn sẽ được hỏi mật khẩu truy cập. Mật khẩu này được lưu trong file: `~/.vnc/passwd`

### 2. Cài đặt máy Mac

- Tải và Cài đặt VNC Viewer
Link: [https://www.realvnc.com/en/connect/download/viewer/](https://www.realvnc.com/en/connect/download/viewer/)
Đây là một sản phẩm của RealVNC - Trên 20 năm kinh nghiệm chuyên cung cấp công nghệ truy cập và điều khiển thiết bị từ xa qua phương thức VNC.
![](/assets/img/posts/2020-01-14-how-to-remote-access-ubuntu-from-mac-os/vnc-app.png)

### 3. Cách truy cập từ xa

- Trên máy Ubuntu
  - Khởi động VNC server
  ```terminal
  $ x11vnc -usepw -forever
  ```
  `-forever` giúp X11VNC server luôn luôn chạy ngay cả khi client ngắt kết nối, còn `-usepw` chỉ định việc yêu cầu VNC password mỗi lần kết nối được thiết lập.
  - Lấy địa chỉ IP trên máy Ubuntu
  ```terminal
  $ hostname -I | cut -d' ' -f1
  192.168.18.72
    ```

- Trên máy Mac
  - Tạo mới kết nối
  ![](/assets/img/posts/2020-01-14-how-to-remote-access-ubuntu-from-mac-os/vnc-new-connection.png)
  - Điền thông tin kết nối
  ![](/assets/img/posts/2020-01-14-how-to-remote-access-ubuntu-from-mac-os/vnc-create-connection.png)
  - Tiến hành kết nối tới máy tính Ubuntu
  ![](/assets/img/posts/2020-01-14-how-to-remote-access-ubuntu-from-mac-os/vnc-connect.png)
  - Kết nối thành công
  ![](/assets/img/posts/2020-01-14-how-to-remote-access-ubuntu-from-mac-os/vnc-remote-success.png)

🌥 Lưu ý: Do ở thiết lập mặc định, kết nối VNC không được mã hoá, nên có thể có nguy cơ về bảo mật. Để tăng cường bảo mật cho kết nối bạn có thể cài đặt thêm VPN hoặc sử dụng SSL/TLS tunnel hoặc SSH tunnel.
