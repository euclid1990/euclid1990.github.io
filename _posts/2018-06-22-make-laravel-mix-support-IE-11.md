---
layout: post
title:  "[Laravel] Generate mix javascript bundle file hoạt động trên IE 11"
date:   2018-06-22 13:23:04 +0700
categories: Infrastructure
tags:
  - Laravel
hide_thumbnail: true
image: /assets/img/posts/2018-06-22-make-laravel-mix-support-IE-11/thumbnail.png
---

Hiện tại bên mình đang có 1 dự án code = Laravel Framework 5.6. Hôm nay bị khách hàng feedback về vụ [Sweetalert2](https://github.com/sweetalert2/sweetalert2) không hoạt động được trên `IE 11`, trong khi Chrome, Firefox thì vẫn ngon lành cành đào. Khách hàng toàn dùng `Chrome` cơ mà tự nhiên hôm nay đến ngày nên dùng `IE 11` :expressionless:

Sau khi **Github issues** thần chưởng thì mình tìm được 3 related issues tương tự sau:


- [Promise not defined in IE 11 #436](https://github.com/JeffreyWay/laravel-mix/issues/436)
- [[Suggestion] Adding polyfills on demand to the bundle. #564](https://github.com/JeffreyWay/laravel-mix/issues/564)
- [Sweetalert2 not working properly on IE11 on Windows 10 Pro v1709 #701](https://github.com/sweetalert2/sweetalert2/issues/701)

Ngay trên tài liệu của Sweetalert2 cũng đề cập vấn đề này:

```html
<script src="sweetalert2/dist/sweetalert2.all.min.js"></script>

<!-- Include a polyfill for ES6 Promises (optional) for IE11, UC Browser and Android browser support -->
<script src="https://unpkg.com/promise-polyfill"></script>
```

Tuy nhiên mình có thử và đ** thành công :joy:. Cụ thể sau 1 quá trình compile bởi `Laravel-mix` (`Webpack`) thì file `app.js` sinh ra và sử dụng nhiều function mà IE 11 không hỗ trợ. Vì thế chúng ta cần thêm `polyfill` vào khi mix file.

Nếu bạn còn đang thắc mắc `Polyfill` là clgt thì hãy đọc giải thích sau nhé:

> A polyfill is a browser fallback, made in JavaScript, that allows functionality you expect to work in modern browsers to work in older browsers, e.g., to support canvas (an HTML5 feature) in older browsers.

Cụ thể là rất nhiều `method` hoặc `function` trên bundle file được viết theo ES6 (chẳng hạn) mà không được `IE 11` hỗ trợ. Sau khi thêm thư viện `Polyfill` này vào nó sẽ check thêm `function/method` đó có tồn tại ko mới chạy, hoặc sử dụng `function/method` thay thế hoạt động trên `IE 11`. Chúng ta cũng có thể tự viết thư viện `Javascript Polyfill` của riêng mình, hãy tham khảo bài viết [sau](https://javascriptplayground.com/writing-javascript-polyfill/).

Trở lại vấn đề cần giải quyết, tóm cái váy lại để file `app.js` hoạt động tốt trên `IE 11` ta cần thêm thư viện `Polyfill` vào, ở đây mình sử dụng: [babel-polyfill](https://babeljs.io/docs/en/babel-polyfill.html)

### Bước 1

Cài đặt và [babel-polyfill](https://babeljs.io/docs/en/babel-polyfill.html) & [babel-preset-es2015](https://babeljs.io/docs/en/babel-preset-es2015/)

```terminal
$ yarn add -D babel-polyfill babel-preset-es2015
```

### Bước 2

- Tạo file `.babelrc` cùng cấp với file `webpack.mix.js` trên thư mục dự án.

```js
{
  "presets": [
    "es2015",
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 2%"],
        "uglify": true
      }
    }]
  ]
}
```
- Sửa lại file `webpack.mix.js`, thêm `options` sau vào

```js
const { mix } = require('laravel-mix')
// ...
mix.options({
  polyfills: [
    'Promise'
  ]
})
// ...
```

### Bước 3

Gọi tới package trong `resources/assets/js/app.js`

```js
// ...
require('babel-polyfill');
// ...
```

### Bước 4

Compile lại và tận hưởng thành quả trên `IE 11` :rofl:

```terminal
$ yarn run dev
```

## References

- [What is the meaning of polyfills in HTML5?](https://stackoverflow.com/questions/7087331/what-is-the-meaning-of-polyfills-in-html5)
- [Laravel-mix で、IE11に対応させる](https://qiita.com/acro5piano/items/b9bffd54c55cc4f74d51)

