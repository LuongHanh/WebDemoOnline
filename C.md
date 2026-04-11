# C. Cấu hình docker compose:
Tạo thư mục: ~/myapp
Chuyển vào trong thư mục ~/myapp
Tạo thư mục: ./myweb
Tạo file ./myweb/index.html (với nội dung là thông tin cá nhân của em)
<img width="1346" height="765" alt="image" src="https://github.com/user-attachments/assets/661f2b37-0853-42cc-837a-d9c73bca57a1" />

Tạo file docker-compose.yml để nó sẽ có các dịch vụ sau:
Khai báo sử dụng nodered/node-red, cổng 1880, dữ liệu nằm tại thư mục ./nodered
Khai báo sử dụng nginx, cổng 80, cấu hình trong file ./nginx/nginx.conf
Mount thư mục ./myweb thành thư mục /myweb trong nginx
Mount file ./nginx/nginx.conf vào file /etc/nginx/nginx.conf trong nginx
<img width="1348" height="777" alt="image" src="https://github.com/user-attachments/assets/6a822a89-d7df-437e-973e-2f04742aa0b8" />

```
version: '3.8'

services:
  nodered:
    image: nodered/node-red
    container_name: nodered
    ports:
      - "1880:1880"
    volumes:
      - ./nodered:/data
    restart: always

  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./myweb:/myweb
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - nodered
    restart: always
```

Edit file ./nginx/nginx.conf để:
Cấu hình web server cổng 80
server_name là sub-domain (sub-domain tuỳ ý của em)
location / trỏ tới root là thư mục /myweb
location /api dùng proxy_pass trỏ tới 1 (hoặc nhiều) node http_in của nodered
<img width="1341" height="775" alt="image" src="https://github.com/user-attachments/assets/bec1a062-dd63-46d2-897f-9de7852670eb" />

```
events {}

http {
    server {
        listen 80;
        server_name api.firstmydm.io.vn;

        location / {
            root /myweb;
            index index.html;
        }

        location /api/ {
            proxy_pass http://nodered:1880/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```
Edit file ./nodered/settings.js để nodered bắt buộc đăng nhập
Chạy docker-compose lần đầu để Node-RED tự sinh file cấu hình trong thư mục ./nodered, sau đó mới tiến hành sửa settings.js và restart lại container


