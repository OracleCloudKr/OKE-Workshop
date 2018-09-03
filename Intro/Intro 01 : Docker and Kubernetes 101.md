# 마이크로 서비스의 시작, Docker 와 Kubernetes

Docker(이하: 도커))나 Kubernetes(이하: 쿠버네티스)를 많이 들어봤을 것입니다.
또한 마이크로 서비스도 처음 듣는 단어가 아닐 것입니다.
오늘은 이것들이 무엇이고 왜 생겨났는가, 그리고 오라클은 어떻게 지원 하는가에 대해 다루어 보도록 합니다.

## 애플리케이션 개발과 관리의 변화

애플리케이션의 개발 방법은 계속해서 변화했습니다.
개발은 그냥 하는 것이라고 생각하지만, 실제로 개발을 하기 위해서 생각해야 할 것들, 그리고 다시 고려해야 할 것들이 많습니다.
이런 것들을 어떻게 효율적으로 생각하고 수정하고 작성하는지에 대해 공식화 한 것을 개발 방법론입니다.
![개발 및 운영의 변화](https://image.prntscr.com/image/llVy_JjRTXOPVHOdOqvinw.png)

개발방법론은 그 시대의 소프트웨어 개발 방법에 대해 정리한 것이고, 이는 하드웨어의 상황과 배포상황, 그리고 애플리케이션의 형태에 따라 방법론이 존재하게 됩니다. 그래서 애플리케이션의 아키텍처와 배포방법, 개발 프로세스는 같은 레벨로 변화되어 왔습니다.

기존 모놀로틱 아키텍처에서는 물리서버를 데이터센터에서 운영하는 방법으로 서비스를 수행하였고, 일반적으로 WEB-WAS-DB 형태의 아키텍처인 N-Tier 아키텍처에서는 호스트에서 가상서버의 개념으로 서비스를 수행하였습니다.

하지만 자원의 활용도 측면에서 매우 유용하게 사용을 하지 못했습니다. 그래서 같은 자원이라도 여러가지로 잘라서 쓰려는 의도가 생겨났고, 현재 마이크로 서비스 아키텍처로 개발됨에 따라 애플리케이션은 컨테이너로 배포되고 관리되며 클라우드에서 운영되는 형태가 되었습니다. 또한 DevOps로 애플리케이션 개발, 테스트, 배포 그리고 운영까지 한꺼번에 관리가 되는 형태로 변하였습니다.

작은 단위의 애플리케이션이 만들어 짐에 따라 배포가 빈번히 일어나게 되고 개발/운영이 나뉘어지지 않고 같이 묶는 DevOps로 인해 개발주기가 짧아지고 배포가 자주 일어나게 되었습니다. 이러한 DevOps를 위한  마이크로서비스 아키텍처가 널리 사용되면서 프로그램은 더 잘 쪼개어져 관리는 더 복잡하게 되었었습니다. 그래서 운영관리는 물리적인 서버에서 가상화의 길로 들어서게 되었습니다.

## 서버 가상화

물리적인 서버의 자원을 효과적으로 사용하기 위하서 서버 가상화가 시작되었습니다. 비용 절감의 수단으로 물리적 서버의 수를 줄이는 동시에 기존과 동일한 성능을 내며 CPU의 파워를 충분히 활용하게 되며, 애플리케이션 하나의 가상화 서버를 할당함으로써 애플리케이션에서 발생하는 문제의 원인을 쉽게 발견할 수 있는 이점이 있습니다. 

가상화의 방법으로 `가상화 머신(Virtual Machine)`과 `컨테이너(Container)`의 방법 두가지가 존재합니다. 간단하게 말해서 가상화 머신은 VMWare 나 VirtualBox 같은 것이고 컨테이너는 Docker 입니다.

### 가상화 머신(Virtual Machine)
![Virtual Machine diagram](https://www.docker.com/sites/default/files/VM%402x.png "source:https://www.docker.com/")
가상 컴퓨터는 게스트 운영 체제를 실행합니다. 하이퍼바이저 위에 각 박스마다 게스트 운영 체제를 구동하고 어플리케이션은 이 OS위에서 운영됩니다.
디스크 이미지 및 응용 프로그램 상태는 OS 설정과 설치된 시스템에 종속적이며, 디스크에 모든 것이 닮긴 형태이므로 OS 보안 패치 및 기타 사항을 쉽게 잃을 수 있고 복제하기가 쉽습니다. 대표적으로 VirtualBox 나 VMWare 등입니다.

### 컨테이너(Container)
![Container Diagram](https://www.docker.com/sites/default/files/Container%402x.png   "source:https://www.docker.com/")
컨테이너라 하면 배에 싣는 네모난 화물 수용용 박스가 생각날 것입니다. 각각의 컨테이너 안에는 여러가지 물품들을 넣을 수 있고 규격화 되어있어 다른 컨테이너나 트레일러에 실을 수 있습니다.

서버에서도 마찬가지로 컨테이너 안에 여러 프로그램, 다양한 환경을 구성할 수 있고 동일한 인터페이스로 인해 배포나 관리가 쉬우며 이러한 이미지는 일반PC 및 서버, AWS, Azure, Google Cloud, Oracle Cloud 등에서 어디서든 쉽게 실행할 수 있습니다.

컨테이너는 하이퍼바이저 대신 컨테이너 엔진(Docker 엔진)이 존재하고 실제적인 OS는 Host OS만 존재합니다. Host OS의 커널을 통해 리소스를 제어하게 되는 것이죠. 
컨테이너는 단일 커널을 공유 할 수 있으며 컨테이너 이미지에 있어야하는 정보는 실행 파일과 패키지 종속성이며 호스트 시스템에 절대 설치할 필요가 없습니다. 이러한 프로세스는 네이티브 프로세스와 같이 실행되며, Linux에서 ps를 실행하여 활성 프로세스를 확인하는 것처럼 docker ps와 같은 명령을 실행하여 개별적으로 프로세스를 관리 할 수 있습니다. 그리고, 모든 종속성을 포함하기 때문에 구성 얽힘이 없습니다. 컨테이너 화 된 앱은 `"어디에서나 실행된다."`입니다.대표적으로 Docker 입니다..

위 그림에서 보듯이 VM은 OS단위이고, 컨테이너는 라이브러리 단위라고 생각하면 됩니다. 여기에서는 컨테이너의 대표격인 도커에 대해서 알아보도록 하겠습니다.

## 도커 (Docker)

![Docker](https://www.docker.com/sites/default/files/social/docker_facebook_share.png "source: https://www.docker.com/")

앞서 말 했듯이 도커는 가상화 솔루션 중의 하나입니다. 가볍고 어디서나 동일한 환경에 어디서나 실행이되고 배포가 쉽다는 장점으로 클라우드 환경에서 주력으로 쓰이는 컨테이너 기반의 오픈소스 가상화 플랫폼입니다.도커(Docker)는 docker.com 에서 배포하는 오픈소스입니다. 

도커의 아키텍처는 다음과 같습니다.
![Docker overview](https://docs.docker.com/engine/images/architecture.svg "source: http://www.docker.com")
위 그림에서 보는바와 같이 크게 3부분으로 나뉩니다.
- `Client` : 도커에 대한 명령을 내리는 클라이언트
- `DOCKER_HOST` : 도커가 구동되는 영역
    - `Docker` Daemon : 명령을 받아 수행하는 데몬
    - `Images` : 가상화 대상
    - `Container` : 도커이미지를 구동하는 컨터이너 엔진
- `Registry` : 도커이미지가 저장되어 있는 장소.

간단히 말하면, 클라이언트에서 도커 명령어를 통해 명령을 하면 저장소(Registry)에서 이미지(Image)를 호스트로 가져와 컨테이너에서 수행하는 형식입니다. 그리고 애플리케이션의 배포 단위는 이미지이며 이는 저장소에 저장이 됩니다.

### 도커 이미지

도커 이미지는 애플리케이션이 수행하는 배포단위입니다. 개인이 자신의 호스트에 도커 이미지를 만들어 수행할 수도 있고, 도커 저장소에 저장되어있는 도커 이미지를 다운로드 받아 수행할 수도 있습니다. 또한 도커 저장소에 저장되어 있는 이미지에 자신의 애플리케이션을 더 추가하여 수행할 수도 있습니다. 이 추가한 이미지를 다시 도커 저장소에 저장할 수도 있습니다.

그럼 다음과 같은 의문점이 생길 수 도 있습니다.
- [도커 이미지를 만드는 방법은?](#Q01) 
- [비어있는 이미지를 만드는 방법은?](#Q02)
- [도커 이미지는 현재상태를 그대로 저장하는가?](#Q03)
- [도커 저장소는 어디인가?](#Q04)

이런 의문점에 대해서 얘기해 보도록 하겠습니다.

<a name="Q01"></a>
#### Q. 도커 이미지를 만드는 방법은?

도커의 이미지를 만드는 방법은 두가지가 있습니다.
첫번째는 도커가 제공하는 최소한의 이미지인 scratch를 이용하는 방법이고, 두번째는 전체 전체 이미지를 만드는 방법입니다.

scratch 는 도커에서 사용하는 가장 작은 이미지이며 Dockerfile을 다음과 같이 구성합니다. 내용은 **scratch** 이미지로 부터 hello 라는 디렉토리를 만들고 이동하는 이미지라는 의미입니다.
```dockerfile
FROM scratch
ADD hello /
CMD ["/hello"]
```


처음부터 빈 이미지에서 출발하는 것이 아니라 대부분이 무언가 필요에 의해서 미리 설치된 도커 이미지를 기본으로 자신의 애플리케이션을 설치하기 때문에 [도커 저장소](http://hub.docker.com)에서 검색한 후에 설치하는 것이 좋습니다.또한 개인이 만든 이미지 보다는 공식적으로 배포된 이미지들을 다운로드 받는 것이 좋습니다.

![Alt text](https://monosnap.com/image/bdBo9tEc7Zer5dayUHE5h6SnBmM2v5.png){:width="50%" height="50%"}

<a id="Q02"></a>
#### Q. 비어있는 이미지를 만드는 방법은?

도커의 빈 이미지를 **scratch** 이미지라고 부릅니다. 빈 이미지는 다음과 같이 만듦니다. scratch 이미지는 /dev/null 로 만들어져 아무것도 없기 때문에 아무것도 실행 할 것이 없어 가장 작은 단위의 이미지 입니다.
```sh
$ tar cv --files-from /dev/null | sudo docker import - scratch
```


도커 홈페이지의 Get Started 에서 처음으로 수행하는 도커는 다음과 같습니다. 이것은 scratch 이미지에서 hello-world를 찍는 아주 간단한 것만 포함되어 있는 이미지입니다.
```sh
$ docker run hello-world
```


<a name="Q03"></a>
#### Q. 도커 이미지는 현재상태를 그대로 저장하는가?

도커 이미지는 파일시스템들의 레이어로 만들어져 있다. 

가장 기본 레이어는 boot file system `bootfs`로 일반적인 Linux boot 파일시스템으로 되어있다. 사용자가 이 영역을 사용할 일은 없고 컨테이너가 부팅하고 나면 이 영역은 unmount 된다. 

그 다음에 `rootfs`라는 root 파일 시스템을 레이어로 가진다. `union mount`를 사용해서 여러개의 파일 시스템을 마운트 하되 하나의 파일 시스템만 마운트 된 것 처럼 사용을 하게 하여 기본 이미지를 기준으로 여러개의 이미지 레이어를 가진다. 

![도커 이미지 레이어](https://1.bp.blogspot.com/-Io2aW5QMzUs/WM1ais914bI/AAAAAAAAAfs/9fPX6O1OHOg3HxR-fkXnqW9cGhpQgxlAwCPcB/s1600/Layer3b.png "소스:http://neokobo.blogspot.com/2017/03/docker-container.html")

그래서 도커에서 모든 이미지를 보면 여러개가 보인다.

![Alt text](https://monosnap.com/image/iNDhFHjIQRsHERwBadTFKO5SuTDz0m.png)



<a name="Q04"></a>
#### Q. 도커 저장소는 어디인가?

도커 이미지들을 저장하고 있는 도커 저장소(Docker Registry)의 공식적인 저장소는 http://hub.docker.com 입니다.
Dockerfile과 클라이언트 Docker cli에 의해서 http://hub.docker.com 에 로긴을 하고 이미지를 업로드/다운로드 합니다.


### 도커 실행

도커의 실행은 도커 클라이언트(Docker Client)를 통해서 실행하게 됩니다.

![](https://devopscube.com/wp-content/uploads/2014/12/docker-architecture-techtip39.png)

- 클라이언트를 통해서 명령어(예를 들어 docker pull, docker run, docker image, docker create...등)을 수행하게 되면 호스트에서 수행하게 됩니다.
- 클라이언트 명령을 통해서 도커의 이미지를 다운로드 받거나 기존 도커 이미지에 자신의 애플리케이션을 추가하면 해당 이미지는 자신의 호스트에 저장을 하게 됩니다.
- 클라이언트 명령을 통해서 이미지를 수행시키면 호스트에서 해당 이미지가 수행되게 됩니다.



## 컨테이너 오케스트레이터 (Container Orchestrator)

단 하나의 머신(노드)에서 운영하는 컨테이너라면 큰 문제가 없습니다. 도커 이미지들을 잘 관리할 수 있고 시스템 리소스도 하나의 머신이기 때문에 관리가 잘 됩니다. 하지만 수십/수백대의 머신(노드)에서 운영하고 있다면 컨테이너의 운영이 매우 힘들 것입니다. 각 유후 리소스들을 찾고 도커 이미지들을 시작하고 네트워크를 설정하는 등 많은 어려움이 있을 것입니다.

![Container Orchestrator](https://blog.docker.com/wp-content/uploads/2a78b37d-bbf0-40ee-a282-eb0900f71ba9-2.jpg)
컨테이너 오케스트레이터(Container Orchestrator)는 이러한 여러 머신(노드)에서 운영하는 많은 컨테이너를 관리하기 위해서 만들어졌습니다. 

하나의 컨테이너만 운영하는 것이 아니라 여러개의 컨테이너를 운영을 할 것이고, 하나의 호스트에서 수행되는 컨테이너라면 문제가 없으나 수십개의 수백개의 호스트를 관리하려면 어려운 점이 많습니다. 어느 호스트에서 유휴 자원이 있는지, 각 호스트들 끼리 네트워크는 어떻게 되는지 관리하기가 힘듧니다.

그래서 Container Orchestrator 가 필요합니다. 아래 그림은 Container Orchestrator중이 하나인 Docker Swarm 에 관한 그림입니다.

![](https://i2.wp.com/blog.docker.com/media/2015/11/image00.png?resize=1140%2C595&ssl=1)

컨테이너 끼리는 Docker Compose 로 컨테이너들을 관리를 하고, 여러 호스트에 있는 컨테이너는 Docker Swarm 으로 관리를 하며 호스트 간 관리는 Cluster Manager로 관리를 합니다.

Container Orchestrator는 Docker Swarm 뿐만 아니라 Apache Mesos 그리고 Kubernetes 등이 있습니다.
컨테이너 오케스트레이터 툴은 여러가지가 있습니다. 그 중 가장 널리 사용되고 있는 쿠버네티스(Kubernetes or K8S)에 대해서 집중적으로 알아보도록 합니다.


## 쿠버네테스 (Kubernetes:k8s)

![](https://zdnet2.cbsistatic.com/hub/i/r/2015/07/21/bb0de0fc-5d9c-47c3-96dd-42ed50858fdb/resize/370xauto/8999227b80cc063f94a76f2b628b0499/kubernetes-logo.png)

인터넷에서도 쿠버네테스(줄여서 K8S : K 과 S 사이에 8자의 문자가 있음)에 대한 정보는 쉽게 찾아볼 수 있다. 그 만큼 컨테이너를 운영하는데 널리 이용되는 오케스트레이터이며 관련자료도 많다고 볼 수 있습니다.

### 왜 필요한가?

단일 호스트에서는 도커를 사용하기 위하여 도커 클라이언트로 명령만 치면 사용이 가능합니다.
여러 도커 이미지를 서로 관련시키기 위해서는 docker-compose 를 통해서 사용하면 됩니다.굳이 오케스트레이션을 사용할 필요가 없습니다.

![](https://cdn-images-1.medium.com/max/800/1*PfGIiTw68JLIUyo0FQY2dA.png)

만약, 여러 호스트에서 도커를 사용하고자 한다면 오케스트레이션은 필요합니다.여러 호스트에서 도커를 사용 중이라면 다음과 같은 고민을 하게 됩니다.
- 컨테이너를 생성하고 싶은데 어느 호스트의 자원이 많이 남아서 생성을 해야 할지
- 어느 호스트의 도커 데몬에서 어느 컨테이너가 수행중인지
- 로드밸런스는 어떻게 해야 할건지

이를 위해 여러 호스트에서 도커 컨테이너들을 조율할 필요가 있는데, 이를 수행하는 것이 쿠버르네테스입니다. 쿠버네티스는 애플리케이션 컨테이너의 배치, 확장 및 운영을 자동화 하도록 설계된 오픈 소스 플랫폼 입니다. 쿠버네티스를 사용하면 다음과 같은 이점이 있습니다.
- 애플리케이션을 빠르게 배포할 수 있다
- 쉽게 확장 가능하다
- 아주 작은 리소스를 사용한다

### 쿠버네티스 클러스터

쿠버네티스는 두 형태의 리소스를 가지고 있습니다.
- Master : 클러스터를 코디네이터 하는 노드
- Node : 애플리케이션을 수행하는 노드

![Cluster Diagram](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

- Master

    클러스트터를 관리하는 노드입니다. 애플리케이션의 스케일링, 롤링 업데이트 같은 것을 관리합니다.

- Node

    VM이나 물리적 컴퓨터이며 쿠버네티스 클러스터 내에서 애플리케이션을 수행하는 컴퓨터를 말합니다. 각각의 노드는 Kubelet 이라는 에이전트를 가지고 있으며 이는 Master 와 통신을 하여 정보를 주고 받습니다. 
    
    노드는 Docker 나 rkt 같은 컨테어너를 핸들링 할 수 있는 툴을 가지고 있어야 합니다. 쿠버네티스 클러스터는 최소한 3개의 노드를 관리해야 합니다.

### Kubernetes Deployments

배포를 말하기에 앞서 Pod과 Deployement, Service 에 관한 이해가 필요합니다.

- Kubernetes Pods

    ![pod](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

    하나의 Pod은 다음의 속성을 가집니다.
    - 볼륨과 같은 스토리지 공유
    - 특별한 cluster IP address 같은 네트워크 공유
    - 이미지의 버젼과 포트 같은 각 컨테이너에 대한 정보 공유

    Pod은 다음과 같은 특징을 가지고 있습니다.
    - 1개의 Pod은 내부에 여러개의 컨테이너를 가질 수 있지만 대부분 1~2개의 컨테이너를 가집니다.
    - 1개의 Pod은 여러개의 물리서버에 나눠지는 것이 아니고 1개의 물리서버(Node) 위에 올라갑니다.
    - Pod 내부의 컨테이너들은 네트워크와 볼륨을 공유하기 때문에 localhost 로 통신할 수 있습니다.
    - Pod은 클러스터 내에서 사용할 수 있는 유니크한 네트워크 ip를 할당 받지만 이 ip는 서비스에서 사용하지 않습니다. Pod의 Endpoint를 관리하고 Pod의 외부에서는 이 ‘Service’ 를 통해 Pod에 접근하게 됩니다.
    - 동일 작업을 수행하는 Pod은 ReplicaSet이라는 컨트롤러에 의해 정해진 룰에 따라 복제됩니다. 이때 복수의 Pod이 여러개의 Node에 걸쳐 실행될 수 있습니다.
      

- Kubernetes Deployment 

    ![](https://d33wubrfki0l68.cloudfront.net/152c845f25df8e69dd24dd7b0836a289747e258a/4a1d2/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)

    애플리케이션의 배포/삭제, scale out의 역할 담당합니다. Deployment를 생성하면 pod과 ReplicaSets을 함께 생성됩니다.Pod에 containerized 애플리케이션이들이 포함되고, pod이 생성되면서 애플리케이션이 배포됩니다.

- Kubernetes Service

    ![](https://d33wubrfki0l68.cloudfront.net/cc38b0f3c0fd94e66495e3a4198f2096cdecd3d5/ace10/docs/tutorials/kubernetes-basics/public/images/module_04_services.svg)

    Pod의 논리적 집합과 액세스 정책일 정의하는 추상화된 개념입니다. 소프트웨어 서비스(예를들면 mssql)에 대해서 이름이 부여된 추상적인 개념이며 Service Object는 클러스터 내부에서 접근가능한 port와 외부에서 접근가능한 nodePort를 가집니다.이 포트를 통해 요청이 왔을 경우, Service object에 설정된 selector를 이용하여 요청이 전달 된 pod를 찾습니다.다.

    Service object의 selector값에 해당하는 label을 가진 pod그룹을 찾고, LB의 설정에 따라 특정 pod에 요청이 전달 됩니다.같은 cluster내에서 pod이 어떤 노드에 생성되었는지 상관 없이 Service/label 방식으로 pod을 찾을 수 있습니다.

    pod은 생성/삭제가 쉬워 특정 pod으로 접속하기는 어렵습니다. 그래서 Service를 활용하여 접근합니다. Service object는 서비스 디스커버리와 로드밸런싱 기능을 제공합니다.




일단 쿠버네티스 클러스터가 구성되어 있으면 컨테이너화 된 애플리케이션을 배포할 수 있습니다. 해야 할 것은 쿠버네티스 배포 환경을 구성하면 됩니다. 오라클은 단순한 클릭만으로 쿠버네티스 배포 환경을 구성해 줍니다. 이는 어떻게 애플리케이션의 인스턴스를 생성하고 업데이트할 것인지가 기록되어 있습니다.

애플리케이션 인스턴스가 생성되면 쿠버네티스 Deployment Controller는 계속적으로 이 인스턴스를 모니터링 합니다. 만약 불행하게 인스턴스가 다운되거나 삭제되면 Deployment Controller는 이를 다른것으로 대체합니다. 이렇게 자동으로 머신의 문제를 해결합니다.

#### Deployment Process

![](https://storage.googleapis.com/cdn.thenewstack.io/media/2017/11/07751442-deployment.png)

배포는 Kubectl 명령어를 통해서 수행할 수 있습니다. Kubectl 는 클러스터와 통신하기 위해서 Kuberentes API를 사용합니다.

아래의 예시는 kubectl의 버젼과 현재 동작하고 있는 pod들의 정보를 보여줍니다.

~~~sh
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.0", GitCommit:"91e7b4fd31fcd3d5f436da26c980becec37ceefe", GitTreeState:"clean", BuildDate:"2018-06-27T22:29:25Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-26T16:44:10Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}

$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    43m       v1.10.0
~~~

배포를 하기 위해서 특별한 애플리케이션을을 위한 컨테이터 이미지가 필요하며 몇개를 수행할 것인지에 대한 수가 필요하며 후에 변경 가능합니다.
예로, 첫번째 도커 컨테이너에 Node.js로 구성되어 있는 첫번째 애플리케이션을 배포하는 과정을 알아봅시다.

~~~sh
$ kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080
deployment.apps/kubernetes-bootcamp created
~~~

배포는 `kubectl run`으로 수행을 합니다. 배포 이름(kubernetes-bootcamp)이 필요하며 애플리케이션 이미지의 주소가 필요합니다. 이는 Docker hub에 있습니다. 이렇게 배포가 끝나면 다음을 수행한 것입니다.
- 애플리케이션을 수행 할 안전한 노드를 찾는다. (현재는 1개 노드만 존재)
- 애플리케이션을 노드에서 수행할 준비를 한다.
- 필요하면 새로운 노드에서 인스턴스를 수행하기 위해서 클러스터에 설정한다.

~~~sh
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            0           12s
~~~


kubernetes-bootcamp라는 애플리케이션은 pod에서 수행하고 있습니다. 
~~~sh
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-jq2n9   1/1       Running   1          50m
~~~


## View our app

Pod은 쿠버네티스 내에서 수행이 되며 private, isolated network에서 수행이 됩니다. 기본적으로 같은 쿠버네티스 클러스터에서 다른 pod에서 보여지고 서비스되나 다른 네트워크에서는 그렇지 않습니다. 그래서 우리는 `kubectl`을 사용하여 다른 애플리케이션에서 방금 수행한 애플리케이션을 볼 수 있게 해야 합니다. `kubectl`을 사용하여 cluster-wide 한 proxy를 생성할 수 있습니다. 

~~~sh
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
~~~
위와 같이 proxy를 생성하면 기본 포트인 8001번으로 proxy가 시작하게 됩니다.
다음과 같이 8001 포트로 접속이 가능합니다.
~~~sh
$ curl http://localhost:8001/version
{
  "major": "1",
  "minor": "10",
  "gitVersion": "v1.10.0",
  "gitCommit": "fc32d2f3698e36b93322a3465f63a14e9f0eaead",
  "gitTreeState": "clean",
  "buildDate": "2018-03-26T16:44:10Z",
  "goVersion": "go1.9.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
~~~

수행하고 있는 pod의 정보를 가지고 와서 출력하고, 해당 pod의 proxy를 출력해 보면 다음과 같습니다.
~~~sh
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5c69669756-jq2n9

$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-jq2n9 | v=1
~~~



