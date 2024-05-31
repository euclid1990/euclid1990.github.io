---
layout: post
title: "Bypass internet packages với SSL/TLS Tunnel và SNI/DNS giả"
date: 2024-05-09 09:00:00 +0700
categories: TIL
tags:
  - Research
hide_thumbnail: true

---

Gần đây mình có nhận được một số quảng cáo về việc hack 4G data, ban đầu mình cũng không tin việc đó, vì đơn giản cũng làm trong nghề, nên không nghĩ là cả một nhà mạng với bao nhiêu con người tài năng lại dễ dàng để xảy ra lỗ hổng như vậy. Mặt khác vào mấy trang bán gói 4G giá siêu rẻ chỉ 10/20K 1 tháng cho mình cảm giác như kiểu web lập ra để scam người dùng nạp tiền 😂

Tuy nhiên sau khi tìm hiểu về cơ chế thì mình thấy việc này không hề giả trân 😆 bạn có thể tham khảo paper sau [Bypassing Content-based internet packages with an SSL/TLS Tunnel, SNI Spoofing, and DNS spoofing](https://arxiv.org/abs/2212.05447) được submit vào tháng 12 năm 2022.

<embed src="/assets/img/posts/2024-05-31-bypassing-content-based-internet-packages-with-an-ssltls-tunnel-sni-spoofing-and-dns-spoofing/2212.05447v1.pdf" type="application/pdf" width="100%" height="350">


**Cơ chế đơn giản như sau**

Chuyển hướng toàn bộ lưu lượng web đến một VPS cá nhân, giả mạo thông tin kết nối để ISP tin rằng bạn đang truy cập một máy chủ hợp pháp như máy chủ zoom.us, tiktok.com. Sau khi lưu lượng được chuyển hướng đến VPS, nó sẽ được chuyển đến các điểm đích thông qua một máy chủ proxy hoặc VPN cài đặt trên VPS. Điều này cho phép người dùng sử dụng internet chùa mà không bị hạn chế. :blush:

Về cơ bản thì nhà mạng vẫn làm ngơ thôi, còn nếu muốn chặn thì mình nghĩ chắc vẫn có cơ chế block 🥲
