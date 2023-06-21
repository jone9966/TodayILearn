도커에는 다양한 명령어가 존재한다.
도커의 명령어는
```docker <상위 커맨드(/하위 커맨드)> (옵션) 대상 (인자)```
로 이루어진다.

- 커맨드: 무엇을, 어떻게 할 것인지 나타내는 명령어
    - 상위 커맨드
    container, image, volume, network, checkpoint, node, plugin, secret, service, stack, swarm, system
    
- 옵션: 커맨드의 자세한 설정을 지정하는 용도입니다.
- 대상: 커맨드를 실행할 구체적인 대상
- 인자: 대상에게 전달할 값

컨테이너를 생성하려면 이미지가 존재해야 한다.
이미지는 도커 허브에서 pull 해오거나 Dockerfile을 통해 커스텀하게 빌드할 수 있다.
```
# 이미지 목록 확인
docker images
# 이미지 다운로드
docker pull <NAME:TAG>
# 이미지 빌드
docker build -t <이미지 이름:이미지 버전> <Dockerfile의 경로>
# 이미지 제거
docker rmi <IMAGE NAME>
```

이미지 목록에 존재하는 이미지를 통해 컨테이너를 생성하고 실행할 수 있다.
```
# 컨테이너 실행
docker run <IMAGE:TAG>
# 컨테이너 목록
docker ps
# 컨테이너 중지
docker stop <CONTAINER>
# 컨테이너 제거
docker rm <CONTAINER>
# 중지된 컨테이너 모두 제거
sudo docker ps -a | grep Exit | cut -d ' ' -f 1 | xargs sudo docker rm
```

컨테이너는 격리된 환경을 제공하지만 호스트의 데이터가 필요한 경우가 많다.
또한 컨테이너 안의 데이터는 컨테이너가 삭제될 때 사라지기 때문에 아래 명령어를 통해 컨테이너와 호스트의 데이터를 복사할 수 있다.
```docker
# 호스트 to 컨테이너
docker cp <호스트경로> <컨테이너이름:컨테이너경로>
# 컨테이너 to 호스트
docker cp <컨테이너이름:컨테이너경로> <호스트경로>
```

볼륨 마운트는 도커 엔진이 관리하는 영역 내에 만들어진 볼륨을 컨테이너에 디스크 형태로 마운트 한다.
    
```
# 볼륨 생성 및 제거
docker volume create 볼륨이름
docker volume ls
docker volume rm 볼륨이름
# 볼륨 마운트
docker run -v 볼륨이름:컨테이너경로 (생략)
```
    
바인드 마운트는 호스트의 폴더나 파일을 컨테이너 실행시에 마운트한다.
```docker run -v 호스트경로:컨테이너경로 (생략)```


컨테이너를 사용하다 보면 컨테이너에서 리눅스 명령어를 실행해야 하는 경우가 종종 발생한다.
이 때 컨테이너에 설치된 bash를 통해 명령을 입력할 수 있다.
docker run을 통해 bash를 실행했다면 컨테이너의 프로그램 대신에 bash가 실행되기 때문에 bash를 사용한 조작이 끝나면 컨테이너를 재시작해야 한다.
```
docker run 이미지이름 /bin/bash
docker exec 컨테이너이름 /bin/bash
```


도커 레지스트리를 통해서 이미지를 관리할 수 있다.
도커 허브가 도커의 공식 레지스트리라고 할 수 있고 개인적인 비공개 레지스트리를 만들 수도 있다.
레지스트리에는 자유롭게 이미지를 업로드 할 수 있는데 이미지를 업로드 하기 위해서는 이미지에 태그를 부여해야 한다.
```
docker login <서버> -u
docker tag <이미지이름> <레지스트리/리포지토리:버전>
docker push <레지스트리/리포지토리:버전>
```


컨테이너를 실행할 때 gpu를 할당하고 싶다면 nvidia-docker가 설치된 상태에서 실행 명령에 gpus를 추가해야 한다.
nvidia-docker의 버전에 따라 명령어가 다르다.
```
sudo docker run -it --rm --gpus=all tensorflow:v1 bash
NV_GPU=1 nvidia-docker run -it --rm tensorflow:v1 bash
sudo docker run -it --rm --runtime=nvidia -e NVIDIA_VISIBLE_DEVICES=1 tensorflow:v1 bash
```