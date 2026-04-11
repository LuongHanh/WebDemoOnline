# E. Triển khai (level test) ứng dụng
Chuyển vào trong thư mục ~/myapp

Gõ lệnh để docker compose chạy: sẽ run tất cả các service khai báo trong file docker-compose.yml

Lợi ích: Chỉ cần docker-compose up -d là toàn bộ hệ thống (Web + Node-RED + Tunnel) tự chạy

Trước hết, ta thêm cloudflared trong docker-compose.yml:

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

  cloudflared:
    image: cloudflare/cloudflared
    container_name: cloudflared
    command: tunnel run mytunnel
    restart: always
    volumes:
      - ./cloudflared:/etc/cloudflared
    depends_on:
      - nginx
```

Kiểm tra các container đang chạy trong docker, nếu có cái nào bị restart cần tìm lỗi rồi edit lại docker-compose.yml

Kiểm tra kiểm thử các service đang chạy độc lập thông qua ip và port của nó: ví dụ mở trình duyệt ip_ubuntu:1880 để check nodered đã chạy chưa

<img width="1919" height="540" alt="image" src="https://github.com/user-attachments/assets/1cdef8af-dfd6-4a0b-9ecf-f1bac6e0e594" />

<img width="1913" height="134" alt="image" src="https://github.com/user-attachments/assets/25bba7b8-a68d-4619-ac94-804f52dfd05e" />

Sử dụng nodered: kéo nodered http_in , http_response, function : để tạo api get đơn giản (dùng cho /api proxy_pass của nginx)
<img width="1919" height="838" alt="image" src="https://github.com/user-attachments/assets/e2a21d78-4226-400d-8142-c7a9f99cecd4" />

Sửa file ./myweb/index.html : thêm code html+js để sử dụng được api đã khai báo proxy_pass (thực ra là sử dụng nodered http_in hoặc sử dụng service myapi)
<img width="1905" height="981" alt="image" src="https://github.com/user-attachments/assets/97289699-ba0e-4004-b489-17ed71bd6214" />

**Kết quả sau phần này**
<img width="1914" height="956" alt="image" src="https://github.com/user-attachments/assets/8678e1c7-9e4e-4a7a-baae-d02d9d0812a7" />
