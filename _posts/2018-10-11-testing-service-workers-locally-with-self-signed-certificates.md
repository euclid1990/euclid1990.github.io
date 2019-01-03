---
layout: post
title:  "Testing service workers trên HTTPS sử dụng self signed certificates"
date:   2018-10-11 07:23:04 +0700
categories: Tips
tags:
  - Progressive Web App
  - Service worker
  - Self SSL
  - Chrome
hide_thumbnail: true
image: /assets/img/posts/2018-10-11-testing-service-workers-locally-with-self-signed-certificates/thumbnail.jpg
---

Gần đây mình đang phát triển một ứng dụng web app (PWA). Tuy nhiên, khi debug service worker trên local với địa chỉ `https://localhost:8500` (Mình hết cổng trên máy nên dùng tạm cổng này :rofl: :joy:) mình gặp phải lỗi sau:

```js
An SSL certificate error occurred when fetching the script.
Failed to load resource: net::ERR_CERT_COMMON_NAME_INVALID
```

![](/assets/img/posts/2018-10-11-testing-service-workers-locally-with-self-signed-certificates/chrome_debug.png)

Dẫn tới việc `service worker` không được install và các chức năng đi theo nó không chạy. Sau khi tìm hiểu thì mình được biết nguyên nhân là do bản thân trình duyệt đã quy định [vậy](https://www.chromium.org/Home/chromium-security/prefer-secure-origins-for-powerful-new-features) trừ việc cài đặt từ một số origin đặc biệt như `http://localhost` (không có `s` nhé):

> Chrome is going to make Service Workers available only to secure origins, because it provides the origin with a new, higher degree of control over a user's interactions with the origin over an extended period of time, and because it gives the origin some control over the user's device as a background task.

Cách giải quyết thì các bạn có thể tham khảo tại đây: [Service Worker Debugging](https://www.chromium.org/blink/serviceworker/service-worker-faq)

### Đối với một domain không có HTTP

```terminal
$ /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --unsafely-treat-insecure-origin-as-secure=http://your.insecure.site:8080
```

### Đối với một domain HTTPS sử dụng self-signed certificate

```terminal
$ /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --allow-insecure-localhost https://localhost:8500/install.html
```

Đối với cách khởi động qua terminal thì bạn bắt buộc phải tắt toàn bộ các instance Chrome đang chạy trên máy.

Hoặc còn một cách khác là các bạn mở một tab mới & vào địa chỉ [chrome://flags/#allow-insecure-localhost](chrome://flags/#allow-insecure-localhost) chọn [Enabled] cho `Allow invalid certificates for resources loaded from localhost.` và [Relauch] trình duyệt.

![](/assets/img/posts/2018-10-11-testing-service-workers-locally-with-self-signed-certificates/allow_insecure_localhost.png)

Sau khi khởi động lại ta có kết quả service worker được cài đặt thành công mặc dù trên console có warning: :laughing:

```js
This site does not have a valid SSL certificate! Without SSL, your site's and visitors' data is vulnerable to theft and tampering. Get a valid SSL certificate before releasing your website to the public.
```

![](/assets/img/posts/2018-10-11-testing-service-workers-locally-with-self-signed-certificates/self_ssl_warning.png)

Nếu bạn muốn generate self signed certificate nhanh chóng thì có thể sử dụng command trong repo này nhé: [https://github.com/euclid1990/generate-self-signed-certificate](https://github.com/euclid1990/generate-self-signed-certificate)

DONE :rocket:
