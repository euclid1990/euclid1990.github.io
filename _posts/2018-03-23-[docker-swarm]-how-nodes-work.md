---
layout: post
title:  "[Docker Swarm] Nodes hoạt động như thế nào?"
date:   2018-03-23 13:23:04 +0700
categories: Docker
tags:
  - Swarm
  - Nodes
---

Mở đầu cho chuỗi series giải thích hoạt động của Swarm mode trong Docker. Bài viết này mình xin được giới thiệu về cách thức hoạt động của một **node** trong Docker Swarm mode.

# How nodes work?

Swarm mode lần đầu xuất hiện trọng phiên bản Docker Engine 1.12, cho phép người dùng tạo cluster của một hoặc nhiều Docker Engines (Gọi đơn giản là swarm). Một swarm có chứa một hoặc nhiều nodes: đó là các máy vật lý hay máy ảo đang chạy Docker Engine 1.12 hoặc phiên bản mới hơn ở swarm mode.

Có 2 loại nodes: **managers** và **workers**.

![Swarm mode cluster]({{ '/assets/img/posts/2018-03-23-[docker-swarm]-how-nodes-work/swarm-diagram.png' | relative_url }})

Nếu bạn vẫn chưa biết về swarm mode, hãy đọc qua [tổng quan về swarm mode](https://docs.docker.com/engine/swarm/) và [các khái niệm chính](https://docs.docker.com/engine/swarm/key-concepts/).

## Manager nodes

Manager nodes giữ vai trò quản các task của cluster như:
- Maintaining cluster state
- Scheduling services
- Serving swarm mode HTTP API endpoints

Với việc sử dụng thuật toán đồng thuận [Raft](https://raft.github.io/raft.pdf), các managers duy trì trạng thái nội bộ nhất quản của toàn bộ swarm và tất cả các service đang chạy trên đó.
Với mục đích testing thì việc chạy swarm trên một single-manager là không vấn đề. Nếu như manager bên trong single-manager swarm fail, các service của bạn vẫn tiếp tục chạy, tuy nhiên bạn phải tạo ra một new-cluster để recover(phục hồi) nó.

Để tận dụng lợi thế của tính năng swarm mode’s fault-tolerance, Docker khuyến khích người sử dụng implement một số lượng lẻ các nodes tuỳ theo mục tiêu high-availability mà người dùng mong muốn. Khi bạn có nhiều managers, bạn có thể recover các failure of a manager node mà ko bị rơi vào tình trạng downtime.

- Một nhóm `3` managers, chịu được việc mất đi tối đa `1` manager node.
- Một nhóm `5` managers, chịu được việc mất đi tối đa đồng thời `2` manager nodes.
- Một nhóm `N` managers, chịu được việc mất đi tối đa `(N-1)/2` manager nodes.
- Docker khuyến khích việc thiết lập tối đa 7 manager nodes cho 1 swarm.

> **Lưu ý**: Việc thêm nhiều managers ko có nghĩa là tăng khả năng mở rộng và nâng cao hiệu suất. Nhìn chung, điều ngược lại là đúng.

## Worker nodes

Worker nodes thực chất cũng là một instances của Docker Engine, với mục đích thực thi/chạy các containers. Worker nodes không tham dự vào quá trình phân tán Raft, đưa ra quyết định lập lịch, cũng như việc đảm nhiệm các swarm mode HTTP API.

Bạn có thể tạo 1 swarm của 1 manager node, nhưng lại không thể có 1 worker node mà thiếu vắng đi sự tồn tại của 1 manager node. Mặc định, tất cả các managers cũng có thể là workers. Trong 1 single-manager node cluster, bạn có thể chạy các câu lệnh như `docker service create` và lên lịch trình đặt tất cả task trên local Engine.

Để ngăn không cho trình lên lịch đặt task lên các manager nodes ở multi-node swarm, hãy thiết lập tính sẵn sàng(availability) của manager node là `Drain`. Khi đó trình lên lịch sẽ dừng các task trên các node bật chế độ `Drain` và sắp xếp các task trên node đặt chế độ `Active`. Trình lên lịch không chỉ định tác vụ mới cho các node có tính sẵn sàng là `Drain`.

Tham khảo câu lệnh *[docker node update](https://docs.docker.com/engine/reference/commandline/node_update/)* để biết được các thay đổi availability của một node.

## Change roles

Bạn có thể đẩy một worker node thành manager node bằng cách chạy lệnh `docker node promote`. Ví dụ: bạn có thể muốn đổi một worker node thành manager node, khi manager node đang offline để thực hiện bảo trì. Xem thêm tại [node promote](https://docs.docker.com/engine/reference/commandline/node_promote/).

Bạn cũng có thể giảm cấp manager node thành worker node. Xem thêm tại [demote node](https://docs.docker.com/engine/reference/commandline/node_demote/).

## References

- [How nodes work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/#worker-nodes)

