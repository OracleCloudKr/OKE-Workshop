![](https://ps.w.org/under-construction-page/assets/screenshot-2.png?rev=1840052)

# Kubernetes 용 컨테이너 엔진 개요
Kubernetes 용 Oracle Cloud Infrastructure Container Engine은 완벽하게 관리되고 확장 가능하며 가용성이 높은 서비스로, 컨테이너 화 된 애플리케이션을 클라우드에 전개하는 데 사용할 수 있습니다. 개발 팀이 클라우드 기본 응용 프로그램을 안정적으로 구축, 배포 및 관리하기를 원할 때 Kubernetes 용 Container Engine을 사용합니다 (때로는 OKE로 약칭 함). 애플리케이션에 필요한 컴퓨팅 리소스를 지정하고 Kubernetes 용 컨테이너 엔진은 기존 OCI 테넌트에서 Oracle Cloud Infrastructure에 제공합니다.

Kubernetes 용 Container Engine은 호스트 클러스터 전반에서 컨테이너 화 된 응용 프로그램의 배포, 확장 및 관리를 자동화하는 오픈 소스 시스템 인 Kubernetes를 사용합니다. Kubernetes는 응용 프로그램을 구성하는 컨테이너를 쉽게 관리하고 발견 할 수 있도록 논리 단위 (포드라고 함)로 그룹화합니다. Kubernetes 용 컨테이너 엔진은 클라우드 네이티브 컴퓨팅 Foundation (CNCF)이 승인 한 Kubernetes 버전을 사용합니다.

Kubernetes 용 Container Engine에 액세스하여 Console 및 REST API를 사용하여 Kubernetes 클러스터를 정의하고 작성할 수 있습니다. Kubernetes 명령 줄 (kubectl), Kubernetes 대시 보드 및 Kubernetes API를 사용하여 만든 클러스터에 액세스 할 수 있습니다.

Kubernetes 용 컨테이너 엔진은 고유 한 Oracle Cloud Infrastructure ID 기능으로 손쉬운 인증을 제공하는 Oracle Cloud Infrastructure 신원 및 액세스 관리 (IAM)와 통합되어 있습니다.

소개 자습서는 Kubernetes 용 Oracle Cloud Infrastructure Container 엔진을 사용하여 클러스터 작성 및 샘플 애플리케이션 배치를 참조하십시오.

## Oracle 클라우드 인프라에 액세스하는 방법
콘솔 (브라우저 기반 인터페이스) 또는 REST API를 사용하여 Oracle Cloud Infrastructure에 액세스 할 수 있습니다. 콘솔 및 API에 대한 지침은이 안내서의 주제에 포함되어 있습니다. 사용 가능한 SDK 목록은 Oracle Cloud Infrastructure SDK를 참조하십시오.

콘솔에 액세스하려면 지원되는 브라우저를 사용해야합니다. 이 페이지 상단의 콘솔 링크를 사용하여 로그인 페이지로 이동할 수 있습니다. 클라우드 세입자, 사용자 이름 및 암호를 입력하라는 메시지가 표시됩니다.

API 사용에 대한 일반적인 정보는 REST API를 참조하십시오.

## 자원 식별자
각 Oracle Cloud Infrastructure 리소스에는 Oracle Cloud ID (OCID)라고하는 고유 한 Oracle 할당 식별자가 있습니다. OCID 형식 및 리소스를 식별하는 다른 방법에 대한 자세한 내용은 리소스 식별자를 참조하십시오.

## 인증 및 권한 부여
Oracle Cloud Infrastructure의 각 서비스는 모든 인터페이스 (콘솔, SDK 또는 CLI 및 REST API)에 대한 인증 및 권한 부여를 위해 IAM과 통합됩니다.

조직의 관리자는 어떤 사용자가 어떤 서비스, 리소스 및 액세스 유형에 액세스 할 수 있는지를 제어하는 ​​그룹, 구획 및 정책을 설정해야합니다. 예를 들어 정책은 새 사용자를 만들 수있는 사람, 클라우드 네트워크를 만들고 관리 할 수있는 사용자, 인스턴스를 시작하고 버킷을 만들고 개체를 다운로드 할 수있는 사용자 등을 제어합니다. 자세한 내용은 정책 시작하기를 참조하십시오. 각 서비스 별 정책 작성에 대한 자세한 내용은 정책 참조를 참조하십시오.

귀사가 소유하고있는 Oracle Cloud Infrastructure 리소스를 사용해야하는 일반 사용자 (관리자 아님) 인 경우 관리자에게 문의하여 사용자 ID를 설정하십시오. 관리자는 사용해야하는 구획 또는 구획을 확인할 수 있습니다.

Kubernetes 용으로 Container Engine에서 만든 클러스터에서 특정 작업을 수행하려면 Kubernetes RBAC 역할 또는 clusterrole을 통해 추가 사용 권한이 필요할 수 있습니다. Kubernetes의 액세스 제어 및 컨테이너 엔진 정보를 참조하십시오.

## Kubernetes 기능 및 한계를위한 컨테이너 엔진
임차가 가능한 각 지역에서는 기본적으로 세 개의 클러스터를 만들 수 있습니다. 생성하는 각 클러스터는 최대 1000 개의 노드를 가질 수 있습니다.

## 필수 IAM 서비스 정책
Oracle Cloud Infrastructure를 사용하려면 콘솔 또는 SDK, CLI 또는 기타 도구와 함께 REST API를 사용하는 경우 관리자가 작성한 정책에 필요한 액세스 유형을 제공해야합니다. 작업을 수행하고 권한이 없거나 허가되지 않은 메시지를 받으면 관리자에게 부여한 액세스 유형과 작업해야 할 구획을 확인하십시오.

새로운 정책 인 경우 정책 및 공통 정책 시작하기를 참조하십시오. Kubernetes 용 컨테이너 엔진 정책에 대한 자세한 내용은 Kubernetes 용 컨테이너 엔진에 대한 세부 정보를 참조하십시오.