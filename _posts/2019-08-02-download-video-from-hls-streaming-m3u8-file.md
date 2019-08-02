---
layout: post
title:  "Làm thế nào để tải video từ các trang web sử dụng giao thứ HLS với m3u8 file"
date:   2019-08-02 07:23:04 +0700
categories: Outsource
tags:
  - HLS
hide_thumbnail: true
image: /assets/img/posts/2019-08-02-download-video-from-hls-streaming-m3u8-file/m3u8-file-streaming.png
---

Trước hết, `HLS` là viết tắt của chữ `HTTP Live Streaming`, là một giao thức streaming bitrate được phát triển bởi Apple. Không giống như các kỹ thuật thông thường, `HLS` sử dụng cách chia tệp tin video ra làm nhiều file nhỏ, các file này thường có đuôi `.ts` và được stream tuần tự về phía player của người dùng.

Khi create một video dưới dạng `HLS`, chúng ta sẽ thu được file `.m3u8`, file này chính là file chứ index tới các file `.ts`. Nhờ vào nội dung file này mà player biết được cần tải các file nào về và timing play như thế nào.

Chi tiết hơn về HSL các bạn có thể tham khảo tại [đây](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) và tại [đây nữa](https://developer.apple.com/streaming/).

Để convert một video sang định dạng `HLS` chúng ta có thể sử dụng một công cụ rất mạnh, đó là [FFmpeg](https://ffmpeg.org/) (Tác giả của nó chính là [Fabrice Bellard](https://bellard.org/)).

Đi thẳng vào vấn đề chính, ở đây mình đang tham gia một số khoá học trên [Linux Academy](https://linuxacademy.com/), tuy nhiên do muốn học cả lúc offline trên thiết bị di dộng nên muốn tải video của các course trên site về máy. Sau khi inspect network thì thấy rằng website đang sử dụng giao thức `HLS` để truyền phát video, việc tải về khá đơn giản. Các bạn có thể làm như sau:

### Step 1:
- Cài đặt [FFmpeg](https://ffmpeg.org/), Tải tại [đây](https://ffmpeg.org/download.html)

### Step 2:
- Sau khi hoàn thành bước 1, hãy mở trình duyệt Chrome lên & sử dụng tính năng [Developer Tools] > Chọn tab [Network] > Nhập text `m3u8` vào input filter như trong hình dưới và truy cập tới video bài học.
![](/assets/img/posts/2019-08-02-download-video-from-hls-streaming-m3u8-file/linuxacademy.png)
- Sau đó chỉ cần copy lại URL tới file `.m3u8` và chạy câu lệnh sau trên terminal
```terminal
ffmpeg -i https://path_to_m3u8_file -c copy -bsf:a aac_adtstoasc output.mp4
```
- Đợi cho `ffmpeg` tải file `output.mp4` về máy là xong ! :rofl:
