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