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
```
sudo vim ../web02/index.html
```
- web02 디렉토리의 index.html
```
<h1> Load Balance Test Page 02 </h1>
```
```
sudo vim ../web03/index.html
```
- web02 디렉토리의 index.html
```
<h1> Load Balance Test Page 03 </h1>
```
## WEB_LB 디렉토리 작업

'''
cd ..
sudo vim docker-compose.yaml
'''
- WEB_LB 디렉토리의 docker-compose.yaml
```
version: '3.8'

services:
  web01:
    build: ./web01
    ports:
      - "8001:80"
  web02:
    build: ./web02
    ports:
      - "8002:80"
  web03:
    build: ./web03
    ports:
      - "8003:80"
  nginx_lb:
    build: ./nginx_lb
    ports:
      - "8080:80"
    depends_on:
      - web01
      - web02
      - web03
```

## 디렉토리 및 파일 구조 확인
```
cd ..
tree
```
> ![1](https://github.com/hanmin0512/docker_Load_Balancer/assets/37041208/2d58cb64-611f-4c00-9061-c9c2fee6b7d9)

## docker-compose 실행시키기
```
sudo docker-compose up
```
