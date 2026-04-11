# D. (Bonus - không bắt buộc)
- tạo thư mục ./myapi
- tạo file ./myapi/app.py sử dụng Python + Flask để tính tuổi hiện tại với đầu vào là năm sinh
  <img width="1919" height="847" alt="image" src="https://github.com/user-attachments/assets/bcd3443f-25ad-4d6b-9cf6-facbb5b7bbc0" />

  ```
  from flask import Flask, request, jsonify
  import random
  from datetime import datetime
  
  app = Flask(__name__)
  
  @app.route("/")
  def home():
      return "My API is running 🚀"
  
  @app.route("/age")
  def hi():
      year = request.args.get("year", type=int)
      current_year = datetime.now().year
  
      if not year or year > current_year or year < 1900:
          return jsonify({"error": "Năm sinh không hợp lệ!"})
  
      age = current_year - year
  
      return jsonify({
          "age": age,
          "funny": random.choice([
              "Bạn vẫn còn trẻ 😎",
              "Sắp già rồi đó 😂",
              "Tuổi đẹp để code 🚀"
          ])
      })
  
  if __name__ == "__main__":
      app.run(host="0.0.0.0", port=9630)
  ```
- tạo file ./myapi/requirements.txt chứa các thư viện mà app.py sử dụng (theo như app.py ví dụ thì requirements.txt chỉ cần có nội dung: flask)
- tạo file ./myapi/Dockerfile để khai báo sử dụng Python 3.9 slim
<img width="1919" height="385" alt="image" src="https://github.com/user-attachments/assets/7d577ab7-c57b-4136-91fb-e3c56d119b9c" />

```
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 9630

CMD ["python", "app.py"]
```
Sửa đổi docker-compose để sử dụng myapp (xem phần tham khảo ở dưới)

Thêm phần cấu hình myapi:
```
  myapi:
    build: ./myapi
    container_name: myapi
    ports:
      - "9630:9630"
    restart: always
```
Sửa đổi nginx/nginx.conf để /api trỏ tới service myapp cổng 9630
<img width="1919" height="578" alt="image" src="https://github.com/user-attachments/assets/4245b4d0-f83d-4733-8cf9-d2e19633d826" />
