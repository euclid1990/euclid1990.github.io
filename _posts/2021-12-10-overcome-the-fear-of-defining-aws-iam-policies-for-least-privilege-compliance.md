---
layout: post
title:  "Vượt qua nỗi sợ hãi khi phải xác định AWS IAM Policy tuân thủ nguyên tắc đặc quyền tối thiểu"
date:   2021-12-10 00:49:04 +0700
categories: Go
tags:
  - Design patterns
hide_thumbnail: true
image: /assets/img/posts/2021-12-10-overcome-the-fear-of-defining-aws-iam-policies-for-least-privilege-compliance/principle-of-least-privilege.jpeg
---

## The Principle of Least Privilege - Nguyên tắc đặc quyền tối thiểu

Không biết bạn đã từng nghe tới nguyên tắc này chưa? nhưng về cơ bản khi thực hiện tạo ra người dùng trong hệ thống, chúng ta sẽ chỉ cấp cho người dùng những quyền mà họ cần để hoàn thành công việc, không cấp thừa cũng như cấp thiếu, chỉ cấp đủ. Chỉ khi nào cần chúng ta mới cấp thêm các quyền cần thiết cho họ. Ví dụ: với tài khoản người dùng sử dụng cho mục đích duy nhất là tạo bản sao lưu dữ liệu thì tài khoản này chỉ có quyền chạy các ứng dụng liên quan đến sao lưu. Mọi đặc quyền khác, chẳng hạn như cài đặt phần mềm mới, đều bị không được cấp phép.

Lợi ích của nguyên tắc này khi triển khai trên mã Code cũng như Infrastructure:

- Hệ thống ổn định hơn. Khi các tài khoản sử dụng bị giới hạn quyền tối thiểu, thì tác động không mong muốn ảnh hưởng xấu tới các thành phần trên hệ thống cũng giảm.
- Bảo mật hệ thống hơn. Giảm số lượng lỗ hổng trên toàn hệ thống khi mã code bị khai thác.
- Dễ dàng triển khai. Do quyền cần cấp là tối thiểu, không tốn công sức hoặc thủ tục yêu cầu nâng cao để xin cấp quyền.

Chính vì lẽ đó mà đây là một trong số những nguyên tắc bất kỳ người quản trị hệ thống nào cũng cần tuân thủ. Và khi làm việc với các Cloud Provider như AWS, GCP bạn cũng sẽ đều bắt gặp nó trên Document Guideline:
- [Google Cloud Platform - Don't get pwned: practicing the principle of least privilege](https://cloud.google.com/blog/products/identity-security/dont-get-pwned-practicing-the-principle-of-least-privilege)
- [Amazon web service - Techniques for writing least privilege IAM policies](https://aws.amazon.com/blogs/security/techniques-for-writing-least-privilege-iam-policies/)

## Vấn đề

Đề tuân thủ `nguyên tắc đặc quyền tối thiểu`, thì cách làm rất dễ đó chính là sử dụng **IAM policies** nhằm giới hạn quyền cho **IAM principal** (`user`, `group`, ...). Tuy nhiên có một vấn đề là khi tạo mới các `Policy` bạn cần nắm vững ứng dụng hiện tại cần những quyền nào, để cấp phép các Action tương ứng cho `Policy` đó. Đây là việc không hề đơn giản, đơn cử với 2 bối cảnh sau:

- Bạn tham gia một dự án giữa chừng, và ứng dụng yêu cầu quyền để sử dụng dịch vụ AWS S3. Ở thời điểm hiện tại một service thông dụng của AWS là S3 thì số lượng action tương ứng lên tới con số trên 124 actions.
![](/assets/img/posts/2021-12-10-overcome-the-fear-of-defining-aws-iam-policies-for-least-privilege-compliance/s3-actions.png){: .align-center}
Nếu bạn là người tham gia dự án từ đầu thì công việc này không quá khó, bạn có thể tạo policy và update liên tục trong quá trình phát triển, nhưng nếu chỉ tham gia vào giai đoạn cuối cùng thì đôi lúc sẽ khó liệt kê hết các quyền cần sử dụng nếu đội dự án không tạo tài liệu yêu cầu quyền cụ thể.

- Bạn cần viết `Terraform` chạy CI/CD để dựng môi trường cho dự án môi và ban đầu, để nhanh chóng đưa dự án có thể chạy, bạn sử dụng luôn quyền AdministratorAcess cho `user` dùng trên CI provider như `CircleCI`, `Github Action`, sau đó mỗi khi có thay đổi, bạn apply nó lên dịch vụ AWS. Rồi tới khi dịch vụ của bạn cần cải thiện yếu tố bảo mật để phù hợp với business, bạn mới nhận ra rằng, để xác định quyền tối thiểu cho `user` đã tạo mới thực sự là ác mộng, bởi mỗi lần thử và thất bại để tìm ra quyền tối thiếu ngốn của bạn rất nhiều thời gian.

![](/assets/img/posts/2021-12-10-overcome-the-fear-of-defining-aws-iam-policies-for-least-privilege-compliance/special-hell.jpeg){: .align-center}

Vòng xoáy quanh quẩn này có thể mô tả lại các bước như sau:

- [1] Tạo **IAM** `user` không có quyền gì và secret credentials dùng trên dịch vụ CI/CD.
- [2] **IAM** `user` thực thi câu lệnh `aws` hoặc `terraform`
- [3] CI trả về lỗi authorization (403) nếu như **IAM** `user` thiếu quyền truy cập
- [4] Bạn tiến hành thêm quyền truy cập cho **IAM** `user`
- [5] Lặp lại bước [2] cho tới khi hết lỗi

## Giải pháp

Rất may mắn, trong số nhiều người gặp phải vấn đề này thì cũng có người chịu bỏ thời gian research và sẵn sàng chia sẻ cách khắc phục vấn đề cho cộng đồng 😂

> [**iamlive**](https://github.com/iann0036/iamlive)
>
> Generate an IAM policy from AWS calls using client-side monitoring (CSM) or embedded proxy

