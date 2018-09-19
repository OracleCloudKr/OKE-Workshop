# Lab-100 : Container Engine 환경구성

오라클의 쿠버네티스 클러스터 환경인 Container Engine for Kubernetes. 줄임말로 OKE에 대해서 알아보도록 합니다.

- OCI (Oracle Cloud Infrastructure) 시작하기
- 리소스 제어권한 부여하기
- VCN (Virtual Cluster Network) 생성하기
- 클러스터 세부사항 정의하기
- 노드풀 세부사항 정의하기

## 배경

Kubernetes용 Oracle Cloud Infrastructure Container Engine은 완벽하게 관리되고 확장 가능하며 가용성이 높은 서비스로, 컨테이너화 된 애플리케이션을 클라우드에 전개하는 데 사용할 수 있습니다. 개발팀이 클라우드 기반 응용 프로그램을 안정적으로 구축, 배포 및 관리하기를 원할 때 Kubernetes용 Container Engine을 사용하십시오. 애플리케이션에 필요한 컴퓨팅 리소스를 지정하고 Kubernetes용 컨테이너 엔진은 기존 OCI 테넌트에서 Oracle Cloud Infrastructure에 제공합니다.

이 자습서에서는 VCN 및 관련 리소스를 적절하게 구성하지 않은 경우 먼저 해당 리소스를 만듭니다. 그런 다음 Kubernetes 클러스터를 새로 만듭니다. 노드 풀을 새 클러스터에 추가 한 다음 클러스터에 대한 Kubernetes 구성 파일 (클러스터의 'kubeconfig'파일)을 다운로드합니다. kubeconfig 파일을 사용하면 kubectl과 Kubernetes Dashboard를 사용하여 클러스터에 액세스 할 수 있습니다. 마지막으로 샘플 helloworld 앱을 클러스터에 배포합니다.

## 필요한 것
- Oracle Cloud Infrastructure 사용자 이름 및 비밀번호.
- 임차의 루트 구획 내에서  `allow service OKE to manage all-resources in tenancy` 정책 선언문을 선언하여 컨테이너 엔진이 Kubernetes에게 임차시 리소스에 액세스 할 수 있도록 지정해야합니다.
- 임대 기간 중에 적절하게 구성된 구획이 있어야합니다. 구획에는 자습서 클러스터를 만들고 배포 할 영역에 이미 구성된 필수 네트워크 리소스가 있어야합니다. 즉, 5 개의 `Subnets`, 인터넷 게이트웨이, 경로 테이블 및 2 개의 보안 목록으로 구성되어집니다. 적절한 VCN이 아직없는 경우 지침에 따라 자습서와 함께 사용하도록 특별히 구성된 새 VCN을 만듭니다. 
- 설명 된대로이 자습서를 완료하려면 최소한 세 개의 계산 인스턴스가 있어야합니다. 하나의 계산 인스턴스 만 사용할 수있는 경우 노드 풀에 단일 `Subnets`과 단일 노드가있는 노드 풀이있는 클러스터를 만들 수 있습니다. 그러나 이러한 클러스터는 가용성이 높지 않습니다.
클러스터를 생성 및 / 또는 관리하려면 다음 중 하나에 속해야합니다.
    - 임차인의 관리자 그룹.
    - 정책이 Kubernetes 사용 권한을 위해 적절한 컨테이너 엔진을 부여하는 그룹입니다. 콘솔을 사용하여 클러스터를 만들거나 수정하는 경우 정책은 그룹에 네트워킹 권한 VCN_READ 및 SUBNET_READ도 부여해야합니다.
- 귀하 (귀하가 속한 그룹)는 Oracle Cloud Infrastructure의 ID 및 액세스 관리에서만 정의되어 있어야합니다. Kubernetes 용 컨테이너 엔진은 현재 다른 아이디 공급자 (예 : Oracle Identity Cloud Service 및 Microsoft Active Directory)와 연합 된 그룹 및 사용자를 지원하지 않습니다. 따라서 아직 Oracle Cloud Infrastructure ID 및 액세스 관리에 정의 된 사용자 및 그룹이없는 경우이 자습서를 시작하기 전에 사용자 및 그룹을 정의하십시오 ([Federated users are not supported by Container Engine for Kubernetes](https://docs.cloud.oracle.com/iaas/Content/knownissues.htm#contengfederateduser) 를 참조하십시오).
- 이 튜토리얼의 뒷부분에서 kubeconfig 파일을 다운로드하기 전에 다음을 이미 완료 했어야합니다 (아직 없거나 확신이 없다면 [Downloading a kubeconfig File to Enable Cluster Access](https://docs.us-phoenix-1.oraclecloud.com/Content/ContEng/Tasks/contengdownloadkubeconfigfile.htm)를 참조하십시오). ) :
    - API 서명 키 쌍을 생성했습니다.
    - API 서명 키 쌍의 공개 키 값을 사용자 이름의 사용자 설정에 추가했습니다.
    - Oracle Cloud Infrastructure CLI를 설치 및 구성
Kubernetes 명령 줄 도구 kubectl을 설치하고 구성해야합니다. 아직 수행하지 않았다면, [kubectl documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl)를보십시오.


# OCI (Oracle Cloud Infrastructure) 시작하기
1. 브라우저에서 Oracle Cloud Infrastructure에 로그인하기 위해 제공 한 URL로 이동하십시오. (예: https://console.us-ashburn-1.oraclecloud.com/)

   ![Tenant](https://image.prntscr.com/image/05I_oK2xTJCbV6dysxS_OQ.png)

1. Tenant명에는 제공된 명을 입력합니다.

   ![login](https://image.prntscr.com/image/n4ZodMHLSFK8nC5oRSmD3g.png)

1. 클러스터를 만들 수있는 적절한 권한이있는 임차인을 지정하십시오. 다음 권한 중 하나를 사용하여 이러한 사용 권한을 상속합니다.
    - 임차인의 관리자 그룹에 속함
    - 정책이 Kubernetes 사용 권한을위한 적절한 컨테이너 엔진과 VCN_READ 및 SUBNET_READ 네트워킹 사용 권한을 부여하는 다른 그룹에 속함
1. 사용자 이름과 암호를 입력하십시오.

# 리소스 제어권한 부여하기
Kubernetes를 설정하기 위해서는 권한을 부여해야 한다.

1. 왼쪽 상위의 MENU를 누르고 Identy 항목의 Policies 를 선택한다.

   ![Policies 선택](https://image.prntscr.com/image/X0n6zTtbT-GARx9EYo-EAw.png)
    
1. COMPARTMENT 아래 메뉴를 눌러 (root) 라고 표기된 compartment를 선택한다.

   ![compartment 선택](https://image.prntscr.com/image/gH_TWTO2QMywpI6Ap8XQjw.png)

1. Create Policy 버튼을 클릭하여 새로운 Policy를 만든다.

   ![Policy 생성](https://image.prntscr.com/image/ZF8vpamuSl_sf2FgloIxVw.png)

1. NAME을 쓰고(예:oke-service) STATEMENT에 "`allow service OKE to manage all-resources in tenancy`" 라고 입력한다.

   ![STATEMENT 입력](https://image.prntscr.com/image/xfkQfzYHRz60toO01zjKGw.png)

저장을 하면 Kubernetes 클러스터링을 만들 수 있는 권한이 부여가 되었다.


# VCN (Virtual Cluster Network) 생성하기

클러스터를 만드는 데 적합한 VCN이 아직없는 경우 아래 지침에 따라이 자습서에서만 사용할 기본 설정으로 새 VCN을 만듭니다. 

1. 콘솔에서 왼쪽 상위의 `MENU`를 클릭합니다. `Neworking`의 `Virtual Cloud Network`를 선택하십시오.

   ![VCN Setup](https://image.prntscr.com/image/5mdrunMBRIOXUDtER_TKMA.png)

1. 작업 할 권한이있는 Compartment을 선택하십시오 (이 자습서에 표시된 예제에서는 `Demo`이라는 구획을 가정합니다). 그런 다음 `Virtual Cloud Network`를 클릭하여 가상 클라우드 네트워크 페이지를 표시하십시오.

   ![Choose Compartment](https://image.prntscr.com/image/Fz8cIn14T4S6g9sWgbp11A.png)

1. Create Virtual Cloud Network 버튼을 클릭하십시오.

   ![Create Virtual Cloud Network](https://image.prntscr.com/image/lz1WxmNkQISeCxqC2C4kBg.png)

1. Create Virtual Cloud Network (VCN 생성) 대화 상자에서 새 VCN에 대한 세부 정보를 지정합니다.

    - CREATE IN COMPARTMENT : VCN을 작성할 구획의 이름 (예 : Demo).
    - NAME : 새 VCN을 제공 할 이름입니다. 이 자습서에 표시된 예는 oke-service-vcn 이라는 VCN을 가정합니다.
    - CREATE VIRTUAL CLOUD NETWORK PLUS RELATED RESOURCES : 인터넷 게이트웨이, 기본 경로 테이블 및 세 개의 `Subnets`과 함께 VCN을 만들기 위해 기본값을 사용하려면이 옵션을 선택합니다.

    ![](https://image.prntscr.com/image/SQ8sMgOwRQy09P4sPMDFNQ.png)

1. 하단부의 Create Virtual Cloud Network 버튼을 클릭하여 VCN을 만듧니다. 그리고 VCN 및 관련 리소스가 만들어지면 닫기를 클릭하십시오.

    ![](https://image.prntscr.com/image/K8hLt0cLSzmY4of2yHuX4Q.png)


1. 가상 클라우드 네트워크 페이지에서 새 VCN의 이름을 클릭하면 기본 등록 정보로 생성 된 자원을 볼 수 있습니다.
    - 클러스터의 작업자 노드에 사용할 세 개의 `Subnets`(Subnets)
    - 라우트 테이블(Route Tables)
    - 인터넷 게이트웨이(Internet Gateways)
    - 보안 목록 (Security Lists)
    - DHCP 옵션 (DHCP Options)

    이러한 리소스와 기본 속성으로 대부분 충분합니다. 그러나 다음을 수행해야합니다.
    - 기본 보안 목록에 규칙을 추가하십시오.
    - 로드 밸런서 `Subnets`에서 사용할 보안 목록을 추가로 만듭니다.
    - 로드 밸런서 `Subnets`으로 두 개의 `Subnets`을 추가로 만듭니다.

1. 보안 목록(Security Lists)을 클릭 한 다음 기본 보안 목록의 이름(예:Default Security List for oke-service-run)을 클릭하십시오.

    ![](https://image.prntscr.com/image/td6dXWbfRqGIh8ue5gmOHA.png)

1. 모든 규칙 편집을 클릭하십시오.

    ![](https://image.prntscr.com/image/zWfpEdaJS5qNC_8liTtHag.png)

    이미 기본적으로 생성 된 수신 규칙 및 송신 규칙을 볼 수 있습니다.

1. `Allow Rules for Ingress` 섹션에서 `+ Add Rule` 버튼을 클릭하고 첫 번째 작업자 노드 `Subnets`에 대해 새로운 상태 비 저장 규칙을 지정합니다.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 10.0.10.0/24 (첫 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : All Protocols

    ![](https://image.prntscr.com/image/ocZpitpASZO8TMRPp5zSlw.png)

1. `+ Add Rule` 버튼을 클릭하고 두 번째 작업자 노드 `Subnets`에 대해 새로운 상태 비 저장 규칙을 지정합니다.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 10.0.11.0/24 (두 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : All Protocols
1. `+ Add Rule` 버튼을 클릭하고 세 번째 작업자 노드 `Subnets`에 대해 새로운 상태 비 저장 규칙을 지정합니다.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR :  10.0.12.0/24 (세 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : All Protocols

    ![](https://image.prntscr.com/image/g0T7yknTSOKlHBGNSWNCRQ.png)

1. `+ Add Rule` 버튼을 클릭하고 Kubernetes 용 컨테이너 엔진이 SSH를 통해 작업자 노드에 액세스 할 수있게 해주는 새로운 상태 저장 규칙을 지정합니다.
    - STATELESS : No (확인란을 선택하지 않았습니다).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 130.35.0.0/16
    - IP PROTOCOL : TCP
    - SOURCE PORT RANGE : ALL
    - DESTINATION PORT RANGE : 22

1. `+ Add Rule` 버튼을 클릭하고 Kubernetes 용 컨테이너 엔진이 SSH를 통해 작업자 노드에 액세스 할 수있게하는 두 번째 새로운 상태 저장 규칙을 지정합니다.
    - STATELESS : No (확인란을 선택하지 않았습니다).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 138.1.0.0/17
    - IP PROTOCOL : TCP
    - SOURCE PORT RANGE : ALL
    - DESTINATION PORT RANGE : 22
    
    ![](https://image.prntscr.com/image/ND3BxOGTTfOijLjwXlU5-A.png)

1. `+ Add Rule` 버튼을 클릭하고 기본 NodePort 범위에서 작업자 노드에 액세스 할 수있는 새로운 상태 저장 규칙을 지정합니다.
    - STATELESS : No (확인란을 선택하지 않았습니다).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 0.0.0.0/0
    - IP PROTOCOL : TCP
    - SOURCE PORT RANGE : ALL
    - DESTINATION PORT RANGE : 30000-32767

    ![](https://image.prntscr.com/image/RLbxpPDqSOGC7ixSaMjMQg.png)

1. `Allow Rules for Egress` 섹션에서 `+ Add Rule` 버튼을 클릭합니다.
![](https://image.prntscr.com/image/EFoNSRqhRDyih5a95qL-7Q.png)


1. 첫 번째 작업자 노드 `Subnets`에 대해 새로운 상태 비 저장 규칙을 지정하십시오.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 10.0.10.0/24 (첫 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : All Protocols

1. `+ Add Rule` 버튼을 클릭하고 두 번째 작업자 노드 `Subnets`에 대해 새로운 상태 비 저장 규칙을 지정합니다.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 10.0.11.0/24 (두 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : All Protocols

1. `+ Add Rule` 버튼을 클릭하고 세 번째 작업자 노드 `Subnets`에 대해 새로운 상태 비 저장 규칙을 지정하십시오.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 10.0.12.0/24 (세 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : All Protocols

    ![](https://image.prntscr.com/image/SInCivRbT8GzBuFK9bbXTA.png)

1. 변경 사항을 기본 보안 목록에 저장하려면 `Save Security Rules`를 클릭하십시오.

    이제로드 밸런서 `Subnets`에 대한 새 보안 목록을 만듭니다.

1. `Security Lists`을 클릭 한 다음 `Create Security List`를 클릭하십시오.

    ![](https://image.prntscr.com/image/VJZO3SmxQWm-FNWMQL9TxQ.png)

1. 보안 목록 작성 대화 상자에서 새 보안 목록의 이름을 입력하십시오. 

    이 자습서에 나와있는 예제에서는 새 보안 목록에 `Load Balancers` 라는 이름을 부여한다고 가정합니다.

1. `Allow Rules for Ingress` 섹션에서 새로운 상태 비 저장 규칙을 지정하십시오.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 0.0.0.0/0 (세 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : TCP
    - SOURCE PORT RANGE : ALL
    - DESTINATION PORT RANGE : ALL

1. `Allow Rules for Engress` 섹션에서 새로운 상태 비 저장 규칙을 지정하십시오.
    - STATELESS : Yes (체크 박스가 선택됨).
    - SOURCE TYPE : CIDR
    - SOURCE CIDR : 0.0.0.0/0 (세 번째 작업자 노드 `Subnets`의 CIDR입니다).
    - IP PROTOCOL : TCP
    - SOURCE PORT RANGE : ALL
    - DESTINATION PORT RANGE : ALL
    
    ![](https://image.prntscr.com/image/2YUTC8-bTkuWQfFL18V2Ag.png)


1. 하위의 `Create Security List`를 클릭하여 보안 목록을 만듧니다. 
로드 밸런서를 위한 보안 목록을 만든 다음 로드 밸런서 서브넷을 만듭니다. 이 자습서에 표시된 예에서는 새 부하 분산 장치 서브넷에 LB Subnet1 및 LB Subnet2라는 이름을 부여한다고 가정합니다.

1. `Subnets`을 클릭 한 다음 `Create Subnet` 를 클릭하여 첫 번째 새 부하 분산 장치 서브넷을 만듭니다.
    ![Alt text](https://monosnap.com/image/Q5fErLQ5ETiKv8YEnbF7zYXhTFXyZH.png)


1. 서브넷 만들기 대화 상자에서 첫 번째 새로드 밸런서 서브넷에 대한 세부 정보를 지정합니다.
    - NAME : LB Subnet1
    - AVAILABLITY DOMAIN : 첫 번째 부하 분산 장치 서브넷을 생성 할 가용성 도메인의 이름입니다 (예 : HrCE : PHX-AD-1).
    - CIDR BLOCK : 10.0.20.0/24
    - ROUTE TABLE : Default Route Table for oke-service-vcn
    - SUBNET ACCESS : PUBLIC SUBNET
    - DNS RESOLUTION : 선택됨 (이 서브넷의 DNS 호스트 이름 사용)
    
    ![Alt text](https://monosnap.com/image/vvjlBY3yWg78IaQyLGoxmwl9KL0WSr.png)

    - DHCP OPTION : Default DHCP Options for oke-service-vcn
    - Security Lists : Load Balancers
        
    ![Alt text](https://monosnap.com/image/RtE0PgwPbCiovOq6shZLCosKe1wRHg.png)

1. Create를 클릭하여 지정한 세부 정보가 포함 된 첫 번째 새 부하 분산 장치 서브넷을 만듭니다.
    
1. 두 번째 새 부하 분산 장치 서브넷을 만들려면 `Create Subnets`를 클릭합니다.
1. `Create Subnets` 대화 상자에서 두 번째 새로드 밸런서 서브넷에 대한 세부 정보를 지정합니다.
    - NAME : LB Subnet2
    - AVAILABLITY DOMAIN : 두 번째 부하 분산 장치 서브넷을 생성 할 가용성 도메인의 이름입니다 (예 : HrCE : PHX-AD-2).
    - CIDR BLOCK : 10.0.21.0/24
    - ROUTE TABLE : Default Route Table for oke-service-vcn
    - SUBNET ACCESS : PUBLIC SUBNET
    - DNS RESOLUTION : 선택됨 (이 서브넷의 DNS 호스트 이름 사용)
    - DHCP OPTION : Default DHCP Options for oke-service-vcn
    - Security Lists : Load Balancers

1. `Create`를 클릭하여 지정한 세부 정보가 있는 두 번째 새 부하 분산 장치 서브넷을 만듭니다.

    ![Alt text](https://monosnap.com/image/w5c3SpCvVWNLZqQFGZJbMrbJ9Ftksz.png)

이제 Subnet 5개, Reoute table 1개, Internet Gateway 1개, Security List 2개, DHCP Option 1개를 가진 환경으로 Kubernetes 클러스터를 만들 준비가되었습니다.


# 클러스터 세부사항 정의하기

1. 콘솔에서 탐색 메뉴를 엽니 다. 솔루션, 플랫폼 및 에지에서 개발자 서비스로 이동하여 컨테이너 클러스터를 클릭하십시오.
![Alt text](https://monosnap.com/image/3ulRXs7Gx3yREDYwBHCbJrU4uHtZA2.png)

1. 작업 할 권한이있는 적절하게 구성된 Compartment를 선택한 다음, Clusters를 클릭하십시오.
![Alt text](https://monosnap.com/image/Zq4lBScxFsoG4JghbDjJt9ecyPjLJ5.png)

1. 클러스터 페이지에서 `Create Cluster`를 클릭하십시오. 이 자습서에 표시된 예는 새 클러스터에 `Kube Cluster`라는 이름을 부여한다고 가정합니다.
![Alt text](https://monosnap.com/image/YSXVruSZC4fVKcwkPpOSx1rfWwTn4z.png)

1. Create Cluster 대화 상자에서 새 클러스터에 대한 세부 사항을 지정하십시오.
    - Name: Kube Cluster
    - Version: 목록에 표시된 Kubernetes 중 v1.10.3 버젼 (2018년 09월 현재 v1.11.1 은 약간의 버그가 있음)
    - VCN: 클러스터를 생성 할 VCN의 이름입니다 (예 : oke-service-vcn)
    - Kubernetes Service LB Subnets: 로드밸러스 조정에 사용할 VCN의 두 서브넷 이름 (예 : LB Subnet1 및 LB Subnet2).
    - Kubernetes Service CIDR Block: 10.96.0.0/16 (기본값)
    - Pods CIDR Block: 10.244.0.0/16 (기본값)
    - Kubernetes Dashboard Enabled: Select 
    - Tiller (Helm) Enabled: Select 

    ![Alt text](https://monosnap.com/image/hJV5YSB5oY4ALpcpFV9qA26diUiEGH.png)
    ![Alt text](https://monosnap.com/image/kgRxf1iBLcpeNpB3bkH4SDPYTN5H0F.png)

1. Create를 클릭하여 새 클러스터를 만듭니다.
    ![Alt text](https://monosnap.com/image/DD5BkQxPcUEy0KFfGpbo3lTWje4MRP.png)

    새 클러스터가 클러스터 페이지에 표시됩니다. 작성된 새 클러스터의 상태는 활성입니다. 클러스터를 생성하면 클러스터의 노드 풀과 작업자 노드 (계산 인스턴스)를 정의 할 준비가되었습니다.


# 노드풀 세부사항 정의하기

1. 클러스터 페이지에서 `Kube Cluster`의 이름을 클릭하여 선택하십시오.

1. `Add Node Pool`를 클릭하십시오. 이 자습서에 표시된 예제에서는 새 노드 풀 이름을 `KubeNodePool`로 가정합니다.

    ![Alt text](https://monosnap.com/image/2N0cMzN4kYgecooZM3tTxmztXyEMdD.png)

1. 노드 풀 추가 대화 상자에서 노드 풀에 대한 세부 사항을 지정하십시오.
    - `NAME` : KubeNodePool
    - `VERSION` : 목록에 표시된 Kubernetes 중 v1.10.3 버젼 (2018년 09월 현재 v1.11.1 은 약간의 버그가 있음)
    - `IMAGE` : Oracle-Linux-7.4 (7.5도 선택가능)
    - `SHAPE` : VM.Standard2.2 (사용 가능한 쉐입을 선택합니다.)
    - `SUBNETS` : 작업 노드를 만들 VCN의 세 서브넷 이름 (예 : Public Subnet : US-ASHBURN-AD-1, Public Subnet : US-ASHBURN-AD-AD-2, Public Subnet : US-ASHBURN-AD-AD-3). 이러한 서브넷은 로드 밸런스 조정을 위해 선택한 서브넷과 달라야합니다.
    - `QUANTITY PER SUBNET` : 1
    - `PUBLIC SSH KEY` : 공란으로 두십시오 (이 자습서를 완료하려면 작업자 노드에 대한 SSH 액세스가 필요하지 않습니다)

    ![Alt text](https://monosnap.com/image/OY025iYDYCHbmtrmA3sX3lcq00xc6y.png)

1. 추가를 클릭하여 새 노드 풀을 작성하십시오.

    새 노드 풀은 클러스터 페이지에 표시됩니다. 노드 풀의 작업자 노드 이름은 `Instance Name` 열에 표시됩니다.
    ![Alt text](https://monosnap.com/image/s9Frfy7h0CVeqYKH02TDawx7i40wfO.png)

---
[상위 페이지](https://github.com/OracleCloudKr/OKE-Workshop/)