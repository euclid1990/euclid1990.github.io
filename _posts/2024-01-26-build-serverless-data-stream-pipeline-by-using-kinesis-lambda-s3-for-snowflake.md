---
layout: post
title: "Xây dựng serverless data stream pipeline bằng AWS Kinesis, Lambda, S3 cho Snowflake"
date: 2024-01-26 10:00:00 +0700
categories: Data-Pipeline
tags:
  - AWS
  - Kinesis Data Streams
  - Data Pipeline
  - Snowflake
hide_thumbnail: true
image: /assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/thumbnail.png

---

Trong bài viết này mình sẽ chia sẻ về một mô hình mà mình sử dụng để đề xuất khách hàng trong quá trình thực hiện estimate một dự án trong bộ phận, mà trên thực tế bản thân cũng chưa từng sử dụng (liều ăn nhiều :meat_on_bone:) nên có thể sẽ phát sinh khá nhiều issue 😂.

Ban đầu cá nhân mình cũng rất mong muốn được một Data Engineer có nhiều kinh nghiệm chinh chiến trong nghề giúp sức, nhưng tìm hoài trong công ty không có vị trí này và cũng chưa thấy có ai đã từng vận hành hệ thống kiểu này cho khách, nên thôi tự mình thử implement coi sao 🫣, nếu bạn cũng có kinh nghiệm với các hệ thống xây dựng data pipeline kiểu này, hãy để lại comment góp ý nhé 🚀

## Bối cảnh

Hệ thống của khách hàng có 3 services chính, phục vụ end-user, mỗi service lại sử dụng các 3rd-party khác nhau để thu thập metrics nhằm theo dõi, đo lường, phân tích hành vi người sử dụng.

Mục đích cuối cùng là để có những bản báo cáo, phân tích, giúp bộ phận Marketing có sự đánh giá chính xác hơn về tình hình kinh doanh online và có cơ sở thực tế để thực hiện các chiến dịch Marketing, quảng cáo hiệu quả hơn.

![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/current-system.png)

Hệ thống gồm 3 services, và các event logs được gửi tới nhiều 3rd services phục vụ cho việc lưu trữ. Các event logs này sau đó được biến đổi cấu trúc và lưu lại vào mấy dịch vụ OLAP (Online Analytical Processing) serverless query service đại loại như `GCP BigQuery`, `AWS Athena`.

Dữ liệu từ các OLAP services sẽ được query và thực hiện insert vào `AWS DynamoDB` bởi một *Batch Process* đã được lập lịch.

Để quản trị viên có thể dễ dàng phân tích số liệu qua biểu đồ, phía khách hàng đã build một ứng dụng riêng, quẩy dữ liệu sau cùng từ `AWS DynamoDB` và hiển thị cho quản trị viên, nhân viên bán hàng, ...

## Vấn đề

Hệ thống trên đang tồn tại một số vấn đề như:

- Độ trễ trong việc hiển thị dữ liệu mới nhất, không phù hợp ở thời điểm cần phản hồi nhanh.
- Cấu trúc / Format dữ liệu không đồng nhất giữa các services
- Cần nhiều tài nguyên: thời gian xử lý, cấu hình máy chủ do dữ liệu ứ đọng lớn
- Gánh nặng quản trị hệ thống khi xảy ra lỗi một thành phần nào đó trong hệ thống

Trước những vấn đề đó, khách hàng tìm tới chúng tôi mong muốn có giải pháp khắc phục hiện trạng này. Như vậy cần cải thiện hệ thống để khắc phục các nhược điểm trên.

![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/new-system.png)

## Giải pháp

Để tiết kiệm tiền cho khách, đội dự án đập toàn bộ những phần cũ đi và xậy lại một Data Stream Pipeline 😆

- **Thời Gian Thực (Real-time)**: Cho phép xử lý dữ liệu ngay lập tức khi nó được tạo ra,thông tin mới nhất sẽ có sẵn ngay khi nó xuất hiện.
- **Khả Năng Mở Rộng (Scalability)**: Thích ứng với các biến động trong khối lượng dữ liệu.
- **Phản Hồi Nhanh (Quick Feedback)**: Với khả năng xử lý dữ liệu ngay lập tức, data stream pipeline cung cấp phản hồi nhanh chóng, giúp người dùng có thể đưa ra quyết định dựa trên thông tin mới ngay khi nó có sẵn.
- **Xử Lý Dữ Liệu Liên tục (Continuous Data Processing)**: Không chờ đến khi có một lô dữ liệu đầy đủ như trong batch processing.
- **Tính Linh Hoạt (Flexibility)**: Data stream pipeline thường linh hoạt hơn với khả năng xử lý dữ liệu đa dạng từ nhiều nguồn và định dạng khác nhau. Việc thêm mới và cập nhật dữ liệu có thể được thực hiện dễ dàng hơn.
- **Tiết Kiệm Tài Nguyên (Resource Efficiency)**: Do data stream pipeline thường chỉ xử lý dữ liệu cần thiết và không đợi đến khi có lượng dữ liệu lớn, nó có thể tiết kiệm tài nguyên so với việc xử lý theo batch.

### Kiến trúc Pipeline

![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/data-stream-pipeline-architect.png)

Trong kiến trúc trên thì các services lần lượt đóng vai trò nhiệm vụ sau:

- **AWS Kinesis Data Streams**: `Source Stream`
- **AWS Kinesis Data Firehose**: `Delivery Stream`
- **AWS Lambda**: `Transformer Function`
- **AWS S3**: `Data Lake`
- **Snowflake**: `Data Warehouse`

Tất cả đều là **Fully Managed Services** giúp giảm gánh nặng quản trị hệ thống cho đội ngũ kỹ sư hạ tầng bên khách.

#### Data Flow

- ① Các services hiện tại sẽ tích hợp Kinesis Client và thực hiện `PUT event` vào **Source Stream** (`AWS Kinesis Data Streams`).
- ② Sau đó event sẽ được chuyển tới **Delivery Stream** (`AWS Kinesis Data Firehose`)
  - ③ Tại đây event sẽ được phân loại, validate, transform thông qua **Transformer Function** (`Lambda Function`).
    - Trong trường hợp này chỉ đơn thuần là convert từ `JSON` dang `CSV` format ([*Loading data into Snowflake is fast and flexible. You get the greatest speed when working with CSV files*](https://www.snowflake.com/blog/how-to-load-terabytes-into-snowflake-speeds-feeds-and-techniques/))
- ④ Dữ liệu sau xử lý sẽ đi vào **Data Lake** (`AWS S3`)
- ⑤ Từ `AWS S3`, thông qua Snow Pipe Auto-ingestion, dữ liệu được đồng bộ tới `Database > Table` trên **Data Warehouse** (`Snowflake`)

#### Monitoring

... To be define :relieved:

## Proof of Concept

Toàn bộ code thực hiện demo từ khâu triển khai xây dựng hạ tầng trên AWS = CDK, Cấu hình Snowflake, Thực hiện `PUT` **dummy Orders data** tới Source Stream đều đã được public lên Github cá nhân mình:

⭐ 🌟 **[https://github.com/euclid1990/data-stream-pipeline](https://github.com/euclid1990/data-stream-pipeline)** :bell:

### 1. Chuẩn bị tài khoản

- Đăng ký tài khoản **AWS**: [https://portal.aws.amazon.com/billing/signup](https://portal.aws.amazon.com/billing/signup)
- Đăng ký tài khoản **Snowflake**: [https://signup.snowflake.com](https://signup.snowflake.com)
  - Sau khi hoàn tất đăng ký bạn sẽ có 400$ credit / Trial trong 1 tháng
    ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-signup-bonus.png)

### 2. Cấu hình chung

Trước hết chúng ta cần có source code trên máy

```bash
git clone git@github.com:euclid1990/data-stream-pipeline.git
cd data-stream-pipeline
```

Chỉnh sửa tệp tin **.env**

```bash
$ cp .env.example .env
$ code .env

CDK_STACK="PipelineStack"   >>>  Tên stack sẽ tạo trên AWS
CDK_ACCOUNT="123456789012"  >>>  *AWS Account ID* của bạn / Bắt buộc chỉnh sửa
CDK_REGION="us-east-1"      >>>  Region sẽ thực hiện deploy stack trên
CDK_USER_NAME="cdk"         >>>  Tên user name tạo mới để chạy CDK, tách biệt với default account
CDK_ROLE_NAME="cdk-role"    >>>  Role mà CDK_USER_NAME sẽ assume để có permission cần thiết provision AWS resources
CDK_USER_POLICY_NAME="cdk-inline-policy"   >>>  Tên user name tạo mới để chạy CDK, tách biệt với default account
```

Thêm quyền execute cho toàn bộ Bash script trong thư mục `/scripts`

```bash
chmod +x ./scripts/*/**/*.sh
```

### 3. Tạo AWS User/Role chạy CDK app

```bash
./scripts/create_iam_user_role.sh
```

### 4. Thực hiện deploy AWS Stack

```bash
./scripts/cdk.sh deploy --outputs-file ./outputs.json
```

Kết quả

![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/cdk-outputs.png)

Toàn bộ các Outputs sẽ được save ở file `./outputs.json` phục vụ cho việc setup **Snowflake** cũng như **Produce event** tới **Kinesis Data Streams**

### 5. Cấu hình Snowflake Auto-ingest S3 Data

Chi tiết bước này đã được phía **Snowflake** hướng dẫn rất kỹ tại đây

- [Refreshing External Tables Automatically for Amazon S3](https://docs.snowflake.com/en/user-guide/tables-external-s3)

Mình đã re-write lại các step = **bash script** cho tiện

- #### 5.1. Giới hạn quyền truy cập tới **AWS S3 Bucket** theo **VPC ID** của Snowflake account

  - Truy vấn thông tin **VPC ID** trên **Snowflake** nơi đặt tài khoản Snowflake của bạn trên AWS
    - Nội dung `SQL Query`: `./scripts/snowflake/01-allow-vpc-id/retrieve_snowflake_aws_vpc_id.sql`
    - Kết quả
      ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-account-vpc-id.png)

  - Thiết lập Bucket Policy tương ứng

    ```bash
    ./scripts/snowflake/01-allow-vpc-id/restrict_s3_access_to_vpc.sh <VPC_ID>
    ```

- #### 5.2. Cấu hình quyền truy cập từ **Snowflake** tới **AWS S3**

  - Tạo mới IAM Policy

    ```bash
    ./scripts/snowflake/02-aws-snowflake-s3-access-policy/create.sh
    ```

    ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-s3-access-policy.png)
  - Tạo mới IAM Role trên AWS

    ```bash
    ./scripts/snowflake/03-aws-snowflake-iam-role/create.sh`
    ```

    Output của việc thực thi script

    ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-query-instructions.png)

    Kết quả trên **AWS IAM Roles**

    ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-role.png)

    Ở bước này, toàn bộ các câu **SQL** cần dùng để thực thi trong các step sắp tới đã được generate thành file **`./scripts/snowflake/03-aws-snowflake-iam-role/integration.sql`**.

    ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-auto-generated-sql.png)

    Hãy copy & paste sang ***SQL Worksheet*** trên **Snowflake**.

    - Tạo Database **s3_to_snowflake**
    - Tạo S3 Storage Integration **s3_int**
    - Thực hiện truy xuất ARN của AWS IAM user được tạo tự động cho tài khoản Snowflake của bạn

      ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-desc-s3-integration.png)

      > - STORAGE_AWS_IAM_USER_ARN: AWS IAM user được tạo cho tài khoản Snowflake của bạn. Tất cả các tích hợp lưu trữ S3 đều sử dụng IAM user đó.
      > - STORAGE_AWS_EXTERNAL_ID: External ID cần thiết để thiết lập mối quan hệ tin cậy trust relationship.

    Thực thi theo các bước hướng dẫn từ Output của script trên.

    - Cho phép Snowflake IAM User quyền được truy cập tới Bucket Objects

      ```bash
      ./scripts/snowflake/03-aws-snowflake-iam-role/update.sh <STORAGE_AWS_IAM_USER_ARN> <STORAGE_AWS_EXTERNAL_ID>
      ```

      Kết quả trên AWS, **Trust relationships** đã được cập nhật

      ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-role-trust-relationship.png)

- #### 5.3. Làm mới dữ liệu của external table với Amazon SQS notifications cho S3 bucket

  - Sử dụng nội dung trong **`./scripts/snowflake/03-aws-snowflake-iam-role/integration.sql`** để tạo mới
    - **Stage**
    - **CSV Format**
    - **External Table**
  - Sau đó chúng ta có thể truy xuất thông tin tới ARN của SQS queue cho External table trong cột notification_channel
    ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-notification-channel.png)

  - Thực hiện cấu hình Event Notifications cho **S3 Bucket** với giá trị cột **notification_channel**
    ```bash
    ./scripts/snowflake/04-aws-s3-event-notification/create.sh <notification_channel_arn>
    ```

    Kết quả trên AWS, **S3 Event Notification** đã được cập nhật

    ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/aws-s3-event-notification.png)

### 6. Kiểm tra việc Sync dữ liệu / Auto-ingest sang Snowflake

#### Before

- Dữ liệu trên **AWS S3** và **Snowflake** đều chưa có gì 🚫

  ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/aws-s3-bucket-empty.png)


  ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-table-empty.png)


#### After

- Thực hiện stream 1,800 records tới **AWS Kinesis Data Streams**

```bash
./scripts/produce.sh 1800
```

![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/produce-1800-records-to-kinesis.png)

- Dữ liệu xuất hiện tại **S3 Desitnation Stream**

  ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/aws-s3-bucket-stream-data.png)

- Dữ liệu được auto-ingest tới **Snowflake Table**

  Record ID = 1,800 đã có mặt 😆

  ![](/assets/img/posts/2024-01-26-build-serverless-data-stream-pipeline-by-using-kinesis-lambda-s3-for-snowflake/snowflake-table-data.png)

🍀 Như vậy chúng ta có thể xác nhận luồng dữ liệu hoạt động như mong đợi. Kiểm chứng đề xuất có khả năng hiện thực hoá 💯

Cảm ơn bạn đã quan tâm tới tận cuối bài viết 🚀
