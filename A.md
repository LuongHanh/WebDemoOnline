# A. Đăng ký tên miền xịn cho cá nhân
## Đăng ký domain xịn
Truy cập trang Mắt bão:
<img width="960" height="453" alt="mb-home" src="https://github.com/user-attachments/assets/49752dc0-ef31-4dd6-95c3-4b9613fb467b" />

Sau khi đăng ký, đăng nhập thành công. 
<img width="960" height="540" alt="mb-dangnhap" src="https://github.com/user-attachments/assets/9e337d60-b046-4e31-92ea-678149b1d978" />

Xác thực tài khoản eKYC trước khi đăng ký domain.
<img width="1915" height="845" alt="image" src="https://github.com/user-attachments/assets/ab496feb-53c7-4e89-8bf9-1c80c9a1801f" />

Chọn đăng ký tên miền:
<img width="960" height="450" alt="mb-dktm" src="https://github.com/user-attachments/assets/2a6de8a5-23a3-4d90-96a7-d1b8c0df51cd" />

Tìm kiếm tên miền mong muốn:
<img width="960" height="452" alt="mb-timdomain" src="https://github.com/user-attachments/assets/c540a71e-4519-484f-a852-60e2b9256d15" />

Sau khi chọn tên miền, trang đăng ký tên miền mở ra, có thể bỏ chọn bảo vệ dns. Thực hiện tuần tự các bước và thanh toán (tip: thanh toán bằng ví điện tử sẽ nhanh hơn). 
<img width="960" height="451" alt="mb-hopdong" src="https://github.com/user-attachments/assets/539a26f6-ddee-45ab-8e89-c661efbb3683" />

Sau khi ký hợp đồng, Mắt bão sẽ gửi email về máy. Chú ý là sau 10 ngày không kê khai tên miền thì domain sẽ bị tạm khoá.
<img width="960" height="452" alt="email-note" src="https://github.com/user-attachments/assets/ab7156c4-75cc-4158-af64-686fa51a4432" />

Ta thực hiện khai báo domain:
<img width="960" height="449" alt="mb-bktm" src="https://github.com/user-attachments/assets/7eb84771-af39-4cf2-8d66-c9463f76b852" />

Sau khi khai báo thành công thì ta sẽ nhận được các email thông báo từ công ty CP Mắt bão và Bộ khoa học & Công nghệ:
<img width="960" height="408" alt="email-ok" src="https://github.com/user-attachments/assets/1b4354ac-74b4-43f5-a753-8f0d95986f7c" />

<img width="1919" height="693" alt="image" src="https://github.com/user-attachments/assets/5c66a59f-36cf-4da6-9d48-0d05abc28577" />

Như vậy là đăng ký domain đã hoàn tất.
## Đăng ký tài khoản cloudflare
Trước hết đăng ký tài khoảng cloudflare. Thực hiện tuàn tự các bước là ok.

## Thêm domain đã đăng ký vào trong cloudflare: Nhận 2 dòng namespace
Chọn domain, sau đó ấn nút Orboard domain. Nhập domain vừa đăng ký vào.
<img width="960" height="454" alt="tích hợp domain cloudflare" src="https://github.com/user-attachments/assets/a999aa49-f53b-4b07-90b6-d955f53fe61b" />

Next đến chỗ có 2 dòng nameserver.
<img width="960" height="449" alt="chỉnh nameserver cloudflare" src="https://github.com/user-attachments/assets/13f67cee-f24d-44db-8fb5-0e2ffb2b74d7" />

## Nhập 2 dòng namespace của cloudflare vào trong trang quản lý DNS record của tên miền đăng ký (vd trên mắt bão)
Vào lại trang Mắt bão, vào trang quản trị domain muốn kết nối với cloudflare, dán 2 nameserver vào và ấn Lưu thay đổi:
<img width="1919" height="886" alt="image" src="https://github.com/user-attachments/assets/2c042dcc-888b-4dbe-9e2b-c9fadd6a625e" />

Quay lại trang Cloudflare, ấn I update nameserver ta được:
<img width="960" height="450" alt="chờ" src="https://github.com/user-attachments/assets/481a3426-c7a3-492e-b158-7585491260d8" />

Ấn vào check nameservers now. Chờ 1 lúc sẽ được. Như hình là ok
<img width="960" height="447" alt="check ok" src="https://github.com/user-attachments/assets/176fb995-5ae1-45d2-b7fa-fa6f8c8be039" />

Hoàn tất nhiệm vụ A.
