---
layout: post
title: "Phát hiện tình cờ về timezone của Việt Nam"
date: 2022-03-06 07:49:04 +0700
categories: TIL
tags:
  - Javascript
  - Timezone
hide_thumbnail: true
image: /assets/img/posts/2022-03-06-vietnam-timezone-before-1975-year/thumbnail.png

---

Nửa đêm ngồi code function chuyển đổi hiển thị thời gian cho người dùng trên nhiều vị trí địa lý mới phát hiện 1 sự thú vị không hề nhẹ 😂.

Anh em coder động vào bài toán xử lý các trường datetime trong Database chắc hẳn ai cũng từng phải convert thời gian từ múi giờ này qua múi giờ khác. Trên **PHP** thì có [Carbon](https://carbon.nesbot.com), **Javascript** thì dùng [Moment](https://momentjs.com/timezone), [Date-fns](https://date-fns.org/docs/Getting-Started)... đều là những thư viện khá mạnh phục vụ tận răng.

Để convert thời gian từ múi giờ `UTC` sang múi giờ của `Việt Nam (UTC+07)` thì có mấy timezone có thể dùng:
- _Asia/Bangkok_
- _Asia/Ho_Chi_Minh_
- _Asia/Saigon_

Mình thì trước giờ hay xài đại _Asia/Ho_Chi_Minh_, nay thử convert cái field time trong Database thì kết quả hơi choáng, code mình vẫn viết test đầy đủ mà nay dính case sau:

```javascript
moment.tz('1970-01-01T01:00:00.000Z', 'UTC').tz('Asia/Ho_Chi_Minh').format()
'1970-01-01T09:00:00+08:00'
```

Bạn thấy đấy, **1970-01-01T01:00:00.000Z** ở múi giờ `UTC` bị chuyển thành **1970-01-01T09:00:00_+08:00_** ở múi giờ `UTC+07`. Kết quả chính xác phải là **1970-01-01T08:00:00_+07:00_** chứ 😥 Tại sao timezone của Việt Nam ở thời điểm năm 1970 lại là `UTC+08` WTF !!!!

Vấn đề này làm mình mất 30p debug code, đâm vào cả trong thư viện mà chẳng hiểu sao nó xử lý vậy :joy:, cuối cùng phải search Wikipedia về lịch sử timezone của Việt Nam mới biết:

![](/assets/img/posts/2022-03-06-vietnam-timezone-before-1975-year/vietnam_timezone.png){: .align-center}

Hoá ra trước thời điểm **13/06/1975** các vùng miền của Việt Nam sử dụng múi giờ là khác nhau, chỗ `UTC+08` chỗ `UTC+07`.

Như vậy thì thư viện không hề sai :v Nhưng nếu dùng để convert fiel `TIME()` trong Database ra giờ thì sẽ cần lưu ý để phù hợp với logic.

```javascript
moment.tz('1975-06-13T01:00:00.000Z', 'UTC').tz('Asia/Ho_Chi_Minh').format()
'1975-06-13T08:00:00+07:00'
```

Test lại cho chắc 😎

Tóm lại là đừng có nghĩ **timezone** của một quốc gia luôn cố định ở mọi thời điểm, mà nó có thể thay đổi 🐖
