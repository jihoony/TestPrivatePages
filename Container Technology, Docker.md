# Container Technology, Docker



# Intro

## Cloud, DevOps, Container

### Cloud

초반 컴퓨터 시스템은 원격 데이터 센터를 통하여 이루어 졌다. 기업들은 컴퓨터 하드웨어를 구입하고 유지보수하였으며, 이를 유지 보수하는 것이 비용대비 효율적이지 않았기 때문에 제 3자가 소유하고 운영하는 원격 머신의 컴퓨팅 파워를 공유하는 비즈니스 모델이 등장 했다.

### DevOps

기존 소프트웨어는 개발자는 소프트웨어를 작성하고, 사용환경에서 소프트웨어를 실행하고 관리하는 운영자에게 전달했다. 소프트웨어 개발과 컴퓨터 운영은 중복되는 업무가 없었으며 각자 매우 전문적인 일이었다. 클라우드의 등장으로 분산 시스템의 복잡성 및 시스템 운영 기술은 클라우드 시스템의 설계, 아키텍처, 구현과 분리하기 쉽지 않게 되었다. 더이상 소프트웨어는 기존 인프라 시스템과 밀접한 관계를 가지기 시작하였으며 또한 상호 의존적이게 변질 되었다. 소프트웨어를 작성하는 사람은 소프트웨어가 나머지 시스템과 어떤 관계가 있는지 이해해야 하며 시스템을 운영하는 사람은 소프트웨어의 작동 방식과 실패를 이해해야 한다. DevOps는 이 두 그룹을 하나로 모으기 위한 시도에 있다.

### Container

소프트웨어를 배포하려면 소프트웨어 자체뿐만 아니라 의존성이 필요하다. 의존성은 라이브러리, 인퍼트리터, 서브 패키지, 컴파일러, 확장 등을 의미한다. 구성(설정, 세부정보, 라이선스키, 패스워드 등) 또한 필요하다. 



## Container

![We sell and rent shipping containers and trailers in the Atlanta Metro |  Container Technology, Inc](https://media.graphcms.com/resize=fit:crop,height:630,width:1200/Pg3AI4OkQw6gA8ckUQ5z)

1950년대 말콤 맥린이라는 트럭 운전사가 화물을 배에 싣는 새로운 방법을 제안했다. 트럭 트레일러에서 짐을 내리고 다시 배에 싣는 대신 단순하게 트럭의 짐칸을 떼어내 배에 싣는 방법을 생각했다. 트럭 트레일러는 본질적으로 바퀴가 달린 큰 금속 상자다. 상자(컨테이너)를 운반하기 위해 붙어 있는 바퀴와 섀시를 떼어낼 수만 있다면, 상자를 들어올리거나 내리는 작업이 매우 쉬워지고, 반대편 끝에 있는 배나 다른 트럭으로도 바로 옮겨 실을 수도 있다.

소프트웨어의 컨테이너도 해운 산업의 컨테이너와 같은 개념이다. 일반적이고 널리 보급된 표준 패키징과 배포 형식은 운반 용량을 크게 늘리고, 비용을 절감하며, 규모의 경제를 이루며, 다루기 쉽나. 컨테이너 형식에는 애플리케이션 실행에 필요한 모든 것이 포함되어 있으며 컨테이너 런타임이 실행할 수 있는 이미지 파일에 저장한다.



### Virtual Machine vs Container

![가상머신과 도커](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/vm-vs-docker.png)

가상 머신 이미지도 애플리케이션 실행에 필요한 모든것을 포함한다. 하지만 그 외에도 불필요한 많은 것을 포함한다. 일반적인 가상머신 이미지는 약 1GiB다. 반면 잘 설계된 컨테이너 이미지는 수백 배 더 작다.

가상 머신에는 관련 없는 프로그램, 라이브러리, 애플리케이션이 사용하지 않는 많은 것이 포함되어 있어서 공간 대부분이 낭비된다. 네트워크를 통한 VM 이미지 전송은 최적화된 컨테이너보다 훨씬 느리다.

가상 머신이 실행하는 CPU는 물리 CPU를 실제와 같이 에뮬레이션 하여 구현한다. 이 때문에 가상화 레이어는 성능에 극적이고 부정적인 영향을 미친다. 컨테이너는 일반 바이너리 실행 파일과 마찬가지로 가상화 오버헤드 없이 실제 CPU에서 직접 실행된다. 컨테이너는 필요한 파일만 보유하므로 VM 이미지보다 훨씩 작다. 또한 컨테이너 간에 공유하고 재 사용할 수 있는 주소 지정 방식의 파일 시스템 레이어를 사용한다.



### Container Orchestration

운영팀은 컨테이너로 워크로드를 크게 단순화 할 수 있다. 다양한 종류의 시스템, 아키텍처, 운영 체제를 관리하는 대신에 컨테이너 오케스트레이터만 실행하면 된다. 컨테이너 오케스트레이터는 다양한 머신을 하나의 클러스터로 결합하도록 설계된 소프트웨어의 한 종류다. 컨테이너 오케스트레이터라는 용어는 일반적으로 스케줄링, 오케스트레이션, 클러스터 관리를 담당하는 단일 서비스를 말한다.

- 스케줄링: 사용 가능한 리소스를 관리하고 가장 효율적으로 실행할 수 있는 워크로드를 할당하는 것
- 오케스트레이션: 서비스의 공통적인 목표를 위해 서로 다른 역할을 조정하고 나열 하는 것
- 클러스터 관리: 여러 개의 물리 또는 가상 서버를 신뢰할 수 있고 fault-tolerant(장애 허용)를 유지하는 원활한 그룹으로 통합하는 것



# Docker

Docker는 2013년 3월 산타클라라에서 열린 Pycon Conference에서 dotCloud의 창업자인 Solomon Hykes가 [The future of Linux Containers](https://www.youtube.com/watch?v=wW9CAH9nSLs) 라는 세션을 통하여 처음 세상에 알려졌다. 이후 dotCloud에서 Docker Inc.로 회사명을 변경하였다.



## Docker Architecture

Dockers uses a client-server architecture.

![Docker Architecture Diagram](https://docs.docker.com/engine/images/architecture.svg)



## Container, Container Image

![Docker image](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/docker-image.png)

이미지는 컨테이너 실행에 필요한 파일과 설정 값등을 포함하고 있는것으로 상태값을 가지지 않고 변하지 않는다.

컨테이너는 이미지를 실행한 상태라고 볼수 있고 추가되거나 변하는 값은 컨테이너에 저장된다.

같은 이미지에서 여러개의 컨테이너를 생성 할 수 있고 컨테이너의 상태가 바뀌거나 컨테이너가 삭제되더라도 이미지는 변하지 않고 그대로 남아있다.



### Container and Image

Container Image와 Container는 Class와 Class Instance의 관계와 같다.

 Java프로그램의 경우 Class를 작성하기 위해 java 파일로 파일을 생성하듯 Container도 Dockerfile 파일을 작성하여 Container Image를 생성한다.

java 파일로 작성된 Class는 인스턴스로 생성되는 과정에 속성 값 등을 가지고 인스턴스 이름을 통하여 각 인스턴스를 구분 하듯이 Container Image도 각종 속성, 환경 변수 등을 가지며 컨테이너 이름을 통하여 구분한다.



### Docker Image

#### Layer 저장방식

![Docker Layer](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/image-layer.png)

![Layers of a container based on the Ubuntu image](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

![Containers sharing same image](https://docs.docker.com/storage/storagedriver/images/sharing-layers.jpg)



Docker Image and layers: https://docs.docker.com/storage/storagedriver/



#### Image 경로

![Docker image url](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/image-url.png)



## Docker How to

### Docker Container 만들기

Docker Container를 만들기 위한 명령어 구문을 통하여 Docker Container를 만들수 있다. 아래는 주요 구문을 정리하였다. Dockerfile은 Top-Down 으로 해석된다.

| Syntax     | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| #          | comment                                                      |
| FROM       | 컨테이너의 base image (운영환경)                             |
| MAINTAINER | 이미지를 생선한 사람의 이름 및 정보                          |
| LABEL      | 컨테이너 이미지에 컨테이너의 정보를 저장                     |
| RUN        | 컨테이너 빌드를 위해 base image에서 실행할 명령어            |
| COPY       | 컨테이너 빌드시 호스트의 파일을 컨테이너로 복사              |
| ADD        | 컨테이너 빌드시 호스트의 파일(tar, url 포함)을 컨테이너로 복사. (압축파일의 경우 자동으로 압축 해제됨) |
| WORKDIR    | 컨테이너 빌드시 명령이 실행된 작업 디렉터리 설정             |
| ENV        | 환경 변수 지정                                               |
| USER       | 명령 및 컨테이너 실행시 적용할 유저 설정                     |
| VOLUME     | 파일 또는 디렉토리를 컨테이너의 디렉토리로 마운트            |
| EXPOSE     | 컨테이너 동작 시 외부에서 사용할 포트 지정                   |
| CMD        | 컨테이너 동작 시 자동으로 실행할 서비스나 스크립트 지정      |
| ENTRYPOINT | CMD와 함께 사용하면서 command 지정 시 사용                   |



**Dockerfile example**

```dockerfile
# Dockerfile
FROM node:12
COPY app.js /
CMD ["node", "/app.js"]
```



```javascript
// app.js
const http = require("http");
const hostname = "127.0.0.1";
const port = 3000;

const server = http.createServer((req,res) =>{
  res.statusCode = 200;
  res.setHeader('Content-Type', 'test/plain');
  res.send('Hello World');
});

server.listen(port, hostname, ()=>{
  console.log("Server running at http://${hostname}:${port}/");
});
```



**Docker build example**

```bash
# -t 옵션은 빌드할 이미지 이름을 지정함. tag를 생략시 tag는 자동으로 latest로 기입됨
# .은 현재 디렉토리의 Dockerfile을 기준으로 이미지를 빌드하도록 함
$ docker build -t imagename:tag .

$ docker build -t node_app .
```



### Docker Run Example

Search Image
