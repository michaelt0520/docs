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
### [Creating service containers](https://help.github.com/en/actions/configuring-and-managing-workflows/about-service-containers)

You can use the services keyword to create service containers that are part of a job in your workflow. For more information, see [jobs.job_id.services.](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idservices)

This example creates a service called redis in a job called `container-job`. The Docker host in this example is the `node:10.18-jessie` container.

```yml
name: Redis container example
on: push

jobs:
  # Label of the container job
  container-job:
    # Containers must run in Linux based operating systems
    runs-on: ubuntu-latest
    # Docker Hub image that `container-job` executes in
    container: node:10.18-jessie

    # Service containers to run with `container-job`
    services:
      # Label used to access the service container
      redis:
        # Docker Hub image
        image: redis
        ports: ['6379:6379']
        options: --entrypoint redis-server
```

### [Creating Redis service containers](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-redis-service-containers)
This guide shows you workflow examples that configure a service container using the Docker Hub redis image. The workflow runs a script to create a Redis client and populate the client with data. To test that the workflow creates and populates the Redis client, the script prints the client's data to the console.

**Running jobs in containers**

```yml
name: Redis container example
on: push

jobs:
  # Label of the container job
  container-job:
    # Containers must run in Linux based operating systems
    runs-on: ubuntu-latest
    # Docker Hub image that `container-job` executes in
    container: node:10.18-jessie

    # Service containers to run with `container-job`
    services:
      # Label used to access the service container
      redis:
        # Docker Hub image
        image: redis
        # Set health checks to wait until redis has started
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      # Downloads a copy of the code in your repository before running CI tests
      - name: Check out repository code
        uses: actions/checkout@v2

      # Performs a clean installation of all dependencies in the `package.json` file
      # For more information, see https://docs.npmjs.com/cli/ci.html
      - name: Install dependencies
        run: npm ci

      - name: Connect to Redis
        # Runs a script that creates a Redis client, populates
        # the client with data, and retrieves data
        run: node client.js
        # Environment variable used by the `client.js` script to create a new Redis client.
        env:
          # The hostname used to communicate with the Redis service container
          REDIS_HOST: redis
          # The default Redis port
          REDIS_PORT: 6379
```

### [Creating PostgreSQL service containers](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-postgresql-service-containers)

When you run a job directly on the runner machine, you'll need to map the ports on the service container to ports on the Docker host. You can access service containers from the Docker host using localhost and the Docker host port number.

You can copy this workflow file to the `.github/workflows` directory of your repository and modify it as needed

```yml
name: PostgreSQL Service Example
on: push

jobs:
  # Label of the runner job
  runner-job:
    # You must use a Linux environment when using service containers or container jobs
    runs-on: ubuntu-latest

    # Service containers to run with `runner-job`
    services:
      # Label used to access the service container
      postgres:
        # Docker Hub image
        image: postgres
        # Provide the password for postgres
        env:
          POSTGRES_PASSWORD: postgres
        # Set health checks to wait until postgres has started
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          # Maps tcp port 5432 on service container to the host
          - 5432:5432

    steps:
      # Downloads a copy of the code in your repository before running CI tests
      - name: Check out repository code
        uses: actions/checkout@v2

      # Performs a clean installation of all dependencies in the `package.json` file
      # For more information, see https://docs.npmjs.com/cli/ci.html
      - name: Install dependencies
        run: npm ci

      - name: Connect to PostgreSQL
        # Runs a script that creates a PostgreSQL client, populates
        # the client with data, and retrieves data
        run: node client.js
        # Environment variable used by the `client.js` script to create
        # a new PostgreSQL client.
        env:
          # The hostname used to communicate with the PostgreSQL service container
          POSTGRES_HOST: localhost
          # The default PostgreSQL port
          POSTGRES_PORT: 5432
```

## [Migrating from CircleCI to GitHub Actions](https://help.github.com/en/actions/migrating-to-github-actions/migrating-from-circleci-to-github-actions)