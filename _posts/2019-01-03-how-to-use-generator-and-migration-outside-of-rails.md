---
layout: post
title:  "Sử dụng rails console generator và rails migration độc lập cho dự án (non-Rails)"
date:   2019-01-03 07:23:04 +0700
categories: Rails
tags:
  - Active record
hide_thumbnail: true
image: /assets/img/posts/2019-01-03-how-to-use-generator-and-migration-outside-of-rails/migrations-in-rails.webp
---

Khi lập trình với Go thì mình không có gì để phàn nàn ngoài 1 điều là Go không support sẵn các thư viện hỗ trợ lập trình viên tiết kiệm thời gian cho việc tạo `migration/schema`.

Dạo 1 vòng quanh github search các thư viện cho Go có thể gặp

- [golang-migrate](https://github.com/golang-migrate/migrate)
- [gorm](https://github.com/jinzhu/gorm)
- [goose](https://github.com/pressly/goose)

Nhưng tất cả đều rất khó sử dụng và yêu cầu lập trinh viên phải write `up/down` migration bằng SQL instruction.

Trong khi đó, ở bên kia thế giới `Ruby on Rails` cũng như `Laravel/Symphony` lại hỗ trợ công việc này đến tận răng.

- `Laravel/Symphony` cung cấp sẵn một list command sau:
```
$ php artisan list migrate
...
Available commands for the "migrate" namespace:
  migrate:install   Create the migration repository
  migrate:refresh   Reset and re-run all migrations
  migrate:reset     Rollback all database migrations
  migrate:rollback  Rollback the last database migration
  migrate:status    Show the status of each migration
```
- `Rails` cũng tương tự
```
$ rails console
Running via Spring preloader in process 21714
Loading development environment (Rails 5.2.2)
irb(main):001:0> require 'rake'
=> true
irb(main):002:0> Rails.application.load_tasks; 0
=> 0
irb(main):003:0> Rake::Task.tasks.each do |task|
irb(main):004:1* if 'db:'.in? task.name
irb(main):005:2> puts task.name
irb(main):006:2> end
irb(main):007:1> end; 0
db:_dump
db:abort_if_pending_migrations
db:charset
db:check_protected_environments
db:collation
db:create
db:create:all
db:drop
db:drop:_unsafe
db:drop:all
db:environment:set
db:fixtures:identify
db:fixtures:load
db:forward
db:load_config
db:migrate
db:migrate:down
db:migrate:redo
db:migrate:reset
db:migrate:status
db:migrate:up
db:purge
db:purge:all
db:reset
db:rollback
db:schema:cache:clear
db:schema:cache:dump
db:schema:dump
db:schema:load
db:schema:load_if_ruby
db:seed
db:setup
db:structure:dump
db:structure:load
db:structure:load_if_sql
db:test:load
db:test:load_schema
db:test:load_structure
db:test:prepare
db:test:purge
db:version
=> 0
irb(main):008:0>
```

Vậy nếu muốn sử dụng các công cụ này cho project non-PHP hay non-Rails thì sao ? điều này hoàn toàn không hề khó, bạn có thể quăng hẳn nguyên bộ scaffolder khi dùng `laravel new [app]` hay `rails new [app]` vào thư mục dự án chính. Tuy nhiên dự án chính sẽ trở nên rất cồng kềnh.

Một cách khác, mình sẽ tách phần migrate và generate migration ra bên ngoài.

Với `PHP` thay vì phải wrap lại `Illuminate\Database\Migrations` ta có một cách tương đối dễ, đó là dùng luôn [doctrine/migrations](https://www.doctrine-project.org/projects/migrations.html). Support tận răng 😭 Hoặc nếu bạn muốn sử dụng lại cũng khá dễ, chỉ cần tạo 1 file `#!/usr/bin/env php > .php` trong đó khai báo các command muốn dùng và switch, cách sử dụng cũng đc ghi rất rõ ràng trên github:
[Illuminate Database](https://github.com/illuminate/database#usage-instructions). Hoặc bạn có thể dùng sẵn hàng của người Việt mình tại đây: [https://github.com/xuanquynh/standalone-laravel-database](https://github.com/xuanquynh/standalone-laravel-database) 👍

Với `Rails` thì với một người không rành `Ruby` như mình quả thực đây là một bài toán khó. Tuy nhiên sau khi chày cối học một chút syntax của `Rails\Rake` thì mần vào core của Active Record > [database_tasks.rb](https://github.com/rails/rails/blob/fc2684c9c012b95ce003cce22b378d5ea9ab56d3/activerecord/lib/active_record/tasks/database_tasks.rb), [databases.rake](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/railties/databases.rake). Mình đã tách đc bề nổi thành công 😆

Nếu bạn muốn xem mình đã implement thế nào thì coi github repo này nhé 😄

⭐️ [https://github.com/euclid1990/rails-migration](https://github.com/euclid1990/rails-migration)

DONE :rocket:
