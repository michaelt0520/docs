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

### Life circle
![](https://miro.medium.com/max/2612/1*UbAOuq0K1oXxPOgigV9L9A.png)

### Docker container
Là 1 máy ảo được cấu thành từ 1 image. Có thể run, remove thông qua remote client
### Docker Image
Là 1 template tạo ra các container, Nó có thể gói các cài đặt môi trường, Thành 1 cụm duy nhất. đó là image

#### Một số chú ý
Có 02 cách để tạo ra các các mirror container

- **Cách 1:** Tạo một container, chạy các câu lệnh cần thiết và sử dụng lệnh docker commit để tạo ra image mới. Cách này thường không được khuyến cáo.
- **Cách 2:** Viết một Dockerfile và thực thi nó để tạo ra một images. Thường mọi người dùng cách này để tạo ra image.

Các **image** là dạng **file-chỉ-đọc (read only file).** Khi tạo một container mới, trong mỗi container sẽ tạo thêm một lớp có-thể-ghi được gọi là container-layer. Các thay đổi trên container như thêm, sửa, xóa file... sẽ được ghi trên lớp này. Do vậy, từ một image ban đầu, ta có thể tạo ra nhiều máy con mà chỉ tốn rất ít dung lượng ổ đĩa.

Quy tắt đặt tên images: **[REPOSITORY[:TAG]]**

#### Một số lệnh làm việc với images
##### Kiểm tra hoạt động của docker
Sử dụng lệnh `docker run hello-world` để kiểm tra hoạt động của docker trên host

```
sh root@u14-vagrant:~# docker run hello-world
Hello from Docker! This message shows that your installation appears to be working correctly. ....

```

##### Tìm kiếm immages từ Docker HUB
Sử dụng lệnh `docker search` để tìm kiếm các images trên Docker HUB

`docker search ubuntu`

Hoặc tìm chính xác phiên bản

`docker search ubuntu:14.04`

##### Tải images từ Docker Hub về host
Ví dụ tải images có tên là `ubuntu` về host

`docker pull ubuntu`

Để tạo một container từ `image ubuntu`, sử dụng lệnh `docker run ubuntu`

##### Kiểm tra các images tồn tại trên host.
Sử dụng lệnh `docker images` để kiểm tra danh sách các images

##### Tạo container từ images
Trong các tài liệu thường hay sử dụng lệnh docker run hello-world để chạy một container, sau khi chạy xong container này nó sẽ thoát. Tuy nhiên, đa số chúng ta lại cần tương tác nhiều hơn nữa với container (thao tác nhiều hơn)

Để chạy container và tương tác với container ta sử dụng tùy chọn -it trong lệnh docker run. Ví dụ: `docker run -it ubuntu`

#### Push - Pull images using Docker Hub.
Để push 1 image vừa tạo lên hub để chia sẻ với mọi người, thì ta cần tạo 1 tài khoản docker hub và login vào bằng câu lệnh

```
docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username:cosy294
Password:
Login Succeeded
```

Sau khi login thành công ta tiến hành push image lên hub

```
$ docker push cosy294/test
The push refers to a repository [cosy294/test]
255c5f0caf9a: Image successfully pushed
```

Trong đó: **cosy294/test** là tên images. - Lưu ý: Nếu bạn muốn push images lên docker hub thì tên images phải có dạng: **cosy294/test**, trong đó **cosy294** là id dockerhub và **test** là tên repo

Để Pull một image từ Docker Hub: `docker pull {image_name}`

#### Create and use Docker Registry: Local Images Repo.
Tạo môi trường chứa image. Docker đã hỗ trợ chúng ta cài đặt các môi trường này duy nhất trong 1 container. Rất là đơn giản, chúng ta chạy lệnh sau.

`docker run -d -p 5000:5000 --name registry registry:2`

Khi đó, port 5000 sẽ được listen. Mọi thao tác pull, push image sẽ được thực hiện trên port này.

Cấu hình docker để pull, push image từ registry vừa tạo: vi /etc/default/docker

`DOCKER_OPTS="--insecure-registry 172.16.69.239:5000"`

- Trong đó, 172.16.69.239 là địa chỉ ip máy chứa registry tạo ở trên.
- Sau khi cấu hình xong, ta khởi động lại docker

`service docker restart`

Các thao tác pull và push image tương tự ở như là pull, push ở docker hub.

### Docker engine
Quản lý việc tạo image. chạy container, dùng image có sẵn hay tải về, Kết nối container, thêm sữa xoá image và container
### Docker network
![](https://camo.githubusercontent.com/d2c856d1986260ace4a29c8478e17bba09dacc13/687474703a2f2f692e696d6775722e636f6d2f614e534a3639792e706e67)

Khi chúng ta cài đặt Docker, những thiết lập sau sẽ được thực hiện: - Virtual bridge docker0 sẽ được tạo ra - Docker tìm một subnet chưa được dùng trên host và gán một địa chỉ cho docker0

Sau đó, khi chúng ta khởi động một container (với bridge network), một veth (Virtual Ethernet) sẽ được tạo ra nối 1 đầu với docker0 và một đầu sẽ được nối với interface eth0 trên container.

#### Default network
Mặc định khi cài đặt xong, Docker sẽ tạo ra 3 card mạng mặc định là: - **bridge - host - only**.

Tương ứng với các nền tảo ảo hóa khác, ta có các chế độ card mạng của docker so với các nền tảng đấy là:

|General Virtualization Term|Docker Network Driver|
|---------------------------|:-------------------:|
| NAT Network	               |  bridge |
| Bridged                   | macvlan, ipvlan|
| Private / Host-only       | bridge |
| Overlay Network / VXLAN	 | overlay |

Để xem chi tiết, ta có thể dùng lệnh

`docker network ps`

Mặc định khi tạo container mà ta không chỉ định dùng network nào, thì docker sẽ dùng dải bridge.

##### None Network
Các container thiết lập network này sẽ không được cấu hình mạng.

##### Bridge
Docker sẽ tạo ra một switch ảo. Khi container được tạo ra, interface của container sẽ gắn vào switch ảo này và kết nối với interface của host.

`docker network inspect bridge`

##### Host
Containers sẽ dùng mạng trực tiếp của máy host. Network configuration bên trong container đồng nhất với host.

#### User-defined networks
Ngoài việc sử dụng các network mặc định do docker cung cấp. Ta có thể tự định nghĩa ra các dải network phù hợp với công việc của mình.

Để tạo network, ta dùng lệnh

`docker network create --driver bridge --subnet 192.168.1.0/24 bridgexxx`

Trong đó: 

- **--driver bridge**: Chỉ định dải mạng mới được tạo ra sẽ thuộc kiểu nào: bridge, host, hay none. -
- **--subnet**: Chỉ định địa địa chỉ mạng. - bridgexxx: Tên của dải mạng mới

Khi chạy container chỉ định sử dụng 1 dải mạng đặc biệt, ta dùng lệnh

`docker run --network=bridgexxx -itd --name=container3 busybox`

Trong đó: 

- **--network=bridgexxx**: Chỉ định ra dải mạng bridgexxx sẽ kết nối với container.

Container mà bạn chạy trên network này đều phải thuộc về cùng một Docker host. Mỗi container trong network có thể communicate với các containers khác trong cùng network.

#### An overlay network with Docker Engine swarm mode
- Overlay network là mạng có thể kết nối nhiều container trên các Docker Engine lại với nhau, trong môi trường cluster.
- Swarm tạo ra overlay network chỉ available với các nodes bên trong swarm. Khi bạn tạo ra một service sử dụng overlay network, manager node sẽ tự động kế thừa overlay network tới các nodes chạy các service tasks.

Ví dụ sau sẽ hướng dẫn cách tạo ra một network và sử dụng nó cho một service từ một manager node bên trong swarm:

```
# Create an overlay network `my-multi-host-network`.
$ docker network create \
  --driver overlay \
  --subnet 10.0.9.0/24 \
  my-multi-host-network

400g6bwzd68jizzdx5pgyoe95

# Create an nginx service and extend the my-multi-host-network to nodes where
# the service's tasks run.
$ $ docker service create --replicas 2 --network my-multi-host-network --name my-web nginx

716thylsndqma81j6kkkb5aus
```

#### "Nói chuyện" giữa các container với nhau.
- Trên cùng một host, các container chỉ cần dùng bridge network để nói chuyện được với nhau. Tuy nhiên, các container được cấp ip động nên nó có thể thay đổi, dẫn đến nhiều khó khăn. Vì vậy, thay vì dùng địa chỉ ip, ta có thể dùng name của các container để "liên lạc" giữa các container với nhau.
- Trong trường hợp sử dụng default bridge network thì ta khai báo thêm lệnh --link=name_container.
- Trong trường hợp sử dụng user-defined bridge network thì ta không cần phải link nữa.

##### Trường hợp sử dụng default bridge network để kết nối các container
Giả sử ta có mô hình: `web - db`

`container web` phải link được với `container db`.

```
docker run -itd --name=db -e MYSQL_ROOT_PASSWORD=pass mysql:latest
docker run -itd --name=web --link=db nginx:latest
```

Kiểm tra:

```
docker exec -it web sh
# ping redis
PING redis (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: icmp_seq=0 ttl=64 time=0.245 ms
...
# ping db
PING db (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: icmp_seq=0 ttl=64 time=0.126 ms
...
```

##### Trường hợp sử dụng user-defined bridge network để kết nối các container
Bạn không cần thực hiện thao tác link qua link lại giữa các container nữa.

```
docker network create my-net
docker network ls
NETWORK ID          NAME                DRIVER
716f591e185a        bridge              bridge              
4b0041303d6d        host                host                
7239bb9e0255        my-net              bridge              
016cf6ec1791        none                null

docker run -itd --name=web1 --net my-net nginx:latest
docker run -itd --name=db1 --net my-net -e MYSQL_ROOT_PASSWORD=pass mysql:latest

docker exec -it web1 sh
# ping db1
PING db1 (172.18.0.4): 56 data bytes
64 bytes from 172.18.0.4: icmp_seq=0 ttl=64 time=0.161 ms
# ping redis1
PING redis1 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: icmp_seq=0 ttl=64 time=0.168 ms
```


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


### Docker compose
Compose là công cụ giúp định nghĩa và khởi chạy multi-container Docker applications.

Khởi động tất cả các dịch vụ chỉ với 1 câu lệnh duy nhất.

Với 3 bước cơ bản như sau:

- Định nghĩa các ứng dụng thông qua Dockerfile
- Định nghĩa các ứng dụng chạy tách biệt và khởi động cùng nhau trong docker-compose.yml
- Thực thi câu lệnh docker-compose up -d để hoàn tất

##### Cài đặt
install using pip

`pip install docker-compose`

Một file docker-compose.xml mẫu:

```yml
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Sử dụng docker-compose để quản lý vòng đời ứng dụng cụ thể xem và quản lý trạng thái của các service (Start, stop, rebuild,...); chuyển log của các ứng dụng đang chạy.

##### Ví dụ
Chúng ta sẽ tạo ra 2 containers, 1 containers chứa mã nguồn wordpress và 1 containers chưa cơ sở dữ liệu mysql. Bằng cách định nghĩa trong file compose. Chỉ với 1 dòng lệnh khởi tạo, docker sẽ lập tức tạo ra 2 containers và sẵn sàng cho chúng ta dựng lên wordpress, một cách nhanh chóng.

- Đoạn mã compose: Viết theo cú pháp YAML.

```yml
version: '2'

services:
   db:
     image: mysql:5.7
     volumes:
       - ./data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: wordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_PASSWORD: wordpress
```

- **version**: '2': Chỉ ra phiên bản docker-compose sẽ sử dụng.
- services:: Trong mục services, chỉ ra những services (containers) mà ta sẽ cài đặt. Ở đây, tạo sẽ tạo ra services tương ứng với 2 containers là db và wordpress.
Trong services db:
- **image**: chỉ ra image sẽ được sử dụng để create containers. Ngoài ra, bạn có thể viết dockerfile và khai báo lệnh build để containers sẽ được create từ dockerfile.
- **volumes**: mount thư mục data trên host (cùng thư mục cha chứa file docker-compose) với thư mục /var/lib/mysql trong container.
- **restart**: always: Tự động khởi chạy khi container bị shutdown.
environment: Khai báo các biến môi trường cho container. Cụ thể là thông tin cơ sở dữ liệu.
Trong services wordpress:
- **depends_on**: db: Chỉ ra sự phụ thuộc của services wordpress với services db. Tức là services db phải chạy và tạo ra trước, thì services wordpress mới chạy.
- **ports**: Forwards the exposed port 80 của container sang port 8000 trên host machine.
- **environment**: Khai báo các biến môi trường. Sau khi tạo ra db ở container trên, thì sẽ lấy thông tin đấy để cung cấp cho container wordpress (chứa source code).

Khởi chạy

`docker-compose up`


##### Một vài chú ý
- **dockerfile** dùng để build các image.
- **docker-compose** dùng để build và run các container.
- **docker-compose** viết theo cú pháp YAML, các lệnh khai báo trong docker-compose gần tương tự với thao tác chạy container docker run.
- **docker-compose** cung chấp chức năng Horizontally scaled, cho phép ta tạo ra nhiều container giống nhau một cách nhanh chóng. Bằng cách sử dụng lệnh


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

## 4. Lab
### 1. Tạo một container
Sử dụng lệnh docker run để tạo một docker từ image có sẵn. Cú pháp lệnh:

`docker run [options] [image] [command] [args]`

Trong đó, nếu có nhiều image thì chỉ định image cho docker với repository:tag. Nếu không chỉ định thì mặc định sẽ lấy tag là latest

Ví dụ:

`docker run ubuntu:14.04 ps aux`

Tạo container, đồng thời sử dụng terminal của container thì bổ sung tham số: - -i tham số chỉ định docker kết nối tới STDIN trên container - -t tham số chỉ định tạo ra một giả lập terminal.

Ví dụ:

`docker run -i -t ubuntu /bin/bas`

Có một tham số -d để chỉ định cho container sẽ chạy nền hoặc như một daemo, Ví dụ:

`docker run -d ubuntu ping 127.0.0.1 -c 50`

Tạo container ở trên sẽ sinh ra tên một cách ngẫu nhiên, sử dụng tham số --name để đặt tên cho container. Ví dụ

`docker run -it --name ubuntu_example ubuntu /bin/bash`

### 2. Kiểm tra thông tin container
Một container được tạo ra sẽ có rất nhiều thông tin đi kèm. Kiểm tra thông tin ngắn gọn thì sử dụng lệnh:

`docker ps -a`

Lệnh trên sẽ cho bạn biết thông tin về tất cả các container đang chạy hoặc đã dừng: tên và id của container, image mà container sử dụng, trạng thái, thời gian chạy.

Kiểm tra thông tin chi tiết của container thì ta sử dụng lệnh:

`docker inspect [containerID/containerName]`

Bạn có thể thay ID hoặc tên của container vào lệnh trên. Rất nhiều thông tin được hiển thị ra dưới định dạng JSON

Theo dõi các thông tin log mà container xuất ra bằng lệnh:

`docker logs [containerID]`

### 3. Chạy một container với port NAT chỉ định và gắn Volume
Trong trường hợp muốn chỉ định một port từ máy host vào port dịch vụ mà ứng dụng trong container cung cấp dịch vụ thì cần sử dụng tham số -p. Ví dụ:

`docker run --name apache_test1 -p 8080:80 -p 443:443 -d eboraas/apache`

Nếu bạn có một thư mục chứa code hoặc tham số cấu hình trên máy host, muốn ứng dụng trong container sử dụng được thì sử dụng tham số -v để chỉ dẫn cho container biết. Ví dụ:

```
mkdir -p website && cd website

docker run --name apache_test2 -p 8080:80 -p 443:443 -v $(pwd):/var/www/  -d eboraas/apache
```

Kiểm tra việc chỉnh sửa code trên host có được thay đổi ngay trong docker không bằng cách sau:

```
mkdir html

echo "test website at host" >> index.html

curl localhost:8180
```

### 4. Liên kết giữa các container

Bạn có nhiều ứng dụng đang chạy trên các container riêng lẻ, ví dụ: web server, database thì làm cách nào để chúng liên kết lại với nhau. Chúng ta sử dụng tham số --link.

Bây giờ tôi tạo một Drupal app (apache, php, mysql, drupal) bằng cách sau:

```
mkdir -p drupal-link-example && cd drupal-link-example

docker pull drupal:8.0.6-apache
docker pull mysql:5.5

// Start a container for mysql
docker run --name mysql_example \
           -e MYSQL_ROOT_PASSWORD=root \
           -e MYSQL_DATABASE=drupal \
           -e MYSQL_USER=drupal \
           -e MYSQL_PASSWORD=drupal \
           -d mysql:5.5

// Start a Drupal container and link it with mysql
// Usage: --link [name or id]:alias
docker run -d --name drupal_example \
           -p 8280:80 \
           --link mysql_example:mysql \
           drupal:8.0.6-apache

// Open http://localhost:8280 to continue with the installation
// On the db host use: mysql

// There is a proper linking
docker inspect -f "{{ .HostConfig.Links }}" drupal_example
```

### 5. export/save/load container

```
mkdir -p ~/Docker-presentation

docker pull nimmis/alpine-apache
docker run -d --name apache_example \
           nimmis/alpine-apache

// Create a file inside the container.
// See https://github.com/nimmis/docker-alpine-apache for details.
docker exec -ti apache_example \
            /bin/sh -c 'mkdir /test && echo "This is it." >> /test/test.txt'

// Test it. You should see message: "This is it."
docker exec apache_example cat /test/test.txt

// Commit the change.
docker commit apache_export_example myapache:latest

// Create a new container with the new image.
docker run -d --name myapache_example myapache

// You should see the new folder/file inside the myapache_example container.
docker exec myapache_example cat /test/test.txt

// Export the container as image
cd ~/Docker-presentation
docker export myapache_example > myapache_example.tar

// Import a new image from the exported files
cd ~/Docker-presentation
docker import myapache_example.tar myapache:new

// Save a new image as tar
docker save -o ~/Docker-presentation/myapache_image.tar myapache:new

// Load an image from tar file
docker load < myapache_image.tar
```

### 6. upload image lên repository

```
mkdir -p ~/upload && cd ~/upload
git clone git@github.com:theodorosploumis/docker-presentation.git
cd docker-presentation

docker pull nimmis/alpine-apache
docker build -t tplcom/docker-presentation .

// Test it
docker run -itd --name docker_presentation \
           -p 8480:80 \
           tplcom/docker-presentation

// Open http://localhost:8480, you should see this presentation

// Push it on the hub.docker.com
docker push tplcom/docker-presentation
```