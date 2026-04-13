# G. Câu hỏi về bài làm?
## Tại sao phải dùng Nginx làm Reverse Proxy mà không trỏ thẳng Tunnel vào Node-RED?
Ta thấy rằng tunnel là đường hầm dẫn ra internet, mà ở đầu ở đường hầm cần 1 thằng chịu trách nhiệm phía server, Thằng nginx sẽ làm việc này. có chuyến tàu nào muốn đi vào trong server thì phải qua thằng nginx, khi đó nginx sẽ xem và uỷ quyền cho đi vào nodered, chứ ta không cắm thằng tunnel và nodered (backend).

Cụ thể, sử dụng NGINX làm Reverse Proxy giúp:
- Làm điểm vào duy nhất (entrypoint) cho toàn bộ hệ thống
- Có thể:
  + phân luồng request (/, /api)
  + load balancing
  + bảo mật (ẩn backend)
  + Không expose trực tiếp backend (Node-RED)
  
Nếu trỏ thẳng Tunnel vào Node-RED:
- khó mở rộng hệ thống
- không tách frontend/backend
- kém bảo mật
  
## Sự khác biệt giữa việc Mount file và Mount thư mục trong Docker là gì?

**Mount file**

Chỉ mount 1 file cụ thể: Ví dụ ./nginx/nginx.conf:/etc/nginx/nginx.conf
Dùng cho:
- config
- file nhỏ

Ưu điểm: kiểm soát chính xác

**Mount thư mục**


Mount cả thư mục: Ví dụ: ./myweb:/myweb
Dùng cho:
- source code
- data
- logs

Ưu điểm: thay đổi ngoài host → container update ngay

## Nếu thay đổi file index.html ở máy Ubuntu, nội dung trên web có thay đổi ngay không? Tại sao?
Khi chỉnh sửa nội dung file index.html trên máy chủ Ubuntu thì nội dung web cũng sẽ bị thay đổi theo
<img width="755" height="271" alt="image" src="https://github.com/user-attachments/assets/69b6d67f-27c3-44d2-a0a0-217f5ed29f50" />

Vì file index.html nằm trong mywweb/ mà nginx đang mout thư mục myweb nên nội dung file index.html thay đổi thì web cũng thay đổi.

## docker-compose.yml khai báo các services có phần restart: always hoặc restart: unless-stopped : chúng để làm gì?
Các services có phần restart: always hoặc restart: unless-stopped giúp các service sẽ tự động restart khi mà service đó gặp lỗi, nó sẽ cố gắng restart lại liên tục cho tới khi dịch vụ đó chạy. Cho nên khi ta docker ps thì sẽ thấy có dịch vụ bị restarting liên tục ==> unhealth, và khí đó ta cần docker logs service đó để debug;

Tuy nhiên 2 cấu hình này có sự khác biệt:

restart: always
- luôn restart khi container dừng (kể cả reboot máy)
  
restart: unless-stopped
- giống always nhưng nếu user stop thủ công → không restart lại
## Cách khai báo để tất cả các services đều dùng chung 1 network? lợi ích của việc khai báo này là gì? Sửa đổi file docker-compose để tất cả các service đều dùng chung 1 network.
Hiện tại docker-compose.yml đang như này:
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
    command: tunnel --no-autoupdate run --token eyJhIjoiNTI0YmRjMzY2YjgzZTg3N2RkMWIxM2MyMGJiNmY1YmIiLCJ0IjoiYWU5NzU2MjktZmEyNi00ZWRmLTkxMjAtNmNhODlkOGVkOWUwIiwicyI6Ill6ZzBZekU1T1RVdFpXUXlOaTAwTnpjM0xUZ3pZVFV0WWpJNVlqWXpORGsxWmpnNSJ9
    restart: always
    volumes:
      - ./cloudflared:/etc/cloudflared
    depends_on:
      - nginx
    dns:
      - 8.8.8.8
      - 1.1.1.1

  myapi:
    build: ./myapi
    container_name: myapi
    ports:
      - "9630:9630"
    restart: always

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9630"]
      interval: 30s
      timeout: 10s
      retries: 3

    deploy:
      resources:
        limits:
          memory: 512M
```
Để tất cả các services đều dùng chung 1 network ta cần:

**Khai báo network**

Thêm vào cuối file:
```
networks:
  mynetwork:  #có thể gọi là router ảo, nó kết nối với internet, các service connect và nó
    driver: bridge #kiểu mạng mặc định của Docker, tạo ra 1 mạng LAN nội bộ cho các service
```
 
Gán vào từng service:
```
services:
  nginx:
    ...
    networks:
      - mynetwork

  nodered:
    ...
    networks:
      - mynetwork

  myapi:
    ...
    networks:
      - mynetwork

  cloudflared:
    ...
    networks:
      - mynetwork
```
Lợi ích
- Container gọi nhau bằng tên:
  + http://nginx:80
  + http://myapi:9630
- Tách biệt với bên ngoài
- Dễ quản lý
- Bảo mật hơn

Sau thêm mạng và sửa lại :ro và tạo file .env:
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
    networks:
      - mynetwork

  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./myweb:/myweb
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - nodered
    restart: always
    networks:
      - mynetwork

  cloudflared:
    image: cloudflare/cloudflared
    container_name: cloudflared
    command: tunnel --no-autoupdate run --token ${CF_TOKEN}
    restart: always
    volumes:
      - ./cloudflared:/etc/cloudflared
    depends_on:
      - nginx
    dns:
      - 8.8.8.8
      - 1.1.1.1
    networks:
      - mynetwork

  myapi:
    build: ./myapi
    container_name: myapi
    ports:
      - "9630:9630"
    restart: always

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9630"]
      interval: 30s
      timeout: 10s
      retries: 3

    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - mynetwork

networks:
  mynetwork:  #có thể gọi là router ảo, nó kết nối với internet, các service connect và nó
    driver: bridge #kiểu mạng mặc định của Docker, tạo ra 1 mạng LAN nội bộ cho các service
```
## Tìm cách đưa Cloudflare Token vào trong file .env rồi sau đó thêm .env vào file .gitignore trước khi push code lên github. Tại sao nói đây là điều quan trọng về bảo mật mã nguồn?
Các dev khi xây dựng 1 project thì sẽ phải cần tới biến môi trường, các biến này nằm trong file .env (enviroment), các biến này là các thông tin quan trọng, tuyệt mật cho project. Ở đây Cloudflare Token chính là các mà chứ tất cả thông tin cấu hình về tunnel, một khi thông tin này lộ ra hacker có thể khai thác và chiếm quyền truy cập tunnel này, các dữ liệu đi quan tunnel sẽ gặp nguy hiểm. Hoặc là đánh cắp dữ liệu, hoặc là đánh sập server. Vì vậy không push file .env lên github vì github là kho lưu trữ mã nguồn mở, và github cũng có cảnh bảo và ngăn chặn việc dev push file .env.
## Tại sao chúng ta nên thêm hậu tố :ro khi mount file cấu hình Nginx?
Nó cũng là 1 phương án bảo mật; Việc này chỉ cho phép user đọc (read only) các file :ro ; Điều này sẽ bảo vệ được mã nguồn không bị chỉnh sửa.
## Khi dùng Cloudflare Tunnel: có cần thiết phải mở cổng cho các service nữa không?
Không cần thiết; Vì tunnel đi qua nginx, chỉ cần mở cổng ra cho nginx là ok, nginx sẽ tự proxy pass sang các dịch vụ khác. Ở đây là proxy pass sang nodered (backend)
