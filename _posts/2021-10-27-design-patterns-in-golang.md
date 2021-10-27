---
layout: post
title:  "Áp dụng design patterns với Go"
date:   2021-10-27 00:49:04 +0700
categories: Go
tags:
  - Design patterns
hide_thumbnail: true
image: /assets/img/posts/2021-10-27-design-patterns-in-golang/design-patterns.png
---

Design Pattern (Mẫu thiết kế) giúp chúng ta giải quyết vấn đề một cách tối ưu nhất, có thể coi nó như là kim chỉ nam giúp các lập trình viên giải quyết vấn đề thay vì tự tìm kiếm giải pháp cho một vấn đề đã được chứng minh. Design patterns không phải là ngôn ngữ cụ thể nào cả, và có thể thực hiện được ở phần lớn các ngôn ngữ lập trình. Nhưng thường gặp nó nhất trong lập trình OOP, mà đại diện chính là Java (Một ngôn ngữ OOP quốc dân).

- **Ưu điểm**
  - Code readability: Code dễ hiểu
  - Communication: Dễ dàng tiếp cận trao đổi về giải pháp cho một vấn đề bằng việc sử dụng một design pattern mẫu
  - Code reuse: Code có tính tái sử dụng cao
- **Nhược điểm**
  - More complexity: Làm phức tạp hóa quá trình code, phải mất công suy nghĩ và tìm hiểu nên dùng pattern nào.
  - Different variants: Có nhiều biến thể cho mỗi mẫu đối tượng khiến cho việc áp dụng các pattern đôi khi không đông nhất.

## Phân loại Design Patterns

Có 3 nhóm chính

![](/assets/img/posts/2021-10-27-design-patterns-in-golang/design-patterns-categories.png){: .align-center}

| **Creational Pattern<br>(Nhóm khởi tạo)** | **Structural Pattern<br>(Nhóm cấu trúc)** | **Behavioral Pattern<br>(Nhóm hành vi)** |
|---|---|---|
| Dùng để khởi tạo đối tượng, không phải sử dụng từ khóa new. | Dùng để thiết lập, định nghĩa quan hệ giữa các đối tượng. | Tập trung thực hiện các hành vi của đối tượng. |
| Nhóm này gồm 9 mẫu design là:<br>- Abstract Factory<br>- Builder<br>- Factory Method<br>- Multiton<br>- Pool<br>- Prototype<br>- Simple Factory<br>- Singleton<br>- Static Factory | Nhóm này gồm có 11 mẫu design là:<br>- Adapter/ Wrapper<br>- Bridge<br>- Composite<br>- Data Mapper<br>- Decorator<br>- Dependency Injection<br>- Facade<br>- Fluent Interface<br>- Flyweight<br>- Registry<br>- Proxy | Nhóm này gồm có 12 mẫu design là:<br>- Chain Of Responsibilities<br>- Command<br>- Iterator<br>- Mediator<br>- Memento<br>- Null Object<br>- Observer<br>- Specification<br>- State<br>- Strategy<br>- Template Method<br>- Visitor |

Có thể cả bạn và mình đã gặp và sử dụng các patterns này hàng ngày nhưng vì một lý do nào đó mà chúng ta không rõ hoặc chưa có khái niệm về nó. Trong bài viết này, mình sẽ đi qua một số kiến thức tổng quan Design Pattern, sau đó sẽ tiến hành cài đặt chi tiết về từng Design Pattern bằng ngôn ngữ Go.

---
## Creational patterns

### Singleton

Đảm bảo chỉ có duy nhất một instance được tạo ra và cung cấp method để có thể truy xuất được instance duy nhất đó mọi lúc mọi nơi trong chương trình.

#### Ưu điểm
- Kiểm soát và giới hạn số lượng instance, shared resource. Ví dụ: Tái sử dụng kết nối đến database, kết nối SSH đến một server để thực hiện một số công việc mà không muốn mở một kết nối khác. Hay implement Logger, Configuration, Caching
- Truy xuất instance ở cấp global
- Instance chỉ phải khởi tạo trong lần truy xuất đầu tiên.

#### Nhược điểm
- Trường hợp multithreaded cần có các xử lý đặc biệt để không tạo ra một object Singleton nhiều lần
- Khiến unit test trở nên khó khăn. Ví dụ: Khi bạn chạy unit test thì với mỗi test case có thể thay đổi state của Singleton object. Không thể mock/stubs được Client code sử dụng Singleton vì có thể constructor của Singleton là private hoặc static methods.
- Vi phạm "Single Responsibility Principle". Ví dụ: Ngoài trách nhiệm cơ bản của nó, Singleton object còn kiểm soát số lượng instances hay còn thêm chức năng manage life-cycle.

#### Cài đặt

[Github source code](https://github.com/euclid1990/design-patterns-in-go/tree/main/creational_pattern/singleton)

### Prototype

Cho phép cloning các objects từ đơn giản tới phức tạp mà không cần hiểu chi tiết bên trong chúng. Đối tượng mới là một bản sao có thể giống 100% với đối tượng gốc, chúng ta có thể thay đổi dữ liệu của nó mà không ảnh hưởng đến đối tượng gốc.

#### Ưu điểm
- Tiết kiệm thời gian và chi phí để tạo một object khi bạn đã có một object tương tự tồn tại.
- Giảm độ phức tạp cho việc khởi tạo đối tượng: do mỗi lớp chỉ implement cách clone của chính nó.
- Khởi tạo object mới bằng cách thay đổi một vài thuộc tính của object (các object có ít điểm khác biệt nhau)

#### Nhược điểm
- Đối với các object có nhiều tham chiếu có thể khó thực hiện implement việc clone

#### Cài đặt

[Github source code](https://github.com/euclid1990/design-patterns-in-go/tree/main/creational_pattern/prototype)

### Builder

Xây dựng một đôi tượng phức tạp bằng cách sử dụng các đối tượng đơn giản và sử dụng tiếp cận từng bước (chia một công việc khởi tạo phức tạp của một đối tượng ra riêng rẽ).

#### Ưu điểm
- Có thể tạo ra nhiều biến thể khác nhau cho cùng một đối tượng mà không cần thay đổi construction code.
- Đảm bảo nguyên tắc Single Responsibility Principle. Tránh việc sử dụng contruct phức tạp
- Thực hiện từng bước rõ ràng

#### Nhược điểm
- Tăng độ phức tạp của code (tổng thể) do số lượng class tăng lên. Do phải copy tất cả các thuộc tình từ class Product sang Builder nên dẫn tới duplicate code nhiều.


#### Cài đặt
[Github source code](https://github.com/euclid1990/design-patterns-in-go/tree/main/creational_pattern/builder)

### Factory Method

Đúng nghĩa là một nhà máy, và nhà máy này sẽ "sản xuất" các đối tượng theo yêu cầu của client. Tư tưởng chung là để cho lớp dẫn xuất quyết định lớp nào sẽ được tạo và tham chiếu đến đối tượng mới được tạo ra bằng cách sử dụng một interface chung.

#### Ưu điểm
- Giảm sự phụ thuộc giữa các module, cung cấp 1 hướng tiếp cận với Interface thay vì tiếp cận cách implement. Giúp chuơng trình độc lập với những lớp cụ thể mà chúng ta cần tạo 1 đối tượng, code ở phía client không bị ảnh hưởng khi thay đổi logic ở factory hay sub class.
- Single Responsibility Principle: phần creation code được di chuyển tập trung một chỗ, khi cần mở rộng, chỉ việc tạo ra sub class và implement thêm vào factory method.
- Open/Closed Principle: Thay đổi thêm bớt kiểu đối tượng cần tạo mà không làm ảnh hưởng tới client code.

#### Nhược điểm
- Tăng độ phức tạp của code (tổng thể) do số lượng subclasses và interface cần implement nhiều.

#### Cài đặt
[Github source code](https://github.com/euclid1990/design-patterns-in-go/tree/main/creational_pattern/factory_method)


## Abstract Factory

Là phương pháp tạo ra một Super-factory dùng để tạo ra các Factory khác. Hay còn được gọi là Factory của các Factory. Abstract Factory Pattern là một Pattern cấp cao hơn so với Factory Method đã mô tả ở trên.

#### Ưu điểm
- Bao gồm các ưu điểm tương tự như Factory Method design pattern

#### Nhược điểm
- Bao gồm các nhược điểm tương tự như Factory Method design pattern

#### Cài đặt
Sách sử dụng Abstract Factory để tạo các giao diện người dùng đa nền tảng mà không cần bó buộc souce code client với một kiểu cụ thể, đồng thời giữ cho tất cả các phần tử được tạo nhất quán với hệ điều hành đã chọn.

[Github source code](https://github.com/euclid1990/design-patterns-in-go/tree/main/creational_pattern/abstract_factory)

... (Còn tiếp)
