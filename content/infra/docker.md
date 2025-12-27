---
date: 2025-12-08
title: docker
description: docker 구성
tags:
  - docker
categories:
  - infra
---

![image](/images/docker-logo.webp#center)

## host 구성

### 패키지 설치
{{% alert title="Note" color=primary %}}
docker-compose 패키지 저장소에 추가되었으므로 패키지로 설치해 사용<br>
또한 바이너리명이 변경되었음. docker-compose -> docker compose
{{% /alert %}}
```sh
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo && \
sudo dnf update -y && sudo dnf upgrade -y && \
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin && \
sudo systemctl restart docker && \
sudo systemctl enable docker && \
sudo systemctl status docker.service && \
sudo docker run -it --rm busybox nslookup google.com
```

### ~docker-compose 설치~
```sh
sudo curl -L "https://github.com/docker/compose/releases/download/v2.6.0/docker compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker compose && \
sudo chmod +x /usr/local/bin/docker compose && \
sudo ln -s /usr/local/bin/docker compose /usr/bin/docker compose && \
sudo docker compose --version
```

### 권한 구성
```sh
sudo usermod -aG docker dev
cat /etc/group | grep docker
logout
```

### bridge network
관리 편의를 위해 다수의 컨테이너를 bridge로 구성할 것이다
```sh
sudo docker network create --driver bridge --gateway 172.18.0.1 --subnet 172.18.0.0/24 dev
```

### docker login
```sh
docker login -u e*******;
L***************************************************************
```

### docker 최상위 폴더
{{% alert title="Note" color=primary %}}
> 서비스 데이터를 /home에 저장하는 것을 좋아하지 않습니다. 많은 사람들이 Docker를 사용하지만 그 이유를 이해할 수 없습니다.<br>
> 그것에 대해 특별히 나쁜 것은 없으며, 데이터베이스를 / etc에 저장하는 것과 같은 지저분한 imho입니다. :-)<br>
> /srv 는 응용 프로그램 / 서비스 데이터를 저장하기 위한 것이므로 사용합니다.<br>
> /opt는 원래 많은 상용 공급업체에서 시스템 파일과 혼합되지 않은 장소에 앱/데이터를 설치하기 위해 사용하는 오래된 표준입니다.

docker 최상위 폴더는 현재 뚜렷한 표준이 없음. 여기서는 `/opt/apps/`로 구성
{{% /alert %}}

### 컨테이너 log 크기 제한
```sh
sudo du -h --max-depth=1 /var/lib/docker | sort -rh
```
```sh
cat << EOF | sudo tee /etc/docker/daemon.json
{
  "log-driver": "local",
  "log-opts": {
    "max-size": "10m"
  }
}
EOF
```
```sh
sudo systemctl restart docker && \
docker info --format '{{.LoggingDriver}}' && \
docker inspect -f '{{.HostConfig.LogConfig.Type}}' $(docker ps --format "{{.Names}}")
```

### 디스크 용량 확보
```sh
docker container prune -f && \
docker network prune -f && \
docker volume prune -f && \
docker system prune -f && \
docker buildx prune -af && \
docker builder prune -a && \
docker images -a | grep none | awk '{ print $3; }' | xargs docker rmi --force && \
docker rmi $(docker images -q -f dangling=true)
```
```sh
sudo find /var/lib/docker/overlay2/ -type f -name *-json.log && \
sudo truncate -s 0 /var/lib/docker/containers/*/*-json.log
```

### docker log
```sh
sudo dockerd --debug
```

## License
상업적 이용 제한 없음
- docker-engine: Apache v2 [^1]

유료 구독 필요 (exceeding 250 employees OR with annual revenue surpassing $10 million USD)
- docker-desktop [^2]

## Troubleshooting
{{% alert color=warning %}}
> 'version' is obsolete

docker-compose.yml 버전 명시 일괄 삭제
 ```sh
sudo find /opt/ -type f -name docker-compose.yml -exec sed -i -z 's/version: "3.8"\n//g' {} \;
```
{{% /alert %}}

## References
- https://www.reddit.com/r/selfhosted/comments/yv8npf/best_practice_dockercompose_setup/
- https://www.reddit.com/r/selfhosted/comments/r5i5f9/how_do_you_structure_your_docker_files/
- https://www.reddit.com/r/linuxadmin/comments/18z2cxv/what_is_the_srv_directory_and_why_does_it_exist/
- https://www.reddit.com/r/docker/comments/125ke3b/are_there_any_best_practices_for_volume_storage/
- https://docs.linuxserver.io/general/volumes/#mapping-a-volume-to-your-container
- https://www.reddit.com/r/docker/comments/xe4u9n/where_is_stated_that_docker_engine_not_docker/?tl=ko
- https://docs.docker.com/engine/install/rhel/

[^1]: https://www.docker.com/pricing/
[^2]: https://docs.docker.com/engine/#licensing
