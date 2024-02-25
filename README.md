## docker-compose_Load_Balancer
docker-compose를 사용하여 Load Balance를 직접 구현해보자

## Architecture
> ![Lb_docker-compose-architecture](https://github.com/hanmin0512/docker_Load_Balancer/assets/37041208/1cddeab1-9612-432a-8866-76cd2279f4ba)



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
```
cd ..
cd web01
sudo vim Dockerfile
```

- web01 디렉토리의 Dockerfile

```
FROM nginx:1.18
RUN rm /usr/share/nginx/html/index.html
COPY index.html /usr/share/nginx/html/index.html
```

- web01 디렉토리의 index.html
  
```
<h1> Load Balance Test Page 01 </h1>
```

- web01 디렉토리의 Dockerfile을 web02, web03에 copy
  
```
cp Dockerfile ../web02
cp Dockerfile ../web02
```

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

```
cd ..
sudo vim docker-compose.yaml
```

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

> ![스크린샷 2024-02-25 오후 7 51 31](https://github.com/hanmin0512/docker_Load_Balancer/assets/37041208/67d64c9b-4b34-4f8c-b047-cfa61c5522bc)



## docker-compose 실행시키기

```
sudo docker-compose up
```

> ![3](https://github.com/hanmin0512/docker_Load_Balancer/assets/37041208/b06f950e-ed47-400b-a32f-79e1143c0f9d)

## 브라우저로 접속 후 새로고침 여러번 시행.
> <img width="421" alt="4" src="https://github.com/hanmin0512/docker_Load_Balancer/assets/37041208/bb309e25-8f53-4b4d-bcb7-3b5648fd48c3">
> <img width="413" alt="5" src="https://github.com/hanmin0512/docker_Load_Balancer/assets/37041208/f7bbfda1-349c-48cd-90fa-7203d2789717">
> <img width="420" alt="6" src="https://github.com/hanmin0512/docker_Load_Balancer/assets/37041208/ea4e35f4-6fe3-4a03-b477-fff15830548e">

## 정리
- nginx1.18 버전의 이미지를 각각 nbinx_lb, web01, web02, web03에 빌드한다.
- nginx1.18 버전에서 로드벨런싱 설정은 컨테이너의 /etc/nginx/conf.d/default.conf 파일에서 정의하고 있기에 이 내용을 바꾸는 것을 Dockerfile에 스크립트로 작성하였다.
- nginx_lb는 프록시 서버로 로드 벨런싱을 해주고 라운드로빈 방식으로 web01, web02, web03으로 forwarding을 해준다.
- docker-compose는 여러 컨테이너를 정의하고 실행 하기 때문에 디렉토리 및 파일 구조와 같이 미리 정의를 했다.
- 또한 docker-compose down 명령어를 통해 한번에 컨테이너 stop, remove, remove image 까지 한번에 해준다.
