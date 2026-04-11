# F. Gỡ lỗi:
nếu có lỗi xẩy ra trong quá trình triển khai docker compose up -d
Kiểm tra nhanh: docker compose ps giúp biết container nào đang chạy xem log, ví dụ: docker logs mynginx docker logs myapi

Thêm healthcheck cho myapi trong file docker-compose.yml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:9630"]
giới hạn resource cho một service: (tránh việc 1 service chiếm quá nhiều ram)
deploy:
  resources:
    limits:
      memory: 512M
sử dụng lệnh: docker compose stats để quan sát lượng ram sử dụng bởi mỗi service
