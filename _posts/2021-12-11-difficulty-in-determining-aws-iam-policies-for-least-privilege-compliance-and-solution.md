---
layout: post
title:  "Khó khăn gặp phải khi xác định AWS IAM Policy và cách khắc phục"
date:   2021-12-11 00:49:04 +0700
categories: Cloud
tags:
  - AWS
  - GCP
hide_thumbnail: true
image: /assets/img/posts/2021-12-11-difficulty-in-determining-aws-iam-policies-for-least-privilege-compliance-and-solution/principle-of-least-privilege.jpeg
---

Tiêu đề bài viết khá dài, hiện tại mình vẫn chưa nghĩ ra được cái nào ngắn gọn mà đầy đủ ý hơn 😆
## The Principle of Least Privilege - Nguyên tắc đặc quyền tối thiểu

Không biết bạn đã từng nghe tới nguyên tắc này chưa? nhưng về cơ bản khi thực hiện tạo ra người dùng trong hệ thống, chúng ta sẽ chỉ cấp cho người dùng những quyền mà họ cần để hoàn thành công việc, không cấp thừa cũng như cấp thiếu, chỉ cấp đủ. Chỉ khi nào cần chúng ta mới cấp thêm các quyền cần thiết cho họ. Ví dụ: với tài khoản người dùng sử dụng cho mục đích duy nhất là tạo bản sao lưu dữ liệu thì tài khoản này chỉ có quyền chạy các ứng dụng liên quan đến sao lưu. Mọi đặc quyền khác, chẳng hạn như cài đặt phần mềm mới, đều bị không được cấp phép.

Lợi ích của nguyên tắc này khi triển khai trên mã Code cũng như Infrastructure:

- Hệ thống ổn định hơn. Khi các tài khoản sử dụng bị giới hạn quyền tối thiểu, thì tác động không mong muốn ảnh hưởng xấu tới các thành phần trên hệ thống cũng giảm.
- Bảo mật hệ thống hơn. Giảm số lượng lỗ hổng trên toàn hệ thống khi mã code bị khai thác.
- Dễ dàng triển khai. Do quyền cần cấp là tối thiểu, không tốn công sức hoặc thủ tục yêu cầu nâng cao để xin cấp quyền.

Chính vì lẽ đó mà đây là một trong số những nguyên tắc bất kỳ người quản trị hệ thống nào cũng cần tuân thủ. Và khi làm việc với các Cloud Provider như AWS, GCP bạn cũng sẽ đều bắt gặp nó trên Document Guideline cũng như các bài viết về kỹ thuật trên Blog của họ:
- [Google Cloud Platform - Don't get pwned: practicing the principle of least privilege](https://cloud.google.com/blog/products/identity-security/dont-get-pwned-practicing-the-principle-of-least-privilege)
- [Amazon web service - Techniques for writing least privilege IAM policies](https://aws.amazon.com/blogs/security/techniques-for-writing-least-privilege-iam-policies/)

## Vấn đề

Đề tuân thủ `nguyên tắc đặc quyền tối thiểu`, thì cách làm rất dễ đó chính là sử dụng **IAM policies** nhằm giới hạn quyền cho **IAM principal** (`user`, `group`, ...). Tuy nhiên có một vấn đề là khi tạo mới các `Policy` bạn cần nắm vững ứng dụng hiện tại cần những quyền nào, để cấp phép các Action tương ứng cho `Policy` đó. Đây là việc không hề đơn giản, đơn cử với 2 bối cảnh sau:

- Bạn tham gia một dự án giữa chừng, và ứng dụng yêu cầu quyền để sử dụng dịch vụ AWS S3. Ở thời điểm hiện tại một service thông dụng của AWS là S3 thì số lượng action tương ứng lên tới con số trên 124 actions.
![](/assets/img/posts/2021-12-11-difficulty-in-determining-aws-iam-policies-for-least-privilege-compliance-and-solution/s3-actions.png){: .align-center}
Nếu bạn là người tham gia dự án từ đầu thì công việc này không quá khó, bạn có thể tạo policy và update liên tục trong quá trình phát triển, nhưng nếu chỉ tham gia vào giai đoạn cuối cùng thì đôi lúc sẽ khó liệt kê hết các quyền cần sử dụng nếu đội dự án không tạo tài liệu yêu cầu quyền cụ thể.

- Bạn cần viết `Terraform` chạy CI/CD để dựng môi trường cho dự án mới, thời điểm đầu để nhanh chóng đưa dự án có thể chạy, bạn sử dụng luôn quyền AdministratorAcess cho `user` dùng trên CI provider (`CircleCI`, `Github Action`), sau đó mỗi khi có thay đổi, bạn apply nó lên các dịch vụ AWS. Rồi tới khi sản phẩm của công ty bạn cần cải thiện yếu tố bảo mật để phù hợp với business (HIPAA, GDPR, ...), bạn mới nhận ra rằng, để xác định quyền tối thiểu cho `user` đã tạo thực sự là ác mộng, bởi việc chạy thử CI/CD mỗi lần rồi thất bại để tìm ra quyền tối thiếu ngốn của bạn quá nhiều thời gian.

![](/assets/img/posts/2021-12-11-difficulty-in-determining-aws-iam-policies-for-least-privilege-compliance-and-solution/special-hell.jpeg){: .align-center}

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

Công cụ này sử dụng một trong 2 cơ chế sau để xác định policy:

- Turning Client Side Monitoring ON
  - Do AWS SDKs và AWS CLI đều hỗ trợ Client Side Monitoring (CSM), nếu bạn kích hoạt tính năng này, mỗi API request sẽ đều được report qua UDP port 31000.

    Để kiểm chứng bạn có thể dùng `tcpdump` để `capture` các packet đi qua interface network.

    ```bash
    # List all your network interfaces
    $ ifconfig -a
    # Run a packet trace on that interface.  I’m using lo0 the loopback interface, so I run:
    $ sudo tcpdump -i lo0 -n udp port 31000 -A
    ```

    ```bash
    # Turn on CSM
    $ export AWS_CSM_ENABLED=true
      export AWS_CSM_PORT=31000
      export AWS_CSM_HOST=127.0.0.1
    # Run sample AWS command
    $ aws s3 ls
    ```

    Kết quả

    ![](/assets/img/posts/2021-12-11-difficulty-in-determining-aws-iam-policies-for-least-privilege-compliance-and-solution/tcpdump.png){: .align-center}

- Using an HTTP proxy
  - Cấu hình các biến môi trường `HTTP_PROXY` và `HTTPS_PROXY` với tên miền DNS hoặc địa chỉ IP và cổng mà máy chủ proxy của `iamlive` đang chạy.

    ```bash
    export HTTP_PROXY=http://10.15.20.25:1234
    export HTTP_PROXY=http://proxy.example.com:1234
    export HTTPS_PROXY=http://10.15.20.25:5678
    export HTTPS_PROXY=http://proxy.example.com:5678
    ```

  Bằng cách này, mọi request tới AWS sẽ chạy qua proxy, và chúng ta có thể filter được `permission` cần cấp.

Và đây là kết quả, `Policy` được sinh ra theo thời gian thực mỗi khi bạn gửi request tới máy chủ của dịch vụ AWS thông qua AWS SDK hoặc CLI.

![](/assets/img/posts/2021-12-11-difficulty-in-determining-aws-iam-policies-for-least-privilege-compliance-and-solution/iamlive.gif){: .align-center}

## Hướng dẫn sử dụng

### Cài đặt

```bash
$ brew install iann0036/iamlive/iamlive
```

### Sử dụng

Cá nhân mình khuyến khích sử dụng `iamlive` ở chế độ **CSM Mode**. (Đã thử với một số trường hợp **Proxy Mode** chạy không có output).

Trước hết để bật mode CSM bạn sử dụng 1 trong 2 cách sau

- Export các biến môi trường:

  ```bash
  $ export AWS_CSM_ENABLED=true
    export AWS_CSM_PORT=31000
    export AWS_CSM_HOST=127.0.0.1
  ```

- Edit file `~/.aws/config`:

  ```js
  csm_enabled = true
  ```

Chạy `iamlive` với câu lệnh sau trong terminal window khác:

```bash
$ iamlive --set-ini --profile myprofile --refresh-rate 1 --sort-alphabetical --host 127.0.0.1
```

Lưu ý chọn đúng `profile` **AWS** của bạn nhé !!!

Thực hiện các tác vụ liên quan tới AWS SDK hoặc CLI, đồng thời nhận kết quả về `Policy` mà mình mong muốn. 😄

![](/assets/img/posts/2021-12-11-difficulty-in-determining-aws-iam-policies-for-least-privilege-compliance-and-solution/terraform-demo.png){: .align-center}
