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
