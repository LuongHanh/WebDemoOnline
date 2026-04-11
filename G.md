# G. Triển khai ứng dụng đến End-user
## Trong Cloudflare: Tạo tunnel (đường hầm), chọn loại triển khai cho docker

Có 2 cách tạo và kết nối tunnel:
Trước hết cần cài cloudflared:
<img width="672" height="385" alt="install cloudflare ubuntu" src="https://github.com/user-attachments/assets/f02a48d9-a26e-47f0-b395-0eea8ffaf229" />

**Cách 1: Local-managed (đã dùng cho tunnel: mytunnel).**

Cách này cấu hình 100% bằng dòng lệnh, khá phức tạp vì cần config.yml và file (tunnelID).json, file cert.pem.

Bước 1: Login Cloudflare `cloudflared tunnel login`

Trên windows mở trình duyệt → login → tải file cert.pem:

Sau đó copy file từ Windows sang Ubuntu:
<img width="960" height="446" alt="tải cem" src="https://github.com/user-attachments/assets/dd912d3d-4ffc-4ad7-8fe6-7e9fec7f733d" />

~/.cloudflared/cert.pem

Bước 2: Tạo tunnel `cloudflared tunnel create mytunnel`

sinh ra file: ~/.cloudflared/<tunnel-id>.json

Bước 3: Tạo file config `nano ~/.cloudflared/config.yml`

Nội dung:
```
tunnel: mytunnel
credentials-file: /home/admin/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: api.firstmydm.io.vn
    service: http://nginx:80
  - service: http_status:404
```

Bước 4: Route DNS `cloudflared tunnel route dns mytunnel api.firstmydm.io.vn`
<img width="667" height="322" alt="create tunnel" src="https://github.com/user-attachments/assets/95b46ff3-9aed-43d5-8eb5-6f65f73b2042" />

Bước 5: Test tunnel `cloudflared tunnel run mytunnel`

nếu không lỗi → OK

Bước 6: Đưa vào docker-compose
```
cloudflared:
  image: cloudflare/cloudflared:latest
  container_name: cloudflared
  command: tunnel run mytunnel
  volumes:
    - ./cloudflared:/etc/cloudflared
  restart: always
  depends_on:
    - nginx
```

Bước 7: Cấu hình nginx.conf
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
            #proxy_pass http://nodered:1880/;
            proxy_pass http://myapi:9630/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

**Cách 2: Remote-managed (đã dùng cho tunnel: first-my-tunnel).**

Cách này thì đơn giản vì tạo tunnel bằng Web cloudflared có UI và hướng dẫn, chỉ cần dùng token, không cần config.yml và file (tunnelID).json

Bước 1: Tạo tunnel. Vào Recents==>Tunnels==>Create Tunnel
<img width="960" height="540" alt="connect" src="https://github.com/user-attachments/assets/f513942b-871c-499c-9254-fa40cc23d79e" />

Bước 2: Copy dòng lệnh màu cam, paste vào Ubuntu (tại vị trí ~/myapp)
<img width="1329" height="103" alt="image" src="https://github.com/user-attachments/assets/b88e6493-3560-4bf2-bc3f-d84d1a85d680" />

Bước 3: Convert sang docker-compose
```
cloudflared:
  image: cloudflare/cloudflared:latest
  container_name: cloudflared
  command: tunnel --no-autoupdate run --token <TOKEN>
  restart: always
  depends_on:
    - nginx
```

Bước 4: Cấu hình Public Hostname
<img width="960" height="540" alt="c3" src="https://github.com/user-attachments/assets/92dbcc20-2b25-4e95-b770-3c0062713799" />


Bước 5: Cấu hình lại nginx.conf
<img width="674" height="386" alt="conf" src="https://github.com/user-attachments/assets/f500b7f6-41d0-44b1-929e-91dfb1de5b5e" />
```
events {}

http {
    server {
        listen 80;
        server_name www.firstmydm.io.vn;

        location / {
            root /myweb;
            index index.html;
        }

        location /api/ {
            #proxy_pass http://nodered:1880/;
            proxy_pass http://myapi:9630/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

Như vậy, trước đó là dùng Local-managed để tạo và chạy tunnel, nhưng đến phần này em lại chuyển đổi sang dùng Remote-managed. Vậy là đã thu hoạch được 2 cách easy==>pro.

## Convert lệnh docker run ... sang dạng docker compose
<img width="1909" height="262" alt="image" src="https://github.com/user-attachments/assets/88ebd8e2-23fb-4a26-913c-111b2ea6496e" />
```
cloudflared:
    image: cloudflare/cloudflared
    container_name: cloudflared
    command: tunnel --no-autoupdate run --token eyJhIjoiNTI0YmRjMzY2YjgzZTg3N2RkMWIxM2MyMGJiNmY1YmIiLCJ0IjoiYWU5NzU2MjktZmEyNi00ZWRmLTkxMjAtNmNhODlkOGVkOWUwIiwicyI6Ill6ZzBZekU1T1RVdFpXUXlOaTAwTnpjM0xUZ3pZVFV0WWpJNVlqWXpORGsxWmpnNSJ9
    restart: always
    volumes:
      - ./cloudflared:/etc/cloudflared
    depends_on:
      - nginx
```

## Chạy lại docker compose
<img width="1901" height="316" alt="image" src="https://github.com/user-attachments/assets/a70b3752-86d2-4d66-8883-ea4272259edb" />

## Kiểm tra url sub-domain đã hoạt động public cho mọi end-user
<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/78e8e930-00db-41b4-85b7-e0a5b226a8e9" />


