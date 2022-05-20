# Docker Laravel

1. Cài đặt
    1. Dockerfile
        
        ```bash
        FROM php:7.2-fpm-alpine
        
        WORKDIR /var/www/html
        
        # Install PHP Composer
        RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
        
        COPY . .
        ```
        
        - Không cần CMD vì bên trong php:7.2-fpm-alpine đã có CMD rồi.
    2. build image
        
        ```bash
        $ docker build -t learning-docker/docker-laravel:v1 .
        ```
        
    3. docker-compose.yml
        
        ```bash
        version: '3.4'
        
        services:
          #PHP Service
          app:
            image: learning-docker/docker-laravel:v1
            restart: unless-stopped
            volumes:
              - ./:/var/www/html
        
          #Nginx Service
          webserver:
            image: nginx:1.17-alpine
            restart: unless-stopped
            ports:
              - "8000:80"
            volumes:
              - ./:/var/www/html
              - ./nginx.conf:/etc/nginx/conf.d/default.conf
        ```
        
    4. nginx.conf
        
        ```bash
        server {
            listen 80;
            index index.php index.html;
        
            error_log  /var/log/nginx/error.log;
            access_log /var/log/nginx/access.log;
        
            root /var/www/html/public;
        
            location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass app:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
                fastcgi_hide_header X-Powered-By;
            }
        
            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }
        }
        ```
        
        - **fastcgi_pass app:9000;** có nghĩ khi có request gửi đến nginx, nginx sẽ chuyển request đó tới PHP-FPM đang lắng nghe ở host tên là app và port 9000, app ở đây nắm giữ địa chỉ của service app (container app).
        - EXPOSE 9000: nhằm mục đích chấp nhận các container khác tương tác thông qua tên service và port 9000.
    5. Tạo .env
        
        ```bash
        APP_NAME=Laravel
        APP_ENV=local
        APP_KEY=
        APP_DEBUG=true
        APP_URL=http://localhost
        
        LOG_CHANNEL=stack
        
        DB_CONNECTION=mysql
        DB_HOST=127.0.0.1
        DB_PORT=3306
        DB_DATABASE=laravel
        DB_USERNAME=root
        DB_PASSWORD=
        
        BROADCAST_DRIVER=log
        CACHE_DRIVER=file
        QUEUE_CONNECTION=sync
        SESSION_DRIVER=cookie
        SESSION_LIFETIME=120
        
        REDIS_HOST=127.0.0.1
        REDIS_PASSWORD=null
        REDIS_PORT=6379
        
        MAIL_DRIVER=smtp
        MAIL_HOST=smtp.mailtrap.io
        MAIL_PORT=2525
        MAIL_USERNAME=null
        MAIL_PASSWORD=null
        MAIL_ENCRYPTION=null
        MAIL_FROM_ADDRESS=null
        MAIL_FROM_NAME="${APP_NAME}"
        
        AWS_ACCESS_KEY_ID=
        AWS_SECRET_ACCESS_KEY=
        AWS_DEFAULT_REGION=us-east-1
        AWS_BUCKET=
        
        PUSHER_APP_ID=
        PUSHER_APP_KEY=
        PUSHER_APP_SECRET=
        PUSHER_APP_CLUSTER=mt1
        
        MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
        MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
        ```
        
    6. run
        
        ```bash
        $ docker-compose up
        ```
        
    7. composer install
        
        ```bash
        $ docker-compose exec app composer install
        ```
        
    8. Thay đổi permission với user www-data
        
        ```bash
        $ sudo chown -R 82:82 .
        ```
        
        - để trùng với user www-data trong nginx
2. Khác
    1. Không copy file nginx.conf trong service app (sử dụng .dockerignore)
        1. .dockerignore
            
            ```bash
            nginx.conf
            ```
            
        2. set quyền cho file .dockerignore
            
            ```bash
            $ sudo chown 82:82 .dockerignore
            ```
            
        3. build image
        4. run
3. Tài liệu tham khảo
    1. [https://viblo.asia/p/dockerize-ung-dung-laravel-vyDZOao75wj](https://viblo.asia/p/dockerize-ung-dung-laravel-vyDZOao75wj)