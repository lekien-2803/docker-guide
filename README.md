# 1. Khởi động 1 container
## 2. Hello World với Docker: Chạy một container Docker đơn giản chứa ứng dụng “Hello World”.

Tạo một folder tên "hello-world", trong folder mở terminal lên và chạy câu lệnh:

```go
go mod init hello-world
```

Tiếp theo tạo file `main.go`, nội dung trong file này:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

Tạo một file tên là `Dockerfile` (hãy nhớ file này không có tên mở rộng), nội dung của file này như sau:

```dockerfile
# Sử dụng một hình ảnh chứa Go để xây dựng ứng dụng
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

```
docker build -t hello-golang-app .
```

Sau khi build xong thì ta chạy tiếp câu lệnh:

```
docker run hello-golang-app
```

## 3. Tạo container web server nginx
### 3.1. Tạo web server sử dụng nginx hệ điều hành Alpine lắng nghe ở cổng 8080

Đầu tiên tạo một file `index.html` với nội dung sau:

```html
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
```bash
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
```dockerfile
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
```
docker build -t nginx-alpine .
```

Chạy container này trên cổng 8080:
```
docker run -d -p 8080:8080 nginx-alpine
```

### 3.2. Tạo web server sử dụng nginx hệ điều hành Alpine lắng nghe ở cổng 9000, thư mục web gốc nginx sẽ tham chiếu (mapping volume) vào thư mục trên desk top của bạn chứa một file index.html. Nội dung file index.html in ra dòng chữ “Nginx Docker”

Chuẩn bị `Dockerfile`:
```dockerfile
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

```html
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
```dockerfile
version: '3.8'
services:
  web:
    build: .
    ports:
      - "9000:9000"
    volumes:
      - /path/to/your/desktop/folder:/var/www/html

```

Thay thế `"/path/to/your/desktop/folder"` bằng đường dẫn thực tế đến thư mục trên máy tính của bạn chứa file `index.html`.

Tiếp theo ta build image, mở terminal lên và chạy câu lệnh:
```
docker-compose build
```

Sau khi build xong thì ta chạy container:
```
docker-compose up -d
```

## 4. Tạo container MySQL database
### 4.1. Tạo MySQL lắng nghe ở cổng 3000, có password root là ‘abc123-’

Tạo file `docker-compose.yml` như sau:
```dockerfile
version: '3.8'
services:
  mysql:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: abc123-
    ports:
      - "3000:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

Khởi chạy container với câu lệnh:
```
docker-compose up -d
```

Sau khi container đã chạy, kết nối đến mysql database bằng công cụ quản lý database (dbeaver, mysql workbench, phpMyadmin, v.v...) với các thông tin sau:

    Host: localhost
    Port: 3000
    User: root
    Password: abc123-

### 4.2.Tạo MySQL lắng nghe ở cổng mặc định, có password root là ‘abc123-’, có thêm công cụ quản trị adminer

Tạo file `docker-compose.yml`:
```dockerfile
version: '3.8'
services:
  mysql:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: abc123-
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  adminer:
    image: adminer
    ports:
      - "8080:8080"

volumes:
  mysql_data:
```

Sau đó mở terminal lên và khởi chạy các container:
```
docker-compose up -d
```

Sau khi khởi chạy xong, vào trình duyệt web truy cập đường dẫn `http://localhost:8080`. Tại đây là giao diện của adminer, ta có thể đăng nhập vào MySQL với các thông tin sau:

    System: MySQL
    Server: mysql (Tên dịch vụ MySQL trong docker-compose.yml)
    Username: root
    Password: abc123-

### 4.3. Tạo MySQL lắng nghe ở cổng mặc định, có password root là ‘abc123-’, thư mục data tham chiếu vào thư mục trên desktop của bạn.

Trước hết, hãy tạo một thư mục trên desktop của bạn để lưu trữ dữ liệu MySQL. Giả sử bạn tạo một thư mục có tên là `mysql_data` trên desktop.

Tạo File `docker-compose.yml`:
```dockerfile
version: '3.8'
services:
  mysql:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: abc123-
    ports:
      - "3306:3306"
    volumes:
      - /path/to/your/desktop/mysql_data:/var/lib/mysql
```
Thay thế `"/path/to/your/desktop/mysql_data"` bằng đường dẫn thực tế đến thư mục mysql_data mà bạn đã tạo trên desktop của mình.

Mở terminal hoặc command prompt và điều hướng đến thư mục chứa file `docker-compose.yml`. 

Khởi chạy Container:
```
docker-compose up -d
```

Bây giờ kết nối với database bằng các công cụ quản lý database như MySQL Work Bench, DBeaver, v.v... và kết nối với database, tạo schemas, tạo bảng, tất cả đều được lưu vào thư mục `mysql_data` trên desktop.

## 5. Tạo container Postgresql database
### 5.1. Tạo Postgresql database hệ điều hành Alpine lắng nghe ở cổng mặc định

Đầu tiên ta tạo file `docker-compose.yml` như sau:

```dockerfile
version: '3.8'
services:
  postgres:
    image: postgres:alpine
    environment:
      POSTGRES_PASSWORD: abc123- 
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Tiếp theo ta khởi chạy container:

```
docker-compose up -d
```

### 5.2: Tạo Postgresql cùng phần mềm quản lý Adminer, password root là ‘abc123-’

File `docker-compose.yml` của ta sẽ như sau:

```dockerfile
version: '3.8'
services:
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: root # Đặt username là root
      POSTGRES_PASSWORD: abc123- # Đặt password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  adminer:
    image: adminer
    ports:
      - "8080:8080"

volumes:
  postgres_data:
```

Tiếp theo ta khởi chạy container:

```
docker-compose up -d
```

Sau đó ta vào đường dẫn `http://localhost:8080` và đăng nhập vào postgres với thông tin như sau:
    System: PostgreSQL
    Server: postgres (Tên dịch vụ PostgreSQL trong docker-compose.yml)
    Username: root
    Password: abc123-

### 5.3: Tạo Postgresql cùng phần mềm quản lý Adminer, password root là ‘abc123-’, thư mục data tham chiếu vào thư mục trên desk top của bạn.

Tạo file `docker-compose.yml` như sau:

```dockerfile
version: '3.8'
services:
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: root 
      POSTGRES_PASSWORD: abc123- 
    ports:
      - "5432:5432"
    volumes:
      - /path/to/your/desktop/postgres_data:/var/lib/postgresql/data

  adminer:
    image: adminer
    ports:
      - "8080:8080"

volumes:
  postgres_data:
```

Thay thế /path/to/your/desktop/postgres_data bằng đường dẫn thực tế đến thư mục postgres_data mà bạn đã tạo trên desktop của mình.

Khởi chạy container:
```
docker-compose up -d
```

Sau khi các container đã chạy, ta có thể truy cập Adminer bằng cách mở trình duyệt và điều hướng đến `http://localhost:8080`. Từ đó có thể đăng nhập vào PostgreSQL sử dụng thông tin sau:

    System: PostgreSQL
    Server: postgres (Tên dịch vụ PostgreSQL trong docker-compose.yml)
    Username: root
    Password: abc123-

## 6. Tạo Docker image
### 6.1. Tạo Docker image của một ứng dụng Golang trả về REST API đơn giản ở cổng 8080.

Tôi có một ứng dụng đơn giản là một trang web sách (book-store) tại repo:

```bash
git clone https://github.com/lekien-2803/book-store.git
```

Sau khi clone về, cây thư mục của chúng ta có dạng:
```bash
│   go.mod
│   go.sum
│   main.go
│
├───.idea
│       .gitignore
│       book-management-app.iml
│       modules.xml
│       vcs.xml
│
├───controller
│       book_controller.go
│
├───database
│       book_database.go
│
├───model
│   │   book_model.go
│   │
│   ├───request
│   │       book_request.go
│   │
│   └───response
│           response.go
│
├───repository
│       book_repository.go
│
├───resources
│   ├───static
│   │   └───lib
│   │       └───bootstrap
│   │               bootstrap.bundle.min.js
│   │               bootstrap.min.css
│   │
│   └───views
│           create.html
│           detail.html
│           index.html
│
├───rest
│       book_rest.go
│
├───router
│       book_router.go
│
└───service
        book_service.go
```

Ta sẽ di chuyển con trỏ ra folder `book-store`, nơi có file `main.go` để tạo file `Dockerfile`:

```dockerfile
# Sử dụng base image Golang từ Docker Hub
FROM golang:alpine

# Thiết lập /app làm thư mục làm việc
WORKDIR /app

# Copy tất cả file trong thư mục hiện tại vào /app trong container
COPY . .

# Build ứng dụng Golang
RUN go build -o main .

# Chạy ứng dụng khi container khởi động
CMD ["/app/main"]
```

Tạo xong `Dockerfile` thì ta build docker image với câu lệnh:
```
docker build -t book-store .
```

Trong đó, `book-store` là tên mà bạn muốn đặt cho Docker image.

Sau khi image đã được build, bạn có thể chạy một container sử dụng image này:
* Sử dụng lệnh:
```
docker run -p 8080:8080 book-store
```

### 6.2. Tạo Docker image một ứng dụng Golang trên nền của Docker image Postgresql, ứng dụng Golang truy vấn vào Postgresql và trả về JSON của bảng People

