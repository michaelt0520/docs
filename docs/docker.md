![](https://froylancamacho.com/content/images/2019/02/docker.png)

# Docker
## 1. Khái niệm
- Docker là một nền tảng dành cho các developer và sysadmin để phát triển, triển khai và chạy các ứng dụng bằng các container. Việc đóng gói thành container này giúp cho việc triển khai các ứng dụng trở nên dễ dàng hơn

- Công nghệ container ngày càng phổ biến bởi:

	- **Linh hoạt**: Có thể đóng gói từ ứng dụng đơn giản đến phức tạp
	- **Nhỏ gọn**: Các container tận dụng, sử dụng chung tài nguyên; kernel của host. Có thể chạy ở mọi nơi, mọi nền tảng.
   - **Khả năng thay đổi linh hoạt**: Cập nhật và nâng cấp nhanh chóng.
   - **Khả năng mở rộng**: Dễ dàng tăng và phân tán tự động các container
   - **Phân tầng dịch vụ**: Mỗi dịch vụ khi deploy sẽ được phân tầng, nằm trên các dịch vụ đang có sẵn. Như vậy sẽ không làm ảnh hưởng tới dịch vụ đang chạy.

### Khái niệm containers và images
- Một container được khởi chạy từ image. Như vậy, image là một gói thực thi chứa bên trong là tất cả những gì cần thiết, liên quan để chạy ứng dụng như mã nguồn, các thư viện, runtime, các biến môi trường và các file cấu hình liên quan.
- Một container là một instance đang chạy được khởi tạo từ image.

### So sánh giữa VM với container
- Container chạy trực tiếp trên môi trường máy chủ như một tiến trình và chia sẻ phần kernel bên dưới dùng chung với máy chủ chứa nó
- VM tạo ra một môi trường giả lập hoàn toàn tách biệt như 1 máy hoàn chỉnh thông qua việc phân bổ tài nguyên của máy chủ, do đó sẽ tốn tài nguyên nhiều hơn cho hệ điều hành của máy ảo

![](https://camo.githubusercontent.com/8cb14a3fc605ac71b9aff74ffb9027b7d003c179/68747470733a2f2f69322e77702e636f6d2f626c6f672e646f636b65722e636f6d2f77702d636f6e74656e742f75706c6f6164732f426c6f672e2d4172652d636f6e7461696e6572732d2e2e564d2d496d6167652d312d31303234783433352e706e673f73736c3d31)

## 2. Cấu trúc và thành phần của Docker

Docker bao gồm:

- **Docker Client**: Giao diện để tương tác giữa người dùng với Docker Daemon - HOST
- **Docker Daemon (HOST)**: Lưu trữ image local và khởi chạy container từ những image đó
- **Docker Hub (registry)**: Nơi lưu trữ các images

![](https://camo.githubusercontent.com/948ae5d9498319b4797adc05bf367ddd1646352c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f302a584b54436d6d6f75346654366f732d632e)

### 2.1 Docker Client
> Docker client dùng để tương tác giữa người dùng và Docker Daemon, Daemon sẽ biên dịch và thực thi các câu lệnh đã tương tác qua Docker client.

**1 Chạy một container từ Image**

`docker run -i -t --name web1 ubuntu`

- Khi chạy **docker pull**, mặc định sẽ tìm trên localhost xem có image nào trùng với yêu cầu trong câu lệnh hay không. Nếu image không có sẵn trên localhost, Docker sẽ tìm kiếm và tải về (pull) từ Registry mặc định.

  - **-i**: Vào chế độ tương tác trực tiếp với Container
  - **-t**: Hiển thị tty
  - **--name**: Đặt tên cho container. Mặc định, nếu không đặt thì sẽ có tên ngẫu nhiên.

**2 Khởi động container ở chế độ chạy nền**

`docker run -d --name web1 ubuntu`

**3 Liệt kê các container**

`docker ps -a`

  - **-a hoặc --all**: Hiển thị toàn bộ số container có trên hệ thống
  - **CONTAINER ID**: ID của container
  - **IMAGE**: Tên của Image khởi tạo
  - **COMMAND**: Câu lệnh chính khi khởi động của container/image
  - **CREATE**: Thời gian container được tạo
  - **STATUS**: Trạng thái của container
  - **PORT**: Cổng của container được ánh xạ với host (HOST:CONTAINER)
  - **NAMES**: Tên của container

**4 Liệt kê các image**

`docker images`

**5 Dừng hoạt động của container**

`docker stop web1`

**6 Khởi động của container**

Có thể thêm -i để có thể tương tác trực tiếp với container.

`docker start web1`

**7 Tương tác với Container đang hoạt động**

`docker attach web1`

Hoặc tương tác sử dụng môi trường **/bin/bash**

`docker exec -it web1 /bin/bash`

**8 Xóa container**

Container chỉ bị xóa khi ở trạng thái dừng hoạt động.

`docker rm web1`

### 2.2 Docker Hub (Registry)
Docker Hub hay thường được gọi là Registry, nơi lưu trữ các image được cộng đồng hoặc các nhà phát triển đóng góp và cung cấp miễn phí, chúng ta có thể tìm các bản images tại đây. Điều này vô cùng tiện lợi, chúng ta chỉ cần pull (tải xuống) các image phục vụ cho nhu cầu ở mọi lúc mọi nơi

Ở ví dụ này, ta sẽ tạo một image và tạo một vài file trong đó rồi đóng gói lại thành một image.

##### Bước 1: Sử dụng một image có sẵn nhỏ gọn là alpine
`docker run -it --name myimage1 alpine`

##### Bước 2: Sau khi vào chế độ tương tác của container, tạo vài file bất kỳ và thoát khỏi container

```
touch a b c d e f
ls
exit
```

##### Bước 3: Lưu lại image tại Host local (commit)
`docker commit myimage1 username/myimage1`

- **username:** Tên đăng nhập Hub 
- **myimage1:** Tên image muốn lưu trên Hub. Có thể kèm theo Tag. 
- VD: `myimage1:latest để đánh dấu image bản mới nhất.`

##### Bước 4: Đăng nhập vào Docker Hub

`docker login`

Nhập user/password để đăng nhập

##### Bước 5: Đưa image lên Hub (push)

`docker push username/myimage1`