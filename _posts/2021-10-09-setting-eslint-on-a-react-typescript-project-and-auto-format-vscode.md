---
layout: post
title:  "Thiết lập ESLint cho dự án React sử dụng Typescript và tự động định dạng mã trên VSCode"
date:   2021-10-09 10:00:04 +0700
categories: Tips
hide_thumbnail: true
image:
---

Mỗi lần khởi tạo một dự án Javascript và Typescript thì công việc cài đặt bộ Linting (Xác minh chất lượng mã, tự động định dạng lại source code cho đẹp, ...), thiết lập rules cho Linting Tool là việc cần làm ngay từ thời điểm bắt đầu code.

Có rất nhiều công cụ Lint cho JavaScript hay Typescript khác nhau nhưng tôi sẽ chỉ dùng ESLint mà thôi. Lý do thì bạn cứ dùng các thằng khác rồi so sánh sẽ hiểu 😂.

Tuy nhiên, giờ đã là năm 2021 ~ làm gì cũng phải nhanh gọn nhẹ, bạn sẽ không thể nhớ được hết các công cụ, các packages cần thêm vào file `package.json` cần cài đặt nếu muốn sử dụng để chạy lệnh `yarn add` hay `npm install`. Để đối phó với bài toán này chúng ta thường có 2 phương pháp: Tìm trên mạng hoặc Sử dụng lại cấu hình dự án cũ. Thế nào cũng được, nhưng theo thời gian tôi sợ cấu hình cũ bị Outdate, phải đi sửa lại version ESLint rồi một tá plugin kèm theo 😢

Mục tiêu chính của hướng dẫn này là thiết lập ESLint một cách nhanh và đơn giản nhất. Bạn sẽ không cần nhớ một đống Rules, rồi nhớ cả tá Plugins đi kèm.

Ở đây tôi sẽ ví dụ trường hợp cài đặt ESLint cho dự án React sử dụng Typescript, đồng thời hỗ trợ tự Format (Định dạng) lại source code mỗi lần bạn Save (Lưu) trên Visual Studio Code (VSCode).
  - Cài đặt ESLint trong dự án React của bạn

    `$ yarn add -D eslint`
  - Khởi tạo cấu hình ESLint: *eslintrc.js*

    `$ yarn eslint --init`

    Menu câu hỏi xuất hiện, bạn chọn giống tôi là OK.
    ```bash
    ✔ How would you like to use ESLint? · To check syntax, find problems, and enforce code style
    ✔ What type of modules does your project use? · JavaScript modules (import/export)
    ✔ Which framework does your project use? · React
    ✔ Does your project use TypeScript? · Yes
    ✔ Where does your code run? · Browser
    ✔ How would you like to define a style for your project? · Use a popular style guide
    ✔ Which style guide do you want to follow? · Standard
    ✔ What format do you want your config file to be in? · JavaScript
    ✔ Would you like to install them now with npm? · Yes

    + eslint-config-standard@16.0.3
    + eslint-plugin-promise@5.1.0
    + ... > Chỗ này nó cài cho mình nhiều packages lắm tôi chẳng nhớ nổi :))
    ```

    Đợi một lúc và thế là ESLint đã tự cài đặt, ở đây tôi dùng sẵn bộ Style Guide của Github là [Standard](https://standardjs.com/), nếu bạn không thích có thể dùng của Airbnb hay Google, ...

  - Kiểm tra thư mục dự án đã thấy xuất hiện file: *eslintrc.js*. Ta thêm một số Rules cơ bản vào:

    ```javascript
    rules: {
      'no-use-before-define': 'off',
      '@typescript-eslint/no-use-before-define': ['error'],
      'react/jsx-filename-extension': ['warn', { extensions: ['.tsx'] }]
    }
    ```

    Các bạn có thể bổ sung thêm tại đây [nhé](https://eslint.org/docs/rules/).

  - Cài đặt VS Code ESLint extension:

    ![](/assets/img/posts/2021-10-09-setting-eslint-on-a-react-typescript-project-and-auto-format-vscode/vscode-eslint.png){: .align-center}

  - Thêm Lint và format với scripts:

    Tìm tới file `package.json` > `scripts` và thêm các mã sau vào để dùng trên Terminal hoặc CI service:

    ```javascript
    "scripts": {
      "lint": "eslint 'src/**/*.{js,jsx,ts,tsx}'",
      "lint:fix": "eslint --fix 'src/**/*.{js,jsx,ts,tsx}'",
    }
    ```

    Nếu muốn có thể vô Terminal chạy thử 2 lệnh trên nha. `$ yarn run lint` hoặc `$ yarn run lint:fix`

  - Tự động cấu hình VSCode mỗi khi Save file .tsx hoặc ts hoặc js như sau:

    Tạo file `.vscode/settings.json` trong thư mục dự án:

    ![](/assets/img/posts/2021-10-09-setting-eslint-on-a-react-typescript-project-and-auto-format-vscode/vscode-settings.png){: .align-center}

    ```javascript
    {
      "[javascriptreact]": {
        "editor.codeActionsOnSave": {
          "source.fixAll.eslint": true
        }
      },
      "[typescript]": {
        "editor.codeActionsOnSave": {
          "source.fixAll.eslint": true
        }
      },
      "[typescriptreact]": {
        "editor.codeActionsOnSave": {
          "source.fixAll.eslint": true
        }
      }
    }
    ```

  Bây giờ thử mở một file `.ts` hay `.tsx` ra mỗi lần Save file bạn sẽ thấy code tự format đẹp như tranh vẽ 💯

  ![](/assets/img/posts/2021-10-09-setting-eslint-on-a-react-typescript-project-and-auto-format-vscode/vscode-format.gif){: .align-center}
