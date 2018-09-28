# Lab-500 : Multi-Tier 애플리케이션 배포

이번에는 멀티 티어 애플리케이션을 배포해 보도록 하겠습니다.
그 시작은 간단하게 애플리케이션과 데이터베이스로 이루어진 아키텍처로 구성을 하도록 하겠습니다.


앞단에 node 로 이루어진 CRUD를 하는 웹애플리케이션이 있고 뒷단에 데이터를 저장하는 mysql 이 있습니다.
오늘 사용할 애플리케이션은 연락처 어플인데, 연락처를 등록/수정/삭제하는 애플리케이션입니다. node.js 와 mysql 로 운영이 되고 있습니다.

- 연락처 리스트 보기
  
  ![Alt text](https://monosnap.com/image/vVhNXN6o9uOK51sZ2MUEu9jgPg0hX9.png)

- 새로운 연락처 등록
  
  ![Alt text](https://monosnap.com/image/upDMO1IkYc5BJ3v9KNuMMtBeQswwaH.png)




# Step-1: MySQL 배포하기

1. mysql docker image 찾기

    mysql을 배포하기 위해서 기존에 공식적으로 등록된 도커 이미지을 찾도록 합니다.
    ~~~
    docker search mysql
    ~~~
    ![Alt text](https://monosnap.com/image/ciYJNJhIUVs29eJHuIRfHf4PemaHp2.png)

    위의 결과와 같이 많은 mysql에 관한 도커 이미지들을 볼 수 있습니다. 이 중 `mysql` 이미지를 사용하도록 합니다.

    mysql을 쿠버네티스에 배포하면 pod에서 해당 서비스가 수행되게 됩니다. pod 이 수행을 종료하면 데이터는 사라지게 됩니다.
    따라서 저장이 가능한 영역이 필요하며 이는 쿠버네티스에서 Volume 으로 정의하게 됩니다.

1. mysql용 패포파일 만들기

    다음은 mysql을 배포하기 위한 yaml 파일입니다.

    `mysql-deployment.yaml`
    ~~~
    apiVersion: v1
    kind: Service
    metadata:
    name: mysql
    labels:
        app: oke-sample
    spec:
    ports:
    - port: 3306
    selector:
        app: mysql
    #  clusterIP: None
    ---
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
    name: mysql
    labels:
        app: oke-sample
    spec:
    selector:
        matchLabels:
        app: mysql
    strategy:
        type: Recreate
    template:
        metadata:
        labels:
            app: mysql
        spec:
        containers:
        - image: mysql:5.6
            name: mysql
            env:
            # Use secret in real usage
            - name: MYSQL_ROOT_PASSWORD
            value: password
            ports:
            - containerPort: 3306
            name: mysql
            volumeMounts:
            - name: mysql-persistent-storage
            mountPath: /var/lib/mysql
        volumes:
        - name: mysql-persistent-storage
            persistentVolumeClaim:
            claimName: mysql-pv-claim
    ~~~

    - `mysql` 이라는 도커 이미지를 같은 이름의 `mysql` 이라는 이름으로 배포를 합니다. 
    - 필요한 환경변수인 MYSQL_ROOT_PASSWORD 변수의 값을 `password`로 입력합니다. 
    - 포트는 `3306`으로 서비스를 합니다.
    - /var/lib/mysql 디렉토리를 볼륨으로 마운트 합니다. 이름은 mysql-persistent-storage 입니다.
    - mysql-psersistent-storage 는 PersistentVolumeClaim 으로 연결이 되며 이름은 mysql-pv-claim 입니다.


1. Persistent 저장소 만들기
    
    위와 같은 배포환경에서 추가로 필요한 부분이 mysql-pv-claim 이름의 저장소를 확보하는 것입니다. 이를 위하여 다음의 배포를 준비합니다.

    `mysql-pv.yaml`
    ~~~
    kind: PersistentVolume
    apiVersion: v1
    metadata:
    name: mysql-pv-volume
    labels:
        app: oke-sample
        type: local
    spec:
    storageClassName: manual
    capacity:
        storage: 2Gi
    accessModes:
        - ReadWriteOnce
    hostPath:
        path: "/mnt/data"
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    name: mysql-pv-claim
    labels:
        app: oke-sample
    spec:
    storageClassName: manual
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
        storage: 2Gi
    ~~~

    - PersistentVolume 형식으로 `mysql-pv-volume` 을 만듭니다. 사이즈는 `2Gbyte` 이며 `/mnt/data` 로 마운트 합니다..
    - 하나의 Pod 에서만 사용가능하도록 `ReadWriteOnce` 를 하였으며 storageClassName 은 `manual` 입니다.
    - PersistentVolume을 사용하기 위한 PersistentVolumeClaim의 이름은 `mysql-pv-claim` 입니다.

1. Persistent Volume 배포하기

    PersistentVolme을 만들기 위하여 배포 합니다. 

    PV (PersistentVolume) 를 배포합니다.
    ~~~
    $ kubectl create -f mysql-pv.yaml
    ~~~

    PVC (PersistentVolumeClaim)을 배포합니다.
    ~~~
    $ kubectl create -f mysql-deployment.yaml
    ~~~

    배포 상세 내역을 확인합니다.

    ~~~
    $ kubectl describe deployment mysql

    Name:               mysql
    Namespace:          default
    CreationTimestamp:  Fri, 14 Sep 2018 17:02:03 +0900
    Labels:             <none>
    Annotations:        deployment.kubernetes.io/revision=1
    Selector:           app=mysql
    Replicas:           1 desired | 1 updated | 1 total | 1 available | 0 unavailable
    StrategyType:       Recreate
    MinReadySeconds:    0
    Pod Template:
    Labels:  app=mysql
    Containers:
    mysql:
        Image:      mysql
        Port:       3306/TCP
        Host Port:  0/TCP
        Environment:
        MYSQL_ROOT_PASSWORD:  password
        Mounts:
        /var/lib/mysql from mysql-persistent-storage (rw)
    Volumes:
    mysql-persistent-storage:
        Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
        ClaimName:  mysql-pv-claim
        ReadOnly:   false
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   mysql-7b79cb5c8d (1/1 replicas created)
    Events:          <none>
    ~~~

    pods의 리스트도 살펴봅니다.
    ~~~
    $ kubectl get pods -l app=mysql

    NAME                     READY     STATUS    RESTARTS   AGE
    mysql-7b79cb5c8d-9z7tn   1/1       Running   0          2d
    ~~~

    PersistentVolmeClaim 을 상세히 봅니다.
    ~~~
    $ kubectl describe pvc mysql-pv-claim

    Name:          mysql-pv-claim
    Namespace:     default
    StorageClass:  manual
    Status:        Terminating (lasts 2d)
    Volume:        mysql-pv-volume
    Labels:        <none>
    Annotations:   pv.kubernetes.io/bind-completed=yes
                pv.kubernetes.io/bound-by-controller=yes
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      2Gi
    Access Modes:  RWO
    Events:        <none>
    ~~~


# Step-2: MySQL 접속하기

앞의 YAML 파일은 클러스터의 다른 pod가 데이터베이스에 액세스 할 수있게 해주는 서비스를 생성합니다. Service의 옵션 `clusterIP : None`을 사용하면 서비스 DNS 이름이 pod의 IP 주소로 직접 해석됩니다. 이는 서비스 뒤에 하나의 pod 있고 pod의 수를 늘리지 않으려는 경우에 가장 적합합니다.

실행되어 있는 mysql container 에 접속해서 잘 실행되고 있는지 알아보겠습니다.

~~~
$ kubectl get pod

NAME                     READY     STATUS    RESTARTS   AGE
mysql-7b79cb5c8d-9z7tn   1/1       Running   0          2d
~~~

위와 같이 현재 수행되고 있는 pod 이름을 살펴봅니다.
그리고 해당 pod 에서 bash을 수행합니다.
~~~
$ kubectl exec -it mysql-7b79cb5c8d-9z7tn bash
~~~

프롬프트가 뜨면 mysql에 접속하기 위해서 `mysql -u root -p`를 입력합니다. 패스워드는 `mysql-deployment.yaml`에서 정의한 바와 같이 'password'입니다.

~~~
root@mysql-7b79cb5c8d-9z7tn:/# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.12 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
~~~

또한 kubectl run 으로 실행할 수도 있습니다.
mysql client 를 통하여 mysql에 접근해 보도록 하겠습니다.
~~~
$ kubectl run -it --rm --image=mysql --restart=Never mysql-client -- mysql -h mysql -ppassword

If you don't see a command prompt, try pressing enter.

mysql>
~~~
이 명령은 MySQL 클라이언트를 실행하는 클러스터에 새 Pod를 만들고 이를 Service를 통해 서버에 연결합니다. 연결되면 MySQL 데이터베이스가 실행 중임을 알 수 있습니다.






# Step-3: 애플리케이션용 계정 및 테이블 만들기

애플리케이션에서 필요한 사용자 및 테이블을 생성합니다.

접속된 mysql에서 다음의 명령을 수행합니다.
~~~
CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'Welcome1';

GRANT USAGE ON *.* TO 'test'@'%';

GRANT ALL PRIVILEGES ON *.* TO 'test'@'%';

CREATE DATABASE sample DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

USE sample;

CREATE TABLE IF NOT EXISTS `players` (
  `id` int(5) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(255) NOT NULL,
  `last_name` varchar(255) NOT NULL,
  `position` varchar(255) NOT NULL,
  `number` int(11) NOT NULL,
  `user_name` varchar(20) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  AUTO_INCREMENT=1;
~~~

![Alt text](https://monosnap.com/image/hlifTEahStJsRMnC3wP1gE6LcB38TC.png)



# Step-4: 애플리케이션 프로젝트 만들기

애플리케이션 만들기 위하여 우리는 Oracle Developer Cloud Service를 사용해 보도록 하겠습니다.

자세한 사항은 [Oracle Developer Cloud Service](http://www.oracle.com/webfolder/technetwork/tutorials/infographics/devcs_trial_quickview/index.html)를 참조하시기 바랍니다.

Oracle Developer Cloud Service 를 이용하여 소스를 수정하고 OKE에서 동작하기 위해 설정을 하고 도커 이미지를 만들어 보겠습니다.
먼저 Oracle Developer Cloud Service 를 이용하여 소스를 가져오도록 하겠습니다.
원본 소스는 https://github.com/jonggyoukim/oke-sample-app 에 있습니다.

1. cloud.oracle.com 에 접속하여 로그인을 합니다. 로그인이 되면 대쉬보드가 보여집니다.

    ![Alt text](https://monosnap.com/image/E1nz3BLkrgIgQ0hndz5xgAATbO9Kez.png)

1. 아래의 `Developer` 항목의  햄버거 메뉴를 누르고 `Open Service Console`을 열어 Developer Cloud Service의 콘솔창을 엽니다.

    ![Alt text](https://monosnap.com/image/YLARJLXFZI2kXYzui74u4uLkmj9F3J.png)

1. 인스턴스가 없으면 `인스턴스 생성` 버튼을 눌러 인스턴스를 생성합니다. 인스턴스가 생성되어 있으면 햄버거 메뉴를 눌러 `Access Service Instance`를 선택합니다.

    ![Alt text](https://monosnap.com/image/OPdAU1XzmhekdvahxKRpYYKHAiHb5G.png)

1. 콘솔창이 나오면 `+New Project` 버튼을 눌러 새로운 프로젝트를 생성합니다.

    ![Alt text](https://monosnap.com/image/jxnZdiILBRJ6VKVr78IHrjpeRvrNej.png)

1. `Name` 항목에 `oke-sample`을 적고 Next 버튼을 누릅니다.

1. `Initial Repository`를 선택하고 Next 버튼을 누릅니다.

1. `Initial Repository` 항목에서 `Import existing repository`를 선택하고 다음값을 입력해 줍니다.
    ~~~
    https://github.com/jonggyoukim/oke-sample-app.git
    ~~~

1. Next 버튼을 누릅니다. 새로운 프로젝트가 생성될 것입니다.



# Step-5: 애플리케이션 변경하기

애플리케이션을 데이터베이스 접속정보를 환경변수로 받는 부분과 도커이미지를 만들기 위한 부분을 실습하겠습니다.

## DB 접속 정보 변경하기

1. 소스파일에서 mysql에 접속하는 부분은 app.js 에 있습니다. app.js 파일을 브라우저에서 클릭합니다.

    ![Alt text](https://monosnap.com/image/lFYwIegs9O67NsTYF5ichGejZqCCYW.png)

    아래와 같은 부분이 보일 것입니다.
    ~~~
    const db = mysql.createConnection ({
        host: 'localhost',
        user: 'test',
        password: 'Welcome1',
        database: 'sample'
    });
    ~~~

    이 부분에서 우리는 쿠버네티스에 배포되어 있는 MySQL을 사용하기 위하여 수정할 것입니다.

1. 위의 부분을 다음과 같이 변경합니다. 변경하기 위해서는 우측 상단의 에디터 아이콘을 클릭합니다.

    ![Alt text](https://monosnap.com/image/stWTwfMM5ZDU3tOodNSU0PDge5j306.png)
    ~~~
    const db = mysql.createConnection ({
        host: process.env.MYSQL_SERVICE_HOST,
        user: process.env.MYSQL_SERVICE_USER,
        password: process.env.MYSQL_SERVICE_PASSWORD,
        database: process.env.MYSQL_SERVICE_DATABASE
    });
    ~~~
    
    - node.js 에서 환경변수를 받기 위해서는 `process.env.환경변수` 로 처리합니다.
    - 4가지의 환경 변수들은 모두 배포를 위한 yaml 파일에서 정의될 것입니다.

1. 우측 상단의 `Commit`을 누르고 팝업되는 다이얼로그창에서 다시 Commit를 누릅니다.

    ![Alt text](https://monosnap.com/image/lgTdfJyXXxu4bJ5T4Vyx3ZWrswYHwT.png)

1. 소스가 업데이트 되어있는지 확인합니다.

1. 다음은 해당 애플리케이션이 포함된 도커 이미지를 만들 것입니다.


## 애플리케이션 도커 이미지로 만들기 

쿠버네티스로 배포하기 위하여 애플리케이션은 도커 이미지화 해야 합니다. 
만드는 방법은 Oracle Developer Cloud Service를 이용해서 만들어도 되고, 자신의 컴퓨터에서 도커를 설치하고 만들어도 됩니다.

    자신의 컴퓨터에 도커가 설치되어 있지 않다면 DevCS를 사용하도록 합니다.


### Oracle Developer Cloud Service 사용하여 도커 이미지 만들기

도커 이미지를 만들기 위해서는 Dockerfile 이 필요합니다.
애플리케이션을 도커 이미지로 만들기 위해 다음의 순서로 처리하면 됩니다.

1. Dockerfile 만들기

    새로운 파일을 만들기 위하여 우측 아래의 `+ File`을 선택합니다. 
    
    ![Alt text](https://monosnap.com/image/TzdgskalLlsnYEhGhT3xxK1A7kPN5P.png)
    
    File name 을 `Dockerfile` 이라고 입력하고 아래의 항목을 내용으로 써 넣습니다.

    Dockerfile 을 다음의 내용으로 만듭니다.
    ~~~
    # Node 버젼 8의 이미지를 기본으로 합니다.
    FROM node:8

    # 애플리케이션이 위치할 디렉토리를 생성합니다.
    WORKDIR /user/src/app

    # npm을 이용하여 필요한 패키지를 설치합니다.
    COPY package*.json ./
    RUN npm install

    # 모든 애플리케이션 파일을 복사합니다.
    COPY . .

    # 포트를 익스포즈 합니다.
    EXPOSE 8080

    # 애플리케이션를 실행합니다.
    CMD ["npm", "start"]
    ~~~

1. .dockerignore 파일 만들기

    역시 파일을 만들기 위하여 우측 아래의 `+ File`을 선택합니다. 

    `.dockerignore` 파일은 도커 이미지에 들어가지 않아야 할 항목들을 나열합니다.
    npm 으로 만들어지는 모듈들과 로그는 빼도록 합니다.

    ~~~
    node_modules
    npm-debug.log
    ~~~

1. 다음과 같이 두개의 파일이 만들어 졌는지 확인합니다.

    ![Alt text](https://monosnap.com/image/7U4a1W8RffDISpf7ag6MpZ0SXAefwr.png)


1. 좌측 메뉴의 `Build`를 클릭하고 `+ New Job`을 클릭하여 새로운 빌드처리를 만듭니다.

    ![Alt text](https://monosnap.com/image/LjRmUK15uPPjVpBrJuqH2UFY1yu4eG.png)

1. Job Name에 적당한 이름(예:build image)를 주고 하위 Software Template 에서 기존에 만들어 두었던 템플릿을 선택하고 Create Job 을 클릭합니다.

    ![Alt text](https://monosnap.com/image/Z45h3fpe5LKfl55wJ86PQFMpObfKuw.png)

1. `Source Control` 탭을 누르고 `Add Source Control`을 눌러 Repository에서 만들어져 있는 `oke-sample.git`을 선택합니다.

    ![Alt text](https://monosnap.com/image/xdxdtYyKKWBRWZajg5qN48uMtM1f0E.png)

1. `Builders` 탭을 누르고 `Add Builder` > `Unix Shell Builder`을 선택합니다.

    ![Alt text](https://monosnap.com/image/KhbuiFqw22Tb67hJEwPfQjKr0r6ltU.png)

1. 다음과 같이 입력합니다.

    ~~~
    docker login -u <태넌트>/<로그인아이디> -p <토큰값> iad.ocir.io
    docker build -t iad.ocir.io/<태넌트>/<저장소명>/oke-sample-app .
    docker push iad.ocir.io/<태넌트>/<저장소명>/oke-sample-app
    ~~~

    예를 들면 다음과 같습니다.
    ~~~
    docker login -u gse00014941/jonggyou.kim -p xxxxxxx iad.ocir.io
    docker build -t iad.ocir.io/gse00014941/project01/oke-sample-app .
    docker push iad.ocir.io/gse00014941/project01/oke-sample-app
    ~~~

    ![Alt text](https://monosnap.com/image/OK2RluHjK34tCgAq1TgMGo52oD7sUO.png)

1. 우측 상단의 `Save`를 눌러 저장을 합니다.

1. Build의 콘솔창이 보이면 우측 상단의 `Build Now`를 눌러 실행합니다.

    ![Alt text](https://monosnap.com/image/qo39ULkgLZIyHOyKENDi1gJDDVfLKb.png)

1. `Build Log` 아이콘을 눌러 현재 빌드되고 있는 상황을 모니터링 합니다.

    ![Alt text](https://monosnap.com/image/KT5jUrgJFOiU8ZOql6d5XQ7gnymmAv.png)

    Docker Registry 에 로그인을 하고, Docker image를 만들고, Registry 에 image를 푸시가 완료되었음을 알 수 있습니다.

1. OCI의 Container Registry 에서 해당 docker image가 등록되어있음을 볼 수 있습니다.
    
    ![Alt text](https://monosnap.com/image/jKZAqM8rwT0FpRXkVFozb3FQamnCe6.png)



### 로컬 컴퓨터에서 도커 이미지 만들기 (스킵가능)

쿠버네티스로 배포하기 위하여 애플리케이션은 도커 이미지화 해야 합니다. 

도커 이미지를 만들기 위해서는 Dockerfile 이 필요합니다.
애플리케이션을 도커 이미지로 만들기 위해 다음의 순서로 처리하면 됩니다.

1. Dockerfile 만들기

    File name 을 `Dockerfile` 이라고 입력하고 아래의 항목을 내용으로 써 넣습니다.

    Dockerfile 을 다음의 내용으로 만듭니다.
    ~~~
    # Node 버젼 8의 이미지를 기본으로 합니다.
    FROM node:8

    # 애플리케이션이 위치할 디렉토리를 생성합니다.
    WORKDIR /user/src/app

    # npm을 이용하여 필요한 패키지를 설치합니다.
    COPY package*.json ./
    RUN npm install

    # 모든 애플리케이션 파일을 복사합니다.
    COPY . .

    # 포트를 익스포즈 합니다.
    EXPOSE 8080

    # 애플리케이션를 실행합니다.
    CMD ["npm", "start"]
    ~~~

1. .dockerignore 파일 만들기

    .dockerignore 파일은 도커 이미지에 들어가지 않아야 할 항목들을 나열합니다.
    npm 으로 만들어지는 모듈들과 로그는 빼도록 합니다.

    ~~~
    node_modules
    npm-debug.log
    ~~~

1. 도커 이미지 만들기

    Dockerfile을 이용하여 도커 이미지를 만들도록 하겠습니다.

    ~~~
    $ docker build -t <your username>/oke-sample-app .
    ~~~

    위의 \<your username>은 hub.docker.com의 아이디입니다. 이 이미지는 모든 사용자가 사용할 수 있는 이미지이므로 본 실습에서는 이전 랩에서 만든 Oracle의 Container Registry를 사용하기 위해 다음과 같이 입력합니다.

    \<your username> 항목이 OKE에서는 `<리즌코드>.ocir.io/<태넌트ID>` 가 됩니다.
    ~~~
    $ docker build -t iad.ocir.io/gse00014941/project01/oke-sample-app .
    ~~~

1. 도커 이미지 확인

    ~~~
    $ docker images
    ~~~
    아래와 같이 해당 도커 이미지가 생성되었음을 알 수 있습니다.

    ![Alt text](https://monosnap.com/image/JmlD2JbyIzxLJ56ucqKB4t6SFt6uET.png)

1. 도커 이미지 푸시

    로컬에 만들어진 도커이미지를 저장소에 저장해 보겠습니다.
    ~~~
    $ docker push iad.ocir.io/gse00014941/project01/oke-sample-app

    The push refers to repository [iad.ocir.io/gse00014941/project01/oke-sample-app]
    25cb017835c5: Pushed
    ac936bfe35ef: Pushed
    b016d32ca91b: Pushed
    999a9548cb3e: Pushed
    d9fdc5af195e: Pushed
    245ce6af2e7b: Pushed
    373fc5310302: Pushed
    25494c62cf78: Pushed
    d714f65bc280: Pushed
    fd6060e25706: Pushed
    d7ed640784f1: Pushed
    1618a71a1198: Pushed
    latest: digest: sha256:c0fa33bc199831bdfa360437b2dd998b61b0650fdfe231635002c7c1bfc29db1 size: 2840
    ~~~

1. 아래에서 보는 바와 같이 OKE의 Registry에서 확인이 가능합니다.

    ![Alt text](https://monosnap.com/image/W5aN7wePigj02URR1kpGlMsYxVnqb7.png)


# Step-6: 애플리케이션 배포하기

1. secret 만들기

    애플리케이션은 멀티로 존재할 수 있기 때문에 쿠버네티스가 docker registry 에서 해당 이미지를 가져올 수 있어야 합니다. 그래서 docker registry에 접근할 수 있는 secret을 먼저 만들도록 합니다.
    ~~~
    $ kubectl create secret docker-registry ocirsecret --docker-server=iad.ocir.io --docker-username='gse00014941/jonggyou.kim' --docker-password='토큰' --docker-email='shiftyou@gmail.com'

    secret/ocirsecret created
    ~~~

    다음과 같이 생성되어 있음을 확인할 수 있습니다.
    ~~~
    $ kubectl get secrets

    NAME                  TYPE                                  DATA      AGE
    default-token-mhr8d   kubernetes.io/service-account-token   3         8d
    ocirsecret            kubernetes.io/dockerconfigjson        1         17s
    ~~~

2. 애플리케이션 배포 yaml 만들기

    다음과 같이 배포용 yaml 파일을 만듭니다.
    `app-deployment.yaml`
    ~~~
    apiVersion: v1
    kind: Service
    metadata:
    name: oke-sample
    labels:
        app: oke-sample
    spec:
    ports:
    - port: 8080
    selector:
        app: oke-sample
        tier: frontend
    type: LoadBalancer

    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: oke-sample
    labels:
        app: oke-sample
    spec:
    selector:
        matchLabels:
        app: oke-sample
        tier: frontend
    strategy:
        type: Recreate
    #  replicas: 1
    template:
        metadata:
        labels:
            app: oke-sample
            tier: frontend
        spec:
        containers:
        - image: iad.ocir.io/gse00014941/project01/oke-sample-app
            name: oke-sample
            env:
            - name: MYSQL_SERVICE_HOST
            value: "mysql"
            - name: MYSQL_SERVICE_USER
            value: "test"
            - name: MYSQL_SERVICE_PASSWORD
            value: "Welcome1"
            - name: MYSQL_SERVICE_DATABASE
            value: "sample"
            ports:
            - containerPort: 8080
            name: oke-sample
        imagePullSecrets:
        - name: ocirsecret
    ~~~
    - 접속되는 포트는 8080 입니다.
    - MYSQL_SERVICE_HOST 부분에 mysql 만 적어주면 이전에 서비스로 수행중인 mysql이 접속됩니다.
    - docker image를  pull 할 secret의 이름은 방금 전 생성한 ocisecret 입니다.


1. 애플리케이션 배포하기

    애플리케이션을 배포합니다.
    ~~~
    $ kubectl create -f ./app-deployment.yaml
    ~~~

    배포된 애플리케이션을 확인합니다.
    ~~~
    $ kubectl get deployment,service,pod
    ~~~
    ![Alt text](https://monosnap.com/image/VoDSlQE4z1USgtQmVT9eA2PPMAmuIF.png)

    - oke-sample 서비스가 LoadBalancer 타입으로 배포되었습니다.
    - EXTERNAL-IP에 적혀있는 129.213.77.2 의 주소로 8080 포트로 현재 서비스 중입니다.
    - mysql 과 oke-sample 은 pod 에서 1개씩 Running 중입니다.

## Step-7: 애플리케이션 테스트하기

서비스로 접근하기 위해서는 service 의 EXTERNAL-IP로 접근을 해야 합니다. 다음의 명령어로 알 수 있습니다.
~~~
$ kubectl get service
~~~
![Alt text](https://monosnap.com/image/zYsBHJz8rZsV3Qcb9inHrGSc7LlNpc.png)

129.213.77.2 으로 서비스 하고 있으며 포트는 8080 입니다.

브라우저에서 `http://129.213.77.2:8080`으로 접속을 합니다.

![Alt text](https://monosnap.com/image/bsQDe3IzFy9zG7CgW88DqdW7bkOK01.png)

![Alt text](https://monosnap.com/image/7bln0Vq0UeZWUQ8GN1W4pQEpBEi7mg.png)

![Alt text](https://monosnap.com/image/lgILdTQ6H1TfJFS4aRtWYs8GfaRrT5.png)

화면의 'IP Address' 항목은 해당 애플리케이션이 수행되고 있는 서버의 로컬 ip address 이며 다음과 같이 일치함을 알 수 있습니다.
~~~
$ kubectl get pod
~~~
![Alt text](https://monosnap.com/image/xn81bhK6i6SqeYFlLoZwjWLSbizqR9.png)

애플리케이션이 실행중인 pod의 내용을 자세히 봅니다.
~~~
$ kubectl describe pod oke-sample-bfc77d9bd-sstff
~~~
![Alt text](https://monosnap.com/image/ROMWhYQY3fwVT3dXZyyLUA4h3kYM2p.png)

애플리케이션이 실행하고 있는 ip 가 보입니다.


수고하셨습니다.

다음 랩은 애플리케이션의 확장과 업데이트를 위한 랩입니다.

---
[상위 페이지](https://github.com/OracleCloudKr/OKE-Workshop/)