---
layout: post
title: "Tại sao nên dùng String Union Types thay cho String Enums trong Typescript"
date: 2023-10-30 12:00:00 +0700
categories: Tips
tags:
  - Typescript
hide_thumbnail: true
image: /assets/img/posts/2023-10-30-why-should-you-use-string-literal-union-types-instead-of-string-enums-in-typescript/thumbnail.png

---

Khi sử dụng TypeScript, chúng ta thường gặp phải trường hợp xác định kiểu của một biến mà giá trị của nó rơi vào một hay nhiều khả năng khác nhau.

Ví dụ, status của một button có thể là: `"hidden"`, `"enabled"`, `"disabled"`.

Với Typescript chúng ta có ít nhất hai cách

```typescript
// Kiểu string enums
export enum ButtonStatus {
  HIDDEN = 'HIDDEN',
  ENABLED = 'ENABLED',
  DISABLED = 'DISABLED',
};

// Kiểu string literal union types
type ButtonStatus = 'HIDDEN' | 'ENABLED' | 'DISABLED';
```

Có cũng có kiểu enum cơ bản (không có phần `= 'HIDDEN'`), khi đó các giá trị sẽ chạy từ 0 → 2, nhưng ta sẽ không bàn về nó trong bài viết so sánh này.

Nào, hãy bắt đầu so sánh coi 🥰

## Declaring & Accessing

- Tính ngắn gọn trong khai báo
  - [✔] Union Type
  - [✘] String Enums
- Autocompletion trên Editor/IDE
  - [✔] Union Type
    - Tự động nhận trên **VsCode**
    ![](/assets/img/posts/2023-10-30-why-should-you-use-string-literal-union-types-instead-of-string-enums-in-typescript/vscode-auto-completion.gif)
  - [✘] String Enums
    - Cần exported và imported mới hoạt động

**Chiến thắng với 2 points cho Union types!**

|Union Type|String Enums|
|---|---|
|2|0|

## Extending
- [✔] Union Type
  ```typescript
  type DynamicButtonStatus = ButtonStatus | 'LOADING';
  ```
- [✘] String Enums
  - Sử dụng trick [spread operator](https://github.com/Microsoft/TypeScript/issues/17592#issuecomment-320028196)
    ```typescript
    export enum ButtonStatus {
      HIDDEN = 'HIDDEN',
      ENABLED = 'ENABLED',
      DISABLED = 'DISABLED',
    };

    // extend enum using "spread"
    const NewButtonStatus = {
        ...ButtonStatus,
        SHOW: 'SHOW',
        BLINK: 'BLINK'
    };
    ```

**Thêm 1 point cho Union types!**

|Union Type|String Enums|
|---|---|
|3|0|

## TypeError

- [✔] Union Type
  - [Playground Link](https://www.typescriptlang.org/play?#code/PTAEHUEsBcAtQK4DtIHsmmgTwA4FMBnUVAM1AOgCdIkBzUAGxj0oEMGCAobfUAIQTRo6AMrRW0BEQC8oAOQAJAJIARFQFEAcnNAAfeVoCCfADLqVO-XJVKRxsxYDcnTiWQBjaGgwAVQtABVFHQfXDwACgoJKQAufkFhJDFoggBKUABvTlByAHcYd3hI8Uk0zOyc0HdWAjx5ZTUtOTjKPElKDABGZ0qqmrq5I1NzZtBW9owAJh7K6tr5Gzthixa2hA7QAGYZnJBQTVQvdzq4CUxYFjw5IiRUUAATPBJWBAZoPtqKgF9OH849gDCqAAtjhIAw6gQEO5jgQiJAyDgakQAG7sSD3cglKSgfJwUBBbygUL4TjudAEVAQgB0DFQtHCfgohJCYXCilUGm0qVSzn+YCBoPBdRYlFQlFACNASLhkqQaKYmKipVxMHgLN8YTJFKpeFp9MZ-g1JIichECgA8uA5DzHEA)
    ```typescript
    // With union types of string literals
    type ButtonStatus = 'HIDDEN' | 'ENABLED' | 'DISABLED';

    function TestUnionType(status: ButtonStatus) {
      switch (status) {
        case 'HIDDEN': return 1;
        case 'ENABLED': return 2;
        case 'DISABLED': return 3;
        // Notice that there's no default case
      }
    }

    // Compile success if pass valid status with Union Type
    console.log(TestUnionType('HIDDEN'));

    // Compile error if pass invalid status with Union Type
    console.log(TestUnionType('SHOW'));
    ```
  - Error được phát hiện từ bước **Transpile**
    ```
    Argument of type '"SHOW"' is not assignable to parameter of type 'ButtonStatus'.
    ```
- [✘] String Enums
  - [Playground Link](https://www.typescriptlang.org/play?#code/PTAEHUEsBcAtQM7QE6QHYHNQFM0FcBbBAKF0NACE9poB7NAZWgENo8FQBvY0UACQCSAESEBRAHKgAvKADkgkRNkAaHqAkBBCgBlRQ6XM069KtUIEMtu-TNnnLxobOIBfANzFPAMzxoAxtCQ9KAAKthITKiYovhEABRIrOwAXJTUdIwsbAgAlFxqCADuMH7wCVnsedy8vH7MCNhpNPRMSQgAdApi4qnI2GzIaKAAjB41oHUNTRmt2e1G1r39eIOgAExjNZONVM2Zbe32VnpLA0MAzJu8ACbYXsx4ADbQpytDACybLq6efvRIiAqCGGBnkwm6slA9WmLSBHmIIFAAGFaAQAA6QR6NBB4Px+cIcSBeUBo+ocABuzEekGugLaoGKcFAkXQWBihBIfzQCFoWPaj1oGDiYQiKFZ7PiiWywxyOXhXIBUvYa1BDD4AHlwJDobsZnDPIjxOrkaiMVicMhkLRkKAiSSyba0JTqbSlRxGfAWZh1LFOf9edh+YLheFoF6MBKEOU2mtZW4gA)
    ```typescript
    // With string enums
    enum ButtonStatus {
      HIDDEN = 'HIDDEN',
      ENABLED = 'ENABLED',
      DISABLED = 'DISABLED'
    };


    function TestStringEnums(status: ButtonStatus) {
      switch (status) {
        case ButtonStatus.HIDDEN: return 1;
        case ButtonStatus.ENABLED: return 2;
        case ButtonStatus.DISABLED: return 3;
        default: return 4;
      }
    }

    const status1 = 'HIDDEN' as ButtonStatus;

    // Compile success if pass valid status with String Enums
    console.log(TestStringEnums(status1));

    const status2 = 'SHOW' as ButtonStatus;

    // NO Compile error if pass invalid status with String Enums
    console.log(TestStringEnums(status2));
    ```
  - Truyền sai giá trị `status2` nhưng chương trình vẫn được transpiled và run bình thường.

*Chỉ có 1 cách duy nhất để sử dụng **union types**, trong khi với **enums** bạn có thể sử dụng `ButtonStatus.HIDDEN` hoặc giá trị trực tiếp `'HIDDEN'` mà không gặp phải compiler complaining.*

**Thêm 1 point về Typesafe cho Union types!**

|Union Type|String Enums|
|---|---|
|4|0|

## Iterating

- [✘] Union Type
  ```typescript
  // Union types
  // ? Can't iterate over a type, defining an array is needed
  const ButtonStatus = ['HIDDEN', 'ENABLED', 'DISABLED'] as const;
  // Notice that the array and type can have the same name if you want
  type ButtonStatus = typeof buttonStatuses[number];

  for(const status of buttonStatus) {
    // ? the type of status here is ButtonStatus
    // ? always iterate in order
  }
  ```
- [✔] String Enums
  ```typescript
  // String enums
  // ? Just do it
  for(const status in ButtonStatus) {
    // ? The type of status here is string
    // ? iteration order not guaranteed with "in"
  }
  ```

**Trả lại 1 point cho String Enums**

|Union Type|String Enums|
|---|---|
|4|1|

## Renaming

- [✘] Union Type
  - **Find and replace** toàn Project
- [✔] String Enums
  - **Right click > Rename symbol**
    ![](/assets/img/posts/2023-10-30-why-should-you-use-string-literal-union-types-instead-of-string-enums-in-typescript/rename-symbol.gif)

**Rõ ràng String Enums tiện hơn, trả thêm 1 point cho String Enums**

|Union Type|String Enums|
|---|---|
|4|2|

## Tổng kết

Trong bài viết so sánh trên quan điểm cá nhân mình thì **Union Type** nhận được **4 điểm** và **String Enums** nhận được **2 điểm**!

Hãy sử dụng **Union types by default** :laughing:

Nếu bạn có quan điểm khác, hãy để lại comment dưới bài viết này, xin chân thành cảm ơn các góp ý từ bạn đọc 🌺
