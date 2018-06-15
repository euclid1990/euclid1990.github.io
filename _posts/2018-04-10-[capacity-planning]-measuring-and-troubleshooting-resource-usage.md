---
layout: post
title:  "[Capacity Planning] Đo lường và khắc phục sự cố sử dụng tài nguyên"
date:   2018-04-10 10:23:04 +0700
categories: Linux
tags:
  - LPIC-2
  - Monitoring
hide_thumbnail: true
image: /assets/img/posts/2018-04-05-capacity-planning-measuring-and-troubleshooting-resource-usage/thumbnail.jpg
---

Có rất nhiều thành phần trong một hệ điều hành Linux có thể gây ảnh hưởng tới hiệu năng của hệ thống. Việc chủ động giám sát các (monitoring) components, sẽ là cách duy nhất để bảo vệ hệ thống của bạn. Bài viết này tôi sẽ đề cập tới các công cụ và tiện ích giúp bạn có thể giám sát hệ thống dễ dàng.

---
## 1. GPU Monitoring

> Khi giám sát CPU, điều quan trọng không chỉ là việc xác định CPU(s) có bị overloaded (quá tải) hay không, mà còn là nguyên nhân gây ra sự quá tải đó. Ví dụ đó là `user process` hay `system process`? Hay nếu làm việc trong môi trường máy ảo, nguyên nhân có phải là do `hypervisor`? Việc xác định được câu trả lời cho những câu hỏi tương tự đã đề cập sẽ giúp bạn khắc phục các vấn đề về system performance.

### 1.1. Basic CPU Load Information

Không dài dòng nữa, hãy cùng bắt đầu với câu lệnh **uptime**, cho chúng ta biết hệ thống đã chạy bao lâu. Quan trọng hơn nữa, câu lệnh này cho chúng ta một tấm ảnh chụp nhanh về số lượng người dùng trên hệ thống, và tải trung bình (load average) vào thời điểm 1, 5 và 15 phút gần nhất.

```terminal
[root@centos ~]# uptime
 15:09:39 up 31 days,  3:46,  2 user,  load average: 0.00, 0.01, 0.05
```

Khái niệm `load average` dễ gây nhầm lẫn cho người quản trị :confused: , giá trị này được thiết kế để mô tả CPU load.
- Với một hệ thống single CPU (đơn CPU). Ví dụ:
  - $$ \text{load average} = 0.50 \Rightarrow $$ *50% CPU được sử dụng trong khoảng. thời gian đó.*
  - $$ \text{load average} = 1.50 \Rightarrow $$ *CPU bị overtasked, các process requests bị kẹt ở queue vì CPU đang bận handling process khác.*
- Với một hệ thống từ 2 CPUs (đa CPUs). Ví dụ:
  - $$ \text{load average} = 0.50 \Rightarrow $$ *25% CPUs được sử dụng trong khoảng thời gian đó.*
  - $$ \text{load average} = 1.50 \Rightarrow $$ *75% CPUs được sử dụng trong khoảng thời gian đó.*

Tóm lại chúng ta có công thức sau:

$$ \text{load average} = \frac {\text{load value}} {\text{number of CPUs}} * 100 \text{ (%)} $$

### 1.2. Detailed CPU Load Information

Một công cụ hữu ích khác để monitoring CPU là **iostat**. Công cụ này còn được sử dụng để monitoring disk I/O.

Khi bạn sử dụng câu lệnh **iostat** với -c option, sẽ cho ta kết quả thống kê liên quan tới việc sử dụng CPU từ thời điểm system booted :rocket:

```terminal
[root@centos ~]# iostat -c
Linux 3.10.0-327.22.2.el7.x86_64 (centos)  2018年04月05日     _x86_64_    (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.16    0.00    0.23    0.00    0.00   99.62
```

Phần đầu tiên trong output cung cấp thông tin tóm lược về hệ thống bao gồm:
```
Kernel verion: Linux 3.10.0-327.22.2.el7.x86_64
Hostname: centos
Date of report: 2018年04月05日
Kernel type: _x86_64_
Number of CPUs: 2 CPU
```

Phần thứ hai, là report về các số liệu thống kê liên quan tới CPUs. Các giá trị được cung cấp gồm có:

- **%user —** Giá trị này biểu diễn phần trăm CPU được sử dụng khi ứng dụng được chạy ở mức user-level (các processes chạy bởi tài khoản user bình thường)
- **%nice —** Các câu lệnh thường xuyên được người dùng chạy bằng **nice** command để thay đổi ưu tiên proccess CPU
- **%system —** Giá trị này biểu diễn phần trăm CPU được sử dụng bởi các processes của kernel.
- **%iowait —** Giá trị này biểu diễn phần trăm CPU được sử dụng khi CPU đang chờ hoạt động disk I/O hoàn thành trước khi chuyển qua hành động kế tiếp.
- **%steal —** Giá trị này chỉ gắn liền với virtual CPUs. Trong một vài trường hợp, virtual CPU phải chờ `hypervisor` xử lý các requests từ virtual CPUs khác. Giá trị này chỉ ra phần trăm thời gian chờ `hypervisor` xử lý virtual CPU's request.
- **%idle —** Giá trị này biểu diễn phần trăm thời gian CPU không xử lý bất kỳ request nào ? Ngồi chơi :satisfied:

※ Output của câu lệnh **iostat** với giá trị **iowait** cao:
```terminal
[root@centos ~]# iostat -c
Linux 3.10.0-327.22.2.el7.x86_64 (centos)  2018年04月05日     _x86_64_    (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.84    0.00    1.18   45.33    0.00   51.65
```
Ở ví dụ này, output của **%iowait** có giá trị cao hơn mức thông thường. Nếu xảy ra các vấn đề làm cho tiến trình xử lý, service phản hồi chậm, thì đây cũng là một giá trị đáng xem xét

**iostat** có 2 options thường rất hay được dùng là:
- `iterval`: Thời gian chờ (đơn vị: giây (s)) giữa những lần chạy lệnh **iostat**.
- `count`: Số lần cần report.

※ Output của câu lệnh **iostat** được repeat 3 lần:
```terminal
[root@centos ~]# iostat -c 1 3
Linux 3.10.0-327.22.2.el7.x86_64 (centos)  2018年04月05日     _x86_64_    (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.16    0.00    0.23    0.00    0.00   99.62

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.50    0.00    0.00    0.00    0.00   99.50

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.50    0.00    0.00   99.50
```

Một câu lệnh khác cũng cung cấp các thống kê tương tự **iostat** là **sar**. Tuy nhiên câu lệnh **sar** hiển thị các thông tin theo khoảng thời gian 10 phút một.

```terminal
[root@centos ~]# sar | head
Linux 3.10.0-327.22.2.el7.x86_64 (centos)  2018年04月05日     _x86_64_    (2 CPU)

00時00分01秒     CPU     %user     %nice   %system   %iowait    %steal     %idle
00時10分01秒     all      0.04      0.00      0.02      0.00      0.00     99.94
00時20分01秒     all      0.03      0.00      0.02      0.00      0.00     99.95
00時30分01秒     all      0.03      0.00      0.02      0.00      0.00     99.95
00時40分01秒     all      0.03      0.00      0.02      0.00      0.00     99.95
00時50分01秒     all      0.03      0.00      0.02      0.00      0.00     99.95
01時00分01秒     all      0.03      0.00      0.02      0.00      0.00     99.95
01時10分02秒     all      0.03      0.00      0.02      0.00      0.00     99.94
```

Nếu bạn muốn có thông tin thống kê theo từng CPU thay vì giá trị trung bình của tất cả CPUs, chúng ta có thể dùng lệnh **mpstat**:

```terminal
[root@centos ~]# mpstat -P ALL
Linux 3.10.0-327.22.2.el7.x86_64 (centos)  2018年04月05日     _x86_64_    (2 CPU)

19時27分07秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
19時27分07秒  all    0.15    0.00    0.22    0.00    0.00    0.00    0.00    0.00    0.00   99.62
19時27分07秒    0    0.13    0.00    0.15    0.00    0.00    0.00    0.00    0.00    0.00   99.72
19時27分07秒    1    0.18    0.00    0.30    0.00    0.00    0.00    0.00    0.00    0.00   99.52
```

---
## 2. Memory Monitoring

Một hệ thống được trang bị CPU tốc độ cao vẫn có thể trở nên trì trệ nếu như gặp vấn đề về bộ nhớ. Cần lưu ý một điều: khi bạn giám sát lượng bộ nhớ sử dụng, bạn cần nhìn vào cả RAM và Swap space.

### 2.1. Basic Memory Usage Information

Câu lệnh đầu tiên mà tôi nghĩ tới nghi cần cung cấp thông tin về virtual memory (RAM + swap space) là **free**:

Ví dụ output của **free** command
```terminal
[root@centos ~]# free
              total        used        free      shared  buff/cache   available
Mem:        4039816      219076     1940072       57968     1880668     3472192
Swap:             0           0           0
```
- `Mem`: Dòng này mô tả về RAM
- `Swap`: Dòng này mô tả về Virtual Memory

Các cột:
- **total —** Giá trị này biểu diễn tổng dung lượng bộ nhớ của hệ thống.
- **used —** Dung lượng bộ nhớ hiện đang được sử dụng
- **free —** Dung lượng bộ nhớ còn trống.
- **shared —** Giá trị này biểu diễn dung lượng bộ nhớ được sử dụng bởi `tmpfs`. `tmpfs` là một `filesystem` thường xuyên hiện hữu trên hard disk, nhưng thực sự lưu trữ dữ liệu trong memory.
- **buff/cache —** Buffer hoặc Cache là các vị trí lưu trữ tạm thời.
- **available —** Giá trị này biểu diễn bao nhiêu dung lượng bộ nhớ trống cho một tiến trình mới (new processes).

Mặc định, các giá trị hiển thị với đơn vị là **kilobytes**. Tuy nhiên bạn có thể hiển thị chính xác giá trị hơn với option **-b**/**--bytes**, hoặc hiển thị theo megabytes **-m**/**--mega**, hoặc gigabytes **-g**/**--giga**. Ngoài ra còn có option **-h**/**--human** hiển thị bất cứ giá trị nào thích hợp cho "human readable sizes"

```terminal
[root@centos ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3945         213        1894          56        1836        3390
Swap:             0           0           0
[root@centos ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.9G        213M        1.9G         56M        1.8G        3.3G
Swap:            0B          0B          0B
```

### 2.1. Detailed Memory Usage Information

Khi cần thông tin chi tiết hơn output mà lệnh **free** cung cấp, hãy nhớ tới câu lệnh **vmstat**

```terminal
[root@centos ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 1939056    712 1879776    0    0     1     1    4    5  0  0 100  0  0           0B          0B          0B

[root@centos ~]# vmstat -S M
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 3  0      0   1894      0   1835    0    0     1     1    4    5  0  0 100  0  0
```

- **Procs —**
  - r: Số process đang chạy hoặc chờ chạy.
  - b: Số process trong trạng thái uninterruptible sleep.

- **Memory —**
  - swpd: Dung lượng virtual memory đã sử dụng.
  - free: Dung lượng memory trống.
  - buff: Dung lượng memory được sử dụng làm buffer.
  - cache: Dung lượng memory được sử dụng làm cache.
  - inact: Dung lượng memory inactive (-a option).
  - active: Dung lượng memory active (-a option).

- **Swap —**
  - si: Số lượng memory đổi chỗ từ disk (/ s).
  - so: Số lượng memory đã được hoán đổi sang disk (/ s).

- **IO —**
  - bi: Block nhận từ block device (blocks/s).
  - bo: Block gửi  block device (blocks/s).

- **System —**
  - in: Số interupt trên giây. Bao gồm cả clock interrupt.
  - cs: Số lượng context switch trên giây.

- **CPU —** Tỷ lệ phần trăm của tổng thời gian CPU.
  - us: Thời gian sử dụng cho việc chạy ngoài Kernel-code (bao gồm cả **user** time và **nice** time).
  - sy: Thời gian sử dụng để chạy Kernel-code.
  - id: Idle time. Phiên bản Linux 2.5.41 trở về trước bao gồm cả thời gian chời IO.
  - wa: Thời gian chờ IO. Phiên bản Linux 2.5.41 trở về trước hiển thị 0.
  - st: Thời gian bị lấy từ một máy ảo. Phiên bản Linux 2.6.11 trở về trước, không được xác định.

---
## 3. Disk I/O Monitoring

Như đã mô tả ở [#1.2](#12-detailed-cpu-load-information), để monitoring disk I/O ta có thể sử dụng command **iostat**, với **-d** option.

Output của câu lệnh **iostat -d**

```terminal
[root@centos ~]# iostat -d
Linux 3.10.0-327.22.2.el7.x86_64 (centos)  2018年04月06日     _x86_64_    (2 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.09         1.15         1.11    3197228    3103049
dm-0              0.04         0.92         0.34    2563425     940439
dm-1              0.00         0.06         0.03     179351      92321
```

- **sda —** 1st SATA drive.
- **dm-0 —** 1st LVM logical volume.
- **dm-1 —** 2nd LVM logical volume.

Ngoài ra để dễ hình dung về hệ thống các thiết bị đĩa vật lý (physical disk device) ta sử dụng câu lệnh **lsblk**lsblk (list block device):

```terminal
[root@centos ~]# lsblk
NAME                                                                                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0                                                                                        2:0    1    4K  0 disk
sda                                                                                        8:0    0   15G  0 disk
└─sda1                                                                                     8:1    0   15G  0 part /
sr0                                                                                       11:0    1 1024M  0 rom
loop0                                                                                      7:0    0  100G  0 loop
└─docker-8:1-18189904-pool                                                               253:0    0  100G  0 dm
  └─docker-8:1-18189904-8a772e0b6b668d6821501582485f3baaf49d6dc44d52f9e38d1d507f4aca27f9 253:1    0   10G  0 dm   /var/lib/docker/devicemapper/mnt/8a772e0b6b668d6821501582485f3baaf49d6dc44d52f9e38d1d507f4aca27f9
loop1                                                                                      7:1    0    2G  0 loop
└─docker-8:1-18189904-pool                                                               253:0    0  100G  0 dm
  └─docker-8:1-18189904-8a772e0b6b668d6821501582485f3baaf49d6dc44d52f9e38d1d507f4aca27f9 253:1    0   10G  0 dm   /var/lib/docker/devicemapper/mnt/8a772e0b6b668d6821501582485f3baaf49d6dc44d52f9e38d1d507f4aca27f9
```

Có thể bạn đã biết về các devices như `sda`, `sr0`. Tuy nhiên, tên device `LVM` có thể cần hơi khéo léo hơn chút. Command như **lsblk** và **mount** hiển thị tên `device mapper` của logical volume trong khi **iostat** hiển thị tên `real device`.

Trong output của command **lsblk** bạn có thể thấy `docker-8:1-18189904-pool` là một `lvm` logical volume, tương ứng với `dm-0` hoặc `dm-1`.

Tất cả các device mapper names được lưu trữ tại thư mục **/dev/mapper**.

```terminal
[root@centos ~]# ls -l /dev/mapper/
合計 0
crw------- 1 root root 10, 236  3月  5 11:23 control
lrwxrwxrwx 1 root root       7  3月  7 16:50 docker-8:1-18189904-8a772e0b6b668d6821501582485f3baaf49d6dc44d52f9e38d1d507f4aca27f9 -> ../dm-1
lrwxrwxrwx 1 root root       7  3月  5 11:23 docker-8:1-18189904-pool -> ../dm-0
```

Nếu bạn còn chưa hiểu các mô tả trong output của command **iostat** thì hãy xem giải thích dưới đây nhé :parasol_on_ground:

- **tps —** Giá trị transfer/s cho mỗi device.
- **kB_read/s —** Số kilobytes/s đọc từ device.
- **kB_wrtn/s  —** Số kilobytes/s ghi vào device.
- **kB_read —** Số kilobytes đọc từ device.
- **kB_wrtn —** Số kilobytes ghi vào device.

Ngoài việc hiển thị lịch sử thông tin liên quan tới CPU, câu lệnh **sar** được nhắc tới ở phần trước, cũng hiển thị cả thông tin giống với **iostat** :point_down:

```terminal
[root@centos ~]# sar -d | head
Linux 3.10.0-327.22.2.el7.x86_64 (centos)  2018年04月06日     _x86_64_    (2 CPU)

00時00分01秒       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
00時10分01秒    dev8-0      0.05      0.00      0.45      9.10      0.00      2.27      1.23      0.01
00時10分01秒  dev253-0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00時10分01秒  dev253-1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00時20分01秒    dev8-0      0.05      0.00      0.37      7.47      0.00      1.43      0.80      0.00
00時20分01秒  dev253-0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00時20分01秒  dev253-1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00時30分01秒    dev8-0      0.03      0.00      0.31      9.30      0.00      0.70      0.45      0.00
```

### Listing Open Files

**lsof** command cũng cung cấp các thông tin chi tiết về disk I/O, nhưng nếu chạy không chỉ định tham số hoặc options, sẽ rất khó để người quản trị nắm bắt được. Ví dụ hãy xem kết quả của 2 câu lệnh bên dưới:

```terminal
[root@centos ~]# lsof | head -2
COMMAND     PID   TID           USER   FD      TYPE             DEVICE  SIZE/OFF       NODE NAME
systemd       1                 root  cwd       DIR                8,1      4096        128 /

[root@centos ~]# lsof | wc -l
5463
```

Câu lệnh đầu tiên hiển thị 2 dòng, bao gồm header và thông tin mô tả một file được mở, với mỗi dòng được thêm vào sau đó tương ứng với một file khác được mở.

Ở câu lệnh thứ hai, bạn có thể thấy một con số chính là tổng số dòng output cho câu lệnh **lsof**. Có nghĩa là hiện tại có 5,463 file đang mở. Và dĩ nhiên ko một ai muốn đọc hết thông tin này :joy:

Rất may, chúng ta còn một cứu cánh với câu lệnh **lsof** và **-u** option:

```terminal
[root@centos ~]# lsof -u vuongnv
COMMAND   PID    USER   FD   TYPE             DEVICE  SIZE/OFF     NODE NAME
sshd    37157 vuongnv  cwd    DIR                8,1      4096      128 /
sshd    37157 vuongnv  rtd    DIR                8,1      4096      128 /
sshd    37157 vuongnv  txt    REG                8,1    819520 17716356 /usr/sbin/sshd
sshd    37157 vuongnv  DEL    REG                0,4            9790810 /dev/zero
sshd    37157 vuongnv  mem    REG                8,1     37328 18790695 /usr/lib64/libnss_sss.so.2
sshd    37157 vuongnv  mem    REG                8,1     15472 50586020 /usr/lib64/security/pam_lastlog.so
...
bash    37249 vuongnv  mem    REG                8,1     26254    89132 /usr/lib64/gconv/gconv-modules.cache
bash    37249 vuongnv    0u   CHR              136,0       0t0        3 /dev/pts/0
bash    37249 vuongnv    1u   CHR              136,0       0t0        3 /dev/pts/0
bash    37249 vuongnv    2u   CHR              136,0       0t0        3 /dev/pts/0
bash    37249 vuongnv  255u   CHR              136,0       0t0        3 /dev/pts/0
```

- **COMMAND —** Câu lệnh mở file.
- **PID —** Process ID của câu lệnh mở file.
- **USER —** Người dùng chạy câu lệnh.
- **FD —** Các thông tin về việc file được sử dụng như thế nào, ...
- **TYPE —** Kiểu của node (file) được mở. Ví dụ `DIR` là directory, `REG` là regular node, `CHR` là character special node, `IPv4` là một IPv4 socket node, ...
- **DEVICE —** Đây là giá trị device number. Giá trị này liên quan tới device vật lý, ví dụ SATA hard drive hoặc pseudo drive. Nếu bạn muốn xác định rõ device hãy sử dụng câu lệnh sau:
```terminal
[root@centos ~]# ls -l /dev | grep "8, *1"
brw-rw---- 1 root disk      8,   1  3月  5 11:23 sda1
```
Kí tự `*` là bắt buộc phải có, do có spaces trong device name.
- **SIZE/OFF —** File size hoặc File offset. Đơn vị byte.
- **NODE —** Giá trị của node phụ thuộc vào kiểu của node.
- **NAME —** Đường dẫn tuyệt đối tới node.

Dưới đây là một số tuyệt kỹ khi dùng **lsof** :dagger:

:owl: Hiển thị các nodes của các processes đang chạy trên cổng network được chỉ định. Ví dụ với cổng SSH: **lsof -i TCP:22**

:octopus: Liệt kê các nodes của processes liên quan tới IPv4: **lsof -i 4**

:turtle: Liệt kê các command mở kết nối mạng: **lsof -i**
```terminal
[root@centos ~]# lsof -i
COMMAND     PID    USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
chronyd     651  chrony    1u  IPv4    20615      0t0  UDP localhost:323
chronyd     651  chrony    2u  IPv6    20616      0t0  UDP localhost:323
dhclient    720    root    6u  IPv4    22181      0t0  UDP *:bootpc
dhclient    720    root   20u  IPv4    22160      0t0  UDP *:23109
dhclient    720    root   21u  IPv6    22161      0t0  UDP *:8635
dhclient    722    root    6u  IPv4    21023      0t0  UDP *:bootpc
dhclient    722    root   20u  IPv4    21013      0t0  UDP *:55470
...
```

:lizard: Loại trừ các nodes thuộc về user đã chỉ định: **lsof -u ^root**
```terminal
[root@centos ~]# lsof -p 100
COMMAND  PID USER   FD      TYPE DEVICE SIZE/OFF NODE NAME
rcuos/26 100 root  cwd       DIR    8,1     4096  128 /
rcuos/26 100 root  rtd       DIR    8,1     4096  128 /
rcuos/26 100 root  txt   unknown                      /proc/100/exe
```

:scorpion: Liệt kê dự trên Process ID, ví dụ PID 100: **lsof -p 100**
```terminal
[root@centos ~]# lsof /usr/bin/bash
COMMAND   PID    USER  FD   TYPE DEVICE SIZE/OFF  NODE NAME
bash    21814 vuongnv txt    REG    8,1   960376 89151 /usr/bin/bash
bash    21857    root txt    REG    8,1   960376 89151 /usr/bin/bash
```

:blowfish: Hiển thị output liên quan tới một file nhất định. Ví dụ `/usr/bin/bash`: **lsof /usr/bin/bash**
```terminal
[root@centos ~]# lsof /usr/bin/bash
COMMAND   PID    USER  FD   TYPE DEVICE SIZE/OFF  NODE NAME
bash    21814 vuongnv txt    REG    8,1   960376 89151 /usr/bin/bash
bash    21857    root txt    REG    8,1   960376 89151 /usr/bin/bash
```

:boar: Hiển thị output của tất cả các file trong một thư mục. Ví dụ `/usr/bin`: **lsof +d /usr/bin**
```terminal
[root@centos ~]# lsof +d /usr/bin
COMMAND     PID           USER  FD   TYPE DEVICE SIZE/OFF    NODE NAME
lsmd        625 libstoragemgmt txt    REG    8,1    24016 2046072 /usr/bin/lsmd
dbus-daem   629           dbus txt    REG    8,1   441176  514706 /usr/bin/dbus-daemon
abrt-watc   632           root txt    REG    8,1    15336 2025405 /usr/bin/abrt-watch-log
vmtoolsd    639           root txt    REG    8,1    44792  553313 /usr/bin/vmtoolsd
dockerd     790           root txt    REG    8,1 33368760 1403499 /usr/bin/dockerd
tuned       794           root txt    REG    8,1     7136  278959 /usr/bin/python2.7
docker-co   990           root txt    REG    8,1  7115584 1403493 /usr/bin/docker-containerd
...
```

---
## 4. Network I/O Monitoring

Như đã trình bày ở phần trước, câu lệnh **lsof** cũng rất hữu dụng khi bạn muốn xác định những gì đang xảy ra với network. Và để hiển thị chi tiết thông tin về network I/O hơn ta lại dùng câu lệnh **netstat** :joy:

```terminal
[root@centos ~]# netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         gateway         0.0.0.0         UG        0 0          0 eno16777984
10.21.0.0       0.0.0.0         255.255.248.0   U         0 0          0 eno16777984
172.16.0.0      192.168.104.254 255.252.0.0     UG        0 0          0 eno33557248
172.17.0.0      192.168.104.254 255.255.255.0   UG        0 0          0 eno33557248
172.20.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
172.21.0.0      0.0.0.0         255.255.0.0     U         0 0          0 br-130d53424141
...
```

Thực sự mà nói đây không phải là một phương pháp giám sát mạng mà chỉ cung cấp cho chúng ta các thông tin hữu ích về route. Bằng cách thêm **-s** option, chúng ta có đầu ra hiển thị tóm tắt thống kê network dựa trên protocol:
```terminal
[root@centos ~]# netstat -s
Ip:
    1621763 total packets received
    1398076 forwarded
    0 incoming packets discarded
    158829 incoming packets delivered
    1599558 requests sent out
    24 dropped because of missing route
Icmp:
    0 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
    58635 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 58635
IcmpMsg:
        OutType3: 58635
Tcp:
    36061 active connections openings
    403 passive connection openings
    35528 failed connection attempts
    26 connection resets received
    1 connections established
    187665 segments received
    194697 segments send out
    537 segments retransmited
    0 bad segments received.
    35887 resets sent
...
```

Dưới đây là một số tuyệt kỹ khi dùng **netstat** :dagger:
- **-l —** Hiển thị network sockets đang trong trạng thái lắng nghe.
- **-lt —** Hiển thị TCP sockets đang trong trạng thái lắng nghe.
- **-lu —** Hiển thị UDP sockets đang trong trạng thái lắng nghe.
- **-p —** Hiển thị program name và PID trong output.
- **-n —** Tăng tốc câu lệnh **netstat**, giảm trễ khi DNS chưa respond.
- **-c —** Update output realtime.

---
## 5. Additional Monitoring Tools

### Listing processes

Sử dụng câu lệnh **ps** với option **-fe** để hiện thị thông tin chi tiết của các processes:

```terminal
[root@centos ~]# ps -fe
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0  3月05 ?      00:00:42 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2     0  0  3月05 ?      00:00:00 [kthreadd]
root         3     2  0  3月05 ?      00:00:02 [ksoftirqd/0]
root         7     2  0  3月05 ?      00:00:00 [migration/0]
root         8     2  0  3月05 ?      00:00:00 [rcu_bh]
root         9     2  0  3月05 ?      00:00:00 [rcuob/0]
...
root       137     2  0  3月05 ?      00:00:00 [rcuos/63]
root       138     2  0  3月05 ?      00:00:10 [watchdog/0]
root       139     2  0  3月05 ?      00:00:07 [watchdog/1]
root       140     2  0  3月05 ?      00:00:00 [migration/1]
root       141     2  0  3月05 ?      00:00:01 [ksoftirqd/1]
root       144     2  0  3月05 ?      00:00:00 [khelper]
root       145     2  0  3月05 ?      00:00:00 [kdevtmpfs]
...
vuongnv  22058 22056  0 12:27 ?        00:00:00 sshd: vuongnv@pts/1
vuongnv  22059 22058  0 12:27 pts/1    00:00:00 -bash
root     22107     2  0 12:30 ?        00:00:00 [kworker/1:1]
root     22108     2  0 12:30 ?        00:00:00 [kworker/0:0]
root     22109 22059  0 12:32 pts/1    00:00:00 ps -fe
...
```

Để hiện thị mối liên hệ của processes theo cấu trúc cây, ta dùng lệnh **pstree**

```terminal
systemd─┬─NetworkManager─┬─2*[dhclient]
        │                └─2*[{NetworkManager}]
        ├─abrt-dbus───2*[{abrt-dbus}]
        ├─abrt-watch-log
        ├─abrtd
        ├─3*[agetty]
        ├─atd
        ├─auditd───{auditd}
        ├─chronyd
        ├─crond
        ├─dbus-daemon
        ├─dockerd─┬─docker-containe─┬─docker-containe─┬─redis-server───2*[{redis-server}]
        │         │                 │                 └─9*[{docker-containe}]
        │         │                 ├─docker-containe─┬─mysqld───22*[{mysqld}]
        │         │                 │                 └─9*[{docker-containe}]
        │         │                 ├─docker-containe─┬─bash─┬─apache2───7*[apache2]
        │         │                 │                 │      └─supervisord
        │         │                 │                 └─9*[{docker-containe}]
        │         │                 ├─docker-containe─┬─bash───node───9*[{node}]
        │         │                 │                 └─7*[{docker-containe}]
        │         │                 ├─docker-containe─┬─node───9*[{node}]
        │         │                 │                 └─8*[{docker-containe}]
        │         │                 ├─docker-containe─┬─java───30*[{java}]
        │         │                 │                 └─8*[{docker-containe}]
        │         │                 └─17*[{docker-containe}]
        │         ├─3*[docker-proxy───4*[{docker-proxy}]]
        │         ├─docker-proxy───5*[{docker-proxy}]
        │         └─23*[{dockerd}]
        ├─irqbalance
        ...
        └─wpa_supplicant
```

Để hiển thị các thông tin cơ bản của hệ thống như running time, average system load, số lượng process đang chạy,  CPU statistics, thông tin memory, ... Ta dùng lệnh **top**

```terminal
[root@centos ~]# top
top - 12:40:35 up 36 days,  1:17,  2 users,  load average: 0.01, 0.02, 0.05
Tasks: 322 total,   2 running, 320 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.5 us,  0.3 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  4039816 total,   270408 free,  1154400 used,  2615008 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  2507692 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
19259 kmarkov   20   0 3617044 521104  17156 S   1.0 12.9  18:56.09 java
19034 kmarkov   20   0 1319548 138388  12620 S   0.3  3.4   4:41.12 node
20888 root      20   0       0      0      0 R   0.3  0.0   0:02.95 kworker/0:2
21996 root      20   0       0      0      0 S   0.3  0.0   0:11.05 kworker/1:2
22192 root      20   0  157828   2424   1552 R   0.3  0.1   0:00.03 top
    1 root      20   0   41288   3492   2156 S   0.0  0.1   0:42.14 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.74 kthreadd
    3 root      20   0       0      0      0 S   0.0  0.0   0:02.56 ksoftirqd/0
    7 root      rt   0       0      0      0 S   0.0  0.0   0:00.49 migration/0
...
```

## 6. Summary

Việc monitoring system là không thể thiếu khi bạn triển khai hệ thống trên staging cũng như production. Bài viết này chỉ giới thiệu các câu lệnh và khái niệm cơ bản, khi deploy và vận hành hệ thống trên môi trường end-user bạn sẽ còn gặp rất nhiều vấn đề phức tạp, lúc đó ngoài việc monitoring còn yêu cầu setup thêm `Alerting tool`, ... Hơn nữa việc sử dụng các hệ thống monitoring tự động như [Zabbix](https://www.zabbix.com/) hay [Prometheus](https://prometheus.io/) sẽ tiết kiệm chi phí về nhân lực và hạ tầng hơn rất nhiều.

Nếu có thời gian, mình sẽ viết tiếp bài triển khai [Prometheus](https://prometheus.io/) + [Grafana](https://grafana.com/) + [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/) + [cAdvisor](https://github.com/google/cadvisor) + [Swarm Visualizer](https://github.com/dockersamples/docker-swarm-visualizer) trên Swarm Network.

---
## 7. References

[LPIC-2 Cert Guide: (201-400 and 202-400 exams) (Certification Guide)](https://www.amazon.com/LPIC-2-Cert-Guide-201-400-Certification/dp/0789757141)
