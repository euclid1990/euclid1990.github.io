---
layout: post
title:  "Tuỳ biến phím tắt trên Slack và Mac với Karabiner"
date:   2021-12-14 00:49:04 +0700
categories: Tips
tags:
  - MacOS
  - Slack
  - Chatwork
  - Karabiner
hide_thumbnail: true
image: /assets/img/posts/2021-12-14-custom-slack-keyboard-shortcut-with-karabiner-on-mac/thumbnail.jpeg
---

Mình hiện tại đang dùng song song 2 công cụ chat khi làm việc đó là **[Chatwork](https://www.chatwork.com)** và **[Slack](https://slack.com)**. Tuy nhiên khi sử dụng 2 công cụ này mình gặp phải một vấn đề đó là:

- Phím tắt của 2 công cụ chat này không giống nhau
  - Ví dụ: Trên Chatwork để gửi tin nhắn bạn có thể dùng `shift + enter` trong khi trên Slack sử dụng `command + enter`

Việc này có thể không vấn đề với nhiều người dùng nhưng với mình nó lại là cả một vấn đề, việc học và thay đổi một thói quen là một việc khá tốn thời gian và không cần thiết.

Ban đầu mình có hỏi thăm một người bạn, và nhận được lời khuyên sử dụng: [Keyboard Maestro](https://www.keyboardmaestro.com), tuy nhiên phần mềm này có giá ~36$, do đó mình quyết định thử tìm hiểu xem có cách nào mapping lại phím mà đỡ tốn kém hơn không. Kết quả là việc này có thể hoàn thành rất dễ dàng với công cụ có tên là: **[Karabiner Elements](https://karabiner-elements.pqrs.org)**.

Để cài đặt **Karabiner Elements** các bạn truy cập trang web [https://karabiner-elements.pqrs.org](https://karabiner-elements.pqrs.org) và tải về cũng như cài đặt một cách hoàn toàn miễn phí.

![](/assets/img/posts/2021-12-14-custom-slack-keyboard-shortcut-with-karabiner-on-mac/karabiner-simple-modifications.png){: .align-center}

Sau khi cài đặt xong, chúng ta sẽ thấy biểu tượng của phần mềm trên thanh trạng thái hệ thống, và có thể tuỳ biến phím đơn trên bàn phím với giao diện `Simple modifications` sẵn có. Ngoài ra bạn có thể map lại các phím chức năng tiêu chuẩn `F1`, `F2`, `F3`, `F4`, `F5`, `F6`, `F7`, `F8`, `F9`, `F10`, `F11`, `F12` trên bàn phím ngoài sang các phím chức năng của Apple MacOS như:

- `F1` – Giảm độ sáng màn hình
- `F2` – Tăng độ sáng màn hình
- `F3` – Điều khiển nhiệm vụ mở
- `F4` – Mở Launchpad
- `F5` – Giảm độ sáng bàn phím (Chỉ trên máy tính xách tay tương thích)
- `F6` – Tăng độ sáng bàn phím (Chỉ trên máy tính xách tay tương thích)
- `F7` – Bỏ qua lại (Âm thanh)
- `F8` – Tạm dừng / Phát (Âm thanh)
- `F9` – Bỏ qua phía trước (Âm thanh)
- `F10` – Tắt tiếng
- `F11` – Giảm âm lượng
- `F12` – Tăng âm lượng

![](/assets/img/posts/2021-12-14-custom-slack-keyboard-shortcut-with-karabiner-on-mac/karabiner-function-keys.png){: .align-center}

bằng cách sử dụng `Function keys`.

### Tuỳ biến tổ hợp phím

Để tuỳ biến lại tổ hợp phím tắt một cách phức tạp hơn bạn cần sửa config ở file sau

```bash
$ vim ~/.config/karabiner/karabiner.json
```

Bạn tìm tới mục `"rules":`, thêm object sau vào

```json
{
    "description": "shift+enter to command+enter / custom Slack send messages",
    "manipulators": [
        {
            "from": {
                "key_code": "return_or_enter",
                "modifiers": {
                    "mandatory": [
                        "right_shift"
                    ]
                }
            },
            "to": [
                {
                    "key_code": "return_or_enter",
                    "modifiers": [
                        "right_command"
                    ]
                }
            ],
            "type": "basic",
            "conditions": [
                {
                  "type": "frontmost_application_if",
                  "bundle_identifiers": [
                    "^com\\.tinyspeck\\.slackmacgap$"
                  ]
                }
              ]
        }
    ]
}
```

Ở trên mình tiến hành tuỳ biến shortcut key cho việc gửi tin nhắn trên **Slack** sang giống như trên **Chatwork** (Cá nhân mình thích gõ `shift + enter` hơn), ngoài ra bạn có thể tham khảo thêm một số rules mẫu tại [đây](https://github.com/pqrs-org/KE-complex_modifications/tree/main/public/json) hoặc một số do mình chia sẻ tại [đây](https://gist.github.com/euclid1990/ee6ed30164c8936f0cecba0194b1b8b4)

Chia sẻ thêm là trong đoạn tuỳ biến trên có phần `"bundle_identifiers"` để giới hạn phạm vi application sẽ kích hoạt việc tuỳ biến, và `bundle_identifiers` được mình lấy theo cách sau:

- Tìm link app (Qua Google hoặc App store).
  - Ví dụ app **Slack**: [https://apps.apple.com/app/slack/id803453959](https://apps.apple.com/app/slack/id803453959)
- Copy number ID từ URL
  - `803453959`
- Mở đường dẫn [https://itunes.apple.com/lookup?id=xxx](https://itunes.apple.com/lookup?id=xxx), thay xxx thành number ID bạn có
  - [https://itunes.apple.com/lookup?id=803453959](https://itunes.apple.com/lookup?id=803453959).
- Truy cập đường dẫn trên trình duyệt sẽ tải về một file txt, Tìm kiếm từ khoá `"bundleId"` và bạn sẽ có `bundle_identifiers`
  - `com.tinyspeck.slackmacgap`

Chúc các bạn thành công 🦞
