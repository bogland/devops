## traefik

동적으로 reverse proxy를 구현해줌

## https 적용

https://www.valentinog.com/blog/traefik/

- traefik이 자동으로 인증서 발급까지 해준다함
- traefik.prod.toml을 통해서 http를 https로 redirect하도록 만들어줘야함
- Host(`api.ohdev.xyz`)" 여기 부분은 실제 사이트 주소가 들어가야 인증서 받아짐

## acme.json

인증 정보가 담길 파일  
chmod 600 ./acme.json 으로 권한 설정해줌
