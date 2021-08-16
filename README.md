# devops
도커, 쿠버네티스, 제플린 활용 방안 연구  

## DOCKERFILE
1. create React CRA  
FROM node:alpine  
run mkdir my-app  
WORKDIR /home/oh/test/react/my-app  
RUN npx create-react-app .  
CMD ["npm", "start"]  
EXPOSE 3000  

> docker build -t react .  
> docker run -d -p 3000:3000 react  
> docker exec -it angry_lamarr sh  
> docker cp hungry_mcnulty:/home/oh/test/react/my-app ./src  
#컨테이너 To 호스트 파일 이동

## Docker HUB로 보내기
> docker login  
> docker tag react guruwang/react  
#docker push는 사용자명/image명 으로 사용자명이 일치해야 전송됨  
> docker push guruwang/react  

## Nginx 배포
FROM nginx:alpine
COPY . /usr/share/nginx/html
#nginx 설치시 참조하는 html 경로에 있는 index.html을 COPY명령어를 통해서 대체하면  
컨테이너 실행시 변경된 index.html 바로 실행됨  

## GITHUB + Nginx 배포  
```
FROM nginx:alpine  
WORKDIR /usr/share/nginx/html  
RUN apk update && apk upgrade && \  
    apk add --no-cache bash git openssh  
RUN git init  
RUN git config core.sparseCheckout true  
RUN git remote add -f origin https://github.com/bogland/htmlstudy.git  
RUN echo "오늘의집/*"> .git/info/sparse-checkout  
#pull 받을때 특정 폴더만 받기  
RUN git pull origin master  
RUN cp -r 오늘의집/* .  
#mv대신 cp를 이용하면 히든파일까지 보내짐  
RUN rm -rf 오늘의집  
```

## Docker Compose - NGINX + MYSQL  
```
version: "3.9"  
services:  
  web:  
    build: .  
    ports:  
      - "80:80"  
    networks:  
    - backend  
  #api:  
  db: # 서비스 명  
    image: mysql # 사용할 이미지  
    restart: always  
    container_name: mysql-test # 컨테이너 이름 설정  
    ports:  
      - "3306:3306" # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)  <- 컨테이너 내부는 무조건 3306  
    environment: # -e 옵션  
      MYSQL_ROOT_PASSWORD: "0000"  
      # "-"를 쓰지 않는 경우엔 key:value 구조를 사용해야함  
      MYSQL_USER: "test"  
      MYSQL_PASSWORD: "0000"  
      MYSQL_DATABASE: "test"   
      TZ: Asia/Seoul  
    command: # 명령어 실행  
      - --character-set-server=utf8mb4
      #dash를 쓰는 경우엔 key=value 구조를 사용해야함  
      - --collation-server=utf8mb4_unicode_ci  
    volumes:  
      - /var/dbdata:/var/lib/mysql # -v 옵션 (다렉토리 마운트 설정)  
    networks:  
    - backend  

networks: # 가장 기본적인 bridge 네트워크  
  backend:  
    driver: bridge  
```
