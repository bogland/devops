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
