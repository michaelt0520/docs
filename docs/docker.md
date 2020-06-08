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
### Docker network

### Docker volume
- Volume trong Docker được dùng để chia sẻ dữ liệu cho container. 
- Để sử dụng volume trong docker dùng cờ hiệu (flag) -v trong lệnh docker run.

#### Sử dụng volume để gắn (mount) một thư mục nào đó trong host với container.
Ví dụ ta sẽ gắn một thư mục **/var/datahost** vào trong **container**.

```
# Bước 1: Tạo ra thư mục trên host
mkdir /var/datahost

# Bước 2: Khởi tạo một container và chỉ ra volume được gắn
docker run -it -v /var/datahost ubuntu
```
Ở ví dụ trên ta đã thực hiện gắn một thư mục vào trong một container. Ta có thể kiểm tra bằng việc tạo ra dữ liệu trên host và kiểm tra ở container hoặc ngược lại.

#### Sử dụng volume để chia sẻ dữ liệu giữa host và container
Trong tình huống này thư mục trên máy host (máy chứa container) sẽ được mount với một thư mục trên container, dữ liệu sinh ra trên thư mục được mount của container sẽ xuất hiện trên thư mục của host.

Ví dụ: Các bước như sau:

- Tạo ra thư mục bindthis trên host, có đường dẫn /root/bindthis.
- Thư mục /root/bindthis này sẽ được mount với thư mục /var/www/html/webapp nằm trên container.
- Tạo ra 1 file trong thư mục /var/www/html/webapp trên container.
- Kiểm tra xem trong thư mục /root/bindthis trên host có hay không.

```
root@compute3:~# mkdir bindthis
root@compute3:~# ls bindthis/ 
root@compute3:~# docker run -it -v $(pwd)/bindthis:/var/www/html/webapp ubuntu bash
root@13aa90503715:/#
root@13aa90503715:/# touch /var/www/html/webapp/index.html
root@13aa90503715:/# exit
exit
root@compute3:~# ls bindthis/
index.html
root@compute3:~#
```
Có 2 chế độ chia sẻ volume trong docker, đó là read-write (rw) hoặc read-only (ro). Nếu không chỉ ra thì mặc định sử dụng chế độ read-write. Ví dụ chế độ read-only, sử dụng tùy chọn ro.

`docker run -it -v $(pwd)/bindthis:/var/www/html/webapp:ro ubuntu bash`

#### Sử dụng volume để chia sẽ dữ liệu giữa các container
Tạo container chứa volume

`docker create -v /linhlt --name volumecontainer ubuntu`

Tạo container khác sử dụng container *volumecontainer* làm volume. Khi đó, mọi sự thay đổi trên container mới sẽ được cập nhật trong container *volumecontainer*

`docker run -t -i --volumes-from volumecontainer ubuntu /bin/bash`

Trong đó, tùy chọn --volumes-from chỉ ra tên của container sẽ được map volume.

#### Backup và Restore volume.
**Backup**

`docker run --rm --volumes-from volumecontainer -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /linhlt`

Lệnh này sẽ backup thư mục volume là /linhlt trong container volumecontainer và nén lại dưới dạng file backup.tar.

**Restore**

`docker run -v /linhlt --name data-container ubuntu /bin/bash`

Tạo volume /linhlt trên container data-container.

```
docker run --rm --volumes-from data-container -v $(pwd):/backup ubuntu bash -c "cd /linhlt && tar -zxvf /backup/backup.tar"
```
-  Tạo container thực hiện nhiệm vụ giải nén file backup.tar vào thư mục /linhlt. Container này có liên kết với container data-container ở trên.

#### Các chú ý về volume trong Docker
- Đường dẫn trong cờ hiệu -v phải là đường dẫn tuyệt đối, thường dùng $(pwd)/ten_duong_dan để chỉ đúng đường dẫn.
- Có thể chỉ định việc mount giữa thư mục trên host và thư mục trên container ở các chế độ read-wirte hoặc read-only, mặc định là read-write.
- Để chia sẻ volume dùng tùy chọn --volumes-from

### Dockerfile
- Dockerfile là một tập tin dạng text chứa một chuỗi các câu lệnh, chỉ thị để tạo nên một image. Dockerfile bao gồm các câu lệnh liên tiếp thực hiện tự động dựa trên một image có sẵn để tạo ra một image mới

- Trong Dockerfile có các câu lệnh chính sau:

```
FROM
RUN
CMD
....còn nữa
```

##### FROM
Dùng để chỉ ra image được build từ đâu (từ image gốc nào)

```
FROM ubuntu
hoặc có thể chỉ rõ tag của image gốc
FROM ubuntu14.04:lastest
```

##### RUN
Dùng để chạy một lệnh nào đó khi build image, ví dụ về một Dockerfile

```
FROM ubuntu
RUN apt-get update
RUN apt-get install curl -y
```

##### CMD
Lệnh CMD dùng để truyền một lệnh của Linux mỗi khi thực hiện khởi tạo một container từ image (image này được build từ Dockerfile)

```
#Cách 1
FROM ubuntu
RUN apt-get update
RUN apt-get install curl -y
CMD ["curl", "ipinfo.io"]
```

```
#Cách 2
FROM ubuntu
RUN apt-get update
RUN apt-get install wget -y
CMD curl ifconfig.io
```

##### LABEL
`LABEL <key>=<value> <key>=<value> <key>=<value> ...`

Chỉ thị LABEL dùng để add các metadata vào image.

```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

##### MAINTAINER
`MAINTAINER <name>`

Dùng để đặt tên cho tác giả

##### EXPOSE
`EXPOSE <port> [<port>...]`

Lệnh EXPOSE thông báo cho Docker rằng image sẽ lắng nghe trên các cổng được chỉ định khi chạy. Lưu ý là cái này chỉ để khai báo, chứ ko có chức năng nat port từ máy host vào container. Muốn nat port, thì phải sử dụng cờ -p (nat một vài port) hoặc -P (nat tất cả các port được khai báo trong EXPOSE) trong quá trình khởi tạo contrainer.

##### ENV
```
ENV <key> <value>
ENV <key>=<value> ...
```

Khai báo cáo biến giá trị môi trường. Khi run container từ image, các biến môi trường này vẫn có hiệu lực.

##### ADD
```
ADD has two forms:
ADD <src>... <dest>
ADD ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

- Chỉ thị ADD copy file, thư mục, remote files URL (src) và thêm chúng vào filesystem của image (dest)
- src: có thể khai báo nhiều file, thư mục, có thể sử dụng các ký hiệu như *,?,...
- dest: phải là đường dẫn tuyệt đối hoặc có quan hệ với chỉ thị WORKDIR

##### COPY
```
COPY <src>... <dest>
COPY ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

Chỉ thị COPY, copy file, thư mục (src) và thêm chúng vào filesystem của container (dest).

##### ENTRYPOINT
```
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
ENTRYPOINT command param1 param2 (shell form)
```

Hai cái CMD và ENTRYPOINT có tác dụng tương tự nhau. Nếu một Dockerfile có cả CMD và ENTRYPOINT thì CMD sẽ thành param cho script ENTRYPOINT. Lý do người ta dùng ENTRYPOINT nhằm chuẩn bị các điều kiện setup như tạo user, mkdir, change owner... cần thiết để chạy service trong container.

##### VOLUME
`VOLUME ["/data"]`

mount thư mục từ máy host và container. Tương tự option -v khi tạo container.
Thư mục chưa volumes là /var/lib/docker/volumes/. Ứng với mỗi container sẽ có các thư mục con nằm trong thư mục này. Tìm thư mục chưa Volumes của container sad_euclid:

##### USER

`USER daemon`

Set username hoặc UID để chạy các lệnh RUN, CMD, ENTRYPOINT trong dockerfiles.

##### WORKDIR
`WORKDIR /path/to/workdir`

Chỉ thị WORKDIR dùng để đặt thư mục đang làm việc cho các chỉ thị khác như: RUN, CMD, ENTRYPOINT, COPY, ADD,...

##### ARG
`ARG <name>[=<default value>]`

Chỉ thị ARG dùng để định nghĩa các giá trị của biến được dùng trong quá trình build image (lệnh docker build --build-arg =).
biến ARG sẽ không bền vững như khi sử dụng ENV.

##### STOPSIGNAL
`STOPSIGNAL signal`

Gửi tín hiệu cho container tắt đúng cách

##### SHELL
`SHELL ["executable", "parameters"]`

- Chỉ thị Shell cho phép các shell form khác có thể ghi đè shell mặc định.
- Mặc định trên Linux là ["/bin/sh", "-c"] và Windows là ["cmd", "/S", "/C"].

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

## 3. Thực hành