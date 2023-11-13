# docker-guide
## 2. Hello World với Docker: Chạy một container Docker đơn giản chứa ứng dụng “Hello World”.

Tạo một folder tên "hello-world", trong folder mở terminal lên và chạy câu lệnh:

`go mod init hello-world`

Tiếp theo tạo file `main.go`, nội dung trong file này:

```package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

Tạo một file tên là `Dockerfile` (hãy nhớ file này không có tên mở rộng), nội dung của file này như sau:

```# Sử dụng một hình ảnh chứa Go để xây dựng ứng dụng
FROM golang:latest

# Sao chép mã nguồn của bạn vào container
COPY . /app

# Thiết lập thư mục làm việc mặc định
WORKDIR /app

# Biên dịch ứng dụng Golang
RUN go build -o main .

# Chạy ứng dụng khi container được khởi chạy
CMD ["./main"]

```


Sau khi có 3 file như trên, tại thư mục `hello-world`, mở terminal lên và chạy câu lệnh:

`docker build -t hello-golang-app .`

Sau khi build xong thì ta chạy tiếp câu lệnh:

`docker run hello-golang-app`

## 3. Tạo container web server nginx
### 3.1. Tạo web server sử dụng nginx hệ điều hành Alpine lắng nghe ở cổng 8080

Đầu tiên tạo một file `index.html` với nội dung sau:

```
<!DOCTYPE html>
<html>
<head>
    <title>Web Server with Nginx</title>
</head>
<body>
    <h1>Hello, Nginx on Alpine Linux!</h1>
</body>
</html>
```

Tiếp theo ta tạo một file nginx config tên là `nginx.conf` với nội dung:
```
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    server {
        listen 8080;
        server_name localhost;

        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
```

Cuối cùng ta tạo `Dockerfile` với nội dung:
```
# Sử dụng hình ảnh Alpine Linux như hệ điều hành cơ sở
FROM alpine:latest

# Cài đặt Nginx
RUN apk add --update nginx

# Tạo thư mục chứa trang web
RUN mkdir -p /var/www/html

# Sao chép tệp index.html mẫu vào thư mục web
COPY index.html /var/www/html/

# Sao chép cấu hình Nginx tùy chỉnh vào container
COPY nginx.conf /etc/nginx/nginx.conf

# Lắng nghe trên cổng 8080
EXPOSE 8080

# Khởi động Nginx khi container được khởi chạy
CMD ["nginx", "-g", "daemon off;"]
```

Mở terminal lên để build Docker image:
`docker build -t nginx-alpine .`

Chạy container này trên cổng 8080:
`docker run -d -p 8080:8080 nginx-alpine`

### 3.2. Tạo web server sử dụng nginx hệ điều hành Alpine lắng nghe ở cổng 9000, thư mục web gốc nginx sẽ tham chiếu (mapping volume) vào thư mục trên desk top của bạn chứa một file index.html. Nội dung file index.html in ra dòng chữ “Nginx Docker”

Chuẩn bị `Dockerfile`:
```
# Sử dụng Alpine làm base image
FROM alpine:latest

# Cài đặt Nginx
RUN apk add --no-cache nginx

# Tạo thư mục cho PID của Nginx và folder chứa nội dung web
RUN mkdir -p /run/nginx /var/www/html

# Cấu hình Nginx để lắng nghe ở cổng 9000
RUN echo "server { listen 9000; root /var/www/html; }" > /etc/nginx/http.d/default.conf

# Expose cổng 9000
EXPOSE 9000

# Khởi chạy Nginx
CMD ["nginx", "-g", "daemon off;"]
```

Tạo file `index.html`:

```
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to Nginx!</title>
</head>
<body>
    <h1>Nginx Docker</h1>
</body>
</html>
```

Tạo file `docker-compose.yml`:
```
version: '3.8'
services:
  web:
    build: .
    ports:
      - "9000:9000"
    volumes:
      - /path/to/your/desktop/folder:/var/www/html

```

Thay thế /path/to/your/desktop/folder bằng đường dẫn thực tế đến thư mục trên máy tính của bạn chứa file `index.html`.

Tiếp theo ta build image, mở terminal lên và chạy câu lệnh:
`docker-compose build`

Sau khi build xong thì ta chạy container:
`docker-compose up -d`
