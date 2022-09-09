---
layout: post
title: "Nâng cấp phiên bản AWS RDS MySQL từ 5.7 lên 8.0"
date: 2022-09-09 07:49:04 +0700
categories: Migration
tags:
  - Real Life
  - AWS
  - RDS
hide_thumbnail: true
image: /assets/img/posts/2022-09-09-upgrade-aws-rds-mysql-from-57-to-80/thumbnail.png

---

Đã hơn 5 tháng trời trôi qua không viết một bài nào trên Blog, và mình đã quay trở lại rồi đây 😂

Mình sẽ khởi động một chuỗi các bài viết mang tính thực chiến hơn là hướng dẫn chi tiết. Dù cho phương pháp trong bài viết có thể không là tối ưu hay cao siêu nhất ... nhưng chắc chắn không phải kiểu bài viết chưa xài trên môi trường thực tế đã đem đi khè người nghe, khiến người đọc bị Bias theo suy nghĩ của tác giả 🤣

Nội dung bài viết có thể không phải 100% là của mình mà là của anh em đồng nghiệp, bạn bè gần xa để lại. Và toàn bộ kiến thức này được sử dụng trong dự án thực (*nghe vậy thôi chứ anh em cũng đừng tin tưởng 100%* 😆)

Để mở đầu, mình sẽ chia sẻ tới các bạn một issue mà bạn [Trần Văn Tuấn](https://ttuan.xyz/) đồng nghiệp của mình đã điều tra cùng khách nhằm giải quyết bài toán nâng cấp phiên bản **MySQL** từ **5.7** lên **8.0** trên Production (sản phẩm hơn 30 triệu end-users) sử dụng **Amazon Relational Database Service (RDS)**. Nội dung bài viết này mình đã Remix lại một chút :joy:

---

## 1. Vấn đề

Hệ thống hiện tại sử dụng Amazon Relational Database Service (RDS) phiên bản MySQL 5.7 và Khách hàng mong muốn nâng cấp lên phiên bản MySQL 8.0

Mỗi phiên bản của không chỉ **MySQL** mà đa số các phần mềm đều có định dạng như sau `MAJOR.MINOR.PATCH`, do đó việc upgrade MySQL `5.7` → `8.0`, đồng nghĩa với việc bạn thực hiện một MAJOR upgrade. Dĩ nhiên theo như tài liệu từ chính chủ thì:

> Major version upgrades can contain database changes that are not backward-compatible with existing applications. As a result, you must manually perform major version upgrades of your DB instances. [[1]](#references)

→ Tin buồn, bạn buộc phải upgrade bằng tay - upgrade manually ✋

> For major version upgrades, you must manually modify the DB engine version through the AWS Management Console, AWS CLI, or RDS API. [[2]](#references)

→ Và nếu như đang sử dụng AWS RDS, sẽ có những công cụ sau để nâng cấp phiên bản **MySQL**: AWS Console, AWS CLI hoặc RDS API. Và để giảm thiểu rủi ro thì việc sử dụng AWS CLI được khuyến khích hơn cả.

Cho dù dùng công cụ nào bạn cũng sẽ phải đương đầu với bài toán **Downtime** trên **Production**, như vậy nhiệm vụ của chúng ta ngoài việc đảm bảo hệ thống chạy ổn định trên phiên bản mới, còn phải giảm thiểu việc dịch vụ ngừng hoạt động khi đang tiến hành nâng cấp hệ thống.

## 2. Giải pháp

Có 2 hướng tiếp cận để nâng cấp MySQL engine version trên AWS RDS:

### Hướng 1

**Upgrade Directly**, nâng cấp trực tiếp DB instance hiện tại từ MySQL v5.7 sang v8.0

### Hướng 2

**Use a Read Replica** để nâng cấp, mục tiêu giảm Downtime của MySQL Database:

- Tạo một read replica từ master DB instance.
- Upgrade MySQL engine version của read replica instance.
- Promote read replica instance sang standalone DB instance → **New** master DB instance.
- Đổi DNS endpoint từ RDS endpoint hiện tại sang master DB instance's endpoint mới.

Cho dù chọn cách nào thì bạn cũng cần chuẩn bị và diễn tập trước khi thao tác trên môi trường người dùng 😎

## 3. Chuẩn bị

### 3.1. Prechecks trước khi nâng cấp MySQL 5.7 lên 8.0

> MySQL 8.0 includes a number of incompatibilities with MySQL 5.7. These incompatibilities can cause problems during an upgrade from MySQL 5.7 to MySQL 8.0. So, some preparation might be required on your database for the upgrade to be successful. [[1]](#references)

Để đảm bảo việc nâng cấp suôn sẻ thì bạn cần đảm bảo các yêu cầu sau:

- Không có tables nào sử dụng các kiểu dữ liệu và hàm đã lỗi thời. Nếu có thì bạn cần sử dụng `REPAIR TABLE` trước khi nâng cấp sang bản 8.0.
- Không tồn tại các orphan `*.frm` files, keyword violations
- Trigger nếu có phải đảm bảo không bị thiếu
- Chi tiết hơn anh em xem tại [đây](https://dev.mysql.com/doc/refman/8.0/en/upgrade-prerequisites.html).

Khi chúng ta bắt đầu upgrade thì AWS RDS sẽ thực hiện việc prechecks tự động để xác định các không tương thích. Tất nhiên nếu tồn tại, thì RDS sẽ ngăn ngừa việc upgrade và cho chúng ta một mả log để điều tra.

Cách chuẩn bị tốt nhất là chúng ta nên tạo test instance từ staging db snapshot, sau đó thử tiến hành upgrade, get log file xem có chỗ nào không tương thích không và tiến hành fix trên local trước.

MySQL cũng có sẵn các hướng dẫn xử lý vấn đề mà chúng ta có thể tham khảo:

- [Upgrading MySQL](https://dev.mysql.com/doc/refman/8.0/en/upgrading.html)
- [Upgrading to MySQL 8.0? Here is what you need to know](https://dev.mysql.com/blog-archive/upgrading-to-mysql-8-0-here-is-what-you-need-to-know/)

### 3.2. Bật sao lưu dự phòng

Hãy đảm bảo rằng tính năng Auto backup được kích hoạt (hoặc giá trị của `backup retention period` ≥ 0).

> Amazon RDS takes two DB snapshots during the upgrade process. The first DB snapshot is of the DB instance before any upgrade changes have been made. If the upgrade doesn't work for your databases, you can restore this snapshot to create a DB instance running the old version. The second DB snapshot is taken when the upgrade completes.

> Amazon RDS only takes DB snapshots if you have set the backup retention period for your DB instance to a number greater than 0. To change your backup retention period, see [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html). [[1]](#references)

Nếu option này không được kích hoạt thì chúng ta sẽ cần tạo snapshot thủ công.

> A snapshot is taken (if backups are enabled) while the instance is still running on the previous version. If Amazon RDS doesn't find any recent backups, a full snapshot is taken during the upgrade process, which can impact the overall upgrade time. [[3]](#references)

Việc chủ động tạo một bản snapshot trước khi upgrade sẽ giúp giảm thời gian upgrade đi đáng kể.

Chúng ta có thể dùng aws-cli với câu lệnh sau:

```bash
aws rds create-db-snapshot \
    --db-instance-identifier $db_instance_name \
    --db-snapshot-identifier $db_snapshot_name
```

### 3.3. Một số điểm cần lưu ý khác

1. Upgrade DB instance OS (Optional)

 > To perform a major version upgrade for a MySQL version 5.6 DB instance on Amazon RDS to MySQL version 5.7 or later, first perform any available OS updates. After OS updates are complete, upgrade to each major version: 5.6 to 5.7 and then 5.7 to 8.0. MySQL DB instances created before April 24, 2014, show an available OS update until the update has been applied. For more information on OS updates, see [Applying updates for a DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Maintenance.html#USER_UpgradeDBInstance.OSUpgrades). [[1]](#references)

 Sẽ tốt hơn nếu hệ điều hành của DB instance được cập nhật.

2. Recheck targets which are needed to be upgraded
 > If your MySQL DB instance is using read replicas, you must upgrade all of the read replicas before upgrading the source instance.
 >
 > If your DB instance is in a Multi-AZ deployment, both the primary and standby replicas are upgraded. Your DB instance will not be available until the upgrade is complete.

 Chúng ta sẽ cần phải nâng cấp Read Replicas instances và DB instances ở cả các AZ khác.

## 4. Tiến hành Upgrade

**Highlight Recommend: [Testing an upgrade](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.MySQL.html#USER_UpgradeDBInstance.MySQL.UpgradeTesting) trước khi bạn quyết định sử dụng phương pháp nào**

### 4.1. Upgrade Directly

- Sử dụng command sau:

  ```sh
  aws rds modify-db-instance \
      --db-instance-identifier $db_instance_name \
      --engine-version $new_mysql_version \
      --allow-major-version-upgrade \
      --apply-immediately
  ```

- Thời gian Downtime

  > Database engine upgrades require downtime. The duration of the downtime varies based on the size of your DB instance.
  >
  > MySQL major version upgrades typically complete in about 10 minutes. Some upgrades might take longer because of the DB instance class size or because the instance doesn't follow certain operational guidelines in Best practices for Amazon RDS. [[1]](#references)

  Chạy thử với free-tier db instance `db.t3.micro`, mất khoảng ~12 minutes để upgrade thành công.

  Tuy nhiên trong quá trình upgrading, db instance sẽ bị shuts down ở một chế độ đặc biệt có tên là `a slow shutdown` nhằm đảm bảo tính nhất quán (consistency). Chính vì lý do đó mà trong 5 phút đầu của quá trình upgrade, ứng dụng của chúng ta vẫn có thể kết nối được tới DB.

- Xác nhận

  Để đảm bảo DB instance đã được upgrade lên đúng phiên bản mong muốn, sử dụng lệnh sau:

  ```sh
  aws rds describe-db-instances \
    --db-instance-identifier $db_instance_name \
    --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,DBInstanceStatus:DBInstanceStatus,EngineVersion:EngineVersion}'
  ```

### 4.2. Using a read replica to reduce downtime

Hướng tiếp cận này đòi hỏi DB instance phải được bật tính năng `Auto Backup`.

Các bước được dùng trong upgrade bạn có thể tham khảo ([chi tiết tại đây](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.MySQL.html#USER_UpgradeDBInstance.MySQL.ReducedDowntime)):

- Để xây dựng môi trường test chúng ta có thể làm theo các bước sau

  - Create a read replica of your MySQL 5.7 instance.

    ```sh
    read_replica_db_instance_name="read-replica-rds-upgrade-testing"

    # Create read replica
    aws rds create-db-instance-read-replica \
        --db-instance-identifier $read_replica_db_instance_name \
        --source-db-instance-identifier $db_instance_name

    # Wait util read replica db instance becomes available (~8 minutes)
    aws rds wait db-instance-available --db-instance-identifier $read_replica_db_instance_name
    ```

  - (Không bắt buộc) Khi read replica được khởi tạo và `Status` hiển thị `Available`, chuyển đổi read replica sang Multi-AZ deployment và bật tính năng backups.

  - Khi read replica `Status` hiển thị `Available`, Nâng cấp read replica sang MySQL 8.0.

    ```sh
    # Upgrade
    aws rds modify-db-instance \
        --db-instance-identifier $read_replica_db_instance_name \
        --engine-version $new_mysql_version \
        --allow-major-version-upgrade \
        --apply-immediately

    # Wait util read replica db instance becomes available (~13 minutes)
    aws rds wait db-instance-available --db-instance-identifier $read_replica_db_instance_name

    # Ensure that upgraded read replica has latest MySQL engine version
    aws rds describe-db-instances \
      --db-instance-identifier $read_replica_db_instance_name \
      --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,DBInstanceStatus:DBInstanceStatus,EngineVersion:EngineVersion}'

    ```

  - Khi quá trình upgrade hoàn thành `Status` hiển thị `Available` hãy xác nhận lại xem read replica có được cập nhật với phiên bản MySQL 5.7 DB chưa.

  - (Không bắt buộc) Bạn có thể tạo thêm read replica để tăng HA.

  - (Không bắt buộc) Cấu hình các custom DB parameter group cho read replica.

  - Promote read replica lên standalone DB instance

    ```sh
    # Promote read replica
    aws rds promote-read-replica \
        --db-instance-identifier $read_replica_db_instance_name

    # Wait util read replica db instance become available (~5 minutes)
    aws rds wait db-instance-available --db-instance-identifier $read_replica_db_instance_name

    ```

    Tuy nhiên bạn cần lưu ý sau
    > When you promote your MySQL 8.0 read replica to a standalone DB instance, it no longer is a replica of your MySQL 5.7 DB instance. We recommend that you promote your MySQL 8.0 read replica during a maintenance window when your source MySQL 5.7 DB instance is in read-only mode and all write operations are suspended. When the promotion is completed, you can direct your write operations to the upgraded MySQL 8.0 DB instance to ensure that no write operations are lost.

    > In addition, we recommend that, before promoting your MySQL 8.0 read replica, you perform all necessary data definition language (DDL) operations on your MySQL 8.0 read replica. An example is creating indexes. This approach avoids negative effects on the performance of the MySQL 8.0 read replica after it has been promoted. [[1]](#references)

  - Đổi kết nối của ứng dụng sang new MySQL 8.0 DB instance. (Update DNS endpoint or DB setting in app)

- Thời gian Downtime
  - Short downtime: Trong khi read replica instance tiến hành upgraded, ứng dụng của chúng ta vẫn có thể truy cập vào `primary` db instance. Sau quá trình upgrade tất cả các dữ liệu sẽ được đồng bộ từ `primary` sang `read replica` instance.

 > `Downtime = Time to promote read replica to standalone instance + Time to change DNS endpoint/DB setting app`.

 Thực tế với free-tier db instance, sẽ mất khoảng ~5 phút để promote read replica instance.

- Lợi ích

  Easy to rollback: Nếu chẳng may có bất kỳ vấn đề nào xảy ra trong quá trình nâng cấp, chúng ta có thể dễ dàng rollbackk lại bằng cách thay đổi DNS endpoint/DB endpoint sang `v5.7` DB instance.

#### Tham khảo thêm các bài viết về hướng tiếp cận này

* [Upgrading to RDS MySQL 8.0 with minimum downtime](https://blogs.halodoc.io/upgrading-to-rds-mysql-8-0-with-minimum-downtime/)
- [Upgrading a MySQL DB on AWS RDS](https://technologytickles.com/2019/03/24/upgrading-a-mysql-db-on-aws-rds/)
- [Our MySQL RDS Upgrade Journey: Cutting Down Downtime by 11,200% and Lessons Learned](https://medium.com/99dotco/our-mysql-rds-upgrade-journey-cutting-down-downtime-by-11200-and-lessons-learned-1fa828e6009c)
- [Zero Downtime Maintenances on MySQL RDS](https://workmarket.tech/zero-downtime-maintenances-on-mysql-rds-ba13b51103c2)

## 5. Hậu nâng cấp

### 5.1. Kiểm tra lại tiến trình nâng cấp đã hoàn thành chưa

Chúng ta có thể sử dụng câu lệnh sau để kiểm tra xem trạng thái của DB instance đã sẵn sàng chưa

```
aws rds wait db-instance-available --db-instance-identifier $db_instance_name
```

### 5.2. Retest the system

Kết nối ứng dụng vào new DB instance, và thực hiện retest hành vi hệ thống.

### 5.3. Rollback trong trường hợp upgrade failed

#### 5.3.1. Trường hợp Upgrade directly

> After the upgrade is complete, you can't revert to the previous version of the database engine. If you want to return to the previous version, restore the first DB snapshot taken to create a new DB instance. [[1]](#references)

- Chúng ta sẽ không thể rollback engine hiện tại (v8.0) về engine phiên bản trước (v5.7). Mà sẽ buộc phải upgrade latest snapshot được tạo trước khi tiến hành nâng cấp:

 ```sh
 new_db_instance_name="new-rds-upgrade-testing"

 aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier $new_db_instance_name \
  --db-snapshot-identifier $db_snapshot_name
  ```

- Cập nhật lại endpoints / Route 53 trỏ về new RDS, và connection endpoint trong ứng dụng sang new MySQL RDS instance.

#### 5.3.2. Trường hợp Upgrade using read replica

- Chúng ta chỉ cần upgrade DNS endpoint sang DB endpoint (v5.7) cũ vì DB instance này chạy hoàn toàn độc lập với RDS master instance (v8.0).

## 6. Kết luận

- Phương án 2 được khuyến khích sử dụng vì những lợi ích sau
  - Short downtime duration
  - Easy to rollback.

- Dù dùng cách gì thì chúng ta cũng cần cẩn thận kiểm tra, diễn tập trước khi tiến hành, và đây là các đầu mục lớn cần làm:
  - Prechecks for upgrades.
  - Backup data to snapshot.
  - Testing an upgrade with test DB instance. ( Tạo một snapshot từ staging db instance => Tạo một DB instance mới từ snapshot này. Thử tiến hành upgrade trên DB instance này).
  - Để hạn chế lỗi, hãy biến toàn bộ các thao tác về scripts: backup, upgrade and rollback vì làm như vậy sẽ giảm thiểu rủi ro, nhớ nhớ quên quên.
- Tính toán sơ bộ thời gian dùng để upgrade: việc này có thể phụ thuộc vào số lượng instance, kích thước database, phương án upgrade, database schema, engine, ...

## References

- [[1] AWS Official User Guide - Upgrading the MySQL DB engine](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.MySQL.html#USER_UpgradeDBInstance.MySQL.Major)
- [[2] AWS Official User Guide - Upgrading a DB instance engine version](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Upgrading.html)
- [[3] AWS Premium Support - I want to upgrade my Amazon RDS MySQL version](https://aws.amazon.com/premiumsupport/knowledge-center/rds-mysql-version-upgrade/)
- [[4] Best Practices for Upgrading Amazon RDS for MySQL and Amazon RDS for MariaDB](https://aws.amazon.com/blogs/database/best-practices-for-upgrading-amazon-rds-for-mysql-and-amazon-rds-for-mariadb/)
