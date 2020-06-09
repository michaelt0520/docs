![](https://res.cloudinary.com/practicaldev/image/fetch/s--WXHQFqpz--/c_imagga_scale,f_auto,fl_progressive,h_900,q_auto,w_1600/https://thepracticaldev.s3.amazonaws.com/i/vmeforsb7sbldnxw0v58.png)
# Github Action
## Overview
- Github Actions cho phép chúng ta tạo workflows vòng đời phát triển phần mềm cho dự án trực tiếp trên Github repository của chúng ta
- Github Actions giúp chúng ta tự động hóa quy trình phát triển phần mềm tại nơi chúng ta lưu trữ code và kết hợp với pull request và issues. 
- Với Github Actions chúng ta có thể tích hợp continuous integration (CI) và continuous deployment (CD) trực tiếp trên repository của mình

## Getting started
### Sử dụng biến và khoá bí mật trong flow
#### Tạo và lưu trữ encrypted secrets
**About encrypted secrets**

> Biến secrets là các biến môi trường được mã hóa mà bạn tạo trong kho lưu trữ hoặc tổ chức. Các biến secrets bạn tạo có sẵn để sử dụng trong quy trình công việc GitHub Action. GitHub sử dụng libsodium để giúp đảm bảo rằng các biến  secrets được mã hóa trước khi chúng đến GitHub và vẫn được mã hóa cho đến khi bạn sử dụng chúng trong quy trình làm việc.

**Quy tắc đặt tên**

- Tên chỉ có thể chứa các ký tự chữ và số **([a-z], [A-Z], [0-9])** or **underscores (_)**. Spaces are not allowed.
- Tên không bắt đầu bằng **GITHUB_ prefix** hoặc **Number**

##### [Tạo encrypted secrets trong repo](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets)

1. On GitHub, navigate to the main page of the repository.
2. Under your repository name, click Settings.
![](https://help.github.com/assets/images/help/repository/repo-actions-settings.png)
3. In the left sidebar, click Secrets.
4. Click Add a new secret.
5. Type a name for your secret in the Name input box.
6. Enter the value for your secret.
7. Click Add secret.

[Decrypt](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#limits-for-secrets)