## docker_guide
도커를 사용하여 Load Balance를 직접 구현해보자

## 환경 설정
- install tools
```
sudo apt update
sudo apt install docker.io
sudo apt install docker-compose
```
- directory 생성
```
mkdir WEB_LB
cd WEB_LB
mkdir nginx_lb
mkdir web01
mkdir web02
mkdir web03
```
## nginx_lb 디렉토리 작업
```
cd nginx_lb
sudo vim Dockerfile
```
- nginx_lb 디렉토리의 Dockerfile
```
FROM nginx:1.18
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
```
- nginx_lb 디렉토리의 nginx.conf 파일 작성
```
upstream backend-lb {
                server 172.17.0.1:8001;
                server 172.17.0.1:8002;
                server 172.17.0.1:8003;
}

server {
        location / {
                proxy_pass http://backend-lb;
        }
}
```

## web01 디렉토리 작업
'''
cd ..
cd web01
sudo vim Dockerfile
'''

- web01 디렉토리의 Dockerfile
'''
FROM nginx:1.18
RUN rm /usr/share/nginx/html/index.html
COPY index.html /usr/share/nginx/html/index.html
'''

- web01 디렉토리의 index.html
'''
<h1> Load Balance Test Page 01 </h1>
'''

- web01 디렉토리의 Dockerfile을 web02, web03에 copy
'''
cp Dockerfile ../web02
cp Dockerfile ../web02
'''

## web02, web03 디렉토리 작업


- docker-compose.yaml
```
version : '3.3'
services:
  mydb:
    image: mariadb:10.4.6
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress 
      MYSQL_DATABASE: wordpress
      MYSQL_USER : userwordpress
      MYSQL_PASSWORD : userwordpress
    volumes:
      - /home/kali/db-data:/var/lib/mysql
    ports:
      - "3306:3306"

  wordpress:
    depends_on:
      - mydb
    image: wordpress:latest
    ports:
      - "8080:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: mydb:33306
      WORDPRESS_DB_USER: userwordpress
      WORDPRESS_DB_PASSWORD: userwordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - /home/kali/web-db:/var/www/html
```
