# Cluster 환경 탐색

쿠버네티스 클러스터를 살펴보기 위하여 다음과 같은 항목들을 수행하도록 합니다.

- Oracle Cloud Infrastructure CLI 설치 및 구성
- API 서명 키 쌍의 공개 키 값을 사용자 이름의 사용자 설정
- 클러스터 용 kubeconfig 파일 다운로드
- kubectl 및 Kubernetes 대시 보드에 대한 클러스터 액세스 확인

# Oracle Cloud Infrastructure CLI 설치 및 구성
쿠버네티스 클라이언트를 위한 CLI 및 필수 소프트웨어를 설치하는 단계에 대해 설명합니다.

## 설치 옵션
CLI를 설치하는 데는 두 가지 방법이 있으며 사용자 환경에 가장 적합한 방법을 사용할 수 있습니다. 다음 옵션 중 하나를 사용하십시오.

- CLI 설치 프로그램에서 CLI 및 종속성 자동 설치
    - 아래에서 설명합니다.
- 가상 환경에서 CLI 및 종속성 수동 설치
    - [Installing the CLI](https://docs.cloud.oracle.com/iaas/Content/API/SDKDocs/cliinstall.htm?TocPath=Developer%20Tools%20|Command%20Line%20Interface%20(CLI)%20|_____1) 를 참조 바랍니다.

## CLI 설치 프로그램 사용
설치 프로그램은 스크립트를 사용하여 필요한 CLI 및 프로그램을 설치합니다. 설치 프로그램을 사용하여 대부분의 수동 단계를 제거하여 CLI를 설치할 수 있습니다. 설치 프로그램 스크립트 :

- 파이썬 설치
    설치 중에 바이너리 및 실행 파일 설치 위치를 제공하라는 메시지가 표시됩니다. Python이 컴퓨터에 설치되어 있지 않거나 설치된 Python 버전이 CLI와 호환되지 않으면 Python을 설치하라는 메시지가 나타납니다.
    설치 프로그램은 MacOS 컴퓨터에 Python을 설치하려고하지 않습니다. 그러나 컴퓨터의 Python 버전이 너무 오래되었거나 호환되지 않는 경우 스크립트에서 알려줍니다.
- virtualenv를 설치하고 가상 환경을 만듭니다.
- 최신 버전의 CLI를 설치합니다.
    설치 프로그램은 기존 설치를 겹쳐 쓰도록 선택하면이를 덮어 씁니다. CLI를 최신 버전으로 업그레이드하라는 메시지가 나타나면 Y로 응답하십시오.
    설치 중에 PATH를 업데이트 할 것인지 묻는 메시지가 나타납니다. PATH를 업데이트하면 PATH에 CLI 실행 파일 ( "oci.exe")이 추가됩니다. oci.exe를 PATH에 추가하면 실행 파일의 전체 경로를 제공하지 않고 CLI를 호출 할 수 있습니다. PATH를 업데이트하려면 프롬프트가 표시되면 Y로 응답하십시오. 터미널 세션을 닫고 다시 시작할시기는 사용자에게 통보됩니다.

### Windows 컴퓨터에 CLI 설치
1. 관리자 권한으로 실행 옵션을 사용하여 PowerShell 콘솔을 엽니 다.
1. 설치 프로그램은 스크립트를 설치하고 실행하여 자동 완성을 사용 가능하게합니다. 이 스크립트를 실행하려면 RemoteSigned 실행 정책을 사용해야합니다.
1. PowerShell에 대한 원격 실행 정책을 구성하려면 다음 명령을 실행합니다.
    ~~~
    Set-ExecutionPolicy RemoteSigned
    ~~~
1. 설치 프로그램 스크립트를 실행하려면 다음 명령을 실행하십시오.
    ~~~
    powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient) .DownloadString ( 'https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.ps1) '))) "
    ~~~
1. 설치 프로그램의 프롬프트에 응답하십시오.

### Bash가 설치된 컴퓨터에 CLI 설치
1. 터미널을 엽니다.
1. 설치 프로그램 스크립트를 실행하려면 다음 명령을 실행하십시오.
    ~~~
    $ bash -c "$ (curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
    ~~~
1. 설치 프로그램의 프롬프트에 응답하십시오.
1. 현재 터미널에 적용하기 위해서 profile을 다시 수행합니다.
    ~~~
    $ . ./.bash_profile
    ~~~





## CLI 설정하기

OCI CLI에 대한 설정환경을 만들기 위해서 다음과 같이 명령합니다.
~~~sh
$ oci setup config
~~~
![Alt text](https://monosnap.com/image/XadFIHn2xMvM1crIRiQXDZWuYppX2V.png)
수행을 하면 다음과 같이 질문사항이 나옵니다.
- Enter a location for your config [/Users/user/.oci/config]:
    - 설정파일이 존재할 디렉토리 - 기본값으로 엔터를 입력합니다.
- Enter a user OCID: 
    - user OCID 값은 User Settings 화면의 중간에 User Information 항목에 있습니다. Copy 버튼을 누르면 복사됩니다.
    ![Alt text](https://monosnap.com/image/a2wldUsviprPPy3uTO43ikShG80LCM.png)
    ![Alt text](https://monosnap.com/image/nHcp3sPnoJ4OJiQWMFNSLOooshGjXz.png)
- Enter a tenancy OCID: 
    - tenancy OCID 값은 콘솔 화면의 하위에 있습니다. 
    ![Alt text](https://monosnap.com/image/feihVfMby4CoQmqMCXiz0GNPF7gSsm.png)
- Enter a region (e.g. eu-frankfurt-1, uk-london-1, us-ashburn-1, us-phoenix-1): 
    - region은 상단 우측에 존재합니다.
    ![Alt text](https://monosnap.com/image/qcV8dVveQKnHLX7BMIkFfP2c4uiEDE.png)
- Do you want to generate a new RSA key pair? (If you decline you will be asked to supply the path to an existing key.) [Y/n]:
    - 키페어를 생성합니다 - 기본값으로 엔터를 입력합니다.
- Enter a directory for your keys to be created [/Users/user/.oci]:
    - 키페어가 위치할 디렉토리를 지정합니다 - 기본값으로 엔터를 입력합니다.
- Enter a name for your key [oci_api_key]:
    - 개인키명 - 기본값으로 엔터를 입력합니다.
- File /Users/user/.oci/oci_api_key_public.pem already exists, do you want to overwrite? [y/N]: y
    - 기존에 존재하고 있으면 새로 생성해서 갱신할 여부를 묻습니다 - 기존에 있다면 갱신을 위해서 y를 입력하고 엔터를 입력합니다.
- Public key written to: /Users/user/.oci/oci_api_key_public.pem
    - 공개키명 - 기본값으로 엔터를 입력합니다.
- Enter a passphrase for your private key (empty for no passphrase):
    - 패스워드를 입력할 지 여부 - 패스워드 필요없이 엔터를 입력합니다.
- File /Users/user/.oci/oci_api_key.pem already exists, do you want to overwrite? [y/N]: y
    - 기존에 존재하고 있으면 새로 생성해서 갱신할 여부를 묻습니다 - 기존에 있다면 갱신을 위해서 y를 입력하고 엔터를 입력합니다.


# API 서명 키 쌍의 공개 키 값을 사용자 이름의 사용자 설정
PEM 공개 키는 https://console.us-ashburn-1.oraclecloud.com 에있는 콘솔에서 업로드 할 수 있습니다. 콘솔에 대한 로그인 및 암호가 없으면 관리자에게 문의하십시오.

1. 콘솔을 열고 로그인하십시오.
1. 키 쌍으로 API를 호출 할 사용자의 세부 정보를 봅니다.
    ![Alt text](https://monosnap.com/image/a2wldUsviprPPy3uTO43ikShG80LCM.png)
    - 이 사용자로 로그인 한 경우 콘솔의 오른쪽 상단에서 사용자 이름을 클릭 한 다음 사용자 설정을 클릭합니다.
    - 다른 사용자를 위해이 작업을 수행하는 관리자 인 경우 ID 대신 사용자를 클릭하고 목록에서 사용자를 선택하십시오.
1. 공개 키 추가를 클릭하십시오.
    ![Alt text](https://monosnap.com/image/c3SMFI80UxPr5XcpEIZKyWTubvkIXu.png)
    공개키는 윗부분에서 언급한 바와 같이 다음과 같이 복사할 수 있습니다.
    ~~~
    cat ~/.oci/oci_api_key_public.pem | pbcopy
    ~~~
1. 대화 상자에 PEM ​​공개 키의 내용을 붙여 넣은 다음 추가를 클릭합니다.

키의 지문이 표시됩니다. 예:
~~~
12:34:56:78:90:ab:cd:ef:12:34:56:78:90:ab:cd:ef
~~~

첫 번째 공개 키를 업로드 한 후 UploadApiKey API 작업을 사용하여 추가 키를 업로드 할 수도 있습니다. 사용자 당 최대 3 개의 API 키 쌍을 가질 수 있습니다. API 요청에서 키의 지문을 지정하여 요청에 서명하는 데 사용할 키를 나타냅니다.


# 클러스터 용 kubeconfig 파일 다운로드

1. 이미 다음 작업을 완료했는지 확인하십시오.
    - API 서명 키 쌍을 생성했습니다.
    - API 서명 키 쌍의 공개 키 값을 사용자 이름의 사용자 설정에 추가했습니다.
    - Oracle Cloud Infrastructure CLI를 설치 및 구성했습니다.
    
    위의 작업 중 하나 이상을 수행하지 않았거나 확실하지 않은 경우, Kubernetes 설명서의 Container Engine에서 [kubeconfig 파일을 다운로드하여 클러스터 액세스를 활성화하는 방법](https://docs.cloud.oracle.com/iaas/Content/ContEng/Tasks/contengdownloadkubeconfigfile.htm) 항목을 참조하십시오.

1. `Kube Cluster`에 대한 세부 정보가 표시된 클러스터 페이지에서 Access Kubeconfig를 클릭하여 Kubeconfig 액세스 방법 대화 상자를 표시합니다.
    ![Alt text](https://monosnap.com/image/vhd4fUiyL5yZPJLsjH1IEOTxO8FwE6.png)

1. 단말기 창에서 kubeconfig 파일을 포함 할 디렉토리를 작성하여 $HOME/.kube 의 예상 기본 이름을 디렉토리에 제공하십시오. 예를 들어 Linux의 경우 다음 명령을 입력하거나 Kubeconfig 액세스 방법 대화 상자에서 복사하여 붙여 넣으십시오.
    ~~~sh
    $ mkdir -p $HOME/.kube
    ~~~
1. Oracle Cloud Infrastructure CLI 명령을 실행하여 kubeconfig 파일을 다운로드하고 $ HOME/.kube/config의 예상 기본 이름과 위치와 함께 저장하십시오. 이 이름과 위치는 kubeconfig 파일이 터미널 창에서 실행될 때마다 kubectl 및 Kubernetes 대시 보드에 액세스 할 수 있도록합니다. 예를 들어 Linux의 경우 다음 명령을 입력하거나 Kubeconfig 액세스 방법 대화 상자에서 복사하여 붙여 넣으십시오.
    ~~~sh
    $ oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.phx.aaaaaaaaae ... --file $HOME/.kube/config
    ~~~
    여기서 `ocid1.cluster.oc1.phx.aaaaaaaaae ...`는 현재 클러스터의 OCID입니다. 편의상 Kubeconfig 액세스 방법 대화 상자의 명령에 이미 클러스터의 OCID가 포함되어 있습니다.
1. 닫기를 클릭하여 Kubeconfig 액세스 방법 대화 상자를 닫습니다.


# kubectl 및 Kubernetes 대시 보드에 대한 클러스터 액세스 확인

1. kubectl을 이미 설치했는지 확인하십시오. 아직 수행하지 않았다면, [kubectl 문서](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)를보십시오.
1. kubectl을 사용하여 생성 한 새 클러스터에 연결할 수 있는지 확인하십시오. 터미널 창에서 다음 명령을 입력하십시오.
    ~~~sh
    $ kubectl get nodes
    ~~~
    클러스터에서 실행중인 노드의 세부 사항을 볼 수 있습니다. 예 :
    ![Alt text](https://monosnap.com/image/qgi3mxbKZxFWjnRZB71hv6jM6Yd1i3.png)
1. Kubernetes 대시 보드를 사용하여 클러스터에 연결할 수 있는지 확인하십시오.
    1. 터미널 창에서 다음 명령을 입력하십시오.
    ~~~sh
    $ kubectl proxy
    ~~~
    1. 새 브라우저 창을 열고 http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/ 로 이동하여 Kubernetes 대시 보드를 표시하십시오.

    ![Alt text](https://monosnap.com/image/EF82F9HMYNWVpgnuuX0tOtI5FM5LBy.png)

    1. Kubeconfig 옵션을 선택하고 kubeconfig 파일 선택을 클릭 한 다음 이전에 다운로드 한 kubeconfig 파일을 선택하십시오.
    1. 로그인을 클릭하십시오.
    1. 개요를 클릭하여 Kubernetes가 클러스터에서 실행중인 유일한 서비스인지 확인하십시오.

    ![Alt text](https://monosnap.com/image/oyAP3vyI9mzajNrJBFP47QV7h82k6U.png)

    새 클러스터가 정상적으로 작동 중임을 확인했습니다. 이제 응용 프로그램을 클러스터에 배포 할 수 있습니다.


---
[상위 페이지](https://github.com/OracleCloudKr/OKE-Workshop/)