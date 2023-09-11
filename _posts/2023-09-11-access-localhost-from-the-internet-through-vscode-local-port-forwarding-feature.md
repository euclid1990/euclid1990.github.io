---
layout: post
title: "Mở ứng dụng Local trên Internet bằng tính năng VSCode - Local Port Forwarding"
date: 2023-09-11 15:00:00 +0700
categories: Tips
tags:
  - Debug
  - Vscode
  - Port Forwarding
hide_thumbnail: true
image: /assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/thumbnail.png

---

Trong quá trình phát triển ứng dụng, web/app chắc hẳn mỗi chúng ta đều đã từng gặp bài toán cần chia sẻ ứng dụng của mình cho một bên khác thông qua internet. Ví dụ:
- Bạn muốn demo website đang code cho khách hàng truy cập từ xa
- Bạn cần trỏ mobile app > backend API endpoint tới server trên máy cá nhân cho tiện debug
- Bạn muốn test tính năng webhook từ một service bên thứ 3 tới endpoint trỏ tới server trên localhost
- ...

*Việc bạn công khai ứng dụng lên internet sẽ không được khuyến khích vì lý do bảo mật, vì vậy hãy cân nhắc nhiều cách khác nhau, trước khi thực hiện điều đó.*

Trước đây để thực hiện việc export ứng dụng ra ngoài Internet có rất nhiều công cụ hỗ trợ như **[Ngrok](https://ngrok.com/)**, **[Localtunnel](https://theboroer.github.io/localtunnel-www/)**, ...

Trong bài viết này mình xin giới thiệu một công cụ mới, hỗ trợ Developer có thể làm việc này rất dễ dàng và hoàn toàn miễn phí, được cung cấp bởi **[VSCode](https://code.visualstudio.com/)** ➡︎ **[Local Port Forwarding](https://code.visualstudio.com/docs/editor/port-forwarding)**. Tính năng **[Built-in port forwarding - Forward local server ports from within VS Code](https://code.visualstudio.com/updates/v1_82)** được xuất hiện trên **[VSCode](https://code.visualstudio.com/)** từ phiên bản **1.82.0**.

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/release_note_1.82.0.png){: .align-center}

## Các bước thực hiện

### Bước 1

Chuẩn bị ứng dụng, trong bài viết này mình sử dụng một web server chạy trên **Localhost** dùng **Port 3000**

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/access_local_address.png){: .align-center}

Tất nhiên hãy để ý, phiên bản **VSCode** mà bạn đang sử dụng phải là phiên bản **≥ 1.82.0**.

### Bước 2

Sau khi đã khởi chạy server trên máy cá nhân, bạn tiến hành mở Project bằng **VSCode**, sau đó nhấn tổ hợp phím Ctr + Shift + P (Windows/Linux) hoặc Cmd + Shift + P (MacOS), tiếp đó tìm tới **Forward a Port**.

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/command_palette.png){: .align-center}

Bấm vào **Forward A Port** button

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/forward_port.png){: .align-center}

### Bước 3

Nhập **Port** của server trên **Local**. Ở đây Bun server mình đang khởi động chạy trên cổng 3000

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/enter_local_port.png){: .align-center}

Sau đó nhấn **Enter** và chọn **Allow**

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/allow_local_tunnel_port_forwarding.png){: .align-center}

**VSCode** sẽ yêu cầu bạn tiến hành cấp quyền cho ứng dụng có thể truy cập tài khoản **Github** quyền *read-only*

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/vscode_authorize.png){: .align-center}

Chọn **Authorize Visual-Studio-Code** và đợi một chút, bên ứng dụng **VSCode** sẽ hiện Logs khởi tạo địa chỉ cho ứng dụng của bạn trên Internet.

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/port_forwarding_logs.png){: .align-center}

Chúng ta có thể tìm được địa chỉ **Domain Address** tại đây:

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/forwarded_address.png){: .align-center}

### Bước 4

Truy cập thử **Endpoint** mà **Port Forwarding** cung cấp.

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/ms_dev_tunnels_authorize.png){: .align-center}

Sau khi cấp quyền cho **Dev Tunnels** của **Microsoft**, chúng ta đã có thể truy cập ứng dụng từ Internet.

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/access_forwarded_address.png){: .align-center}

:heart_eyes:

**Mặc định** địa chỉ này sẽ Private, tức là chỉ có bạn mới có thể truy cập được, nếu muốn công khai cho người ngoài truy cập thì hãy thiết lập **Public Address** trong cột **Visibility** như hình bên dưới.

![](/assets/img/posts/2023-09-11-access-localhost-from-the-internet-through-vscode-local-port-forwarding-feature/public_adress.png){: .align-center}

## Tổng kết

Trên đây mình đã giới thiệu tới bạn đọc cách chuyển tiếp Port của ứng dụng trên Local lên Internet với **VSCode**, hy vọng sẽ giúp ích cho bạn đọc.

> Ngoài lề, forwarded ports được bảo mật thế nào ?
>
> Mặc định, cả việc hosting và connecting tới tunnel đều yêu cầu authentication từ cùng một GitHub và Microsoft account.
>
> VSCode không thiết lập bất kỳ listener nào ngoài việc kết nối ra tới services được hosted trên Azure.
>
> Tuy nhiên, một khi **Public Port** (Thay vì **Private**), bất kỳ người dùng nào cũng có thể truy cập được. Cần cẩn thận để tránh lưu trữ bất kỳ thông tin bí mật hoặc dịch vụ không an toàn nào trên các cổng đó.
>
> Nếu một tổ chức muốn kiểm soát quyền truy cập vào **Forward Port**, bạn có thể cho phép hoặc từ chối quyền truy cập vào miền `global.rel.tunnels.api.visualstudio.com`

> Tham khảo thêm các giới hạn về Port Forwarding tại **[đây](https://code.visualstudio.com/docs/remote/tunnels#_are-there-usage-limits-for-the-tunneling-service)**
