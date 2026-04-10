# B. Cài đặt Ubuntu + Docker
## Cài đặt hệ điều hành Ubuntu 24.04.4 LTS
### Sử dụng một trong các công cụ để giả lập: VirutualBox (Miễn phí).
Truy cập trang [oracle virtualbox downloads](https://www.oracle.com/latam/virtualization/technologies/vm/downloads/virtualbox-downloads.html)
Chọn bản dành cho window.
<img width="1914" height="899" alt="image" src="https://github.com/user-attachments/assets/2c5237aa-0e7d-42ee-8036-47ff911d0a44" />

Tải về và cài đặt.


### Download file ubuntu
Tải [ubuntu-24.04.4-live-server-amd64.iso](https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso) để cài đặt.

### Cấu hình mạng trong Ubuntu (và công cụ giả lập) để cho phép truy cập SSH vào Ubuntu từ cmd của windows
**Cấu hình network Virtualbox**
Chọn Network ở giao diện chính của Virtualbox. Tab host-only được mở, chọn DHCP ở dưới, bỏ tích enable server để cấu hình ip tĩnh cho các máy ảo.
<img width="1085" height="1002" alt="image" src="https://github.com/user-attachments/assets/7e261a63-2b0e-4cae-9d69-182acaa3bcd6" />

Sang tab Nat network, chuột phải vào khoảng trắng chọn Create, đặt IPv4 Prefix là 10.0.2.0
<img width="1087" height="963" alt="image" src="https://github.com/user-attachments/assets/3a07ebbd-bf3d-4606-97b7-7bd2bb263fc8" />

Xong cấu hình mạng Virtualbox.
**Cấu hình network Ubuntu**
*Bước 1: Trước hết cần tạo máy ảo Ubuntu.*

Tạo 1 máy ảo, bấm Create a new virtual machine.
<img width="546" height="502" alt="vm" src="https://github.com/user-attachments/assets/df1d3d18-ca8e-413c-8b09-17b5dc91b2c0" />

Đặt tên máy ảo là Ubuntu, chọn file iso là file ubuntu-24.04.4-live-server-amd64.iso vừa tải về.
Tới bước tiếp theo, đặt mật khẩu và tên user name tuỳ ý(trong bài này em đặt là admin).
Bước tiếp theo là cấu hình phần cứng tuỳ ý, để phù hợp em chọn: Bộ xử lý 2 nhân, Bộ nhớ 4GB, Disk 25GB.
Sau khi tạo ta được máy ảo có cấu hình như hình:
<img width="1091" height="938" alt="image" src="https://github.com/user-attachments/assets/fa48659e-6db7-44f6-9c1d-146f11582d23" />

*Bước 2: Cấu hình network cho Ubuntu.*

Chọn Setting (bánh răng màu vàng cam). Chọn Network.
Ở Adapter 1: Chọn Attached to: Nat network, Name: NatNetwork vừa tạo.
<img width="1177" height="557" alt="image" src="https://github.com/user-attachments/assets/121d67fc-e5db-44d2-97e5-e3a462635d1a" />

Ở  Adapter 2: Chọn Attached to: Host-only Adapter, Name mặc định
<img width="1193" height="535" alt="image" src="https://github.com/user-attachments/assets/a77acbc7-9ff4-4a22-a1db-2868ff3db903" />

*Bước 3: Cấu hình ip tĩnh cho Ubuntu.*

Trước hết, strart Ubuntu. Chờ khoảng 10p cho nó chạy các file setup. Sau đó gõ 
`reboot` để khởi động lại.

Sau khi reboot, thì sẽ thiết lập login và password.

Tiếp theo gõ
`sudo nano /etc/netplan/*.yaml`

Để mở file 50-cloud-init.yaml (đây là file cấu hình mạng mặc định được tạo khi mà đã tạo thành công máy ảo ubuntu); Sau đó gõ đoạn code sau.

```
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true

    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.56.10/24
```
Lưu file lại dùng tổ hợp phím Ctrl+O, enter để lưu, Crtl+X để thoát.

Cuối cùng gõ lệnh `sudo netplan apply` để đưa cấu hình mạng mới thiết lập đi vào hoạt động.

*Bước 4: Check ip Ubuntu.*

Gõ lệnh `ip -4 addr` để kiểm tra:

- Ta thấy có ip 192.168.56.10/24 trong enp0s8 là ip Ubuntu đã đáp ứng điều kiện cần cho ssh.
- Ta thấy có ip 10.0.2.x/24 trong enp0s3 là đã sẵn sàng kết nối internet. Ta có thể kiểm tra bằng cách ping tới google.com bằng lệnh `ping 8.8.8.8`. Các gói tin gửi đi như hình là ok.
<img width="968" height="965" alt="image" src="https://github.com/user-attachments/assets/36ef1fbc-4d0b-4a6c-be6d-df971d6058f4" />

*Bước 5: Cài SSH cho Ubuntu.*

Ta gõ lần lượt các lệnh:
`
sudo apt update
sudo apt install openssh-server -y
`

Có thể kiểm tra ssh đã cài thành công chưa bằng lệnh `sudo systemctl status ssh` ra như hình là ok. Đừng lo khi thấy inactive(dead), do chưa có client nào kết nối đến nên nó hiện vậy.
<img width="640" height="457" alt="cài ssh" src="https://github.com/user-attachments/assets/99d7c41f-70ee-418e-a297-801d29b1eb46" />

Sau khi đá có SSH server rồi thì ta có thể làm việc trên CMD của window sẽ dễ dàng hơn (nó cho phép cuộn chuột, zoom to nhỏ, copy bằng Ctrl+C và pate bằng Chuột phải).

Mở CMD trên window gõ lệnh `ssh admin@192.168.56.10` (lưu ý, username của Ubuntu đặt là gì thì sẽ thay admin=cái đó). Sau đó nó bắt chọn yes/no thì gõ yes (Cần tắt gõ tiếng việt), tiếp theo nó bắt nhập mật khẩu, ta nhập mật khẩu đá đặt cho Ubuntu rồi enter là ok. Màn hình chào mừng của Ubuntu hiện ra.
<img width="1350" height="743" alt="image" src="https://github.com/user-attachments/assets/531c0928-c8d3-48a2-96d7-1779418b545c" />

Khi này ta có thể check lại lệnh `sudo systemctl status ssh` sẽ thấy nó active ngon lành.
<img width="965" height="379" alt="image" src="https://github.com/user-attachments/assets/af7f5ab7-410c-4527-995f-ceea849a2766" />

Ok. Thiết lập mạng xong!
## Tìm hiểu các lệnh cơ bản của ubuntu
Các lệnh cần tìm hiểu:

Liệt kê các file trong thư mục: ls
Tạo thư mục: mkdir nameFolder
Chuyển thư mục làm việc: cd path
Copy file: cp file_nguồn path/file_đích
Thay đổi quyền thao tác file: sudo chmod xxx filename
Edit file: sudo nano tenfile
CTRL+o : lưu nội dung sau khi edit
CTRL+x : thoát edit file
Xem ip của máy ubuntu: ip -4 addr
## Cài đặt docker cho Ubuntu
Kiểm tra phiên bản docker vừa cài đặt, kiểm tra phiên bản của docker compose
Cấu hình để docker chạy mà không cần tiền tố sudo
Tìm hiểu tập lệnh của docker và docker compose
Đảm bảo tường lửa trên Ubuntu đã cho phép các cổng 80, 1880, 9630 (Lệnh: sudo ufw allow ...)
