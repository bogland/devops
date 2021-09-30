# devops
도커, 쿠버네티스, 제플린 활용 방안 연구  

## DOCKERFILE
1. create React CRA  
```
FROM node:alpine  
run mkdir my-app  
WORKDIR /home/oh/test/react/my-app  
RUN npx create-react-app .  
CMD ["npm", "start"]  
EXPOSE 3000  
```
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
```
FROM nginx:alpine
COPY . /usr/share/nginx/html
```
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

## Docker Compose - NGINX + EXPRESS +  MYSQL  
```
version: "3.9"
services:
  web:
    build: ./todayhouse
    ports:
      - "80:80"
    networks:
      - frontend
  api:
    build: ./api
      - "5000:5000"
    networks:
      - frontend
      - backend
  db: # 서비스 명
    image: mysql # 사용할 이미지
    restart: always
    container_name: mysql-test # 컨테이너 이름 설정
    # ports: #컨테이너에서만 접근할 수 있도록 주석처리
    #   - "3307:3306" # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)  <- 컨테이너 내부는 무조건 3306
    environment: # -e 옵션
      MYSQL_ROOT_PASSWORD: "0000"
      MYSQL_USER: "test"
      MYSQL_PASSWORD: "0000"
      MYSQL_DATABASE: "test" 
      TZ: Asia/Seoul
    command: # 명령어 실행
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    volumes:
      - /var/dbdata:/var/lib/mysql # -v 옵션 (다렉토리 마운트 설정)
    networks:
      - backend

networks: # 가장 기본적인 bridge 네트워크
  frontend:
    driver: bridge
  backend:
    driver: bridge
```
## Reverse Proxy  
https://minholee93.tistory.com/entry/Nginx-Reverse-Proxy  

## Jenkins  
1. 환경 구성  
http://jmlim.github.io/docker/2019/02/25/docker-jenkins-setup/  
2. github token 등록  
https://uchupura.tistory.com/63  
3. githook 설정  
https://uang.tistory.com/14  
4. 배포 자동화  
https://www.youtube.com/watch?v=Z2Sw1WiREFw    
https://nerd-mix.tistory.com/37  
-> 여길 aws private 주소로 넣어줘야함  
https://nerd-mix.tistory.com/38?category=824214
주의점 몇가지 적어놈
1. docker의 ~/.ssh/rsa.pub key값을 host  ~/.ssh/ authorized_keys에 넣어줘야 한다. 그리고 ssh ubuntu@[내부망ip로] 테스트    
2. jenkins 구성에서 GitHub hook trigger for GITScm polling 체크하고 저장    
3. github에서 hook 설정이랑 jenkins 퍼블릭키 등록  
4. jenkins 구성에서 빌드에 서버 소스는 **/*, 명령어는 cd ~/htmlstudy git pull  
5. 서버 연동을 위한 plugin은 over ssh 랑 github integration(이거 필요 없을지도)   

Failed to add SSH key. Message [invalid privatekey:  
-> https://blog.gizmo80.com/101  
