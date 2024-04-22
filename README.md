# Laravel x Zerops

```yaml
#yamlPreprocessor=on
project:
  name: laravel-jetstream
  tags:
    - laravel

services:
  - hostname: db
    type: postgresql@16
    mode: NON_HA
    priority: 10

  - hostname: redis
    type: keydb@6
    mode: NON_HA
    priority: 10

  - hostname: storage
    type: object-storage
    objectStorageSize: 2
    priority: 10

  - hostname: mailpit
    type: go@1
    buildFromGit: https://github.com/zeropsio/recipe-mailpit
    enableSubdomainAccess: true
    ports:
      - port: 8025
      - port: 1025
        httpSupport: false
    minContainers: 1

  - hostname: adminer
    type: php-apache@8.0+2.4
    buildFromGit: https://github.com/zeropsio/recipe-adminer@main
    enableSubdomainAccess: true
    minContainers: 1
    maxContainers: 1

  - hostname: app
    type: php-nginx@8.3+1.22
    buildFromGit: https://github.com/fxck/zerops-laravel-hello-world
    enableSubdomainAccess: true
    envSecrets:
      APP_NAME: ZeropsLaravel
      APP_DEBUG: true
      APP_ENV: production
      APP_FAKER_LOCALE: en_US
      APP_FALLBACK_LOCALE: en
      APP_KEY: <@generateRandomString(<32>)>
      APP_LOCALE: en
      APP_MAINTENANCE_DRIVER: file
      APP_MAINTENANCE_STORE: database
      APP_TIMEZONE: UTC
      APP_URL: ${zeropsSubdomain}
      ASSET_URL: ${APP_URL}
      VITE_APP_NAME: ${APP_NAME}

      DB_CONNECTION: pgsql
      DB_DATABASE: db
      DB_HOST: db
      DB_PASSWORD: ${db_password}
      DB_PORT: 5432
      DB_USERNAME: ${db_user}

      AWS_ACCESS_KEY_ID: ${storage_accessKeyId}
      AWS_REGION: us-east-1
      AWS_BUCKET: ${storage_bucketName}
      AWS_ENDPOINT: ${storage_apiUrl}
      AWS_SECRET_ACCESS_KEY: ${storage_secretAccessKey}
      AWS_URL: ${storage_apiUrl}/${storage_bucketName}
      AWS_USE_PATH_STYLE_ENDPOINT: true

      LOG_CHANNEL: syslog
      LOG_LEVEL: debug
      LOG_STACK: single

      MAIL_FROM_ADDRESS: hello@example.com
      MAIL_FROM_NAME: ZeropsLaravel
      MAIL_HOST: mailpit
      MAIL_MAILER: smtp
      MAIL_PORT: 1025

      BROADCAST_CONNECTION: redis
      CACHE_PREFIX: cache
      CACHE_STORE: redis
      QUEUE_CONNECTION: redis
      REDIS_CLIENT: phpredis
      REDIS_HOST: redis
      REDIS_PORT: 6379
      SESSION_DRIVER: redis
      SESSION_ENCRYPT: false
      SESSION_LIFETIME: 120
      SESSION_PATH: /

      BCRYPT_ROUNDS: 12
      TRUSTED_PROXIES: "*"
      FILESYSTEM_DISK: s3
    nginxConfig: |-
      server {
          listen 80;
          listen [::]:80;

          server_name _;

          root /var/www/public;

          add_header X-Frame-Options "SAMEORIGIN";
          add_header X-Content-Type-Options "nosniff";

          index index.php;

          charset utf-8;

          location / {
              try_files $uri $uri/ /index.php?$query_string;
          }

          location = /favicon.ico { access_log off; log_not_found off; }
          location = /robots.txt  { access_log off; log_not_found off; }

          error_page 404 /index.php;

          location ~ \\.php$ {
              fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
              fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
              include fastcgi_params;
          }

          location ~ /\\.(?!well-known).* {
              deny all;
          }

          access_log syslog:server=unix:/dev/log,facility=local1 default_short;
          error_log syslog:server=unix:/dev/log,facility=local1;
      }
    minContainers: 1

```
