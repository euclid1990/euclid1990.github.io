---
layout: post
title:  "Làm thế nào để tải video từ các trang web sử dụng giao thứ HLS với m3u8 file"
date:   2019-08-02 07:23:04 +0700
categories: Tips
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

======================
## Update 09/26
Để download video từ youtube, các bạn có thể sử dụng [youtube-dl](https://github.com/ytdl-org/youtube-dl).

Cách cài đặt khá đơn giản, chạy các câu lệnh sau trên terminal:

```terminal
$ sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
$ sudo chmod a+rx /usr/local/bin/youtube-dl
```

Để tải 1 video từ youtube bất kỳ, bạn chỉ cần sử dụng cú pháp sau:

```terminal
$ youtube-dl https://youtubelink
```

Để lựa chọn các định dạng sẵn có bạn thêm flag `-F` vào:

```terminal
$ youtube-dl -F https://youtubelink
[youtube] XXX: Downloading webpage
[youtube] XXX: Downloading video info webpage
[info] Available formats for XXX:
format code  extension  resolution note
249          webm       audio only tiny   53k , opus @ 50k (48000Hz), 3.02MiB
250          webm       audio only tiny   69k , opus @ 70k (48000Hz), 3.77MiB
140          m4a        audio only tiny  130k , m4a_dash container, mp4a.40.2@128k (44100Hz), 7.89MiB
251          webm       audio only tiny  133k , opus @160k (48000Hz), 6.94MiB
160          mp4        256x144    144p   73k , avc1.4d400c, 30fps, video only, 2.31MiB
278          webm       256x144    144p  121k , webm container, vp9, 30fps, video only, 5.69MiB
133          mp4        426x240    240p  128k , avc1.4d4015, 30fps, video only, 4.25MiB
242          webm       426x240    240p  174k , vp9, 30fps, video only, 5.75MiB
134          mp4        640x360    360p  245k , avc1.4d401e, 30fps, video only, 7.42MiB
243          webm       640x360    360p  284k , vp9, 30fps, video only, 8.43MiB
135          mp4        854x480    480p  351k , avc1.4d401f, 30fps, video only, 10.80MiB
244          webm       854x480    480p  394k , vp9, 30fps, video only, 12.10MiB
136          mp4        1280x720   720p  472k , avc1.4d401f, 30fps, video only, 15.93MiB
247          webm       1280x720   720p  668k , vp9, 30fps, video only, 21.65MiB
137          mp4        1920x1080  1080p 1710k , avc1.640028, 30fps, video only, 72.11MiB
248          webm       1920x1080  1080p 2080k , vp9, 30fps, video only, 92.92MiB
43           webm       640x360    360p , vp8.0, vorbis@128k, 40.76MiB
18           mp4        640x360    360p  377k , avc1.42001E, mp4a.40.2@ 96k (44100Hz), 22.98MiB
22           mp4        1280x720   720p  390k , avc1.64001F, mp4a.40.2@192k (44100Hz) (best)
```

Để download định dạng mong muốn:

```terminal
$ youtube-dl -f 248+251 https://youtubelink
```

Để có thể download riêng video và audio, sau đó merge lại thành 1 file thì bạn cần cài thêm FFMPEG hoặc Arconv. Cách cài đặt thì mình đã ghi trong đầu bài viết.

Câu lệnh dùng để download chuẩn video+audio và merge lại thành một file như sau:

```terminal
$ youtube-dl -f 'bestvideo[ext=webm]+bestaudio[ext=webm]' --merge-output-format webm https://youtubelink
hoặc
$ youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]' --merge-output-format mp4 https://youtubelink
```

Done !
