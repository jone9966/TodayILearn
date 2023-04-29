# 쿠버네티스 설치 및 세팅
### 쿠버네티스 설치
쿠버네티스를 설치하기 위해 우선 도커가 설치되어 있어야 한다.

<details>
    <summary>쿠버네티스 설치 코드</summary>

    ```
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl
    sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    
    # turn off swap memory
    swapoff -a
    
    # set docker daemon driver
    sudo cat > /etc/docker/daemon.json <<EOF
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m"
      },
      "storage-driver": "overlay2"
    }
    EOF

    sudo mkdir -p /etc/systemd/system/docker.service.d
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    
    # k8s install
    sudo apt-get update
    sudo apt-get install -y kubelet=1.22.17-00 kubeadm=1.22.17-00 kubectl=1.22.17-00

    # 버전 고정
    sudo apt-mark hold kubelet kubeadm kubectl
    ```
    - kubeadm: 클러스터를 구성하기 위한 기능 제공
    - kubelet: 클러스터 내에서 실행되는 작업을 수행하는 컨테이너 핸들러
    - kubectl: 클러스터를 조작하기 위한 CLI 도구
</details>

### 쿠버네티스 클러스터링
```
# master node
sudo kubeadm init
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 토큰 재생성
kubeadm token create --print-join-command
```

### Install CNI(container network interface)
쿠버네티스 내부의 네트워크 설정을 전담하는 CNI를 설치한다.
calico.yaml 에서 CALICO_IPV4POOL_CIDR 부분의 값을 수정하여 IP 풀 설정이 가능하다.
```
wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml
kubectl apply -f calico.yaml
```

### Install NVIDIA-PULGIN
```
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta4/nvidia-device-plugin.yml
```


## Nvidia setting
쿠버네티스는 기본적으로 Docker-CE를 Default Container Runtime으로 사용한다.
따라서, Docker Container 내에서 NVIDIA GPU를 사용하기 위해서는 NVIDIA-Docker 를 Container Runtime 으로 사용하여 pod를 생성할 수 있도록 Default Runtime을 수정해 주어야 한다.
```
sudo cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "insecure-registries": ["docker-registry.default.svc.cluster.local:30000"],
  "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
EOF

sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl restart docker
```