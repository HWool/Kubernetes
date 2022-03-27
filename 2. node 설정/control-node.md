# Contorl-node 설정
<pre>
$ kubeadm init

$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

$ kubectl get nodes -o wide 
# version 확인. 
</pre>


---
> 오류 발생 
- init 명령어를 사용하고 나서 

![hosterror](https://media.vlpt.us/images/chrisantus/post/0955c2a5-f2a9-4f16-81eb-73a6fbb8db7f/image.png)

<br>

1. 위에 사진처럼 <b>hostname "호스트 네임" could not be reached</b> 경고가 발생하면 /etc/hosts 부분의 master node의 hostname이 일치하지 않거나 없기때문에 오류가 발생한 것.

<br>

- 1. 해결법 <br> 
master-node와 hostname의 이름을 동일하게 변경 <br>
※ hostname 명령어로 hostname 확인가능 ※

<br>

2. 위에 사진처럼 <b> NunCPU </b> 에러가 발생하는 이유는 쿠버네티스의 최소 요구사항 중 하나가 cpu 2개여서 이런 오류가 발생.

<br> 

- 2. 해결법
<pre>kubeadm init --ignore-preflight-errors=NumCPU</pre>

<br>

3. <b>'curl -sSL http://localhost:10248/healthz'</b> 라는 오류 발생시

<br>

- 3. 해결법

<pre>
cat << EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF

systemctl enable docker
systemctl deamon-reload
systemctl restart docker

kubeadm reset
</pre>
---