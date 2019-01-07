---
layout: post
title:  "Vietnam Kubernetes Community - First meetup"
date:   2019-01-07 07:23:04 +0700
categories: Meetup
tags:
  - Kubernetes
hide_thumbnail: true
image: /assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/thumbnail.jpeg
---

Trong những năm gần đây meetup giữa những người làm công nghệ ngày càng trở nên phổ biến. Rất nhiều hoạt động meetup của cộng đồng các kỹ sư thuộc các mảng công nghệ được tổ chức.

Được đối mặt và trò chuyện trực tiếp với những người có cùng đam mê và sở thích thực sự đem lại cho cá nhân mình rất nhiều lợi ích. Đó không chỉ là kiến thức chúng ta thu được từ việc lắng nghe và chia sẻ, kiểm chứng lại cái mình hiểu và đang dùng đúng hay sai, mà meetup còn giúp mỗi cá nhân mở rộng các mối quan hệ giúp ích cho careerer path của bạn rất nhiều.

Trong một năm qua cá nhân mình cũng chỉ tham dự được 4~5 cái meetup nho nhỏ, và sang năm mới 2019, nhân dịp [`Vietnam Kubernetes Group`](https://vietkubers.github.io/) tổ chức first meetup, mình có cơ hội được giao lưu và gặp gỡ những kỹ sư đang vận hành các hệ thống sử dụng `Kubernetes` đồng thời cũng là những contributor cho open source này.

![](/assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/standee.jpg)

**Kubernetes là gì ?** Nếu bạn đã biết có thể bỏ qua, còn nếu chưa biết thì mình xin được giải thích một chút:

`Kubernetes` viết tắt là K8s ( Cách viết tắt này bạn có thể gặp rất nhiều như Internationalization - I18n, Localization - L10n, Translation - T9n ... Tất cả đều là ký tự đầu, số ký tự ở giữa và ký tự cuối ), là một công cụ giúp điều phối và quản lý các container trên các cluster. Hiện tại k8s được sử dụng như một service trên các cloud provider như GKE, AKS và EKS. K8s giúp tăng tốc quá trình phát triển hệ thống, hỗ trợ auto deploy, updates (rolling-update), auto scaling up/down, quản lý hoạt động của các apps và service (zero downtime). Ngoài ra k8s còn cung cấp thêm một tính năng rất quan trọng đó là self-healing, khởi động lại các service khi process crash bên trong container.

![](/assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/kubernetes_architecture_explained.jpg)

`Kubernetes` được phát triển bởi Google, và chính thức open-source vào năm 2014. Năm 2015, K8s được release v1.0 và trở thành project đầu tiên trong [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/). Bốn năm sau khi chính thức ra mắt, K8s đã và đang có những bước tiến rất xa, khi đứng sau công cụ này không chỉ có đông đảo cộng đồng mà còn là các công ty công nghệ lớn như Google, IBM, Amazon, Microsoft, Oracle, ...

Trở lại nội dung buổi meetup của Vietnam Kubernetes group, event lần này được tổ chức tại tầng 24, Keangnam. Lịch trình như sau:

![](/assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/agenda.jpg)

Mở đầu buổi Meetup là bài giới thiệu sơ lược về group, Group chỉ mới được thành lập vào tháng 11-2018, bởi 12 kỹ sư chủ yếu hoạt động trong lĩnh vực liên quan tới OpenStack. Tuy vậy, hiện tại con số thành viên trong nhóm đã vượt quá 300 members, các topic chủ yếu thảo luận về công nghệ và các vấn đề phát sinh khi triển khai hệ thống trên K8s.

![](/assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/kubernetes_recap.jpg)

Bài thuyết trình đầu tiên tới từ speaker Cao Xuân Hoàng, hiện đang là kỹ sư tại Genesis Cloud, người có kinh nghiệm triển khai K8s trên production với con số lên tới hàng ngàn node. Tuy nhiên ở slide trong buổi meetup lần này không đi quá sâu vào các bài toán thực tế mà chủ yếu là các khái niệm liên quan tới docker/rkt container, kiến trúc K8s, các kiến thức và công cụ được coi là essential với một kỹ sử khi sử dụng K8s. Có một điều hơi đáng tiếc là phần workshop đi kèm bài thuyết trình được tác giả quảng cáo là rất hay lại không được thực hành vì lý do hạn chế về thời gian.

![](/assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/microservice_deployment_scaling_with_kubernetes.jpg)

Ở bài thuyết trình sau đó, anh Đào Công Tiến giới thiệu các khái niệm về monolithic, SOA (Service oriented architecture), microservices. Các điểm mạnh, yếu của từng kiến trúc, và hoàn cảnh sử dụng sao cho hợp lý, cách mà các microservice communicate với nhau. Đồng thời tác giả cũng chuẩn bị sẵn một demo về Microservice trên K8s, rolling-update, auto scaling up và down container theo chiều ngang trên cụm Cluster.

Sau bài thuyết trình này, tea-break cũng là lúc mọi người có thời gian riêng trao đổi về công việc và kỹ thuật với các cá nhân khác cũng tham gia meetup

![](/assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/tea_break.jpg)

Kết thúc có lẽ là bài chia sẻ nhận được nhiều câu hỏi và ý kiến nhất, tới từ 3 thành viên hiện đang là software engineer tại Fujitsu Việt Nam với tựa đề: “Deep Dive into Container Networking v1.0”. Tuy còn các thành viên còn rất trẻ nhưng thành tích mà nhóm giới thiệu không hề nhỏ, trong 2 năm nhóm đã contribute ~ 30k Lines of code vào Openstack, chủ yếu ở các dự án liên quan tới Network. Sau khi chuyển hướng nghiên cứu sang network bên trong K8s, nhóm tiến hành tổng hợp lại các kiến thức về container networking, điểm giống và khác nhau trong cách thức hoạt động của docker, k8s network. Cách mà các Pod, Service, Cluster giao tiếp với nhau trong mạng LAN, cách xây dựng Loadbalancer và Ingress network, iptables và bpf trong tương lai ...

![](/assets/img/posts/2019-01-05-vietnam-kubernetes-first-meetup/deep_dive_into_container_networking_v1.0.jpg)

## Tham gia meet up nên hay không ?

Thực tế tồn tại một điều là rất nhiều kỹ sư ngại tham gia meetup, tự thu mình lại. Nếu bạn là một software engineer thì chắc chắn sẽ hiểu một nỗi khổ là kỹ thuật thay đổi từng giờ từng ngày, chỉ cần không theo kịp 3~6 tháng thôi là rất có thể chúng đã trở thành người cổ đại trong 1 mảng nào đó. Mình lấy một ví dụ đơn giản như công nghệ container engine hiện tại là cuộc chiến giữa [rkt](https://coreos.com/rkt/) vs [docker](https://www.docker.com/), tuy nhiên chỉ mới đây thôi đã lại xuất hiện một nhân tố mới đó là: [kata container](https://katacontainers.io/) được bảo trợ bởi Intel, công nghệ mới xuất hiện từng ngày, nhu cầu cập nhật là tuỳ thuộc ở mỗi cá nhân. Khi bắt đầu là một sinh viên, bạn được học rất nhiều trên ghế nhà trường, nhưng khi đi làm bạn sẽ nhận ra những kiến thức đó là không đủ, requirement và các bài toán trên market khác xa với lý thuyết rất nhiều và cùng với đó bạn học được không ít kinh nghiệm thực tế từ đồng nghiệp, khách hàng, ... Tuy nhiên nếu chỉ dừng lại ở phạm vi trong công việc liệu có đủ ? Cá nhân mình nghĩ là chưa 😑, và việc không ngại tham gia các buổi meetup, thu nhận được rất nhiều kinh nghiệm xương máu từ những người đi trước cũng là một cách giúp bản thân mình tránh bị rơi vào bi kịch "dốt mà lại không biết mình dốt". Đây có lẽ là dốt ở mức độ cao nhất 😂
