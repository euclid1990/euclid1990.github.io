---
layout: post
title: "Ngăn chặn truy cập tới ứng dụng web đặt sau AWS CloudFront theo Client IP"
date: 2023-12-26 10:00:00 +0700
categories: Cloud
tags:
  - AWS
  - CloudFront
  - WAF
  - Nginx
  - DDos
hide_thumbnail: true
image: /assets/img/posts/2023-12-26-blocking-access-to-application-behind-cloudfront-by-real-client-ip/thumbnail.png

---

Trong bài viết này mình mô tả một sự cố gần đây xảy ra trong hệ thống mà mình gặp phải khiến RDS Database CPU/Memory luôn gặp phải tình trạng quá tải, ảnh hưởng tới trải nghiệm người dùng. Đồng thời chia sẻ về các giải pháp khả thi mà các thành viên trong đội dự án đã thử triển khai thành công. Thực tế, còn có nhiều giải pháp xịn hơn, nếu bạn đã từng gặp phải vấn đề này, hãy để lại comment chia sẻ nhé 🚀

Hệ thống của mình là một web app thông thường được host trên AWS với mô hình giản lược sau:

![](/assets/img/posts/2023-12-26-blocking-access-to-application-behind-cloudfront-by-real-client-ip/web-architect.png)

Toàn bộ Traffic sẽ đi qua các thành phần `CloudFront`, `ALB` trươc khi tới `ECS Cluster`.


Cụm `ECS` gồm có 3 thành phần Container chính

- NGINX: Đóng vai trò reverse proxy nhận request
- Web Application: Dynamic web app nhận request từ Nginx
- Fluentd: Push log từ `NGINX` và `Web` Container sang `Firehorse` > `Opensearch` > `Kibana` phục vụ cho việc phân tích log. ([Custom log routing
](https://docs.aws.amazon.com/AmazonECS/latest/userguide/using_firelens.html) / [Streaming AWS ECS logs via Fluentd using AWS FireLense](https://faun.pub/streaming-aws-ecs-logs-via-fluentd-fluent-bit-using-aws-firelense-log-driver-14f2782fc885))

## Vấn đề

Vào một ngày bình thường như bao ngày, AWS liên tục gửi alert về việc % sử dụng CPU của RDS Database liên tục trên ngưỡng 90% đáng báo động 😢

Trước hoàn cảnh đó, cả team cần gấp rút tìm ra nguyên nhân tại sao lại xảy ra hiện tượng như vậy. Chúng ta hãy sang phần tiếp theo xem đội dự án đã xử lý ra sao.

## Quá trình điều tra

### Giai đoạn 1

Thuận tự nhiên, lỗi ở RDS thì engineer vào AWS **RDS Performance Insight** > **TOP SQL**. Phát hiện ra câu query chính khiến tải RDS tăng.

```sql
SELECT `articles`.`id`
FROM `articles`
WHERE `articles`.`deleted_at` IS NULL
ORDER BY `articles`.`id`
DESC LIMIT 8 OFFSET 357678
/*controller:article,action:index*/
```
*Câu query thực gây ra việc tăng tải của hệ thống đã được thay đổi*

Hàng loạt các câu query tương tự như trên xuất hiện trong bảng xếp hạng TOP những câu query chiếm tài nguyên của RDS.

Ở giai đoạn này chúng tôi phán đoán ở `ArticleController` > `Index` action đang phát sinh nhiều request hơn thực tế.

### Giai đoạn 2

Điều tra log người dùng truy cập hệ thống xem bất thường gì không.

Với thiết kế ban đầu, hệ thống sẽ lưu toàn bộ dữ liệu log người dùng truy cập để phục vụ cho *Marketing Insight* và *Debug*. Logs được đẩy vào **Opensearch** và Visualize trên **Kibana**.

Filter log truy cập url `article` bởi `NGINX` theo khoảng thời gian, team phát hiện số lượng request tăng đột biết từ 10~15k request trong khoảng thời gian phát sinh sự cố. Đặc biết khi chú ý tới giá trị `OFFSET` câu query `SQL` ở giai đoạn 1, và query parameter phân trang `?page=`. Nhận thấy các số trong page đều chạy từ nhỏ đến rất lớn (>= 10,000), hành vi này không giống với thực tế end-user sử dụng mà thay vào đó khá giống từ **Crawler Bot** đi thu thập dữ liệu.

Trích xuất thông tin từ dữ liệu log, đội dự án thu được 2 thông tin chính:

- Dải IP của BOT (Check thông tin tại [ipinfo](https://ipinfo.io/123.100.101.123))
  - 123.100.101.0/24
  - 123.100.102.0/24
  - ...
  - *Đã chỉnh sửa khác với giá trị thực tế*
  - Để get network IP / CIDR 24 từ danh sách các IP có thể dụng đoạn code `Javascript/NodeJS` sau:

    ```javascript
    const ipList = [
      '123.100.101.1',
      '123.100.101.2',
      '123.100.102.5',
      ...
    ];

    function getCIDR(ip) {
      const ipParts = ip.split('.');
      return ipParts.slice(0, 3).join('.') + '.0/24';
    }

    const cidrArray = ipList.map(getCIDR);

    const uniqueCIDRs = Array.from(new Set(cidrArray));

    console.log(uniqueCIDRs);
    ```

- Header `User-Agent` của BOT

  ```conf
  Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36
  ```

### Giải pháp

Kết thúc giai đoạn 2, đội dự án phán đoán nguyên nhân RDS tăng tải do xuất hiện nhiều request từ **Crawler Bot**.

Đầu tiên cần xác định quan điểm chặn, phạm vi ảnh hưởng tới người dùng thực:

- Chặn những page mà chỉ có Bot muốn vào, người dùng hiếm vào (Hành vi)
- Chặn những page có performance thấp, ảnh hưởng RDS nếu tần suất truy cập nhiều

Từ đó đưa ra các giải pháp block access / chặn truy cập từ Crawler Bot như.

- Chặn truy cập theo header trên **ALB listener rules**
  - Condition match header của Bot
  - Action return fixed response 503
- Chặn truy cập từ các client IP thuộc dải IP của Bot / 1 trong 2 cách
  - Chặn truy cập trên **WAF** > **WebACLs**
    - Sử dụng `Rule builder` > Block hoặc Rate Limit các truy cập từ IP set của Bot
      - [Rate-based rule statement](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html)
      - Cấu hình các quy tắc - [Aggregation options and keys](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based-aggregation-options.html)
      - Ví dụ:

        ![](/assets/img/posts/2023-12-26-blocking-access-to-application-behind-cloudfront-by-real-client-ip/waf-webacls-rule.png)

        Mặc định mỗi **WebACLs** sẽ có 5000 WCU (WAF web ACL capacity units). 1 Rule trên sẽ mất 7 WCU. Nếu dùng quá 1500 WCU thì bạn cần lưu ý thêm **Price** của AWS WAF tại [đây](https://aws.amazon.com/waf/pricing/), bởi lúc đó AWS sẽ charge phí sử dụng.

    - Có thể sử dụng thêm Match regex URI / Query string với condition `?page=` để giảm thiểu độ ảnh hưởng
  - Chặn truy cập ở **NGINX Layer**
    - Sử config của NGINX bằng cách thêm phần **map** là IP range của Bot. ($http_cf_client_ip là real client IP được thêm vào header từ CloudFront)

    ```conf
    http {
      map $http_cf_client_ip $block_access {
        ~*^123\.100\.101\..*$   1;
        ~*^123\.100\.102\..*$   1;
        default                 0;
      }

      server {

        location / {
          if ($block_access) {
            return 403;
          }
          try_files $uri @app;
        }

        location @app {
          proxy_pass http://app_service;
          ...
        }
      }
    }
    ```

Sau nhiều cân nhắc thì kết quả đội dự án quyết định sử dụng giải pháp **AWS WAF - WebACLs** với khả năng block request linh hoạt và hiệu quả.

### Kiểm tra

Sử dụng conditions với điều kiện hẹp hơn, và các công cụ benchmark request như **[ApacheBench](https://httpd.apache.org/docs/2.4/programs/ab.html)**

```bash
$ ab -A basicAuthUsername:basicAuthPassword -n 100 -c 10 "https://example.com/"

-n requests: Số lượng request được gửi
-c concurrency: Số lượng request được thực hiện 1 thời điểm
```

---

**Bổ sung kiến thức:** `CloudFront` **không** pass `Client IP` sang `ALB` và `NGINX`, vậy làm thế nào để **có thể** get được `Client IP` trên các Layer này ??

![](/assets/img/posts/2023-12-26-blocking-access-to-application-behind-cloudfront-by-real-client-ip/web-architect.png)

Trên lý thuyết AWS cung cấp header "CloudFront-Viewer-Address" - [Adding CloudFront request headers
](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/adding-cloudfront-headers.html). Tuy nhiên muốn pass header này qua **origin* cần **disable caching** với Cache Policy.

Xem lại sơ đồ kiến trúc hệ thống, chúng ta thấy CloudFront module có trigger **Lambda@Edge Function** mỗi khi có request tới (Event Type = Origin-request). Về cơ bản để thêm header thì chỉ cần một đoạn code Javascript như sau:

```javascript
'use strict';

exports.handler = (event, context, callback) => {
    const request = event.Records[0].cf.request;
    const clientIp = event.Records[0].cf.request['clientIp'];
    const headerName = 'CF-Client-IP';
    request.headers[headerName.toLowerCase()] = [{ key: headerName, value: clientIp }];
    callback(null, request);
    return;
};
```

Tham khảo thêm về [Adding HTTP Security Headers Using Lambda@Edge and Amazon CloudFront](https://aws.amazon.com/blogs/networking-and-content-delivery/adding-http-security-headers-using-lambdaedge-and-amazon-cloudfront/)

Trên **NGINX** layer config như sau để log ra **Client IP**

```conf
log_format main escape=json
  '{'
    ...
    '"request":"$request",'
    '"request_method":"$request_method",'
    '"host":"$host",'
    '"uri":"$uri",'
    '"status":"$status",'
    '"http_cf_client_ip":"$http_cf_client_ip",'
    ...
  '}';

access_log  /var/log/nginx/access.log  main;
```

Cảm ơn bạn đã quan tâm tới tận cuối bài viết 😆
