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

### Docker container
Là 1 máy ảo được cấu thành từ 1 image. Có thể run, remove thông qua remote client
### Docker Image
Là 1 template tạo ra các container, Nó có thể gói các cài đặt môi trường, Thành 1 cụm duy nhất. đó là image
### Life circle
![](https://miro.medium.com/max/2612/1*UbAOuq0K1oXxPOgigV9L9A.png)

### Docker engine
Quản lý việc tạo image. chạy container, dùng image có sẵn hay tải về, Kết nối container, thêm sữa xoá image và container
### Docker volume

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

**Tập hợp các lệnh sử dụng trong Docker**

##### Pull một image từ Docker Hub
`docker pull {image_name}`

##### Liệt kê các images hiện có
`docker images`

##### Xóa một image
`docker rmi {image_id/name}`

##### Liệt kê các container đang chạy
`docker ps`

`docker ps -a #Liệt kê các container đã tắt`

##### Xóa một container
`docker rm -f {container_id/name}`

##### Đổi tên một container
`docker rename {old_container_name} {new_container_name}`

##### Khởi động một container
`docker start {new_container_name}`

`docker exec -it {new_container_name} /bin/bash`

##### Tạo mới một container, đồng thời khởi động với tùy chọn cổng và volume
```
docker run --name {container_name} -p {host_port}:{container_port} -v {/host_path}:{/container_path} -it {image_name} /bin/bash
```
##### Xem các thay đổi trên container
`docker diff {container_name}`

##### Commit các thay đổi trên container và image
`docker commit -m "message" {container_name} {image_name}`

##### Save image thành file .tar
`docker save {image_name} > {/host_path/new_image.tar}`

##### Tạo một image mới từ file .tar
`cat musashi.tar | docker import - {new_image_name}:latest`

##### Xem lịch sử các commit trên image
`docker history {image_name}`

##### Khôi phục lại images từ IMAGE_ID
`docker tag {iamge_id} {image_new_name}:{tag}`

##### Build một image từ container
`docker build -t {container_name} .`

Dấu . ở đây ám chỉ Dockerfile đang nằm trong thư mục hiện tại.

##### Copy file từ host vào container
`docker cp foo.txt mycontainer:/foo.txt`

##### Copy file từ container vào host
`docker cp mycontainer:/foo.txt foo.txt`

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