# Cài đặt Node KVM

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

Thực hiện các lệnh sau để cài đặt gói node và khởi động lại libvirt để sử dụng tệp cấu hình được cung cấp bởi OpenNebula:

```
# sudo yum install -y opennebula-node-kvm
# sudo systemctl restart libvirtd
```

Để biết thêm về việc cấu hình, bạn có thể truy cập hướng dẫn cụ thể [cấu hình KVM](http://docs.opennebula.org/5.8/deployment/open_cloud_host_setup/kvm_driver.html#kvmg).

## Bước 4: Cấu hình truy cập SSH mà không sử dụng mật khẩu

OpenNebula Front-end kết nối với các máy chủ Node thông qua SSH. Bạn cần phải phân phối *public key* của người dùng `ondeadmin` từ tất cả các máy chủ Node vào tệp `/var/lib/one/.ssh/authorized_keys` trên tất cả các máy.
