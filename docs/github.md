![](https://res.cloudinary.com/practicaldev/image/fetch/s--WXHQFqpz--/c_imagga_scale,f_auto,fl_progressive,h_900,q_auto,w_1600/https://thepracticaldev.s3.amazonaws.com/i/vmeforsb7sbldnxw0v58.png)
# Github Action
## Overview
- Github Actions cho phép chúng ta tạo workflows vòng đời phát triển phần mềm cho dự án trực tiếp trên Github repository của chúng ta
- Github Actions giúp chúng ta tự động hóa quy trình phát triển phần mềm tại nơi chúng ta lưu trữ code và kết hợp với pull request và issues. 
- Với Github Actions chúng ta có thể tích hợp continuous integration (CI) và continuous deployment (CD) trực tiếp trên repository của mình

## Using variables and secrets in a workflow
### [Tạo và lưu trữ encrypted secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets)

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

### [Sử dụng biến môi trường](https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables)
Để tuỳ chỉnh các biến môi trường, bạn cần chỉ định các biến trong workflow. Bạn có thể xác định các biến môi trường cho `step`, `job` hoặc toàn bộ `workflow` bằng cách sử dụng các `jobs`

Ví dụ:

```
steps:
  - name: Hello world
    run: echo Hello world $FIRST_NAME $middle_name $Last_Name!
    env:
      FIRST_NAME: Mona
      middle_name: The
      Last_Name: Octocat
```

**Bạn cũng có thể sử dụng `set-env` để đặt biến môi trường**
`::set-env name={name}::{value}`

Tạo hoặc cập nhật một biến môi trường cho bất kỳ hành động nào chạy tiếp theo trong `job`

```
- if: failure()
  run: echo "::set-env name=MESSAGE::Release staging failed, Please try again or release manually"
```

### [Authenticating with the GITHUB_TOKEN](https://help.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token)

## Caching and storing workflow data
**Using the cache action**

The cache action will attempt to restore a cache based on the key you provide. When the action finds a cache, the action restores the cached files to the path you configure.

If there is no exact match, the action creates a new cache entry if the job completes successfully. The new cache will use the key you provided and contains the files in the path directory.

You can optionally provide a list of restore-keys to use when the key doesn't match an existing cache. A list of restore-keys is useful when you are restoring a cache from another branch because restore-keys can partially match cache keys. For more information about matching restore-keys, see "Matching a cache key."

**For more information, see [actions/cache](https://github.com/actions/cache).**

Input 

- **key**: Required The key created when saving a cache and the key used to search for a cache. Can be any combination of variables, context values, static strings, and functions. Keys have a maximum length of 512 characters, and keys longer than the maximum length will cause the action to fail.
- **path**: Required The file path on the runner to cache or restore. The path can be an absolute path or relative to the working directory.
   - With v2 of the cache action, you can specify a single path, or multiple paths as a list. Paths can be either directories or single files, and glob patterns are supported.
   - With v1 of the cache action, only a single path is supported and it must be a directory. You cannot cache a single file.
- **restore-keys**: Optional An ordered list of alternative keys to use for finding the cache if no cache hit occurred for key.

Output

- **cache-hit**: A boolean value to indicate an exact match was found for the key.

**Example Using Cache**

```yml
name: Caching with npm

on: push

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Cache node modules
      uses: actions/cache@v2
      env:
        cache-name: cache-node-modules
      with:
        # npm cache files are stored in `~/.npm` on Linux/macOS
        path: ~/.npm
        key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-build-${{ env.cache-name }}-
          ${{ runner.os }}-build-
          ${{ runner.os }}-

    - name: Install Dependencies
      run: npm install

    - name: Build
      run: npm build

    - name: Test
      run: npm test
```

When key matches an existing cache, it's called a cache hit, and the action restores the cached files to the path directory.

When key doesn't match an existing cache, it's called a cache miss, and a new cache is created if the job completes successfully. When a cache miss occurs, the action searches for alternate keys called restore-keys.

- If you provide restore-keys, the cache action sequentially searches for any caches that match the list of restore-keys.
   - When there is an exact match, the action restores the files in the cache to the path directory.
If there are no exact matches, the action searches for partial matches of the restore keys. 
   - When the action finds a partial match, the most recent cache is restored to the path directory.
- The cache action completes and the next workflow step in the job runs.
- If the job completes successfully, the action creates a new cache with the contents of the path directory.

## Using databases and service containers