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

