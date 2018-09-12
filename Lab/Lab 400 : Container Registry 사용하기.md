# Container Registry 사용하기

Container Registry는 Docker image가 저장되는 장소입니다.
Oracle Container Registry를 사용하기 위해서 password 에 해당하는 토큰이 필요하며 토큰 생성 후에 docker image를 저장소에 저장해 봅니다.

## 인증 토큰 가져 오기

1. 콘솔의 오른쪽 상단에서 사용자 아이콘을 클릭 한 다음 `User Settings`을 클릭합니다.
    ![Alt text](https://monosnap.com/image/eoOorBQDsnuacQFwZJUW0PDFrg6clk.png)
1. `Auth Tokens` 페이지에서 `Genereate Token`을 클릭합니다.
    ![Alt text](https://monosnap.com/image/cUgcfFDBX4Vr7jyAmR36K3qvft3HPr.png)
1. 토큰에 대한 설명으로 Tutorial auth token을 입력하고 Generate Token을 클릭하십시오. 새 인증 토큰이 표시됩니다.
1. 콘솔에서 인증 토큰을 다시 볼 수 없으므로 인증 토큰을 나중에 검색 할 수있는 안전한 위치로 즉시 복사하십시오.
    ![Alt text](https://monosnap.com/image/408nqJHx6XMvqLk5ESH6bRgFdjNYQX.png)
1. 토큰 생성 대화 상자를 닫으십시오.
1. Oracle Cloud Infrastructure Registry에 액세스 할 수 있는지 확인하십시오. 콘솔에서 탐색 메뉴를 엽니다. Solutions, Platform and Edge 의 Developer Services 의 Registry를 클릭하십시오.
    ![Alt text](https://monosnap.com/image/XvqxqjRFZIlS62YkwXUpR57d0JiIt1.png)

    ![Alt text](https://monosnap.com/image/kSOmCKx6iwGiMFxsS3YJru99b4IaUP.png)

1. 위의 화면이 보이면 액세스 가능한 환경입니다.

## Docker CLI에서 Oracle Cloud Infrastructure Registry에 로그인
1. Docker를 실행하는 클라이언트 시스템의 터미널 창에서 다음을 입력하여 Oracle Cloud Infrastructure Registry에 로그인하십시오.
    ~~~
    docker login <region-code>.ocir.io
    ~~~

    여기서 \<region-code>는 다음과 같이 사용중인 Oracle Cloud Infrastructure Registry 영역의 코드에 해당합니다.

    - 프랑크푸르트 지역 코드로 fra를 입력하십시오.
    - 애쉬 번 지역 코드로 iad를 입력하십시오.
    - 런던의 지역 코드로 lhr을 입력하십시오.
    - 피닉스 지역 코드로 phx를 입력하십시오.

1. 메시지가 표시되면 \<tenancy_name>/\<username> 형식으로 사용자 이름을 입력하십시오. 예 : acme-dev/jdoe@acme.com.
1. 메시지가 나타나면 앞에서 복사 한 인증 토큰을 암호로 입력하십시오.

    ![Alt text](https://monosnap.com/image/ToTNsH8a0sCicdLUHXeFjdT9rHNPoV.png)

## DockerHub에서 hello-world 이미지를 가져옵니다.

1. Docker를 실행하는 클라이언트 컴퓨터의 터미널 창에서 docker pull hello-world를 입력하여 DockerHub에서 최신 버전의 hello-world 이미지를 검색합니다.

    ![Alt text](https://monosnap.com/image/gYNTTr00ZaVaYbfFgcZ9pIvyWPfIB7.png)

1. hello-world 이미지의 여러 레이어가 각각 차례대로 가져옵니다.


## 이미지 푸시를 위한 태그
1. Docker를 실행중인 클라이언트 시스템의 터미널 창에서 다음을 입력하여 Oracle Cloud Infrastructure Registry로 푸시 할 이미지에 태그를 제공하십시오.
    ~~~
    docker tag hello-world <region-code>.ocir.io/<tenancy-name>/<repo-name>/<image-name>
    ~~~
    항목:
    - \<region-code>는 fra, iad, lhr 또는 phx 중 하나입니다.
    - ocir.io는 Oracle Cloud Infrastructure 레지스트리 이름입니다.
    - \<tenancy-name>은 이미지를 푸시 할 태넌시의 이름입니다 (예 : acme-dev). 사용자는 태넌시에 액세스 할 수 있어야 합니다.
    - \<repo-name> (지정된 경우)은 이미지를 푸시 할 저장소의 이름입니다 (예 : project01). 저장소 지정은 선택 사항입니다. 저장소 이름을 지정하지 않으면 이미지의 이름이 Oracle Cloud Infrastructure Registry의 저장소 이름으로 사용됩니다.
    - \<image-name>은 Oracle Cloud Infrastructure Registry에서 이미지에 부여 할 이름입니다 (예 : hello-world).
    - \<tag>는 Oracle Cloud Infrastructure Registry에서 이미지에 부여 할 이미지 태그입니다. 지정하지 않으면 latest 입니다. (예 : latest).
    
    예 :
    ~~~
    docker tag hello-world iad.ocir.io/gse00014941/project01/hello-world
    ~~~

    ![Alt text](https://monosnap.com/image/9UQcccmn2MOR1L1sUVV06dgSzcsh3h.png)

1. 다음을 입력하여 사용 가능한 이미지 목록을 검토하십시오.
    ~~~
    docker images
    ~~~

    ![Alt text](https://monosnap.com/image/3vo0gNorZqs3XByFbk5tADWpTw4r00.png)

    태그된 hello-world 이미지가 있음이 확인됩니다.    
    
## Oracle Cloud Infrastructure Registry에 hello-world 이미지 푸시

1. Docker를 실행중인 클라이언트 시스템의 터미널 창에서 다음을 입력하여 클라이언트 시스템의 Docker 이미지를 Oracle Cloud Infrastructure Registry로 푸시하십시오.
    ~~~
    docker push <region-code>.ocir.io/<tenancy-name>/<repo-name>/<image-name>:<tag>
    ~~~

    항목:
    - \<region-code>는 fra, iad, lhr 또는 phx 중 하나입니다.
    - ocir.io는 Oracle Cloud Infrastructure 레지스트리 이름입니다.
    - \<tenancy-name>은 사용자가 이미지를 푸시하려는 저장소 (예 : acme-dev)를 소유하는 태넌시의 이름입니다. 사용자는 태넌시에 액세스 할 수 있어야 합니다.
    - \<repo-name> (지정된 경우)은 이미지를 푸시 할 저장소의 이름입니다 (예 : project01). 저장소 지정은 선택 사항입니다. 저장소 이름을 지정하지 않으면 이미지의 이름이 Oracle - Cloud Infrastructure Registry의 저장소 이름으로 사용됩니다.
    - \<image-name>은 Oracle Cloud Infrastructure Registry에서 이미지에 부여 할 이름입니다 (예 : hello-world).
    - \<tag>는 Oracle Cloud Infrastructure Registry에서 이미지에 부여 할 이미지 태그입니다. 지정하지 않으면 latest 입니다.(예 : latest).
    
    예 :
    ~~~
    docker push iad.ocir.io/gse00014941/project01/hello-world
    ~~~

    ![Alt text](https://monosnap.com/image/8lWsmVytbauUKPAf3kwxwhXgOzisrP.png)

1. helloworld 이미지가 푸시됩니다. 만약 여러 레이어의 이미지면 서로 다른 레이어가 차례로 푸시됩니다.

## 이미지가 Oracle Cloud Infrastructure Registry로 푸시되었는지 확인

1. 레지스트리 페이지가 표시된 콘솔을 표시하는 브라우저 창에서 다시로드를 클릭하십시오. hello-world 이미지를 푸시 할 때 생성 된 개인 Hello-world 저장소를 포함하여 액세스 권한이있는 레지스트리의 모든 저장소를 볼 수 있습니다.

    ![Alt text](https://monosnap.com/image/kIkdkGRSH2EodnbROhuEqE5qsyW2Xq.png)

1. 방금 푸시 한 이미지가 포함 된 hello-world 저장소의 이름을 클릭하십시오.
    ![Alt text](https://monosnap.com/image/iNhyr01DEkJMb3JF0NXrnZ44eThDw1.png)
    - 저장소에 여러개의 이미지가 있을 수 있습니다. 현재는 이미지가 하나뿐입니다.
    - 저장소 작성자, 생성시기, 크기, 공개 저장소 또는 개인 저장소인지 여부 등 저장소에 대한 세부 정보를 보십시오.
    - 좌측 하부의 Readme는 저장소와 관한 설명입니다. 현재 아직 readme가 없습니다.

1. 최신 이미지 태그를 클릭하십시오. 세부 정보 섹션에는 이미지 크기, 눌렀을 때의 사용자 수 및 이미지를 가져온 횟수가 표시됩니다.
    ![Alt text](https://monosnap.com/image/AHlYKGuOEN3NY8zuKJJTsxKJBTVFYa.png)

1. (선택 사항) 나중에 이미지를 가져 오려면 이미지 이름 옆의 `Actions`버튼을 클릭하고 `Copy Pull Command`를 선택합니다. 예를 들어, 명령은 다음과 같을 수 있습니다.
    ~~~
    docker pull iad.ocir.io/gse00014941/project01/hello-world:latest
    ~~~



축하합니다! 
성공적으로 DockerHub에서 hello-world 이미지를 가져 와서 태그를 달고 Docker CLI를 사용하여 Oracle Cloud Infrastructure Registry로 푸시했습니다. 이미지가 성공적으로 푸시 되었는지 확인했습니다.

---
[상위 페이지](https://github.com/OracleCloudKr/OKE-Workshop/)