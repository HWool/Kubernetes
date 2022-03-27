# Worker-node 설정

- control-node에서 kubeadm 설정 후 출력되는 join 문 입력하기.
<pre>
kubeadm join 192.168.43.5:6443 --token <token 값>
--discovery-token-ca-cert-hash sha256:<hash 값>
</pre>

---
> 오류 발생 
- join 명령어를 사용하고 나서 

![hosterror](https://media.vlpt.us/images/chrisantus/post/0955c2a5-f2a9-4f16-81eb-73a6fbb8db7f/image.png)

<br>

1. 위에 사진처럼 <b>hostname "호스트 네임" could not be reached</b> 경고가 발생하면 /etc/hosts 부분의 master node의 hostname이 일치하지 않거나 없기때문에 오류가 발생한 것.

<br>

- 1. 해결법 <br>
master-node와 hostname의 이름을 동일하게 변경 <br>
※ hostname 명령어로 hostname 확인가능 ※

<br>

2. <b>'curl -sSL http://localhost:10248/healthz' q\'</b> 라는 오류 발생시

<br>

- 2. 해결법

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

- reset이 되지 않는다면, cni를 삭제할 것.
---