---
layout: post
title:  "[Docker Swarm] Services hoạt động như thế nào?"
date:   2018-03-26 11:23:04 +0700
categories: Docker
tags:
  - Swarm
  - Services
---

Tiếp tục cho chuỗi series giải thích hoạt động của Swarm mode trong Docker. Bài viết này mình xin được giới thiệu về cách thức hoạt động của một **service** trong Docker Swarm mode.

# How services work?

Để deploy một image của ứng dụng, trong khi Docker Engine đang chạy ở chế độ swarm mode, bạn có thể tạo ra một service. Thông thường, một service là một image của một microservice trong bối cảnh chúng ta có một ứng dụng lớn. Ví dụ như các services có thể chứa một HTTP server, database hoặc bất kỳ một chương trình nào mà bạn muốn chạy trên môi trường phân tán (distributed environment).

Khi bạn chạy một service, bạn cần chỉ rõ container image nào được sử dụng & câu lệnh nào cần được thực thi trong các container đang chạy. Các options có thể được định nghĩa trong serivce bao gồm:

- Cổng (port) để các service có thể truy cập và sử dụng bên ngoài swarm
- Overlay network để các service có thể kết nối tới service khác bên trong swarm
- Giới hạn CPU và memory
- Rolling update policy
- Số lượng các bản sao (replica) của image sẽ chạy trong swarm.

## Services, tasks, and containers

Khi bạn deploy các services lên swarm, *swarm manager* cho phép định nghĩa các services của bạn như là trạng thái mong muốn cho service. Sau đó chúng lập lịch cho từng service trên các nodes trong mạng swarm chẳng hạn như 1 hoặc nhiều replica tasks. Các tasks này chạy độc lập với các tasks khác trong swarm.

Ví dụ, giả sử bạn muốn có 1 load balance giữ 3 instances của một HTTP listener. Diagram (biểu đồ) sau sẽ biểu diễn một HTTP listener với 3 replicas. Mỗi một instance trong 3 listener là 1 task trong swarm.

![Services diagram]({{ '/assets/img/posts/2018-03-26-[docker-swarm]-how-services-work/services-diagram.png' | relative_url }})

Mỗi một container là một tiến trình được cô lập (isolated process). Trong mô hình swarm mode, mỗi task gắn kết với một container. Một task giống như một "slot" mà ở đó, scheduler đặt một container. Khi container hoạt động (live), scheduler nhận định rằng task đó đang có trạng thái running. Nếu container thực hiện health checks thất bại hoặc bị kết thúc (terminate), thì task sẽ bị kết thúc.

## Tasks and scheduling

Một task được coi là một atomic unit (đơn vị nguyên tử ~ chắc mang ý nghĩa ko thể phân chia được nữa :expressionless:) của scheduling trong một swarm. Khi bạn khai báo một trạng thái service mong muốn bằng cách tạo ra hoặc cập nhật một service, trình điều khiển (orchestrator) sẽ nhận ra trạng thái mong muốn bằng cách lập lịch trình các tasks.

Ví dụ, bạn định nghĩa một service mà chỉ thị cho các orchestrator giữ 3 instances của một HTTP listener chạy toàn thời gian. Orchestrator phản hồi bằng cách tạo ra 3 tasks. Mỗi task là một "slot" mà trình lên lịch fill đầy bằng cách spawn (đẻ :joy:) ra một container. Container là sự khởi tạo của task. Nếu một HTTP listener task bị fail health check hoặc crashes, orchestrator sẽ tạo ra một tác vụ bản sao mới (new replica task) tạo ra một container mới.

Task có cơ chế một chiều (one-directional mechanism). Nó tiến hành đơn điệu thông qua một loạt các trạng thái: được phân công (assigned), chuẩn bị (prepared), chạy (running), vv.. Nếu task không thành công, orchestrator sẽ loại bỏ task và container của nó, sau đó tạo ra một task mới để thay thế nó theo trạng thái mong muốn được chỉ định bởi service.

Logic cơ bản của Docker swarm mode là mục đích của scheduler & orchestrator. Bản thân các service và task abstractions không nhận thức được về các container chúng implement. Theo giả thuyết, bạn có thể implement các loại tasks như: tác vụ máy ảo (virtual machine tasks) hoặc tác vụ không chứa container (non-containerized process tasks). Scheduler và orchestrator không thể biết các loại tác vụ này. Tuy nhiên, phiên bản hiện tại của Docker chỉ hỗ trợ các container tasks :sunglasses:.

Diagram dưới đây cho thấy swarm mode chấp nhận service tạo yêu cầu và lên lịch các task cho các nodes làm việc.

![Services flow]({{ '/assets/img/posts/2018-03-26-[docker-swarm]-how-services-work/service-lifecycle.png' | relative_url }})

### Pending services

Một service có thể được cấu hình theo cách mà không có node hiện tại nào trong swarm có thể chạy các tác vụ (task) của nó. Trong trường hợp này, dịch vụ vẫn đang trong trạng thái đang chờ xử lý (`pending`). Dưới đây là một vài ví dụ về thời điểm dịch vụ có thể vẫn ở trong trạng thái `pending`.

> **Lưu ý:** Nếu mục đích của bạn chỉ là ngăn không cho một service được deploy thì hãy scale service về 0 thay vì cố gắng câis hình service theo cách mà nó thành trạng thái `pending`.

- Nếu tất cả các node bị `paused` hoặc `drained`, và bạn tạo một service, nó sẽ được chờ cho đến khi một node trở thành `available`. Trong thực tế, node đầu tiên luôn sẵn sàng (`available`) cho tất cả các task, do đó, đây không phải là một điều tốt để làm trên môi trường production.

- Bạn có thể dự trữ một số lượng bộ nhớ cụ thể cho một service. Nếu không có node nào trong swarm có số lượng bộ nhớ yêu cầu, dịch vụ vẫn ở trong trạng thái `pending` cho đến khi một node có thể chạy các tác vụ của nó. Nếu bạn chỉ định một giá trị rất lớn, chẳng hạn như 500 GB, task sẽ vẫn ở trạng thái chờ đợi mãi mãi :confused:, trừ khi bạn thực sự có một node có thể đáp ứng nó.

- Bạn có thể áp đặt các ràng buộc vị trí trên service và những ràng buộc này có thể không đạt được tại một thời điểm nhất định.

Hành vi này chỉ ra rằng các yêu cầu và cấu hình task của bạn không chặt chẽ với trạng thái hiện tại của swarm. Là quản trị viên của một swarm, bạn định nghĩa tình trạng mong muốn của swarm, và manager node + worker node sẽ làm việc cùng nhau để tạo ra state đó. Bạn không cần phải quản lý vi mô (micro-manage) các task trên swarm.

## Replicated and global services

Có 2 loại service khi deploy, đó là: **replicated** và **global**.

- Với một **replicated** service, bạn chỉ định một số lượng task mong muốn chạy. Ví dụ, bạn quyết định deploy một HTTP service với 3 replicas, mỗi replica đều serving chung một nội dung.

- Trong khi đó **global** service là một service chạy một task trên các node. Không có số lượng task được chỉ định. Mỗi khi bạn thêm một node vào swarm, orchestrator tạo ra một task và trình lập lịch (scheduler) sẽ phân công (assigns) task này cho node mới. Ứng cử viên cho các **global** service thường là monitoring agents, anti-virus scanners, hoặc các loại container mà bạn muốn chạy trên toàn bộ các node trong swarm.

Diagram dưới đây biểu diễn bộ *3-service replica* (màu vàng) và *global service* (màu xám).

![Global vs replicated services]({{ '/assets/img/posts/2018-03-26-[docker-swarm]-how-services-work/service-lifecycle.png' | relative_url }})

## References

- [How services work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)






