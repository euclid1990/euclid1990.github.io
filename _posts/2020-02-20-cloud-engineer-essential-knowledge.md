---
layout: post
title:  "Một số kiến thức cơ bản mà Cloud Engineer cần nắm được"
date:   2020-05-10 08:00:04 +0700
categories: Cloud
tags:
  - AWS
  - GCP
hide_thumbnail: true
image: /assets/img/posts/2020-02-20-cloud-cloud-engineer-essential-knowledge/thumbnail.png
---
(_Bài viết còn đang chưa hoàn thành: 10/05/2020_)

Trong bài viêt này mình muốn giới thiệu với các bạn các kiến thức cần thiết khi làm việc như một Cloud Engineer hay Solution Architect trong dự án mà mình cảm thấy là cần phải biết và hiểu. Đôi khi chỉ đa phần là kiến thức cơ bản, nhưng nếu không hiểu rõ thì sẽ rất dễ nhầm lẫn và đưa ra quyết định sai khi đứng trước những lựa chọn.

Đầu tiên, chúng ta thử tìm hiểu xem mình mong muốn trở thành một người thế nào nhé. Với vai trò của mình, bạn sẽ cần phải đứng giữa Customers/Technical Teams/Manager Teams, để hiểu những ai, cần những gì, vì không phải ai cũng giống nhau. Bạn cần tìm cách xác định thứ mà mỗi bên mong muốn thực sự. Sau khi đã xác định được mục tiêu, bạn cần phải chuyển đổi mục tiêu của bên này (khách hàng) sang bên kia (đội ngũ developer) qua việc phân tích, thiết kế hệ thống ... Đây là lúc bạn cần vận dụng rất nhiều kiến thức của bản thân và của cả đội ngũ phát triển để có thể đạt được mục tiêu chung đã đề ra.

![](/assets/img/posts/2020-02-20-cloud-cloud-engineer-essential-knowledge/types-of-technical-people.png){: .align-center}

Trên đây là 1 số kiểu SA (Solution Architect) Engineer 🐖
- Kiểu "I" (chuyên gia theo định nghĩa cũ) thường là những người có kiến thức ở một lĩnh vực nhất định và đi rất sâu trong lĩnh vực đó.
- Kiểu "ー" (nếu xoay ngang kiểu "I") đại diện cho một kiểu người hiểu biết trải rộng trên nhiều lĩnh vực, nhưng không có cái nào đi sâu.
- Kiểu "T" (kết ở kiểu "I" và "ー") là những người có kiến thức tổng quát ở nhiều lĩnh vực cùng khả năng phát triển thêm các hiểu biết sâu trong một vài lĩnh vực cụ thể. Họ có thể trở thành các short-term chuyên gia khi cần.
- Kiểu "M" (trùm cuối 🤣) không chỉ mang một khối lượng kiến thức sâu và rộng, còn là nhóm người có rất nhiều kinh nghiệm thực tế.

Kiến thức đầu tiên mà mình muốn đề cập, đó đơn giản chỉ là một số đơn vị đo lường:

### 1. MB, Mb và MiB

Nghe 3 từ này thì có vẻ giống hệt nhau nhưng chắc chắn một điều là không !!!. Trước hết bạn cần nắm được các term trên là viết tắt của từ gì:

![](/assets/img/posts/2020-02-20-cloud-cloud-engineer-essential-knowledge/megabyte-vs-megabit.jpg){: .align-center}

- `MB` = `Megabyte`
  - Megabyte (MB) là một thuật ngữ dùng để nói về khả năng lưu trữ dữ liệu của các phần cứng chuyên dụng như ổ đĩa 500MB, ổ đĩa 1024MB (1GB), 2048 MB (2GB), ...
- `Mb` = `Megabit`
  - Megabit (Mb) lại hay dùng để nói về chuyện truyền tải dữ liệu, chúng ta thường rất hay gặp Mbps (Megabit per second) được hiểu là tốc độ truyền dữ liệu trên mỗi giây.
    - Ví dụ, tốc độ Internet của bạn là 15.5Mbps, có nghĩa là 15.5 Megabit đang được truyền đi mỗi giây.
- `MiB` = `Mebibyte` (Đôi lúc có thể viết là `MIB`)
  - Mebibyte là một bội số của đơn vị byte trong đo lường khối lượng thông tin số.  Tiền tố nhị phân mebi nghĩa là $$ 2^{20} $$, bởi vậy $$ 1 \text{ mebibyte} = 2^{20} = 1048576 \text{ bytes} = 1024 \text{ kibibyte} $$.
    - Đơn vị này được giới thiệu bởi International Electrotechnical Commission (IEC) năm 1998 nhằm thay thế megabyte, được sử dụng trong nhiều bối cảnh để đại diện cho $$ 2^{20} $$ byte, thay cho định nghĩa của tiền tố mega của International System of Units (SI) là bội số của $$ 10^6 $$ byte.



Để rõ hình dung hơn, chúng ta có các công thức về mối tương quan như sau:
- 1 Byte = 8 Bits
- 1 Megabyte = 1,000,000 Bytes
- 1 Megabit = 1,000,000 Bits == 125,000 bytes
- 1 MebiByte = $$ 2^{20} $$ Bytes = 1,048,576 Bytes
- 1 Megabyte = 8 Megabit

Mặc dù `12.5MB = 8*12.5 = 100Mb`, nhưng xét về góc độ kinh doanh, nếu cùng một gói cước mạng viễn thông, nhà cung cấp quảng cáo "gói cước mạng 100 Mbps" thay vì "gói cước mạng 12.5 MBps" rõ ràng khách hàng nào không hiểu sẽ thấy con số 100 trông ấn tượng hơn rất nhiều, một thủ thuật nhỏ nhưng mang lại hiệu quả maketing vô cùng lớn, giúp thu hút được nhiều khách hàng hơn. 💯

... Còn nữa ...

## Tham khảo
https://www.atlassian.com/incident-management/kpis/sla-vs-slo-vs-sli
https://www.ibm.com/support/knowledgecenter/SSPHQG_7.2/concept/ha_concepts_fault.html
https://searchdisasterrecovery.techtarget.com/tip/How-to-determine-your-disaster-recovery-objectives
https://docs.ukcloud.com/articles/other/other-ref-gib.html
https://www.perfmatrix.com/latency-bandwidth-throughput-and-response-time/
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances.html
https://aws.amazon.com/blogs/database/understanding-burst-vs-baseline-performance-with-amazon-rds-and-gp2/
