# Cài đặt Front-end

## Bước 1: Tắt SELinux trên CentOS/RHEL 7

SELinux có thể chặn một số hoạt động do OpenNebula Front-end khởi xướng, gây ra lỗi. 
Nếu quản trị viên không có kinh nghiệm về cấu hình SELinux, bạn nên tắt chức năng này để tránh các lỗi không mong muốn. 
Bạn có thể bật SELinux bất cứ lúc nào sau này khi bạn hoàng thành cài đặt và cài đặt hoạt động.

### Vô hiệu hoá SELinux (được khuyến nghị)

Thay đổi dòng sau trong file `/etc/selinux/config` để vô hiệu hoá SELinux:

```
SELINUX=disabled
```

Sau khi thay đổi, bạn cần phải khởi động lại máy.

```
# sudo systemctl reboot
```

### Kích hoạt SELinux

Thay đổi dòng sau trong file `/etc/selinux/config` để kích hoạt SELinux với trạng thái `enforcing`: 

```
SELINUX=enforcing
```

Khi thay đổi từ trạng thái `disable`, cần phải kích hoạt lại tệp hệ thống ở lần khởi động tiếp theo bằng cách tạo tệp `/.autorelable`
, ví dụ:

```
$ touch /.autorelabel
```

Sau khi thay đổi, bạn nên khởi động lại máy.

> Thực hiện theo [Hướng dẫn của người dùng và quản trị viên của SELinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/) để biết thêm thông tin về cách định cấu hình và khắc phục sự
> cố của SELinux.

## Bước 2: Thêm kho lưu trữ OpenNebula

Để thêm kho lưu trữ OpenNebula, thực hiện lệnh sau với với quyền root:

```
# cat << EOT > /etc/yum.repos.d/opennebula.repo
[opennebula]
name=opennebula
baseurl=https://downloads.opennebula.org/repo/5.8/CentOS/7/x86_64
enabled=1
gpgkey=https://downloads.opennebula.org/repo/repo.key
gpgcheck=1
#repo_gpgcheck=1
EOT
```

## Bước 3: Cài đặt phần mềm

> Kích hoạt kho lưu trữ [EPEL](http://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F) (Gói bổ sung cho Enterprise Linux) trước khi cài đặt OpenNebula.
>
> Trên CentOS 7, việc bật EPEL dễ dàng như cài đặt gói cấu hình kho lưu trữ bổ sung:
>
> ```
> # yum install epel-release
> ```

Cài đặt CentOS/RHEL OpenNebula Front-end với các gói từ kho lưu trữ của OpenNebula bằng cách thực hiện lệnh sau dưới quyền root:

```
# yum install opennebula-server opennebula-sunstone opennebula-ruby opennebula-gate opennebula-flow
```

##### Mô tả gói CentOS/RHEL

Các gói cho OpenNebula Front-end và máy chủ ảo hoá như sau:

* __opennebula__: Giao diện dòng lệnh.
* __opennebula-server__: Trình nền chính của OpenNebula, trình lập lịch biểu, v.v.
* __opennebula-sunstone__: [Sunstone](#) (GUI) và [API EC2](#).
* __opennebula-ruby__: Ruby Bindings.
* __opennebula-java__: Ràng buộc Java.
* __python-pyone__: Ràng buộc Python.
* __opennebula-gate__: Máy chủ [OneGate](#) cho phép giao tiếp giữa VM và OpenNebula.
* __opennebula-flow__: [OneFlow](#) quản lý các dịch vụ và sự mềm dẻo.
* __opennebula-node-kvm__: Gói meta cài đặt người dùng oneadmin, libvirt và kvm.
* __opennebula-common__: Các tệp phổ biến cho các gói OpenNebula.

> Các tập tin cấu hình thường được đặt trong `/etc/one` và `/var/lib/one/remotes/etc`.

## Bước 4: Cài đặt Ruby Runtime

Một số thành phần OpenNebula cần thư viện Ruby. OpenNebula cung cấp một tập lệnh mà các cài đặt yêu cầu gems cũng như
một vài gói thư viện phát triển cần thiết.

Thực hiện lệnh sau dưới quyền root:

```
# /usr/share/one/install_gems
```

Script trước đó được chuẩn bị để phát triển các bản phân phối Linux phổ biến và cài đặt các thư viện cần thiết.
Nếu không tìm thấy các gói cần thiết trong hệ thống của bạn, hãy cài đặt thủ công các gói này:

* sqlite3 development library
* mysql client development library
* curl development library
* libxml2 and libxslt development libraries
* ruby development library
* gcc and g++
* make

Nếu bạn muốn chỉ cài đặt một bộ gems cho một thành phần cụ thể, hãy đọc [Xây dựng từ mã nguồn](http://docs.opennebula.org/5.8/integration/references/compile.html#compile)
nơi nó được giải thích sâu hơn.

## Bước 5: Cài đặt và kích hoạt MySQL/MariaDB (Tuỳ chọn)

Bạn có thể bỏ qua bước này nếu bạn chỉ muốn triển khai OpenNebula càng nhanh càng tốt, vì mặc định OpenNebula sẽ sử dụng SQLite
làm cơ sở dữ liệu. Tuy nhiên nếu bạn đang triển khai phần này để sản xuất, kinh doanh, hoặc trong một môi trường quan trọng hơn, hãy đảm bảo thực hiện bước này để tạo cơ sở dữ liệu với MySQL/MariaDB.

> Lưu ý rằng, chúng ta có thể chuyển từ SQLite sang MySQL/MariaDB, nhưng vì nó là khó và cồng kềnh để thực hiện và hoàn thành việc
> chuyển cơ sở dữ liệu, nên nếu không chắc chắn bạn đủ khả năng thực hiện, hãu sử dụng MySQL/MariaDB ngay từ đầu.

### Cài đặt

Trước hết, bạn cần một máy chủ MySQL/MariaDB hoạt động. Bạn có thể triển khai một bản để cài đặt OpenNebula hoặc sử dụng lại mọi
bản cài đặt có sẵn để triển khai và truy cập cho OpenNebula Front-end.

Nếu bạn cần cài đặt mới, thực hiện các lệnh sau:

```
# sudo yum -y install mariadb-server mariadb
# sudo systemctl enable mariadb
# sudo systemctl start mariadb
```

Thiết lập mật khẩu của tài khoản root cho MariaDB sử dụng lệnh sau:

```
# sudo mysql_secure_installation
```

#### Cấu hình MySQL

Bạn cần tạo cơ sở dữ liệu mới, ví dụ `opennebula`, thêm người dùng mới và cấp đặc quyền đối với cơ sở dữ liệu này:

```
# mysql -u root -p
Enter password:
Welcome to the MySQL monitor. [...]

mysql> CREATE DATABASE opennebula;
Query OK, 1 row affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin' IDENTIFIED BY '<thepassword>';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```

#### Cấu hình OpenNebula

Trước khi bạn chạy OpenNebula lần đầu tiên, bạn cần thiết lập cấu hình tại `/etc/one/oned.conf` với các chi tiết kết nối vào cơ sở
dữ liệu mà bạn đã tạo và cấp đặc quyền.

Chỉnh sửa dòng sau

```
DB = [ BACKEND = "sqlite" ]
```
sang

```
# DB = [ BACKEND = "sqlite" ]
# Cấu hình mẫu cho MySQL 
DB  =  [  backend  =  "mysql" , 
       server   =  "localhost" , 
       port     =  0 , 
       user     =  "oneadmin" , 
       passwd   =  "<thepassword>" , 
       db_name  =  "opennebula"  ]
```

Các trường:

* __server__: Địa chỉ máy chủ MySQL.
* __port__: Cổng kết nối đến máy chủ MySQL. Nếu đặt là 0, cổng mặc định sẽ được sử dụng.
* __user__: MySQL user-name.
* __passwd__: MySQL password.
* __db_name__: Tên của cơ sở dữ liệu mà OpenNebula sẽ sử dụng.

## Bước 6: Bắt đầu OpenNebula

> Nếu bạn đang thực hiện nâng cấp, hãy bỏ qua bước này và các bước tiếp theo và quay lại tài liệu hướng dẫn nâng cấp.

Cấu hình đăng nhập người dùng `oneadmin` theo các bước sau:

Tệp tin `/var/lib/one/.one/one_auth` sẽ được tạo với một mật khẩu đăng nhập được tạo ngẫu nhiên cho tài khoản `oneadmin`. Nội dung tệp tin sẽ là `oneadmin:<password>`. Vui lòng thay đổi mật khẩu trước khi bắt đầu OpenNebula. Ví dụ, thực hiện lệnh sau:

```
# echo "oneadmin:mypassword" > /var/lib/one/.one/one_auth
```

> Điều này sẽ đặt mật khẩu cho tài khoản `oneadmin` tại lần khởi động đầu tiên. 
> Từ thời điểm đó, bạn phải sử dụng lệnh `passwd oneuser` để thay đổi mật khẩu của `oneadmin`.
> Thông tin thêm về các thay đổi mật khẩu của `oneadmin` [tại đây](http://docs.opennebula.org/5.8/operation/users_groups_management/manage_users.html#change-credentials)

Bây giờ, máy chủ của bạn đã sẵn sàng để bắt đầu OpenNebula. Bạn có thể sử dụng lệnh `systemctl` như sau:

```
# systemctl start opennebula opennebula-sunstone
```

Hoặc sử dụng lệnh `service` như sau:

```
# service opennebula start
# service opennebula-sunstone start
```

## Bước 7: Xác minh cài đặt

Sau khi OpenNebula được khởi động lần đầu tiên, bạn nên kiểm tra xem các lẹnh có thể kết nối với OpenNebula daemon được hay không. Bạn có thể thực hiện trong Linux CLI hoặc với giao diện người dùng Sunstone.

### Trong Linux CLI

Tại Font-end, hãy chạy lệnh sau dưới dạng tài khoản oneadmin:

```
# oneuser show
USER 0 INFORMATION
ID              : 0
NAME            : oneadmin
GROUP           : oneadmin
PASSWORD        : 3bc15c8aae3e4124dd409035f32ea2fd6835efc9
AUTH_DRIVER     : core
ENABLED         : Yes

USER TEMPLATE
TOKEN_PASSWORD="ec21d27e2fe4f9ed08a396cbd47b08b8e0a4ca3c"

RESOURCE USAGE & QUOTAS
```

Nếu bạn nhận được một thông báo lỗi, thì OpenNebula daemon đã không thể khởi động đúng cách:

```
# oneuser show
Failed to open TCP connection to localhost:2633 (Connection refused - connect(2) for "localhost" port 2633)
```

Nhật ký OpenNebula được đặt trong thư mục tại `/var/log/one`, bạn nên coi ít nhất là các tệp nhật ký lõi `oned.log` và lịch trình `sched.log`. Trong file `oned.log`, các thông báo lỗi được đánh dấu bằng tiền tố `[E]`.

### Với giao diện người dùng Sunstone

Bây giờ, bạn có thể thử đăng nhập vào giao diện web của Sunstone. Để làm điều này, điều huóng trình duyêt đến địa chỉ
`http://<frontend_ip_address>:9869`. Nếu mọi thứ đều ổn, bạn sẽ nhìn thấy một form đăng nhập. Hãy thử đăng nhập bằng tên người dùng là `oneadmin` và mật khẩu bạn đã nhập trong tệp `/var/lib/one/.one/one_auth` trong Font-end của bạn.

Nếu trang không tải được, hãy kiểm tra nhật ký lỗi tại `/var/log/one/sunstone.log` và `/var/log/one/sunstone.error`. Ngoài ra, đảm bảo rằng cổng TCP 9869 được cho phép thông qua bởi tường lửa.

#### Với `firewall`, thực hiện như sau:

Kiểm tra xem dịch vụ `firewall` có được kích hoạt hay không:

```
# sudo firewall-cmd --state
running
```

Nếu bạn nhận được kết quả là *running*, nghĩa là dịch vụ được kích hoạt, hãy mở cổng `9869` bằng các lệnh sau:

```
# sudo firewall-cmd --add-port=9869/tcp --permanent
# sudo firewall-cmd --reload
```

## Cấu trúc thư mục

Bảng sau liệt kê một số đường dẫn đáng chú ý có sẵn trong Front-end của bạn sau khi cài đặt:


|                 Path                |                                     Description                                      |
|-------------------------------------|--------------------------------------------------------------------------------------|
| `/etc/one/`                         | Các file cấu hình                                                                    |
| `/var/log/one/`                     | Các file nhật ký, đáng chú ý: `oned.log`, `sched.log`, `sunstone.log` và `<vmid>.log`|
| `/var/lib/one/`                     | Thư mục gốc tài khoản `oneadmin`                                                     |
| `/var/lib/one/datastores/<dsid>/`   | Lưu trữ kho dữ liệu                                                                  |
| `/var/lib/one/vms/<vmid>/`          | Các file hoạt động cho VMs (deployment file, transfer manager scripts, etc...)       |
| `/var/lib/one/.one/one_auth`        | Thông tin xác thực tài khoản `oneadmin`                                              |
| `/var/lib/one/remotes/`             | Probes and scripts that will be synced to the Hosts                                  |
| `/var/lib/one/remotes/hooks/`       | Hook scripts                                                                         |
| `/var/lib/one/remotes/vmm/`         | Virtual Machine Manager Driver scripts                                               |
| `/var/lib/one/remotes/auth/`        | Authentication Driver scripts                                                        |
| `/var/lib/one/remotes/im/`          | Information Manager (monitoring) Driver scripts                                      |
| `/var/lib/one/remotes/market/`      | MarketPlace Driver scripts                                                           |
| `/var/lib/one/remotes/datastore/`   | Datastore Driver scripts                                                             |
| `/var/lib/one/remotes/vnm/`         | Networking Driver scripts                                                            |
| `/var/lib/one/remotes/tm/`          | Transfer Manager Driver scripts                                                      |

## Cấu hình tường lửa

Danh sách các cổng được sử dụng bởi OpenNebula, các cổng này cần được mở để OpenNebula có thể làm hoạt động tốt:

|                 Port              |                     Description                                                          |
|-----------------------------------|------------------------------------------------------------------------------------------|
| `9869`                            | Sunstone server.                                                                         |
| `29876`                           | VNC proxy port, used for translating and redirecting VNC connections to the hypervisors. |
| `2633`                            | OpenNebula daemon, main XML-RPC API endpoint.                                            |
| `2474`                            | OneFlow server. This port only needs to be opened if OneFlow server is used.             |
| `5030`                            | OneGate server. This port only needs to be opened if OneGate server is used.             |

OpenNebula kết nối với các trình ảo hoá thông qua ssh (cổng 22). Ngoài ra, `oned` có thể kết nối với [Chợ tài nguyên của OpenNebula](https://marketplace.opennebula.systems/) và [kho chứa các ảnh tạo sẵn của Linux](https://images.linuxcontainers.org)
để có danh sách các thiết bị khả dụng. Bạn nên mở các kết nối gửi đến các cổng giao thức này. Lưu ý: Đây là các cổng mặc định, mỗi thành phần có thể được cấu hình để liên kết với các cổng cụ thể hoặc sử dụng Proxy HTTP.
