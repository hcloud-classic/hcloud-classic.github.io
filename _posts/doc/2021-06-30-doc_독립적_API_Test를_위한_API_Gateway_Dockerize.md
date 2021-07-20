---
title: 독립적 API Test를 위한 API Gateway Dockerize
subtitle: 
layout: post
icon: fa-book
order: 3
---
# 독립적 API Test를 위한 API Gateway Dockerize

Hcloud Classic에서는 미들웨어의 배포와 이식성을 향상 하기 위해서 Dockerize를 진행 하고 있습니다. 어느정도 개발이 완료된 뒤에 할 예정 이였지만, 백업/복구 소프트웨어인 [Timpani](https://github.com/hcloud-classic/timpani) 와 함께 API테스를 진행하기 위해 독립된 API Gateway를 제공해야 했기 때문에 기존의 Gateway와는 독립적으로 동작하는 서비스를 Docker로 제공 하게 되었습니다.

사실 서비스 자체는 기존의 환경에서 잘 동작 했었기 때문에 Dockerize하는 과정에서 크게 신경 쓸 부분은 없었습니다.

# Dockerize

## Build

`Dockerfile`

```docker
FROM alpine:3.12
MAINTAINER codex <codex@innogrid.com>
RUN apk add --no-cache libc6-compat
RUN mkdir -p /usr/local/etc/piccolo
ENTRYPOINT ["/usr/local/etc/piccolo/piccolo"]
```

기본 이미지를 만들기 위해 `alpine:3.12` 이미지를 사용합니다.

alpine 운영체제는 가볍고 간단한, 보안성을 목적으로 개발한 리눅스 배포판으로 docker container 에서 많이 사용하는데, 특히 **"가볍다"** 라는 점이 가장 큰 장점 인것 같습니다.

기본 Alpine Container 의 Docker Hub를 참고해 보면, 3MB도 안되는 용량으로 Linux container를 실행 할 수 있습니다.



![0630_1](/assets/images/docs/2021-06-30-doc_독립적_API_Test를_위한_API_Gateway_Dockerize/0630_1.png)



`Docker hub Alpine Container`

물론 애플리케이션 실행 환경과 바이너리 파일도 어느정도 공간을 차지 하겠지만, 이노그리드의 바이너리 파일은 Go 로 컴파일 되어 있어서 실행환경을 따로 구축 할 필요가 없습니다.

Go언어 네이티브 바이너리를 사용 할 수있다는게 Continer 를 최소의 실행환경으로 구축 할 수 있게 해주는 가장큰 장점인것 같습니다.

```docker
CONTAINER ID   IMAGE                    COMMAND                  CREATED        STATUS       PORTS                            NAMES                     SIZE
8e60a40443da   api_gateway_test:latest   "/etc/hcc/piccolo/pi…"   2 hours ago    Up 2 hours   0.0.0.0:48600->48600/tcp          api_gateway_test          834B (virtual 5.61MB)
```

## Docker Compose

배포의 용이성을 위해 Docker file 이 업데이트 되었다면 새로 빌드 하는것이 좋습니다.

volume 부분이 있을텐데, 배포시에 바이너리 및 설정 파일을 같이 배포하게 됩니다.

```yaml
version: "3.9"  # optional since v1.27.0
services:
  piccolo:
    container_name: "api_gateway_test"
    ports:
      - "48600:48600"
    volumes:
      - ./:/usr/local/etc/piccolo:rw
    image: api_gateway:${TAG:-latest}
    build:
      context: ./
      dockerfile: ./Dockerfile
    hostname: api_gateway_test
    networks:
      - piccolo_vswitch
# 가상 스위치가 없다면 하단의 설정 대로 생성되게 됩니다.
networks:
  piccolo_vswitch:
    name: "Module_vSwitch"
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: "10.255.254.0/24"
```

테스트를위해서 메뉴얼하게 빌드하여 실행해보는 것도 좋을 것 같습니다.

## Docker Deploy

Hcloud Classic에서는 CI/CD 를 위해 내부에서 Gitlab CI/CD 기능을 사용하고 있습니다.

Deploy하는 부분 을 공유 드리면 될 것 같습니다.

```yaml
{% raw %}
docker_build:
  stage: build
  script:
    # Piccolo build
    - make
    - sudo rm -rf $PWD/docker/piccolo
    - cp $PWD/piccolo $PWD/docker/
    - Docker_Gateway=$(sudo docker network inspect --format='{{range .IPAM.Config}}{{.Gateway}} {{end}}' Module_vSwitch| tr -d '\\n ')
    - sed -i "s/Docker_Gateway/$Docker_Gateway/g" docker/piccolo.conf
    - export TERM=xterm
    - for items in $(sudo docker ps -a --format {{.Names}});do if [ "$items" = "api_gateway_test" ];then echo "$items Find!"; sudo docker stop api_gateway_test; sudo docker rmapi_gateway_test; break; fi;done
    - sudo rm -rf $Docker_Path
    - sudo mkdir -p $Docker_Path
    - sudo cp -r $PWD/docker/* $Docker_Path/
    - echo "Build Complete"
  only:
    - docker/api_gateway

docker_run:
  stage: run
  script:
    - sudo docker-compose -f $Docker_Path/docker-compose.yml up -d
    - echo "Finished"
  only:
    - docker/api_gateway
{% endraw %}
```

Build Stage 을 보면 실행하는 단계가 상당히 많은데요.

Container를 배포하기 전에 기존에 실행되고 있던 동일한 Container 를 찾아서 삭제해 주기위함입니다.

물론 상황마다 적절하게 Piccolo의 gRPC 및 GrapqhQL의 설정을 바꿔야 해서 `Docker_Gateway` 의 네트워크 값을 바꿔 줍니다.

마지막 Deploy Stage에서는 배포된 docker compose 설정을 기준으로 배포를 하게 됩니다.

그리고 docker container 전용 브랜치이기 때문에 다른 브랜치에서 커밋했을 때는 Stage 가 동작하지 않게 하기 위해서 `docker/api_gateway` 에서만 동작하게 설정을 했습니다.

## Running Container

정상적으로 동작이 되는지 접속하여 확인해 봅니다.

![0630_2](/assets/images/docs/2021-06-30-doc_독립적_API_Test를_위한_API_Gateway_Dockerize/0630_2.png)

api gateway는 기관에서 이노그리드의 API테스트를 용이하게 하기 위해 GraphQL의 Playground를 제공합니다.

물론 모든 API를 오픈하는것은 아닙니다.

각 기관에서 필요한 필수 API에 대해서만 테스트 가능하며, 보안성을 위해서 반드시 Token인증을 받아야 합니다.

모든 API는 Token없이 동작 할 수 없으니 1차적으로는 API보안이 보장 됩니다.

오늘은 API Gateway인 Piccolo의 일부 기능을 Dockerize하여 배포하는 방법에 대해 공유 했는데요.

추후에는 시스템에 직접 동작해야 하는 모듈을 제외하고 모든 모듈을 Dockerize 할 계획입니다.

관리 부터 배포 및 운영까지 사용자 입장에서의 편리성 증대를 위해서 Container 관리를 적용할 예정 입니다.

\#docker #docker-compose #container #Alpine #CICD #Dockerize #API #APIgateway #Debian #Ubuntu #sds #softwaredefinedserver #소프트웨어정의서버