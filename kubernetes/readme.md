## 쿠버네티스 설치  
https://lindarex.github.io/kubernetes/ubuntu-kubernetes-installation-with-kubeadm/    
> sudo kubeadm init --control-plane-endpoint < Master public IP >:6443 --upload-certs     
> kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"  
master는 init  
worker는 join  

## 쿠버네티스 포트 개방  
Master : 6443,2379-2380,10250-10252  
Worder : 10250,30000-32767  

## 방화벽 제거  
```
systemctl stop firewalld  
systemctl disable firewalld  
```

## Worker Node Join 에러시
```
accepts at most 1 arg(s), received 2
To see the stack trace of this error execute with --v=5 or higher
```
> kubeadm token create --print-join-command

## Worker Node NotReady  
> kubectl describe node <node 이름>

## 쿠버네티스 40s 에러시  
https://blusky10.tistory.com/473  
```
> sudo vi /etc/docker/daemon.json
{
  "exec-opts":["native.cgroupdriver=systemd"]
}
systemctl daemon-reload
systemctl restart docker

로그 확인
> journalctl -xeu kubelet
```

## 쿠버네티스 에러  
failed to create API client configuration from kubeconfig: invalid configuration: [context was not found for specified context: default  
```
sudo -i
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## 쿠버네티스 에러 - the connection to the server localhost:8080 was refused
```
sudo docker stop $(sudo docker ps -aq)
sudo docker rm $(docker ps -a -q) 

일반사용자 권한 추가
sudo mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

root권한에서 환경변수 등록
export KUBECONFIG=/etc/kubernetes/admin.conf
sudo chown <user> /etc/kubernetes/admin.conf^
```

## 쿠버네티스 에러   
```
Error in configuration: 
* unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for system:node:kube.master.node due to open /var/lib/kubelet/pki/kubelet-client-current.pem: permission denied
```
```
systemctl daemon-reload
systemctl restart kubelet
```

## 쿠버네티스 EC2에서 Join
nc -zv 1.236.xxx.xxx 6443 으로 포트 접근 가능한지 확인  
*no route to host 에러시  
서버쪽에서 다음 명령어 실행  
> sudo iptables -D  INPUT -j REJECT --reject-with icmp-host-prohibited  
> sudo iptables -D  FORWARD -j REJECT --reject-with icmp-host-prohibited  

## 쿠버네티스 EC2에서 JOIN (2)
기본적으로 외부 IP 접근은 JOIN 불가하기에 IP를 추가해줘야함  
https://blog.dudaji.com/kubernetes/2020/04/08/add-ip-to-kube-api-cert.html



## 쿠버네티스 시작시 에러 문구 무시  
- 2GB CPU 2GB RAM 제한 푸는법, prefight 에러 메세지 무시(스펙 제한걸리면 중지되는거 무시  
```
sudo kubeadm init --ignore-preflight-errors=NumCPU \
--ignore-preflight-errors=Mem \
--ignore-preflight-errors strings \
```

## 쿠버네티스 토큰 만료시  
```
kubeadm token list
kubeadm token delete 토큰이름
kubeadm token create --print-join-command
```
