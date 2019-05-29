---
layout: post
title:  "Hướng dẫn sử dụng Namecheap domain với AWS Route53 và AWS EC2"
date:   2019-05-29 07:23:04 +0700
categories: Outsource
tags:
  - AWS
  - DNS
hide_thumbnail: true
image: /assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/thumbnail.png
---

Nếu bạn đang đọc bài viết này, thì rất có thể bạn đang gặp khó khăn khi di chuyển (transfer) tên miền (domain) từ các nhà cung cấp tên miền (domain providers) như [Namecheap](https://www.namecheap.com), [GoDaddy](https://www.godaddy.com/), [Porkbun](https://porkbun.com/), [Domain.com](https://www.domain.com/) ... về [Amazon Web Service (AWS) Route53](https://aws.amazon.com/route53/).

Có hai lý do khiến bạn gặp vấn đề này đó là:
- Trước khi triển khai dịch vụ trên AWS, bạn đã sở hữu tên miền trên một nhà cung cấp khác mà không phải là AWS.
- Bạn muốn tiết kiệm chi phí duy trì domain cũng như muốn tận dụng các mã giảm giá từ các dịch vụ cung cấp domain.

Đừng lo, trong bài viết này tôi sẽ hướng dẫn các bạn khắc phục các vấn đề từng bước một để thiết lập thành công một domain từ nhà cung cấp dịch vụ khác tới Route53 và EC2 instance.

## Yêu cầu

Trước khi bắt đầu, hãy chắc chắn rằng bạn có đủ 2 điều kiện sau:
- Sở hữu Namecheap domain. Vì bài viết này mình chỉ hướng dẫn với domain mua tại Namecheap, tuy nhiên các nhà cung cấp dịch vụ khác cũng tương tự.
![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/namecheap.png)
- AWS Account với tối thiểu một EC2 instance đang chạy Web Server. Ở đây mình sử dụng `busybox http` làm web server, cài đặt rất đơn giản và dễ sử dụng, trên thực tế bạn có thể dùng `Apache` hay `Nginx` hay `Caddy`, ...

## Thực hiện

### Bước 1: Tạo EC2 instance & Elastic IP

Bạn đăng nhập vào `AWS Management Console` > `EC2`. Việc bạn cần làm tạo 1 instance EC2 và gán cho nó một `Elastic IP`. Đây là công việc khá đơn giản, mình ko làm hướng dẫn chi tiết phần này.

Đừng lo lắng vì `Elastic IP` là hoàn toàn miễn phí, và AWS sẽ chỉ charge tiền trên sự lãng phí của bạn mà thôi. Tức là khi bạn tạo một `IP` mà nó lại không thuộc về bất kỳ instance nào.

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/ec2-management-console.png)

Để cài đặt `busybox` và create 1 html web page đơn giản, bạn `SSH` vào bên trong EC2 & chạy lệnh sau:

```terminal
$ wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64
$ sudo mv busybox-x86_64 /usr/local/bin/busybox
$ sudo chmod +x /usr/local/bin/busybox
$ cat > index.html <<EOF
<html>
<head><title>Hello World</title></head>
<body><h1>Hello, World</h1></body>
</html>
EOF
$ sudo /usr/local/bin/busybox httpd -p 80
```

Và đây là kết quả, hãy gõ địa chỉ `Elastic IP` của EC2 lên browser:

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/busybox-http.png)

### Bước 2: Tạo mới Public Hosted Zone

Truy cập [Route53](https://console.aws.amazon.com/route53/home) > [Hosted zones](https://console.aws.amazon.com/route53/home#hosted-zones:)

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/hosted-zone.png)

Tạo mới `Public Hosted Zone`

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/create-new-hosted-zone.png)

Tạo mới `A record set` cho `www`.

Ở phần `Value` các bạn hãy thay giá trị `IP: 198.51.100.234` thành `Elastic IP` của EC2 instance mà bạn đã tạo trước đó nhé.

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/create-www-a-record-set.png)

Tạo mới `A record set` empty cho tên miền `non-www`.

Lưu ý chọn Alias là `[yes]` & ở mục `Target` hãy gõ tên miền `wwww` trong record trước đó.

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/create-empty-a-record-set.png)

Kết quả sau cùng, chúng ta có các record sau:

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/create-2-a-record-set.png)

### Bước 4:

Tiến hành copy toàn bộ DNS Servers trong Hosted Zone của [Route53 AWS](https://console.aws.amazon.com/route53/home#hosted-zones:)

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/route53-4-dns-server.png)

sang NameCheap DNS Servers. Các bạn vô [Domain List](https://ap.www.namecheap.com/domains/list/) và bấm `[Manage]` chọn domain tương ứng

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/namecheap-domain-list.png)

sau đó Paste các DNS mà bạn vừa copy từ `Route53` qua `[NAMESERVERS]` (Nhớ chọn `Custom DNS` nhé)

![](/assets/img/posts/2019-05-29-pointing-namcheap-domain-to-aws-route53-ec2/namecheap-custom-dns.png)

**Lưu ý: Loại bỏ các ký tự '`.`' ở cuối mỗi DNS copy từ Hosted Zone trước khi paste sang DNS của Namecheap.**

```
ns-125.awsdns-15.com.           ----> ns-125.awsdns-15.com
ns-644.awsdns-16.net.           ----> ns-644.awsdns-16.net
ns-1436.awsdns-51.org.          ----> ns-1436.awsdns-51.org
ns-1650.awsdns-14.co.uk.        ----> ns-1650.awsdns-14.co.uk
```

### Bước 5: Tận hưởng thành quả 🤣

Mở trình duyệt lên và truy cập vào domain của bạn nhé, nếu hiện web page `Hello, World` là mọi công việc setup đã thành công.

Nếu như bạn đã hoàn thành toàn bộ các step mà website vẫn chưa hoạt động hãy cố gắng đợi khoảng 1 ngày (24h) nhé, vì việc change DNS giữa các Name Server đôi lúc phải mất một khoảng thời gian cho các ISP cập nhật.

*Ở thời điểm bài viết được đăng, chi phí cho Route 53 là khá nhỏ:*
- $0.50 / hosted zone / month với 25 hosted zones đầu tiên.
- $0.10 / hosted zone / month với các hosted zones thêm vào sau đó.

Tất nhiên còn các chi phí cho Queries, Traffic, Health Checks nhưng hoàn toàn không đáng kể mấy 💵
