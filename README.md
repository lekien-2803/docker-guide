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

```bash
docker build -t hello-golang-app .
```

Sau khi build xong thì ta chạy tiếp câu lệnh:

```bash
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
```bash
docker build -t nginx-alpine .
```

Chạy container này trên cổng 8080:
```bash
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
```yaml
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
```bash
docker-compose build
```

Sau khi build xong thì ta chạy container:
```bash
docker-compose up -d
```

## 4. Tạo container MySQL database
### 4.1. Tạo MySQL lắng nghe ở cổng 3000, có password root là ‘abc123-’

Tạo file `docker-compose.yml` như sau:
```yaml
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
```bash
docker-compose up -d
```

Sau khi container đã chạy, kết nối đến mysql database bằng công cụ quản lý database (dbeaver, mysql workbench, phpMyadmin, v.v...) với các thông tin sau:

    Host: localhost
    Port: 3000
    User: root
    Password: abc123-

### 4.2.Tạo MySQL lắng nghe ở cổng mặc định, có password root là ‘abc123-’, có thêm công cụ quản trị adminer

Tạo file `docker-compose.yml`:
```yaml
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
```bash
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
```yaml
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
```bash
docker-compose up -d
```

Bây giờ kết nối với database bằng các công cụ quản lý database như MySQL Work Bench, DBeaver, v.v... và kết nối với database, tạo schemas, tạo bảng, tất cả đều được lưu vào thư mục `mysql_data` trên desktop.

## 5. Tạo container Postgresql database
### 5.1. Tạo Postgresql database hệ điều hành Alpine lắng nghe ở cổng mặc định

Đầu tiên ta tạo file `docker-compose.yml` như sau:

```yaml
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

```bash
docker-compose up -d
```

### 5.2: Tạo Postgresql cùng phần mềm quản lý Adminer, password root là ‘abc123-’

File `docker-compose.yml` của ta sẽ như sau:

```yaml
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

```bash
docker-compose up -d
```

Sau đó ta vào đường dẫn `http://localhost:8080` và đăng nhập vào postgres với thông tin như sau:
    System: PostgreSQL
    Server: postgres (Tên dịch vụ PostgreSQL trong docker-compose.yml)
    Username: root
    Password: abc123-

### 5.3: Tạo Postgresql cùng phần mềm quản lý Adminer, password root là ‘abc123-’, thư mục data tham chiếu vào thư mục trên desk top của bạn.

Tạo file `docker-compose.yml` như sau:

```yaml
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
```bash
docker-compose up -d
```

Sau khi các container đã chạy, ta có thể truy cập Adminer bằng cách mở trình duyệt và điều hướng đến `http://localhost:8080`. Từ đó có thể đăng nhập vào PostgreSQL sử dụng thông tin sau:

    System: PostgreSQL
    Server: postgres (Tên dịch vụ PostgreSQL trong docker-compose.yml)
    Username: root
    Password: abc123-

# 2. Tạo Docker image
## 1. Tạo Docker image của một ứng dụng Golang trả về REST API đơn giản ở cổng 8080.

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
```bash
docker build -t book-store .
```

Trong đó, `book-store` là tên mà bạn muốn đặt cho Docker image.

Sau khi image đã được build, bạn có thể chạy một container sử dụng image này:
* Sử dụng lệnh:
```bash
docker run -p 8080:8080 book-store
```

## 2. Tạo Docker image một database Postgresql database, cần thực hiện file SQL lần đầu tiên khi khởi động database

Ta có file sql đặt tên là `init-data.sql` như sau:
```sql
-- Create the 'humanresource' database
CREATE DATABASE humanresource;

-- Create the 'people' table
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    gender VARCHAR(10),
    date_of_birth DATE
);

-- Create the 'university' table
CREATE TABLE university (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    location VARCHAR(255)
);

-- Create the 'department' table
CREATE TABLE department (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    description TEXT
);

-- Create the 'salary_history' table
CREATE TABLE salary_history (
    id SERIAL PRIMARY KEY,
    person_id INT,
    salary DECIMAL(10, 2),
    effective_date DATE,
    FOREIGN KEY (person_id) REFERENCES people(id)
);
   -- Insert data into the 'people' table
INSERT INTO people (first_name, last_name, gender, date_of_birth)
VALUES
    ('John', 'Doe', 'Male', '1990-01-15'),
    ('Jane', 'Smith', 'Female', '1985-03-20'),
    ('Michael', 'Johnson', 'Male', '1995-07-10'),
    ('Emily', 'Brown', 'Female', '1992-09-25'),
    ('David', 'Lee', 'Male', '1988-12-05');

-- Insert data into the 'university' table
INSERT INTO university (name, location)
VALUES
    ('University of ABC', 'Cityville'),
    ('XYZ University', 'Townsville'),
    ('ABC Institute of Technology', 'Tech City');

-- Insert data into the 'department' table
INSERT INTO department (name, description)
VALUES
    ('Human Resources', 'Manage personnel and hiring'),
    ('Finance', 'Manage financial transactions'),
    ('Computer Science', 'Teaching computer science courses');

-- Insert data into the 'salary_history' table
INSERT INTO salary_history (person_id, salary, effective_date)
VALUES
    (1, 55000.00, '2022-01-01'),
    (2, 60000.00, '2022-01-01'),
    (3, 58000.00, '2022-01-01'),
    (4, 62000.00, '2022-01-01'),
    (5, 53000.00, '2022-01-01');
```

Tạo file `docker-compose.yml`:
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: abc123-
      POSTGRES_DB: demo
    ports:
      - "5432:5432"
    volumes:
      - ./init-data.sql:/docker-entrypoint-initdb.d/init-data.sql
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Mở terminal lên tại thư mục chứa `docker-compose.yml` và `init-data.sql` rồi chạy lệnh:
```bash
docker-compose up -d
```

Mở các công cụ quản lý database lên và kiểm tra xem, ta sẽ thấy các bảng đã có dữ liệu ở trong database tên là `demo`.

## 3. Tạo Docker Image dự án bài số 2, sau đó thêm vào một ứng dụng Golang APP truy vấn bảng people bằng câu lệnh SELECT * FROM people rồi trả về JSON ở cổng 8080.

Ta đã có image ở bài số 2, công việc bây giờ là tạo ra một app Golang có thể truy vấn vào bảng `people` và trả về dữ liệu JSON.

Đầu tiên thì trong app Golang đó ta cần có một struct `Person` có các trường dữ liệu tương ứng với các cột trong bảng `people` bao gồm: `ID`, `FirstName`, `LastName`, `Gender`, `DateOfBirth`.

Tiếp theo ta sẽ kết nối với `postgres`, lấy dữ liệu với câu truy vấn:
```sql
SELECT * FROM people
```

Trả về dữ liệu dạng JSON tại api `/people`

Vì là app Golang đơn giản, nên ta có thể để luôn phần struct này trong file `main.go` như sau:

```go
package main

import (
    "database/sql"
    "encoding/json"
    "log"
    "net/http"

    _ "github.com/lib/pq"
)

type Person struct {
    ID         int    `json:"id"`
    FirstName  string `json:"first_name"`
    LastName   string `json:"last_name"`
    Gender     string `json:"gender"`
    DateOfBirth string `json:"date_of_birth"`
}

func main() {
    connStr := "postgres://postgres:abc123-@postgres:5432/demo?sslmode=disable"
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        log.Fatal(err)
    }

    http.HandleFunc("/people", func(w http.ResponseWriter, r *http.Request) {
        rows, err := db.Query("SELECT * FROM people")
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
        defer rows.Close()

        var people []Person
        for rows.Next() {
            var p Person
            if err := rows.Scan(&p.ID, &p.FirstName, &p.LastName, &p.Gender, &p.DateOfBirth); err != nil {
                http.Error(w, err.Error(), http.StatusInternalServerError)
                return
            }
            people = append(people, p)
        }

        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(people)
    })

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

Tiếp theo ta tạo file `Dockerfile`:
```dockerfile
FROM golang:alpine

WORKDIR /app

COPY . .

RUN go build -o main .

CMD ["/app/main"]
```

Sửa lại file `docker-compose.yml` ở trên:
```yaml
version: '3.8'
services:
  postgres:
    # Cấu hình postgres giống như trước

  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - postgres
```

Giờ ta chạy lệnh:
```bash
docker-compose up --build
```

Mở trình duyệt lên và vào đường dẫn `http://localhost:8080/people`, ta sẽ thấy dữ liệu được hiển thị dưới dạng JSON.