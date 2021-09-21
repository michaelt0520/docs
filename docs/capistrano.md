![](https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcSkddLpg0l5CqPr0-SHfF1iWTYU56otjuQKd9aM0C4TmsILnolZ&usqp=CAU)

# Capistrano deploy
[Capistrano](https://capistranorb.com/documentation/getting-started/local-tasks/) | [Github](https://github.com/capistrano/capistrano)

### Step 1: Add gem && bundle install
* **Gemfile**

```ruby
group :development
  gem 'capistrano', '~> 3.14', require: false
  gem 'capistrano-rails', '~> 1.6', require: false
  gem 'capistrano-rbenv'
  gem 'capistrano-bundler', '~> 2.0', require: false
  gem 'capistrano3-puma', require: false
  gem 'capistrano-sidekiq', require: false
  gem 'capistrano-yarn', require: false
end
```

`bundle install`

* Make sure your project doesn't already have a "Capfile" or "capfile" present. Then run:

`bundle exec cap install`

* To customize the stages that are created, use:

`bundle exec cap install STAGES=local,sandbox,qa,production`

### Step 2: Config Capfile & deploy.rb staging.rb
* **Capfile**

```ruby
# Load DSL and set up stages
require 'capistrano/setup'

# Include default deployment tasks
require 'capistrano/deploy'
require 'capistrano/rbenv'
require 'capistrano/bundler'
require 'capistrano/sidekiq'
install_plugin Capistrano::Sidekiq
install_plugin Capistrano::Sidekiq::Upstart
require 'capistrano/rails/migrations'
require 'capistrano/rails/assets'
require 'capistrano/puma'
install_plugin Capistrano::Puma
install_plugin Capistrano::Puma::Daemon
require 'seed-fu/capistrano3'
require 'capistrano/scm/git'
install_plugin Capistrano::SCM::Git

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }

```

* **Deploy.rb**

```ruby
# config valid only for current version of Capistrano
lock '~> 3.14.1'

# Default value for :format is :airbrussh.
set :format, :airbrussh

# You can configure the Airbrussh format using :format_options.
set :format_options, command_output: true, log_file: 'log/capistrano.log', color: :auto, truncate: :auto

# Default value for :pty is false
set :pty, false

# Default value for :linked_files is []
set :linked_files, ['config/database.yml', 'config/master.key', ".env.#{fetch(:stage)}"]

# Default value for linked_dirs is []
set :linked_dirs, ['log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'bundle', 'public/packs', 'config/puma', 'node_modules', 'public/uploads']
# Default value for default_env is {}
# set :default_env, { path: "/opt/ruby/bin:$PATH" }

# Default value for keep_releases is 5
set :keep_releases, 3
set :local_user, 'wkx2'
set :use_sudo, false

namespace :yarn do
  desc 'Run rake yarn install'
  task :install do
    on roles(:web) do
      within release_path do
        execute("cd #{release_path} && yarn install")
      end
    end
  end
end

namespace :assets do
  desc 'Precompile assets locally and then rsync to deploy server'
  task :precompile do
    on roles(:all) do
      run_locally do
        execute "RAILS_ENV=#{fetch(:stage)} bundle exec rails assets:precompile"
        execute "rsync -av ./public/packs/ #{fetch(:user)}@#{fetch(:server)}:#{current_path}/public/packs/"
        execute 'rm -rf public/packs'
      end
    end
  end
end

namespace :deploy do
  desc 'Upload yml file.'
  task :upload_yml do
    on roles(:app) do
      execute "mkdir -p #{shared_path}/public"
      execute "mkdir -p #{shared_path}/config"
      execute "mkdir -p #{shared_path}/config/puma"
      upload!('package.json', "#{shared_path}/package.json")
      upload!("config/puma/#{fetch(:stage)}.rb", "#{shared_path}/config/puma/#{fetch(:stage)}.rb")
      upload!(".env.#{fetch(:stage)}", "#{shared_path}/.env.#{fetch(:stage)}")
      upload!('config/database.yml', "#{shared_path}/config/database.yml")
      upload!('config/master.key', "#{shared_path}/config/master.key")
    end
  end
end

namespace :maintenance do
  task :on do
    on roles(:app) do
      invoke 'puma:stop'
      execute "cd #{shared_path}/public && touch maintenance"
    end
  end

  task :off do
    on roles(:app) do
      execute "cd #{shared_path}/public && rm -rf maintenance"
    end
  end
end

namespace :db do
  desc 'Seed the database.'
  task :seed do
    on roles(:app) do
      within current_path do
        with(rails_env: fetch(:stage)) do
          execute :bundle, :exec, :rake, 'db:seed_fu'
        end
      end
    end
  end

  desc 'Reset database'
  task :reset do
    on roles(:app) do
      within current_path do
        with(rails_env: fetch(:stage)) do
          execute :bundle, :exec, :rake, 'db:drop db:create db:migrate'
        end
      end
    end
  end
end

task :log do
  on roles(:app) do
    execute "cd #{shared_path}/log && tail -f #{fetch(:stage)}.log"
  end
end

task :update_crontab do
  on roles(:app) do
    execute "cd #{current_path} && whenever --update-crontab --set environment=#{fetch(:stage)}"
  end
end

before('deploy', 'maintenance:on')

after('deploy:finished', 'maintenance:off')

```

### Step 3: Create instace EC2 in AWS
* Set group security with ssh port **22** and http port **80**
* Before create you should download file **your-instance.pem**

```
chmod 400 your-instance.pem
ssh -i path/to/your-instance.pem ubuntu@your-public-dns
```
### Step 4: Update OS

`sudo apt-get update && sudo apt-get -y -upgrade `

* Create a new user & set password in ubuntu server

```
sudo useradd -d /home/username -m username
sudo passwd username
```

### Step 5: Run sudo visudo command in server
`sudo visudo`

* chỉnh phân quyền cho user username **username ALL=(ALL:ALL)ALL**

### Step 6: Login as deploy user and create public key

```
sudo su - username
ssh-keygen
```

### Step 7: Copy your local machine public key and paste in server authorized_keys
* Trên máy local

```
ssh-keygen -t rsa -C <local-name>
cd ~/.ssh
ssh-add ~/.ssh/id_rsa
cat id_rsa.pub
```

* Copy id_rsa bỏ trong **authorized_keys** trong **~/.ssh** trên server

```
cd ~/.ssh
nano authorized_keys
```

```
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys
restorecon -r -vv .ssh/authorized_keys
```

* Trên máy local

```
ssh-keygen -t rsa -C key_rsa_name
cd ~/.ssh
ssh-add ~/.ssh/key_rsa_name
cat key_rsa_name.pub
```

* Copy key_rsa_name.pub bỏ trong **authorized_keys** trong **~/.ssh** trên server

```
cd ~/.ssh
nano authorized_keys
```

* Copy key_rsa_name.pub bỏ trong repo setting deploy keys

### Step 8: Install git
```
sudo apt-get update
sudo apt-get install git
```

### Step 9: Install node (version 12) & yarn
```
sudo apt-get install curl autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm6 libgdbm-dev libdb-dev

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install nodejs -y

curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get install yarn

exec $SHELL
```

### Step 10: Install minimagick
```
sudo apt-get install ruby-mini-magick
```

### Step 11: Install letsencrypt
```
sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
sudo certbot --nginx -d domain.name
```

### Step 12: Setup htpasswd
```
sudo apt install apache2-utils

sudo htpasswd -c /password/.htpasswd username
```

### Step 13: Nginx in server

```
sudo apt-get install nginx
```

* Edit your nginx config file

```
sudo rm /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/conf.d/cms.conf
```

**Nginx file**

```bash
upstream wkx2_puma {
  server unix:/data/staging/wakuwaku-cms/shared/tmp/sockets/puma.sock fail_timeout=0;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2 ipv6only=on;
  ssl_certificate /etc/letsencrypt/live/admin.stg-wakuwaku.net/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/admin.stg-wakuwaku.net/privkey.pem; # managed by Certbot

  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

  server_name admin.stg-wakuwaku.net editor.stg-wakuwaku.net translator.stg-wakuwaku.net;

  root /data/staging/wakuwaku-cms/current/public;

  charset utf-8;

  # Include the basic h5bp config set
  # include h5bp/basic.conf;

  try_files $uri/index.html $uri @wkx2_puma;

  error_page 503 /503.html;

  location @wkx2_puma {
    if (-f /data/staging/wakuwaku-cms/shared/public/maintenance) {
      return 503;
    }

    auth_basic "Restricted";
    auth_basic_user_file /password/.htpasswd;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_pass http://wkx2_puma;
  }

  location /cable {
    proxy_pass http://wkx2_puma;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";

    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto https;
    proxy_redirect off;
 }

  location ~ \.(?:json|ico|jpg|css|jpeg|png|js|swf|woff|otf|eot|svg|ttf|html|gif)$ {
    access_log  off;
    log_not_found off;
    add_header  Pragma "public";
    add_header  Cache-Control "public";
    expires     365d;
  }

  if ($scheme = http) {
    return 301 https://$server_name$request_uri;
  }

  location = /ads.txt {
    return 403;
  }

  client_max_body_size 100M;
  keepalive_timeout 10;
}
```

```
sudo nano /etc/nginx/conf.d/assets.conf
```

**Nginx file**
```bash
server {
  listen 443 ssl;

  server_name assets.stg-wakuwaku.net;
  root /data/staging/wakuwaku-cms/current/public;

  error_page 503 /503.html;
  ssl_certificate /etc/letsencrypt/live/admin.stg-wakuwaku.net/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/admin.stg-wakuwaku.net/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

  location /packs/ {
    root /data/staging/wakuwaku-cms/current/public;
    expires max;
    break;
  }

  location ~ \.(?:json|ico|jpg|css|jpeg|png|js|swf|woff|otf|eot|svg|ttf|html|gif)$ {
     access_log  off;
     log_not_found off;
     add_header  Pragma "public";
     add_header  Cache-Control "public";
     expires     365d;
  }

}
```

### Step 14: Postgres Install
* Install postgres

`sudo apt-get install postgresql postgresql-contrib libpq-dev`

* Create user in postgres

`sudo -u postgres createuser -s "user-name-data"`

* Set password

```
sudo -u postgres psql
\password "user-name-data"
```

* Creata database

`sudo -u postgres createdb -O "user-name-data" "database-name"`

### Step 15: Install rbenv && ruby
```
curl -sL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer | bash -
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL
rbenv install 2.7.2
rbenv global 2.7.2
```

### Step 16: Setup timezone
```
sudo timedatectl set-timezone Asia/Ho_Chi_Minh
```

### Step 17: Create project folder, database.yml, application.yml, master.key

```
pwd
mkdir /data/project_app
mkdir - p /home/username/project_app/shared/config
```

* Database

`nano project_app/shared/config/database.yml`

**database.yml**

```ruby
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

production:
  <<: *default
  database: <%= ENV['DATABASE_NAME'] %>
  username: <%= ENV['DATABASE_USER'] %>
  password: <%= ENV['DATABASE_PASSWORD'] %>
  host: <%= ENV['DATABASE_HOST'] %>
  port: 5432
  connect_timeout: 1
  checkout_timeout: 5
  variables:
    statement_timeout: 5000
```

**application.yml**

* generate serect_key and copy to application.yml in server
* thư mục project run RAILS_ENV=production rake secret SECRET_KEY_BASE: "<your_secret_key_base>"

`nano project_app/shared/config/application.yml`

**Rails > 5.2**

`application.yml -> master.key`

#### REST NGINX

`apt-get purge nginx*`
