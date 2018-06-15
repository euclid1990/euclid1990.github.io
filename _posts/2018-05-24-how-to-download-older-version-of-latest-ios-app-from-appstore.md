---
layout: post
title:  "Hướng dẫn cách tải về tệp .ipa bất kỳ phiên bản nào của ứng dụng trên App Store"
date:   2018-05-24 11:23:04 +0700
categories: Tips
tags:
  - .ipa
  - App Store
  - Download
hide_thumbnail: true
image: /assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/thumbnail.png
---

Nếu bạn đang dùng iOS device (iPhone, iPad) thì bài viết này sẽ rất hữu ích.

Lý do mình viết bài này chỉ đơn giản là vì thiết bị iPhone mình đang dùng đã xuống cấp và hiệu năng không thể đáp ứng nổi phiên bản iOs hiện tại (`11`). Do vẫn đứng yên vị tại iOs `9.3.5` nên mỗi lần có ứng dụng có phiên bản mới, mình đều rất đắn đo khi cập nhật latest version :joy:.

Trong một buổi tối đẹp trời, lỡ tay cập nhật ứng dụng `youtube` phiên bản mới (`13.19.7`), có một sự thất vọng không hề nhẹ, chỉ xem một vài video mà máy nóng, pin tụt như tụt quần, tải comment thì giật thôi rồi. So với phiên bản `12.30.12` thì quá tồi. Nên mình thử lên mạng tìm phiên bản Youtube `.ipa` offical cũ để copy vô điện thoại cài lại. Tuy nhiên chẳng thấy chỗ nào có :cry:.

Sau một hồi lần mò đủ thứ, lục tung Google thì cũng đã có giải pháp. Mình sẽ trình bày các tải bên dưới đây. Let's Go :rocket:

## Công cụ

- [iTunes phiên bản `12.63`](https://www.apple.com/itunes/download/). Lưu ý không sử dụng phiên bản mới hơn, do từ `iTunes 12.7` đã bỏ chức năng quản lý app trên iPhone/iPad. Dĩ nhiên thư mục `Mobile Applications` trên máy tính cũng trở nên vô dụng :stuck_out_tongue_closed_eyes: Để thấy sự vô dụng đó bạn có thể xem [thread sau](https://discussions.apple.com/thread/8077773).

- [Charles Proxy phiên bản `4.2.5`](https://www.charlesproxy.com/).

Cả 2 phần mềm này bạn có thể tải về bản cài đặt trên Windows tại [Onedrive](https://1drv.ms/f/s!AsY96NdxxwbYghITV2k4tPJx3woV), trên Mac thì vui lòng tự tìm nhé :stuck_out_tongue_closed_eyes:. Sau khi tải về máy tính hãy cài đặt chúng.

## Tiến hành

### Bước 1

- Khởi động `iTunes`
- Tắt cập nhật `iTunes`
![Disable iTunes auto update]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step01-Disable_Auto_Update_iTunes_01.png' | relative_url }})
Chọn `Edit > Preferences`.
![Disable iTunes auto update]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step01-Disable_Auto_Update_iTunes_02.png' | relative_url }})
Bỏ check ở ô `Check for new software updates automatically` và chọn `OK`.

### Bước 2

- Vào App Store trên `iTunes`
![Go to Appstore on iTunes]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step02-Go_to_Appstore.png' | relative_url }})
- Chọn `iPhone` App nếu bạn muốn tải `.ipa` cho iPhone.
![Go to Appstore and select iPhone]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step02-Go_to_Appstore_iPhone_App.png' | relative_url }})

### Bước 3

- Tìm app cần tải `.ipa` về máy. Bài viết này mình sẽ tải lại `youtube` phiên bản cũ.
![Search app to download]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step03-Search_app_to_download_ipa.png' | relative_url }})
- Bấm vào để xem chi tiết app
![View app detail]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step03-View_detail_of_App.png' | relative_url }})

### Bước 4

- Mở phần mềm `Charles Proxy` lên.
![Open Charles Proxy]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step04-Open_Charles.png' | relative_url }})

### Bước 5

- Quay lại ứng dụng `iTunes`, và bấm vào nút <button>+ GET</button>. Có thể sẽ phải nhập thông tin iCloud, nhập vào và click và GET nhé.
![Get Youtube App]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step05-Click_button_[GET]_to_download_to_[Mobile_Applications]_folder.png' | relative_url }})

### Bước 6

- Dừng việc tải file `.ipa` về máy tính trên `iTunes`
![App downloading status]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step06-Pause_download_01.png' | relative_url }})
Bấm vào nút <button>❚❚</button> để dừng tải.
![Pause app download status]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step06-Pause_download_02.png' | relative_url }})

### Bước 7

- Chuyển qua ứng dụng `Charles Proxy`, ta sẽ có 1 danh sách các request như bên dưới. Chú ý cái **Buy Server** `p12-buy.itunes.apple.com`, chỗ `p12` thì số `12` có thể là 1 con số bất kỳ nhé bạn, ví dụ `p39`, `p20`, ...
![Apple Buy Server]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step07-Enable_SSL_Proxying_01.png' | relative_url }})
- Cài đặt `Charles Root Certificate`
![Install Charles Root Certificate]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step07-Enable_SSL_Proxying_02.png' | relative_url }})
![Install Charles Root Certificate]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step07-Enable_SSL_Proxying_03.png' | relative_url }})
![Install Charles Root Certificate]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step07-Enable_SSL_Proxying_04.png' | relative_url }})
![Install Charles Root Certificate]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step07-Enable_SSL_Proxying_05.png' | relative_url }})
- Chuột phải vào **Buy Server** và **ENABLE SSL PROXYING**
![Enable SSL Proxying]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step07-Enable_SSL_Proxying_06.png' | relative_url }})

### Bước 8

- Quay lại ứng dụng `iTunes` và tiến hành download lại `.ipa`. Lại bấm vào nút <button>+ GET</button> nhé :tired_face:
![Get app again]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step08-Redownload_App_witth_SSL_Proxying_01.png' | relative_url }})
![Get app again]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step08-Redownload_App_witth_SSL_Proxying_02.png' | relative_url }})
- Chú ý, khi bắt đầu download thì ta lại `pause` nó lại.
![Get app again]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step08-Redownload_App_witth_SSL_Proxying_03.png' | relative_url }})
- Quay sang ứng dụng `Charles Proxy`. Sẽ thấy **Buy Server** được request qua proxy.
![Show content]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step08-Redownload_App_witth_SSL_Proxying_04.png' | relative_url }})

    Phần đóng khung đỏ ở trên (tab `Content`) chính là phiên bản `.ipa` mà `iTunes` sẽ tải về máy tính.

    Phần đóng khung đỏ bên dưới (tab `XML Text`) chính là danh sách các phiên bản `.ipa` của ứng dụng, thứ tự từ cũ (trên cùng) tới mới (dưới cùng).

### Bước 9

- Sao chép toàn bộ phiên bản của App sang Notepad để dễ tham chiếu sau này.
![Select all app version]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step09-Copy_version_information_to_Notepad_01.png' | relative_url }})
![Paste all app version to Notepad++]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step09-Copy_version_information_to_Notepad_02.png' | relative_url }})

### Bước 10

- Mở và bấm vào button cái bút (compose an HTTP request)
![Compose new HTTPS request]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step10-Compose_new_request_buyProduct_01.png' | relative_url }})
- Sau đó sửa lại version app muốn tải về (đóng khung màu đỏ).
![Compose new HTTPS request]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step10-Compose_new_request_buyProduct_02.png' | relative_url }})
Sau khi sửa xong nhấn vào nút <button>Execute</button>
- Tap **Breakpoints** sẽ xuất hiện
![Breakpoints HTTPS request]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step10-Compose_new_request_buyProduct_03.png' | relative_url }})
- Nhớ vào **Edit Request** và sửa lại phiên bản muốn tải về rồi nhấn vào nút <button>Execute</button>
![Breakpoints HTTPS request]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step10-Compose_new_request_buyProduct_04.png' | relative_url }})
- Quay lại ứng dụng `iTunes` cửa sổ nhập thông tin sẽ xuất hiện, nhập lại user iCloud và password và **GET** :sunglasses:
![Get app again]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step10-Compose_new_request_buyProduct_05.png' | relative_url }})
- Sau đó trên `Charles Proxy` sẽ xuất hiện thêm 1, 2 cái **Breakpoints**, nếu có đoạn:
    ```xml
    <key>appExtVrsId<key>
    <string>...</string>
    ```
nhớ sửa lại phiên bản của mấy cái đó về bản cần tải, rồi nhấn vào nút <button>Execute</button>, còn không thì cứ cứ nhấn vào nút <button>Execute</button> đại đi =)) :rofl:
![Execute again and again]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step10-Compose_new_request_buyProduct_06.png' | relative_url }})

### Bước 11

- Sau khi hết cái để <button>Execute</button> trên `Charles Proxy`, chuyển qua `iTunes` bạn sẽ thấy điều kỳ diệu, file `.ipa` cần tải đã sắp có hàng.
![Confirm download status on iTunes]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step11-Confirm_download_status_on_iTunes.png' | relative_url }})

### Bước 12

- Kiểm tra tệp tin `.ipa` trên ổ cứng
![Confirm download status on disk]({{ '/assets/img/posts/2018-05-24-how-to-download-older-version-of-latest-ios-app-from-appstore/Step12-Confirm_download_file_in_Mobile_Applications_folder.png' | relative_url }})
   - **Windows 7**: `C:\Users\Username\My Music\iTunes\iTunes Media\Mobile Applications\`
   - **Mac OS X**: `~/Music/iTunes/iTunes Media/Mobile Applications/`

Các bạn thấy đấy, mình đã tải được file `.ipa` của `youtube` iOs App từ đời nào ý :rofl:

Do quá phấn khích nên mình đã tải về rất nhiều `.ipa` của `youtube` các phiên bản từ 10.43.9, ..., 11.21.8, ... 12.33.9, ... 13.19.7 =)) Mọi người có thể lấy về tại [link này](https://1drv.ms/f/s!AsY96NdxxwbYggOEaR5hItwrgrFB).

Nếu bạn không có nhiều budget cho việc đầu tư thiết bị mới với hiệu năng tốt hơn, hay trung thành với những phiên bản app tuy cũ nhưng stable, less bug thì hãy dùng cách này nhé. File `.ipa` sau khi tải về có thể cài đặt thông qua `iTunes` luôn :satisfied:
