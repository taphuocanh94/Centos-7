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

OpenNebula Front-end kết nối với các máy chủ Node thông qua SSH. Bạn cần phải phân phối *public key* của người dùng `ondeadmin` từ tất cả các máy chủ Node vào tệp `/var/lib/one/.ssh/authorized_keys` trên tất cả các máy. Có nhiều cách để thực hiện việc phân phối  các `SSH key` đến các máy, trong hướng dẫn này, chúng ta sẽ thực hiện thủ công thông qua lệnh `scp`.

Sau khi tiến hành cài đặt các gói trên Front-end, một SSH key được tạo và lưu trữ tại file `authorized_keys`. Chúng ta sẽ đồng bộ 
các file `id_rsa`, `id_rsa.pub` và `authorized_keys` từ Front-end đến các máy chủ node. Ngoài ra, chúng ta cần tạo file `known_host` và đồng bộ nó cùng với tất các các máy node. Để tạo file `known_host`, chúng ta cần thực thi lệnh này với người dùng `oneadmin` trên máy chủ Front-end với tất cả tên của các máy chủ node và tên của máy chủ Front-end như là các tham số:

```
# ssh-keyscan <frontend> <node1> <node2> ... >> /var/lib/one/.ssh/known_hosts
```
Chúng ta cũng cần kiểm tra máy chủ Front-end đã có một publick key `id_rsa` tại `/var/lib/one/.ssh/` chưa. Nếu chưa hay thực hiện lệnh:

```
# ssh-keygen -t rsa -b 4096 -C "your_email@domain.com"
```

Nhấn `Enter` để nhập đường dẫn sẽ chứa file `id_rsa` được tạo ra.

```
Enter file in which to save the key (/var/lib/one/.ssh/id_rsa):
```

Mặc định khi thực thi lệnh với người dùng oneadmin, đường dẫn sẽ là, `/var/lib/one/.ssh/id_rsa`. Nếu không đúng đường dẫn đó, bạn cần nhập lại chính xác như trên.

Bây giừo chúng ta cần copy thư mục `/var/lib/one/.ssh` đến tất cả các máy chủ node. Phương pháp dễ nhất là đặt một mật khẩu tạm thời cho tài khoản `oneadmin` trong tất cả các máy chủ:

```
# passwd oneadmin
```

Và copy thư mục từ máy chủ Front-end sang với lệnh `scp`:

```
# scp -rp /var/lib/one/.ssh <node1>:/var/lib/one/
# scp -rp /var/lib/one/.ssh <node2>:/var/lib/one/
# scp -rp /var/lib/one/.ssh <node3>:/var/lib/one/
# ...
```

Bạn nên kiểm tra lại việc đăng nhập băng người dùng `oneadmin` từ máy chủ Front-end đến các máy chủ node và đến chính máy chủ Front-end, và từ các máy chủ node đến máy chủ Front-end, nếu việc đăng nhập hoàn thành mà không có một lần nào yêu cầu nhập mật khẩu, nghĩa là việc cấu hình đã hoàn thành. Thực hiện như sau:

```
# ssh <frontend>
# exit
# ssh <node1>
# ssh <frontend>
# exit
# exit
# ssh <node2>
# ssh <frontend>
# exit
# exit
# ssh <node3>
# ssh <frontend>
# exit
# exit
```

> Hãy nhớ rằng bạn cần phải chạy `ssh-agent` với *private key* tương ứng được nhập trước khi OpenNebula được khởi động.
> Bạn có thể bắt đầu `ssh-agent` bằng cách chạy và thêm _private key_ với lệnh:
>
> ```
> # eval "$(ssh-agent -s)"ssh-add /var/lib/one/.ssh/id_rsa
> ```

## Bước 5: Cấu hình mạng

![Network](http://docs.opennebula.org/5.8/_images/network-02.png)

Chúng ta cần một trình kết nối mạng đến máy chủ Front-end của OpenNebula để quản lý, giám sát các máy chủ node và truyền các file Images. Để có được sự ổn định cao, khuyến khích các bạn dùng một cổng mạng riêng, chuyên dụng cho mục đích này.

OpenNebula hỗ trợ nhiều mô hình mạng khác nhau ([Tham khảo tại đây](http://docs.opennebula.org/5.8/deployment/open_cloud_networking_setup/overview.html#nm)).

Đơn giản nhất, bạn có thể sử dụng mô hình mạng Bridge. Đối với mô hình mạng này, bạn cần thiết lập một cầu nối birdge và bao gồm một cổng mạng vật lý vào nó. Sau này, khi xác định mạng trong OpenNebula, bạn sẽ chỉ định tên của cầu nối này và OpenNebula sẽ biết rằng nó sẽ kết nối các máy chủ ảo với cầu nối đó. 

Ví dụ: Với yêu cầu như trên, một máy chủ node điển hình cần có 2 cổng mạng vật lý, một cổng cho sẽ cho địa chỉ Public Ip (ví dụ `eth0` gắn với một NIC, cổng này sẽ dùng để giao tiếp giữa máy chủ Front-end và máy chủ node này; một cổng khác sẽ dành cho các mạng LAN ảo riêng, cụ thể nó sẽ được cắm vào cầu nối bridge được tạo trên máy chủ node để cung cấp kết nối cho bridge này với toàn hệ thống.

```
# brctl show
bridge name  bridge id           STP enabled   interfaces
br0          8000.001e682f02ac   no            eth0
br1          8000.001e682f02ad   no            eth1
```

> Chú ý, hãy nhớ rằng điều này chỉ yêu cầu trên máy chủ node, không phải trên máy chủ Front-end. Và các tên của các cầu nối bridge
> như `br0`, `br1` là không quan trọng, nhưng quan trọng là tên các cầu nối gắn với NIC để kết nối với máy chủ Front-end trên các
> máy chủ node cần phải có cùng tên.

### Cấu hình một cầu nối bridge trên máy chủ node

...


## Bước 6: Cấu hình lưu trữ

Bạn có hoàn toàn có thể bỏ qua bước này nếu bạn chỉ muốn dùng thử OpenNebula, vì nó sẽ được cấu hình mặc định theo cách nó sử dụng
bộ lưu trữ cục bộ của Front-end để lưu trữ Images và bộ lưu trữ cục bộ của bộ ảo hoá làm bộ lưu trữ cho các máy ảo đang chạy.

Tuy nhiên, nếu bạn muốn thiết lập cấu hình lưu trữ khác, như Ceph, NFS, LVM, ..., bạn nên đọc chương [Open Cloud Storage](http://docs.opennebula.org/5.8/deployment/open_cloud_storage_setup/overview.html#storage) của tài liệu hướng dẫn.

## Bước 7: Thêm máy chủ và OpenNebula

Trong bước này, chúng ta sẽ đăng ký node mà đã được cài đặt trong máy chủ Front-end, vì vậy, OpenNebula có thể khởi chạy máy ảo trong đó. Bước này có thể thực hiện trong CLI hoặc trong giao diện web Sunstone. Chỉ cần thực hiện một trong hai phương pháp vì chúng hoàn toàn giống nhau.

### Thông qua giao diện web Sunstone

Truy cập vào giao diện Sunstone như hướng dẫn cài đặt máy chủ Front-end. Trong menu bên trái, vào phần Infrastructure đến phần Host. Nhấp vào nút cộng (`+`) màu xanh lục

![Add host](http://docs.opennebula.org/5.8/_images/sunstone_select_create_host.png)

Điền tên máy chủ node vào trường Hostname

![Fill hostname](http://docs.opennebula.org/5.8/_images/sunstone_create_host_dialog.png)

Cuối cùng trở về danh sách máy chủ đợi đến khi máy chủ chuyển sang trạng thái `ON`, nó sẽ mất khoảng 20 giây đến 1 phút để hoàn thành. Hảy thử nhấp vào nút làm mới để kiểm tra trạng thái thường xuyên hơn.

![refresh](http://docs.opennebula.org/5.8/_images/sunstone_list_hosts.png)

Nếu máy chủ chuyển sang trạng thái `ERR` thay vì `ON`, hãy kiểm tra nhật ký tại file `/var/log/one/oned.log`. Rất có thể đó là một vấn đề với kết nối SSH.

### Thông qua CLI

Để thêm một máy chủ node, chạy lệnh sau dưới người dùng `oneadmin` trong Front-end:

```
# onehost create <node01> -i kvm -v kvm
# onehost list
  ID NAME            CLUSTER   RVM      ALLOCATED_CPU      ALLOCATED_MEM  STAT
   1 localhost       default     0                  -                  -  init

// After some time (20s - 1m)
# onehost list
  ID NAME            CLUSTER   RVM      ALLOCATED_CPU      ALLOCATED_MEM  STAT
   0 node01          default     0       0 / 400 (0%)     0K / 7.7G (0%)  on
```

Nếu máy chủ chuyển sang trạng thái `ERR` thay vì `ON`, hãy kiểm tra nhật ký tại file `/var/log/one/oned.log`. Rất có thể đó là một vấn đề với kết nối SSH.
