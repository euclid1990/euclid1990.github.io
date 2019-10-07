---
layout: post
title:  "Sử dụng Google Spreadsheet hiệu quả hơn với Google Apps Script"
date:   2019-10-07 10:00:04 +0700
categories: Tips
tags:
  - App scripts
  - Daily working
hide_thumbnail: true
image: /assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/thumbnail.jpg
---

Google App Script (GAS) là ngôn ngữ lập trình dựa trên JavaScript với việc biên tập, biên dịch cũng như lưu trữ đều được đảm nhiệm bởi Gooogle. Về tài liệu liên quan tới GAS các bạn có thể tìm thấy rất nhiều tại [đây](https://developers.google.com/apps-script).

Việc hiểu và biết cách sử dụng GAS sẽ giúp cho cả tôi và bạn có thể tận dụng hệ sinh thái của Google hiệu quả cũng như tiết kiệm rất nhiều thời gian khi làm việc với chúng.

Trong bài viết này tôi sẽ không đi quá sâu vào ngôn ngữ GAS mà chỉ nêu ra một trong số các bài toán tôi đã và đang giải quyết nó nhanh gọn bằng GAS trong nhiều năm trở lại đây.

## Bài toán

> Tôi có một spreadsheet quản lý các câu hỏi (từ team Offshore) và câu trả lời (từ khách hàng) & nghiễm nhiên các câu hỏi và câu trả lời đều được dịch song ngữ (JP-VN). Spreadsheet này tạm gọi tắt là `Spreadsheets Q&A`.
>
> Tuy nhiên trong quá trình phát triển thường xảy ra vấn đề khách hàng trả lời câu hỏi trong `Spreadsheets Q&A` nhưng lại không thông báo lại cho phía VN (Qua Brse/Comtor/Team Leader, ...etc) dẫn tới tình trạng ứ đọng việc trao đổi giữa hai bên, tốn thời gian confirm qua lại.

Về nội dung `Spreadsheets Q&A` thì tương tự như ảnh bên dưới 👇
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/project_qa_list_spreadsheets.png)

## Giải pháp A

Ban đầu tôi thường xuyên nhắc nhở khách hàng trên nhóm chat rằng: phải thông báo cho phía VN khi họ trả lời xong một vài Q&A nào đó → Họ cũng ghi nhận, nhưng kết quả là họ chỉ thực hiện tốt việc đó một vài lần đầu 🤣

## Giải pháp B

Sau khi giải pháp A thất bại, tôi nghĩ tới cách "tay to" này, đó là tôi dùng 👀 gửi polling request vào `Spreadsheets Q&A` 😂. Mỗi khi thấy thanh niên khách hàng update là tôi lại hỏi lại nó: "Mày trả lời câu #xyz rồi hả?", thế để tao nhắn lại cho đội VN nhé 😎 Cách này khá OK, nhưng tôi mệt vãi nồi các bạn ạ ! Không thể giải thoát cái đầu mình khỏi việc tracking các thay đổi trên `Spreadsheets Q&A` :cold_sweat:

![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/just_tired.jpg)

## Giải pháp C

Nhờ một thành viên trong team, tracking sự thay đổi trong `Spreadsheets Q&A`, thực sự khó vì méo ai muốn làm công việc nè 👎

![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/your_job_is_not_my_job.jpg)

## Giải pháp D

Sử dụng Bot, giúp tôi watching các thay đổi trên `Spreadsheets`. Nếu như người thay đổi là khách hàng hoặc ô (`cell`) thay đổi nội dung là ô (`cell`) dành cho khách (Các `cell` thuộc column `G` như `Spreadsheets Q&A` tôi có đính kèm ở trên) thì Bot sẽ gửi tin nhắn tới Box chat của dự án, hoặc Box chat private giữa tôi và nó, kèm theo message mô tả nội dung thay đổi.

![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/chatwork_notice_message.png)

Như bạn thấy, ở ảnh trên, con Bot không chỉ làm nhiệm vụ tracking các thay đổi trong `Spreadsheets Q&A` mà còn làm thêm nhiệm vụ tracking các thay đổi trong `Spreadsheets Feedback` (một tài liệu khác). Giảm tải được rất nhiều việc cho tôi ahihi. 💯

Sử dụng Google Apps Script, tất nhiên bạn cần biết lập trình một chút, không cần quá giỏi, chỉ cần đọc hiểu được tài liệu của Google cung cấp sẵn là 👌

- [Google Apps Script + Google Sheets](https://developers.google.com/apps-script/guides/sheets)
- [Tạo Trigger để run Apps Script khi 1 hoạt động nào đó xảy ra](https://developers.google.com/apps-script/guides/triggers)
- [Tài liệu tổng hợp các Spreadsheet Service](https://developers.google.com/apps-script/reference/spreadsheet)
- [URL Fetch - HTTP/HTTPS request trong Apps Script](https://developers.google.com/apps-script/reference/url-fetch)
- [Utilities - Các tiện ích như encoding/decoding, formatting,... trong Apps Script](https://developers.google.com/apps-script/reference/utilities)

Sau khi hiểu một vài khái niệm cũng như các function của Apps Script, trước hết các bạn thực hiện từng bước theo hướng dẫn sau để làm quen trước nhé:

### Cách chạy Apps Script trong Spreadsheet

- Đầu tiên các bạn mở `Spreadsheet` ra, vào mục `Tools > Script editor`
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/script_editor.png)
Đây chính là nơi để paste đoạn code cần thực hiện vào.
- Để chạy đoạn code bạn hãy chọn Function muốn thực hiện và bấm vào icon :arrow_forward:
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/run_script.png)
- Để xem Log của script thì chúng ta các 3 kiểu Log sau:
  - `Excution transcript`:
  > Every time you run a script, Google Apps Script records an execution transcript, which is a record of each call to a Google Apps Script service that is made while the script runs.

    Để truy cập các bạn vào menu `View > Execution transcript`
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/execution_transcript.png)
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/execution_transcript_modal.png)
  - `Apps Script Logger`:
  > Lightweight but persists only for a short time. These logs are intended for simple checks during development and debugging, and do not persist very long.

    Sử dụng `Logger` tương đối dễ, ví dụ đơn giản:
    ```javascript
    // Log the number of Google Groups you belong to.
    var groups = GroupsApp.getGroups();
    Logger.log('You are a member of %s Google Groups.', groups.length);
    ```
    Để truy cập các bạn vào menu `View > Logs`
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/logger.png)
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/logger_modal.png)
  - `Stackdriver Logging`:
  >  Google Cloud Platform (GCP) Stackdriver Logging service (formerly known as "Cloud Logging"). When you require logging that persists for several days, or need a more complex logging solution for a multi-user production environment, Stackdriver Logging is the preferred choice.

    Sử dụng `Stackdriver Logging` tương đối dễ, ví dụ đơn giản:
    ```javascript
    /**
     * Logs the time taken to execute 'myFunction'.
     */
    function measuringExecutionTime() {
      // A simple INFO log message, using sprintf() formatting.
      console.info('Timing the %s function (%d arguments)', 'myFunction', 1);
      ...
    }
    ```
    Để truy cập các bạn vào menu `View > Logs > Apps Script Dashboard`
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/stackdriver_logging.png)
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/apps_script_dashboard.png)
    Tuy nhiên có một hạn chế đó là trên `Apps Script Dashboard` chỉ có thể xem được message log sinh ra khi script được thực hiện bởi chính người đó, còn nếu bạn muốn xem được toàn bộ log thì bạn buộc phải attach `Apps script` vào một `GCP Project`. Xem hướng dẫn tại [đây](https://developers.google.com/apps-script/guides/cloud-platform-projects?hl=en) hoặc như ảnh bên dưới.

    Setting `OAuth consent screen`
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/oauth_consent_screen.png)
    Attach to standard `GCP project`
    ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/resources_attach_gcp.png)

- Trong Apps Script có file `manifest: appsscript.json` theo format `JSON` chứa các cấu hình cơ bản để có thể chạy được code. Ví dụ như `timeZone`, `oauthScopes`, ... để hiển thị nội dung file này, các bạn vào `View > Show manifest file`
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/manifest_file.png)

### Implement Giải pháp D

- **[1.]** Trong file `appsscript.json` các bạn thêm nội dung sau vào:
```javascript
{
  "timeZone": "Asia/Bangkok",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/script.external_request",
    "https://www.googleapis.com/auth/userinfo.email"
  ]
}
```
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/qa_appsscript_json.png)

- **[2.]** Trong file `Code.gs` các bạn sử dụng code sau và chỉnh sửa lại các params cho hợp lý

  ```javascript
  // Please edit following constant before apply this script
  var CHATWORK_TOKEN = 'your_chatwork_token'; // Chatwork API token
  var CHATWORK_ROOM_ID = '12345678';  // Chatwork notice room ID
  var CHATWORK_SLEEP = 30; // Time sleep before notice / Unit: second
  var CHATWORK_MESSAGE = '[To:1] Nguyen Van A\n(F) The answer for question [#%s] has been updated !\nPlease check it out: %s';  // Nội dung message sẽ gửi vào room trên
  var CUSTOMER_INFO = [{name: 'Customer A', email: 'customer.a@example.jp'}, {name: 'Customer B', email: 'customer.b@example.jp'}]; // Thông tin khách hàng thao tác với sheet Q&A
  var OFFSHORE_INFO = [{name: 'ANV', email: 'nguyen.van.a@example.com'}]; // Thông tin VN team thao tác với sheet Q&A
  var SPREADSHEET_SHEET_LOGS = 'Logs';  // Tên sheet dùng để log dữ liệu notice nhằm tránh gửi lại 2 lần một thông báo giống nhau
  var SPREADSHEET_SHEET_WATCHING = [{name: 'Sprint4~QA一覧', subjectColumn: 'A', watchColumn: 'G'}, {name: 'QA一覧', subjectColumn: 'A', watchColumn: 'G'}]; // Thông tin các sheet sẽ tracking sự thay đổi gồm có: Tên sheet/Số thứ tự Q&A/Cột tracking nội dung - Có thể để null nếu muốn tracking toàn bộ nội dung sheet

  // Trigger when user edit sheet
  function onEditTrigger(e) {
    var spreadsheet = e.source;
    var spreadsheetId = spreadsheet.getId();
    var spreadsheetTz = spreadsheet.getSpreadsheetTimeZone();
    var spreadsheetLogs = SPREADSHEET_SHEET_LOGS;
    var sheet = e.range.getSheet();
    var sheetName = spreadsheet.getActiveSheet().getName();
    var url = spreadsheet.getUrl();
    var targetSheet = getItemFromArrayObject(SPREADSHEET_SHEET_WATCHING, 'name', sheetName);
    // We dont care if someone insert/clear logs manually
    // Or do not send message if recently notice
    if ((targetSheet == false) || !isShouldSend(spreadsheetId, spreadsheetTz, spreadsheetLogs, sheet, targetSheet)) {
      return;
    }
    var msg = buildMessage(url, sheet, targetSheet);
    Utilities.sleep(CHATWORK_SLEEP*1000);
    sendMessage(CHATWORK_ROOM_ID, msg);
  }

  // Set header for Logs sheet/ Run once time
  function settingLogsSpreadsheet(spreadsheetId, spreadsheetLogs) {
    if (typeof(spreadsheetId) == 'undefined') {
      spreadsheetId = SpreadsheetApp.getActive().getId();
    }
    if (typeof(spreadsheetLogs) == 'undefined') {
      spreadsheetLogs = SPREADSHEET_SHEET_LOGS;
    }
    var ssLogs = SpreadsheetApp.openById(spreadsheetId).getSheetByName(spreadsheetLogs);
    if (ssLogs == null) {
      ssLogs = SpreadsheetApp.openById(spreadsheetId).insertSheet(spreadsheetLogs);
    } else {
      return '✗ Spreadsheet is already existing.';
    }
    ssLogs.setFrozenRows(1);
    ssLogs.getRange(1, 1, 1, 3)
      .setBackground('lightGray')
      .setFontWeight('bold')
      .setValues([['Subject', 'Editor', 'Datetime']]);
    return '✔ Spreadsheet header was set successfully.';
  }

  // Check condition to decide execute sending notice message or not
  function isShouldSend(spreadsheetId, spreadsheetTz, spreadsheetLogs, sheet, targetSheet) {
    var email = getEditorEmail();
    var editor = getCustomer(email);
    var subject = getSubject(sheet, targetSheet);
    var now = new Date();
    var nowFormatted = Utilities.formatDate(new Date(), spreadsheetTz, "yyyy-MM-dd HH:mm:ss");
    var ssLogs = SpreadsheetApp.openById(spreadsheetId).getSheetByName(spreadsheetLogs);
    var vlookups = ssLogs.getRange(2, 1, ssLogs.getLastRow(), ssLogs.getLastColumn()).getValues();
    var columnLetter = getColumnLetter(sheet.getActiveCell().getColumn());
    // Return if editor is not a customer
    if (editor === false || (targetSheet.watchColumn !== null && targetSheet.watchColumn !== columnLetter)) {
      console.info('This editting is should not send');
      console.info('Email: %s', email);
      return false;
    }
    // Match item have same subject in logs
    for (var i = 0; i < vlookups.length; i++) {
      var rows = vlookups[i];
      // Logs sheet is not empty
      if (rows.length == 3) {
        if (rows[0] == subject) {
          var lastNoticedAt = new Date(rows[2]);
          var diffTime = Math.floor((now.valueOf() - lastNoticedAt.valueOf())/1000/60); // Unit: minute
          // No notice if we recently notice 2 minutes ago
          if (diffTime < 2) return false;
          break;
        }
        continue;
      }
    }
    // Insert log at first
    ssLogs.insertRowBefore(2);
    ssLogs.getRange(2, 1, 1, 3).setValues([[subject, editor.email, nowFormatted]]);
    return true;
  }

  // Get column letter from column index number: A, B, C, ...
  function getColumnLetter(column) {
    var temp, letter = '';
    while (column > 0) {
      temp = (column - 1) % 26;
      letter = String.fromCharCode(temp + 65) + letter;
      column = (column - temp - 1) / 26;
    }
    return letter;
  }

  // Get active sheet header
  function getHeader(sheet) {
    var data = sheet.getRange("A1:1").getValues();
    var header = data[0];
    return header;
  }

  // Get link to cell in spreadsheet
  function getCellUrl(url, sheetId, row, column) {
    var cell = getColumnLetter(column) + row;
    var rows = row + ':' + row;
    return url + '#gid=' + sheetId + '&range=' + rows;
  }

  // Determine edit subject
  function getSubject(sheet, targetSheet) {
    var row = sheet.getActiveCell().getRow();
    var cellName = Utilities.formatString('%s%d', targetSheet.subjectColumn, row);
    var subject = '';
    var cell = sheet.getRange(cellName);
    if (!cell.isPartOfMerge()) {
      subject = cell.getValue();
    } else if (cell.getValue() != "") {
      subject = cell.getValue();
    } else {
      var range = cell.getMergedRanges()[0];
      return range.getDisplayValue();
    }
    return subject;
  }

  // Get editor email and return email if same G-suite domain
  function getEditorEmail() {
    var email = Session.getActiveUser().getEmail();
    return email;
  }

  // Get customer info from email
  function getCustomer(email) {
    if (email == '') {
      // Because we dont retrieve user email now > we temporary hard fix it T.T
      email = 'customer.a@example.jp';
    }
    return getItemFromArrayObject(CUSTOMER_INFO, 'email', email);
  }

  // Get offshore info from email
  function getOffshore(email) {
    return getItemFromArrayObject(OFFSHORE_INFO, 'email', email);
  }

  // Build meaning message body from edit info
  function buildMessage(url, sheet, targetSheet, editor, row, column) {
    var sheetId = sheet.getSheetId();
    var row = sheet.getActiveCell().getRow();
    var column = sheet.getActiveCell().getColumn();
    var cellUrl = getCellUrl(url, sheetId, row, column);
    var subject = getSubject(sheet, targetSheet);
    var email = getEditorEmail();
    var editor = getCustomer(email);
    var message = Utilities.formatString(CHATWORK_MESSAGE, subject, cellUrl);
    return message;
  }

  // Send message to Chatwork room via Chatwork API v2
  function sendMessage(roomId, body) {
    UrlFetchApp.fetch('https://api.chatwork.com/v2/rooms/' + roomId + '/messages', {
      'method' : 'post',
      'payload' : {
        "body" : body
      },
      'headers' : {
        'X-ChatWorkToken': CHATWORK_TOKEN
      }
    });
  }

  // Get element has property equal value in array object list
  function getItemFromArrayObject(arr, key, value) {
    for (var i = 0; i < arr.length; i++) {
      var item = arr[i];
      if (typeof(item[key]) == 'undefined') {
        continue;
      }
      if (item[key] == value) {
        return item;
      }
    }
    return false;
  }
  ```
  Nhớ `execute` function `settingLogsSpreadsheet` để tạo sheet `Logs` nhé. Khi chạy function này, script sẽ yêu cầu một số permission từ gsuite account, hay cho phép **[allow]**.
  ![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/qa_code_gs.png)

- **[3.]** Thiết lập `onEdit` installable trigger theo hướng dẫn [sau](https://developers.google.com/apps-script/guides/triggers/installable?hl=en):
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/current_project_triggers.png)
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/create_new_trigger.png)
![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/add_trigger_for_spreadsheet.png)

Sau khi hoàn tất việc thiết lập trigger, mỗi khi có một hành động chỉnh sửa file `Spreadsheet`, hệ thống sẽ thông báo tới bạn qua Chatwork nếu người chỉnh sửa đó là khách hàng.

### Vấn đề tồn tại

Trong đoạn mã ở file `Code.gs` có method:

```javascript
// Get editor email and return email if same G-suite domain
function getEditorEmail() {
  var email = Session.getActiveUser().getEmail();
  return email;
}
```

sử dụng hàm [`Session.getActiveUser().getEmail()`](https://developers.google.com/apps-script/reference/base/session?hl=en#getActiveUser()) để lấy email người chỉnh sửa, nhằm mục đích phân biệt giữa khách hàng và team offshore. Vì lý do bảo mật thông tin cá nhân, nên hàm này sẽ trả về một `blank string` nếu script được chạy mà không được sự đồng ý của người thao tác trên `Spreadsheet`, đơn giản là trong trường hợp `onOpen(e)` cũng như `onEdit(e)` ta đều không thể lấy được email người dùng. Tuy nhiên trong trường hợp người dùng cùng Gsuite domain với người phát triển script, thì hàm này lại hoạt động rất ok.
- Điều đó dẫn tới với những người cùng Gsuite domain (Offshore team) →　Lấy được email
- Điều đó dẫn tới với những người khác Gsuite domain (Customer team) →　Không lấy được email

Ở script trên tôi mặc định khi không lấy được email của Editor thì sẽ mặc định người đó là Customer :rofl:

![](/assets/img/posts/2019-10-07-productivity-tips-for-google-spreadsheet-app-scripts/it_works_just_dont_touch_it.png)
