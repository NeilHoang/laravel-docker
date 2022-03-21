Để có thể chạy được cần những thành phần sau:

Webserver: Sử dụng Nginx.
Database: Sử dụng MySQL.
PHP-FPM: Trình phiên dịch của PHP.



Cấu trúc thư mục 

    .docker
        data 
            db
            logs
        nginx
            conf.d
                default.conf
        php-fpm
            Dockerfile
        workspace
            Dockerfile
    src
        docker-compose.yml
        README.md
        .dockerignore (ko co cung dc =)))



1  Ở trong thư mục con conf.d, tạo một file là default.conf

   add code vao default.conf
                server {
                    listen 80;
                    index index.php index.html;
                    server_name _;
                    root /var/www/html/public;
                
                    location / {
                        try_files $uri $uri/ /index.php?$query_string;
                    }
                
                    location ~ .php$ {
                        try_files $uri =404;
                        fastcgi_split_path_info ^(.+.php)(/.+)$;
                        fastcgi_pass php-fpm:9000;
                        fastcgi_index index.php;
                        include fastcgi_params;
                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                        fastcgi_param PATH_INFO $fastcgi_path_info;
                    }
                }

2  Tiếp theo  tạo ra một Dockerfile ở trong thư mục .docker/nginx
                    # Sử dụng image nginx:1.19-alpine
                    FROM nginx:1.19-alpine 
                    
                    # Copy toàn bộ nội dung ở trong default.conf vào file default.conf trong container
                    ADD ./.docker/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
                    
                    # Tạo thư mục /var/www/html
                    RUN mkdir -p /var/www/html


3  Đối với PHP-FPM, tạo ra một Dockerfile ở trong thư mục .docker/php-fpm

                    # Sử dụng image php:8.0-fpm-alpine
                    FROM php:8.0-fpm-alpine
                    
                    LABEL maintainer="DucLS &lt;<a class="__cf_email__" href="/cdn-cgi/l/email-protection">[email protected]</a>&gt;"
                    
                    ARG DEBIAN_FRONTEND=noninteractive
                    
                    # Cài đặt những thư viện cần thiết
                    RUN docker-php-ext-install 
                    bcmath 
                    pdo_mysql
                    
                    # Tạo thư mục /var/www/html
                    RUN mkdir -p /var/www/html
                    
                    # Copy toàn bộ file trong thư mục ./src ở máy local vào trong thư mục /var/www/html ở trong container
                    COPY ./src /var/www/html


4   Ngoài 3 thành phần chính trên, sử dụng thêm một container khác, và đặt tên là Workspace, với container này, sẽ cài đặt Composer, Git, Nano, Vim,… Và sẽ thao tác với app Laravel thông qua container này


5  tạo ra Dockerfile ở trong thư mục .docker/workspace

                    FROM composer:2.0
                    
                    FROM php:8.0
                    
                    LABEL maintainer="DucLS &lt;<a class="__cf_email__" href="/cdn-cgi/l/email-protection">[email protected]</a>&gt;"
                    
                    ARG DEBIAN_FRONTEND=noninteractive
                    
                    # Cài đăt composer
                    COPY --from=composer:2.0 /usr/bin/composer /usr/bin/composer
                    
                    RUN apt-get update &amp;&amp; apt-get install -y 
                    software-properties-common locales
                    
                    # Cài đặt các tool cần thiết
                    RUN apt-get update &amp;&amp; apt-get install -y 
                    git 
                    curl 
                    vim 
                    nano 
                    net-tools 
                    pkg-config 
                    iputils-ping 
                    apt-utils 
                    zip 
                    unzip
                    
                    # Cài đặt các thư viện cần thiết
                    RUN docker-php-ext-install 
                    bcmath 
                    pdo_mysql
                    
                    
                    #Tạo thư mục /var/www/html
                    RUN mkdir -p /var/www/html
                    
                    #Copy toàn bộ source code ở folder ./src ở local vào thư mục /var/www/html trong container
                    COPY ./src /var/www/html


6   Tiến hành build và run multi-container bằng cách sử dụng docker-compose

                    version: '3'

                    services:
                    webserver:
                        build:
                        context: .
                        dockerfile: .docker/nginx/Dockerfile
                        container_name: laravel_nginx
                        volumes:
                        - ./src:/var/www/html
                        - .docker/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf
                        - .docker/data/log/nginx:/var/log/nginx
                        ports:
                        - 8080:80
                        depends_on:
                        - mysql
                        - php-fpm

                    php-fpm:
                        build:
                        context: .
                        dockerfile: .docker/php-fpm/Dockerfile
                        container_name: laravel_php-fpm
                        volumes:
                        - ./src:/var/www/html

                    mysql:
                        image: mysql
                        container_name: laravel_mysql
                        restart: unless-stopped
                        volumes:
                        - .docker/data/db:/var/lib/mysql
                        ports:
                        - "33060:3306"
                        environment:
                        MYSQL_DATABASE: api-test
                        MYSQL_ROOT_PASSWORD: root

                    workspace:
                        build:
                        context: .
                        dockerfile: .docker/workspace/Dockerfile
                        container_name: laravel_workspace
                        volumes:
                        - ./src:/var/www/html
                        working_dir: /var/www/html
                        tty: true

                    phpmyadmin:
                        image: phpmyadmin
                        container_name: laravel_admin
                        depends_on:
                        - mysql
                        environment:
                        PMA_HOST: mysql
                        MYSQL_ROOT_PASSWORD: root
                        restart: always
                        ports:
                        - 8081:80
                        volumes:
                        - /sessions
                        - .docker/php/php.ini:/usr/local/etc/php/conf.d/php-phpmyadmin.ini


7  Ngoài docker-compose.yml,  tạo một file là .dockerignore, nhưng thư mục trong file này sẽ được bỏ qua trong quá trình build và run docker, file này có nội dung như sau


            .docker/data/




8 Sau khi đã build xong file docker-compose.yml, sẽ tiến hành build các container bằng command: docker-compose build

Sau khi build xong,  khởi chạy container bằng command: docker-compose up

Sau khi khởi chạy thành công, các bạn cần làm các bước sau:

Copy nôi dụng file .env.example sang file .env: cp .env.example .env.
Config file .env với nội dung sau:


                                DB_CONNECTION=mysql
                                # Bởi vì container chạy MySql mình đặt tên là laravel_mysql
                                DB_HOST=laravel_mysql
                                
                                # Port 3306 đã được mở sẵn trong container 
                                DB_PORT=3306
                                
                                # Tên DB mình config trong docker-compose
                                DB_DATABASE= ten DB
                                DB_USERNAME=root
                                DB_PASSWORD=root



9    chạy câu lệnh sau: docker container exec laravel_workspace php artisan key:generate 
                        docker container exec laravel_workspace php artisan migrate
                        docker container exec laravel_workspace composer install
                        chmod -R 777 ./src/storage
                        dùng trình duyệt vào localhost:8080,