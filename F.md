# F. Gỡ lỗi:
Nếu có lỗi xẩy ra trong quá trình triển khai docker compose up -d

Kiểm tra nhanh: docker compose ps giúp biết container nào đang chạy xem log, ví dụ: docker logs cloudflared

Thực chất trong quá trình làm xảy ra rất nhiều lỗi, trong đó khi chạy nodered thường bị lỗi về quyền sở hữu thư mục khiến nodered bị restating liên tục; Khi gặp phải trường hợp đó ta cần set lại quyền truy cập thư mục bằng lệnh `sudo chown -R 1000:1000 nodered`
Dưới đây là giải thích chi tiết từng thành phần lệnh:
- sudo: Chạy lệnh với quyền quản trị cao nhất (root). Bạn cần quyền này vì việc thay đổi chủ sở hữu các tệp tin hệ thống hoặc của người dùng khác bị hạn chế.
chown: Viết tắt của "change owner". Đây là lệnh dùng để thay đổi chủ sở hữu (user) và nhóm sở hữu (group) của tệp tin hoặc thư mục.
- -R: Viết tắt của "Recursive" (đệ quy). Tham số này cực kỳ quan trọng: nó áp dụng thay đổi cho thư mục nodered và tất cả các tệp/thư mục con bên trong nó.
1000:1000:
- Số 1000 đầu tiên là UID (User ID): Định danh của người dùng. Trong nhiều hệ thống Linux (như Ubuntu, Debian), 1000 thường là ID của người dùng đầu tiên được tạo ra.
- Số 1000 thứ hai là GID (Group ID): Định danh của nhóm người dùng.
- Dấu : ngăn cách giữa User và Group.
- nodered: Tên của thư mục hoặc tệp tin mà bạn muốn áp dụng thay đổi.

Sau khi thay cập nhật quyền sở hữu thư mục đã chạy ok.

Tiếp đến là lỗi quyền truy cập file json trong cloudflared; Ta dùng lênh `chmod 644 ~/myapp/cloudflared/*.json` để fix;
Giải thích lệnh:

Phân tích chỉ số 644

Mỗi con số đại diện cho một nhóm đối tượng cụ thể theo thứ tự: Chủ sở hữu (Owner) : Nhóm (Group) : Người khác (Others). 
- 6 (Chủ sở hữu): Chủ sở hữu (thường là người dùng hiện tại) có toàn quyền đọc và chỉnh sửa nội dung tệp.
- 4 (Nhóm): Chỉ có quyền Đọc. Các thành viên trong cùng nhóm không thể sửa đổi tệp.
- 4 (Người khác): Chỉ có quyền Đọc. Bất kỳ ai khác trên hệ thống cũng chỉ có thể xem nội dung tệp. 

Đường dẫn và Đối tượng
- ~/myapp/cloudflared/: Đường dẫn đến thư mục chứa các tệp cấu hình của bạn. Dấu ~ đại diện cho thư mục Home của người dùng.
- *.json: Ký tự đại diện (*) giúp áp dụng lệnh cho tất cả các tệp JSON trong thư mục đó, thay vì phải gõ tên từng tệp. 

 Ta thấy lỗi:
<img width="1919" height="422" alt="image" src="https://github.com/user-attachments/assets/dc595715-8d1b-4072-b955-61bc283fea34" />

Log lỗi:
<img width="1913" height="313" alt="image" src="https://github.com/user-attachments/assets/7f2bac07-a05f-486e-9322-257a7f27cb99" />

<img width="1896" height="358" alt="image" src="https://github.com/user-attachments/assets/4eb4c07d-e7f7-4f76-b8a3-563ca59e06e6" />

Sau khi fix đã chạy ok.
<img width="1919" height="320" alt="image" src="https://github.com/user-attachments/assets/e91585d3-b535-43cd-8dcb-afe268056f57" />

Thêm healthcheck cho myapi trong file docker-compose.yml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:9630"]

```
healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9630"]   #gọi API
    interval: 30s   #30s check 1 lần
    timeout: 10s   #tối đa 10s
    retries: 3   #fail 3 lần ==> unhealth (không ổn)
```
giới hạn resource cho một service: (tránh việc 1 service chiếm quá nhiều ram)
```
deploy:
  resources:
    limits:
      memory: 512M
```
Sau khi thêm healthcheck và deaploy:
<img width="1284" height="450" alt="image" src="https://github.com/user-attachments/assets/8c130acd-b66e-4ee7-a519-2324fe114022" />

sử dụng lệnh: docker stats để quan sát lượng ram sử dụng bởi mỗi service (các thông số sẽ thay đổi theo thời gian)
<img width="1295" height="188" alt="image" src="https://github.com/user-attachments/assets/939c512e-12e4-4857-9426-aa41d6f8d7b2" />

