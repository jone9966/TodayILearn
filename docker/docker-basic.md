# Docker
## 도커란
컨테이너를 기반으로 리눅스의 응용 프로그램들을 격리된 환경에서 실행하고 관리하게 해주는 가상화 플랫폼

### 컨테이너?
컨테이너는 원하는 프로그램을 충돌없이 실행하기 위해 격리된 환경을 구성한 것

### 가상머신(VM)과 컨테이너의 차이
가상 머신은 가상의 물리 서버를 만드는 것과 같다. 따라서 호스트 운영 체제와 별개로 자체 커널을 포함한 완전히 별개의 운영 체제를 실행한다.
컨테이너는 호스트 운영 체제의 커널 위에서 빌드되며 호스트 OS의 많은 자원들을 컨테이너끼리 공유한다.
따라서 가상 머신에 비해 컨테이너가 훨씬 가볍고 리소스 조절이 쉽지만 보안에서는 완전히 분리된 가상 머신이 더 강력하다.

## 도커 설치
<details>
    <summary>설치 코드</summary>

    ```
    sudo apt-get update
    sudo apt-get install -y ca-certificates curl gnupg lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    # stable version
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # docker install
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    # add sudo group
    sudo groupadd docker
    sudo usermod -aG docker $USER
    sudo -su $USER
    ```
</details>

<details>
    <summary>NVvidia-docker</summary>
    도커 컨테이너에 GPU를 할당하려면 nvidia-docker를 설치해야 한다.

    ```
    # Install NVIDIA Driver
    sudo add-apt-repository ppa:graphics-drivers/ppa
    sudo apt update && sudo apt install -y ubuntu-drivers-common
    sudo ubuntu-drivers autoinstall
    sudo reboot
    
    # install NVIDIA-Docker
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID) && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    sudo apt-get update
    sudo apt-get install -y nvidia-docker2
    sudo systemctl restart docker
    ```
    컨테이너에 gpu를 할당하려면 컨테이너를 실행할 때 gpus를 적어 주어야 한다.
    ```docker run -it --rm --gpus '"device=1"' -v /home/jone/test:/home/jone tnesorflow:v1```
    
</details>

### 이미지
도커 이미지는 컨테이너의 설계도 역할을 하며 이미지를 통해서 같은 구조의 컨테이너를 여러 개 생성할 수 있다.
Dockerfile 에 명세된 정보를 통해 도커 이미지를 만들 수 있으며 이를 이미지를 빌드한다 라고 표현한다.
도커는 이미지를 빌드할 때 레이어라는 개념을 사용하여 효율적으로 용량을 줄일 수 있다.
```
# Dockerfile
FROM  tensorflow/tensorflow:latest-gpu-jupyter
MAINTAINER  jone9966@naver.com

RUN  apt-get update && \
     apt-get upgrade -y && \
     apt-get install -y openjdk-8-jdk wget curl git python3-dev language-pack-ko vim

RUN  locale-gen en_US.UTF-8 && \
     update-locale LANG=en_US.UTF-8

RUN  pip install --upgrade pip
RUN  pip install -r requirements.txt

WORKDIR  /tf
```
위 도커파일은 tensorflow 이미지를 베이스 이미지로 하여 필요한 레이어들을 추가한 예시이다.
아래 명령어를 통해 도커파일을 바탕으로 이미지를 빌드할 수 있다.
```docker build -t tensorflow:v1 .```