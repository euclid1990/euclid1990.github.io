---
layout: post
title:  "Chuyện encoding tiếng Nhật và format khi export CSV trong PHP với fputcsv"
date:   2019-04-24 07:23:04 +0700
categories: Outsource
tags:
  - PHP
hide_thumbnail: true
image: /assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/thumbnail.png
---
Nếu ai đã từng join các dự án outsource cho khách hàng Nhật thì đều gặp bài toán phổ thông lấy dữ liệu từ database và xuất ra report `.csv` cho người dùng. Nghe rất đơn giản, ta chỉ cần dùng hàm có sẵn của `PHP`:

```php
fputcsv ( resource $handle , array $fields [, string $delimiter = "," [, string $enclosure = '"' [, string $escape_char = "\\" ]]] ) : int
```
  - **handle**: File pointer.
  - **fields**: Mảng dữ liệu (row data).
  - **delimiter**: Ký tự ngăn cách các field (Thường là dấu `,` hoặc `;`)
  - **enclosure**: Ký tự dùng để kiểm tra xem một field được bắt đầu và kết thúc.
  - **escape_char**: Ký tự escape (ko biết dịch thế nào cho hợp lý) để tắt cơ chế này thì hãy set giá trị empty ("")

    Ví dụ: `Trong Javascript, sử dụng ký tự \ làm escape character:`
![](/assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/javascript_escape_character.png)

## Encoding, bài toán muôn thuở

Đi tới vấn đề phức tạp hơn:
> Chúng ta có nhận thêm yêu cầu là muốn convert dữ liệu trong csv này từ original `UTF-8` sang `Shift-JIS` ~ WTF!!! 🎱

Lý do người Nhật vẫn dùng `Shift-JIS` chỉ đơn giản gói gọn trong 1 từ khoá **Legacy**. `Shift-JIS`, `EUC-JP` đều là những chuẩn encoding được sử dụng trước khi mà `UTF-8` trở lên phổ biến như ngày hôm nay. Ngoài ra còn một lý do nữa mình nghĩ là cũng quan trọng khi mà `Shift-JIS` chỉ cần dùng `2-byte` mã hoá trong khi `UTF-8` cần tới `3~4-byte` cho việc encoding tiếng Nhật nói chung và `CJK` nói riêng 🎃

Ở đây mình có một base code như sau. Thay vì đọc dữ liệu từ database, chúng được hard code trong 1 array:

```php
function convert($fields) {
    $result = [];
    foreach ($fields as $field) {
        $result[] = mb_convert_encoding($field, 'SJIS', 'UTF-8');
    }
    return $result;
}

function arr2csv($rows) {
    $fp = fopen('php://temp', 'r+b');
    foreach($rows as $fields) {
        // Convert row data from UTF-8 to Shift-JS
        $fields = convert($fields);
        fputcsv($fp, $fields);
    }
    rewind($fp);
    // Convert CRLF
    $tmp = str_replace(PHP_EOL, "\r\n", stream_get_contents($fp));
    fclose($fp);
    return $tmp;
}

function writecsv($filename) {
    // Header of csv
    $header = [
        'ID',
        '従業員コード', // Employee code
        '氏名※',       // Name
        'フリガナ',     // Furigana
    ];
    // Sample data from database
    $data = [
        [
            "id" => 1,
            "code" => "A01",
            "name" => "山",
            "furigana" => "カミソヤマ　ユニ"
        ],
        [
            "id" => 2,
            "code" => "A02",
            "name" => "上曽山　ゆに",
            "furigana" => "ソ　ユニ",
        ],
        [
            "id" => 3,
            "code" => "A03",
            "name" => '上曽山\\"　上曽山',
            "furigana" => "ゆに",
        ],
    ];
    // Prepend header
    array_unshift($data, $header);
    // Open file to write stream data
    $f = fopen($filename, 'w+');
    fwrite($f, arr2csv($data));
    fclose($f);
}

writecsv("output.csv");
```
- Expected:
```terminal
ID,従業員コード,氏名※,フリガナ
1,A01.,山,カミソヤマ　ユニ
2,A02,上曽山　ゆに,ソ　ユニ
3,A03,"上曽山\"　上曽山",ゆに
```
- Output:
```terminal
ID,従業員コード,氏名※,フリガナ
1,A01,山,"カミソヤマ　ユニ"
2,A02,上曽山　ゆに,"ソ　ユニ"
3,A03,"上曽山\"　上曽山",ゆに
```
Để ý sẽ thấy row số 1, cột Furigana: Giá trị `カミソヤマ　ユニ` bị đặt trong dấu `"`, trở thành `"カミソヤマ　ユニ"`. Nếu chỉ xem trên các chương trình như Open Office, MS Excel sẽ rất khó phát hiện. Ở đây mình có 2 ảnh mở trên `Terminal` và `Sublime` sẽ thấy rõ điều đó !
![](/assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/output.csv_on_terminal.png)
![](/assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/output.csv_on_sublime.png)

Trong dữ liệu ngoài row cuối cùng chứa ký tự double quote `"` ra thì các row ở trên nó hoàn toàn không có chứa ký tự đó. Nên chẳng có lý do gì bị wrap lại ấy thế mà 😢

Sau khi thử loại trừ bằng cách xoá đi dần các ký tự bên trong giá trị của các field, mình phát hiện thêm 1 điều:
> Giá trị trên mỗi row của csv chỉ bị dính double quotes add thêm trong trường hợp có chứa ký tự 「ソ」、「十」hoặc ký tự nào nữa cũng chưa rõ ???

```php
$stdout = fopen('php://stdout', 'w');
fprintf($stdout, "UTF-8 ソ: 0x%s\n", bin2hex('ソ'));
fprintf($stdout, "UTF-8 十: 0x%s\n", bin2hex('十'));
fprintf($stdout, "Shift-JIS ソ: 0x%s\n", bin2hex(mb_convert_encoding('ソ', 'SJIS', 'UTF-8')));
fprintf($stdout, "Shift-JIS 十: 0x%s\n", bin2hex(mb_convert_encoding('十', 'SJIS', 'UTF-8')));
```
```terminal
UTF-8 ソ: 0xe382bd
UTF-8 十: 0xe58d81
Shift-JIS ソ: 0x835c
Shift-JIS 十: 0x8f5c
```
Nhìn vào code trên ta có thể thấy UTF-8 cần 3 bytes trong khi Shift-JIS chỉ cần 2 byte cho các ký tự tiếng nhật. Cụ thể:

「ソ」=「0x8F5C」=「0x8F」+「0x5C」

「十」=「0x8f5c」=「0x8f」+「0x5c」

Sau khi đã thử tìm kiếm nguyên nhân trên Google mà vẫn không cho kết quả thoả đáng, thì quyết định cuối cùng là thử mần tới method `php_fputcsv` bên trong `PHP Interpreter > php-src/ext/standard/file.c`:
  ```c
  PHPAPI size_t php_fputcsv(php_stream *stream, zval *fields, char delimiter, char enclosure, int escape_char)
  {
    ...
    smart_str csvline = {0};

    ZEND_ASSERT((escape_char >= 0 && escape_char <= UCHAR_MAX) || escape_char == PHP_CSV_NO_ESCAPE);
    count = zend_hash_num_elements(Z_ARRVAL_P(fields));
    ZEND_HASH_FOREACH_VAL(Z_ARRVAL_P(fields), field_tmp) {
      ...
      /* enclose a field that contains a delimiter, an enclosure character, or a newline */
      if (FPUTCSV_FLD_CHK(delimiter) ||
        FPUTCSV_FLD_CHK(enclosure) ||
        (escape_char != PHP_CSV_NO_ESCAPE && FPUTCSV_FLD_CHK(escape_char)) ||
        FPUTCSV_FLD_CHK('\n') ||
        FPUTCSV_FLD_CHK('\r') ||
        FPUTCSV_FLD_CHK('\t') ||
        FPUTCSV_FLD_CHK(' ')
      ) {
        char *ch = ZSTR_VAL(field_str);
        char *end = ch + ZSTR_LEN(field_str);
        int escaped = 0;

        smart_str_appendc(&csvline, enclosure);
        while (ch < end) {
          if (escape_char != PHP_CSV_NO_ESCAPE && *ch == escape_char) {
            escaped = 1;
          } else if (!escaped && *ch == enclosure) {
            smart_str_appendc(&csvline, enclosure);
          } else {
            escaped = 0;
          }
          smart_str_appendc(&csvline, *ch);
          ch++;
        }
        smart_str_appendc(&csvline, enclosure);
      } else {
        smart_str_append(&csvline, field_str);
      }
      ...
    } ZEND_HASH_FOREACH_END();
    ...
  }
  ```
  References:
  - [`#define PHP_CSV_NO_ESCAPE EOF`](https://github.com/php/php-src/blob/91ef4124e56a8ec52078bdcb5547ea5dbf654566/ext/standard/file.h#L80)
  - [`php-src/ext/standard/file.c`](https://github.com/php/php-src/blob/91ef4124e56a8ec52078bdcb5547ea5dbf654566/ext/standard/file.c#L1886L1893)
  - [`php-src/Zend/zend_smart_str_public.h > typedef smart_str struct`](https://github.com/php/php-src/blob/91ef4124e56a8ec52078bdcb5547ea5dbf654566/Zend/zend_smart_str_public.h#L22L25)
  - [`php-src/Zend/zend_smart_str.h > smart_str_appendc`](https://github.com/php/php-src/blob/91ef4124e56a8ec52078bdcb5547ea5dbf654566/Zend/zend_smart_str.h#L32)

Để ý đoạn `if{block}` xử lý thêm kí tự `enclosure` vào giá trị từng field trên mỗi row-csv có điều kiện sau:
```c
/* enclose a field that contains a delimiter, an enclosure character, or a newline */
if (...
  (escape_char != PHP_CSV_NO_ESCAPE && FPUTCSV_FLD_CHK(escape_char)) ||
  ...
) {
```
Rõ ràng khi `$escape_char` được set giá trị mặc định `"\\"` nó thoả mãn điều kiện `escape_char != PHP_CSV_NO_ESCAPE` vậy còn điều kiện `FPUTCSV_FLD_CHK(escape_char))` tức là sao ?
```java
#define FPUTCSV_FLD_CHK(c) memchr(ZSTR_VAL(field_str), c, ZSTR_LEN(field_str))
// ---> Return:
// A pointer to the first occurrence of value(c) in the block of memory pointed by ptr(field_str).
// If the value is not found, the function returns a null pointer.
```
*Chứng tỏ để có kết quả output kia chắc chắn bên trong giá trị các field có chứ ký tự backslash `\`!*

Test thử với sample sau: `memchr.c`
```c
#include <stdio.h>
#include <string.h>

#define FPUTCSV_FLD_CHK(c) memchr(str, c, strlen(str))

int main()
{
    const char str[] = "We Make It Awesome";
    const char ch = 'e';
    char *ret;

    ret = FPUTCSV_FLD_CHK(ch);

    if (ret != NULL) {
        printf ("'%c' found at position %d.\n", ch, ret-str+1);
        printf("String after |%c| is - |%s|\n", ch, ret);
    } else {
        printf ("'%c' not found.\n", ch);
    }

    return 0;
}
```
- Output:
```terminal
'e' found at position 2.
String after |e| is - |e Make It Awesome|
```

Ok, đọc tới đây chắc hẳn bạn đã hiểu nguyên lý hoạt động của method `memchr` trong `C/C++`. Cụ thể trong trường hợp này dùng để check sự có mặt của ký tự `$escape_char = "\\"` trong giá trị của `$field` hay không, nếu có wrap lại giá trị đó bằng double quotes `""`.

Thử apply trong trường hợp ký tự `「ソ」=「0x8F5C」=「0x8F」+「0x5C」` xem sao nhỉ :arrow_heading_down:

```c
#include <stdio.h>
#include <string.h>

#define FPUTCSV_FLD_CHK(c) memchr(str, c, strlen(str))

int main()
{
    const char str[] = "\x8F\x5C";  /* Important! initialize char array using hex numbers: 0x8F5C = ソ */
    const char ch = '\\';           /* Find backslash char */
    char *ret;

    ret = FPUTCSV_FLD_CHK(ch);

    if (ret != NULL) {
        printf ("'%c' found at position %d.\n", ch, ret-str+1);
        printf("String after |%c| is - |%s|\n", ch, ret);
    } else {
        printf ("'%c' not found.\n", ch);
    }

    return 0;
}
```
- Output:
```terminal
'\' found at position 2.
String after |\| is - |\|
```

Ký tự backslash `'\'` trong `UTF-8` lại xuất hiện trong string `「ソ」=「0x8F5C」=「0x8F」+「0x5C」` - `Shift-JIS`.
Hoá ra `0x5C` trong bảng ASCII lại chính là ký tự `\`.

![](/assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/superise.jpg)

👉 [ASCII Table](http://lwp.interglacial.com/appf_01.htm) :capital_abcd:

Và có lẽ đó chính là lý do mà giá trị của các field trong file `csv` bị wrap bởi dấu `"` một cách bất thường !

![](/assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/done.png)

**Và cuối cùng đây là bí kíp cần ghi nhớ khi động tới csv 📖**

Việc cần làm rất đơn giản, chỉ cần đổi lại thứ tự thực hiện convert encoding sau khi export định dạng CSV là ok.
Và việc đổi thứ tự này cũng giúp cost tính toán giảm đi khá nhiều vì chỉ phải thực hiện câu lệnh encode một lần duy nhất, thay vì `n` lần như version đầu tiên !

```php
function arr2csv($rows) {
    $fp = fopen('php://temp', 'r+b');
    foreach($rows as $fields) {
        fputcsv($fp, $fields);
    }
    rewind($fp);
    // Convert CRLF
    $tmp = str_replace(PHP_EOL, "\r\n", stream_get_contents($fp));
    fclose($fp);
    // Convert row data from UTF-8 to Shift-JS
    return mb_convert_encoding($tmp, 'SJIS', 'UTF-8');
}

function writecsv($filename) {
    // Header of csv
    $header = [
        'ID',
        '従業員コード', // Employee code
        '氏名※',       // Name
        'フリガナ',     // Furigana
    ];
    // Sample data from database
    $data = [
        [
            "id" => 1,
            "code" => "A01",
            "name" => "山",
            "furigana" => "カミソヤマ　ユニ"
        ],
        [
            "id" => 2,
            "code" => "A02",
            "name" => "上曽山　ゆに",
            "furigana" => "ソ　ユニ",
        ],
        [
            "id" => 3,
            "code" => "A03",
            "name" => '上曽山\\"　上曽山',
            "furigana" => "ゆに",
        ],
    ];
    // Prepend header
    array_unshift($data, $header);
    // Open file to write stream data
    $f = fopen($filename, 'w+');
    fwrite($f, arr2csv($data));
    fclose($f);
}

writecsv("output.csv");
```
- Expected:
```terminal
ID,従業員コード,氏名※,フリガナ
1,A01.,山,カミソヤマ　ユニ
2,A02,上曽山　ゆに,ソ　ユニ
3,A03,"上曽山\"　上曽山",ゆに
```
- Output:
```terminal
ID,従業員コード,氏名※,フリガナ
1,A01.,山,カミソヤマ　ユニ
2,A02,上曽山　ゆに,ソ　ユニ
3,A03,"上曽山\"　上曽山",ゆに
```

Output file đúng như mong đợi 👍 💯

## Chuẩn RFC 4180 & Lỗi chưa được fix

[RFC 4180](https://www.ietf.org/rfc/rfc4180.txt) một chuẩn quy ước định dạng cho file CSV được sử dụng để đọc xuất CSV trong rất nhiều ứng dụng, đơn cử như Google Spreadsheet.

Nếu ta có 1 spreadsheet với nội dung sau:
```
"Hello\", World!`
```
![](/assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/csv_rfc_4180_google_spreadsheet.png)

Thì khi chọn `[File] > [Download as] > [Comma-separated values (.csv, current sheet)]` ta sẽ được nội dung file như sau:
```
"""Hello\"", World!"
```
![](/assets/img/posts/2019-04-24-php-fputs-csv-convert-utf8-to-shiftjs-encoding/csv_rfc_4180_csv_download.png)

Thử dùng `PHP` đọc file này coi sao 😃
```php
$fp = fopen('rfc4180.csv', 'r');
$row = fgetcsv($fp, 0, ',', '"');
echo $row[0].PHP_EOL;
```
- Expected:
```terminal
"Hello\", World!
```
- Output:
```terminal
"Hello\"
```

Nguyên nhân kết quả không đúng như mong đợi là do hàm `fputcsv` của `PHP` đang được sử dụng không đúng chuẩn [RFC 4180](https://www.ietf.org/rfc/rfc4180.txt).

> *If double-quotes are used to enclose fields, then a double-quote appearing inside a field must be escaped by preceding it with another double quote.*

Chỉnh sửa lại một chút !
```php
$fp = fopen('rfc4180.csv', 'r');
$row = fgetcsv($fp, 0, ',', '"', '"');    // Add escape_char='"'
echo $row[0].PHP_EOL;
```
- Expected:
```terminal
"Hello\", World!
```
- Output:
```terminal
"Hello\", World!
```

Tuy nhiên khi dùng `fputcsv` với chuẩn RFC_4180 để ghi ra csv file rồi sau đó dùng `fgetcsv` để đọc lại chính file vừa được ghi thì kết quả lại không chính xác.

```php
$fp = tmpfile();
fputcsv($fp, ['"Hello\", World!'], ',', '"', '"');
rewind($fp);
echo fgets($fp, 4096);
rewind($fp);
$row = fgetcsv($fp, 0, ',', '"', '"');
echo $row[0].PHP_EOL;
echo $row[1].PHP_EOL;
```
- Output:
```terminal
""Hello\", World!"    # wrote content
Hello\"               # read content: row_1
 World!"              # read content: row_2
```

Dễ thấy kết quả đọc ra thì dữ liệu lại trở thành 2 row trong khi expect là 1 row mà thôi 🙃.

Đây là một bug tồn tại trong PHP, mà ko được fix ! bạn có thể vô đây coi issue: [50686](https://bugs.php.net/bug.php?id=50686)
Hiện tại mình cũng không biết fix lỗi này kiểu gì 🤣
