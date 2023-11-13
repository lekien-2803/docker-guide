# docker-guide
2. Hello World với Docker: Chạy một container Docker đơn giản chứa ứng dụng “Hello World”.

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

Sau khi chạy xong, tại thư mục `hello-world`, mở terminal lên và chạy câu lệnh:

`docker build -t hello-golang-app .`

Sau khi build xong thì ta chạy tiếp câu lệnh:

`docker run hello-golang-app`