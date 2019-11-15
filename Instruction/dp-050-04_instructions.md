---
lab:
    title: '랩 4 - Azure로 SQL 워크로드 마이그레이션'
    module: '모듈 4: SQL Database로 SQL 워크로드 마이그레이션'
---

# DP-050 - Azure로 SQL 워크로드 마이그레이션

## 랩 4 - Azure로 SQL 워크로드 마이그레이션  

**예상 소요 시간:** 60분

**필수 구성 요소:** 이 랩에서는 수행해야 할 필수 단계가 없습니다.

**랩 파일:** 이 랩용 랩 파일은 없습니다.

## 랩 개요

이 랩에서는 Azure SQL Database로의 마이그레이션을 수행합니다. 먼저 스키마 마이그레이션을 수행하고 필수 구성 요소인 대상 데이터베이스를 만든 다음 SQL 인스턴스에서 실행되는 데이터베이스에서 Azure의 SQL Database로 마이그레이션합니다. 구체적으로는 DMS(Database Migration Service)를 사용하여 온라인 마이그레이션을 수행하며, 데이터 마이그레이션이 중단될 때까지 원본과 대상 데이터베이스의 데이터를 동기화 상태로 유지합니다.

## 랩 목표

이 랩을 마치면 다음 작업을 수행하는 방법을 배울 수 있습니다.

1. Azure Cloud Shell을 사용하여 SQL Database 만들기
1. Azure Data Migration Service 구성
1. Azure SQL Database로 데이터베이스 스키마 마이그레이션
1. 데이터 마이그레이션 서비스를 사용하여 온라인 마이그레이션 수행

## 시나리오

여러분은 데이터 현대화 프로젝트 실행을 준비 중인 AdventureWorks의 선임 데이터 관리 책임자입니다. Azure Virtual Machine의 SQL Server로 데이터베이스 집합을 마이그레이션하는 데 필요한 환경을 준비하고 Data Migration Assistant를 사용하여 테스트 마이그레이션을 수행하려고 합니다.

## 연습 1: Azure Cloud Shell을 사용하여 SQL Database 만들기

이 연습에서는 Azure Cloud Shell을 사용하여 다음 작업을 수행합니다.

- 새 리소스 그룹 만들기

- 새 Azure SQL Database 서버 인스턴스 만들기

- Azure SQL Database 서버 방화벽 구성

- 새 범용 Azure SQL Database 만들기

**예상 소요 시간:** 20분

Azure에는 서비스 설치/관리/배포를 자동화할 수 있는 여러 옵션이 포함되어 있습니다. 이러한 옵션 중 하나는 간단한 플랫폼 간 명령줄 도구인 Azure CLI 명령줄 도구를 설치하는 것입니다. Azure Cloud Shell을 사용하여 해당 작업을 자동화하고 스크립트로 생성하는 옵션도 있습니다.

Azure Cloud Shell은 Azure에서 호스트되며 브라우저를 통해 관리할 수 있는 대화형 셸 환경입니다. Cloud Shell을 사용하면 bash 또는 PowerShell을 통해 Azure 서비스를 사용할 수 있습니다. Cloud Shell의 미리 설치된 명령을 사용하는 경우 로컬 환경에 별도의 도구를 설치하지 않고도 이 문서에 나와 있는 코드를 실행할 수 있습니다.

### 태스크 1: 로그온 및 Azure Cloud Shell 구성

참고: 이 태스크는 Azure Portal에서 모두 완료할 수 있습니다.

1. [Azure Shell](https://shell.azure.com)로 이동합니다.
1. 이 교육에서 사용하는 자격 증명으로 Azure 구독에 로그온합니다.

 ![Azure Shell 시작 > 로그인 창](/images/azureshell.png)

3. 스크립팅 환경으로 Bash를 선택합니다.

![Bash 셸을 선택하는 Azure Shell 화면](/images/bash.png)

### 태스크 2: Azure Cloud Shell용 새 스토리지 계정 및 공유 만들기

1. Azure Cloud Shell용으로 생성된 스토리지 계정이 없다는 메시지가 표시되면 **고급 설정 표시**를 클릭합니다.
1. 다음 세부 정보를 입력합니다.
    1. 구독: **이 랩에 사용하는 Azure 구독을 선택합니다.**
    1. Cloud Shell 지역: **실제 지리적 위치에 가장 가까운 사용 가능한 Cloud Shell 지역을 선택합니다.**
    1. 리소스 그룹:  
        1. **새로 만들기**를 선택합니다.
        1. **dp050lab4rg**를 입력합니다.
    1. 스토리지 계정:
        1. **새로 만들기**를 선택합니다.
        1. **dp050sa&lt;youridentifier&gt;**를 입력합니다. 여기서 **youridentifier**는 고유한 이름입니다.
    1. 파일 공유:
        1. **새로 만들기**를 선택합니다.
        1. **dp050share&lt;youridentifier&gt;**를 입력합니다. 여기서 **youridentifier**는 고유한 이름입니다.
    1. 스토리지 만들기를 선택합니다.

스토리지 만들기가 완료되면 Azure Cloud Shell 명령줄이 열립니다. 

### 태스크 3: 다음 스크립트를 수정하고 실행하여 SQL Database 서버 인스턴스 및 SQL Database를 만듭니다.

1. **{}** 아이콘을 클릭하여 편집기를 엽니다.
1. 아래 스크립트를 붙여넣고 파일로 저장합니다. **CreateSQLDBandServer**

&lt;스크립트 파일은 호스트형 랩 환경의 Labfiles 폴더와 github의 lab starter 폴더에서도 제공됩니다.&gt;

```shell
#bash script to create a new sql db server instance and sql db  

#Disclaimer: this script is a sample script on how to create an Azure database but uses least restrictive firewall settings for lab purposes. Do not use this script  

#defining variable to be passed to Azure CloudShell Bash

resourcegroup=dp-050-labresourcegroup

#edit the variable below to provide a unique server name

servername=dp-050-servername

#edit the variable below to provide the location – azure locations can be listed by typing az account list-locations -o table in the shell command interface

location=eastus

adminuser=sqladmin

password=Pa55w.rd.123456789

firewallrule=dp-050-access

#edit the script to provide a unique database name

labdatabase=dp-050-AdventureworksLT2008R2

#creates a resource group

az group create --name $resourcegroup --location $location

#creates a sql db servername

az sql server create --name $servername --resource-group $resourcegroup --admin-user $adminuser --admin-password $password --location $location

#shows current firewall list

az sql server firewall-rule list --resource-group $resourcegroup --server $servername

#creates a server-based firewall – note – you should restrict the start/end ip range based on your environment

az sql server firewall-rule create --resource-group $resourcegroup --server $servername --name $firewallrule --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255

#creates a general-purpose SQL database

az sql db create --name $labdatabase --resource-group $resourcegroup --server $servername -e GeneralPurpose -f Gen4 -c 1 

```

3. **sh CreateSQLDBandServer**를 입력하여 스크립트를 실행합니다.

위의 단계를 완료하여 스크립트가 정상적으로 실행되면 Azure 계정 구독에 리소스 그룹, 서버 및 데이터베이스가 생성되었는지 유효성을 검사할 수 있습니다.

bash 스크립트 출력에는 아래에 강조 표시된 텍스트와 비슷한 서버 이름이 포함되어 있어야 합니다.

```shell
{ "administratorLogin": "sqladmin",

  "administratorLoginPassword": null,

  "fullyQualifiedDomainName": "dp-050-servername.database.windows.net",

  "id": "/subscriptions/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/resourceGroups/dp-050-labresourcegroup/providers/Microsoft.Sql/servers/dp-050-servername",

  "identity": null,

  "kind": "v12.0",

  "location": "eastus",

  "name": "dp-050-servername",

  "resourceGroup": "dp-050-labresourcegroup",

  "state": "Ready",

  "tags": null,

  "type": "Microsoft.Sql/servers",

  "version": "12.0"

}
```

## 연습 2: Azure SQL Database로 데이터베이스 마이그레이션(오프라인)

이 연습에서는 Azure(이전 랩에서 만든 SQL 2017 VM)에서 실행 중인 SQL Server 인스턴스에 포함된 데이터베이스의 데이터베이스 스키마를 마이그레이션하고, 이전 연습에서 만든 SQL Database에 해당 스키마를 로드합니다.

구체적으로는 다음 작업을 수행합니다.

- Data Migration Assistant를 사용하여 새 마이그레이션 프로젝트 만들기
- 스키마 마이그레이션만 수행

**예상 소요 시간:** 20분

**참고:** SQL Server 2008 R2 가상 머신에서 실행되는 Data Migration Assistant를 사용하여 이 연습을 완료합니다.

### 태스크 1: Data Migration Assistant를 사용하여 마이그레이션 프로젝트 만들기

1. **Microsoft Data Migration Assistant**를 엽니다.
1. **+**를 클릭하여 새 프로젝트를 시작합니다.
1. 프로젝트 유형으로 **마이그레이션**을 선택합니다.
1. 프로젝트 이름으로 **SQLDBMigration**을 입력합니다.
1. 다음 설정의 유효성을 검사합니다.
1. 원본 서버 유형: **SQL Server**
1. 대상 서버 유형: **Azure SQL Database**
1. 마이그레이션 범위: **스키마만**
1. **만들기**를 선택합니다

### 태스크 2: 스키마 마이그레이션 수행

1. 원본 서버에 연결에 **LONDON**을 입력합니다.
1. **연결 암호화** 옵션 선택을 취소합니다.
1. **연결**을 선택합니다.
1. 데이터베이스 목록이 표시되면 **AdventureworksLT2008R2**를 선택합니다.
1. 유효성을 검사한 결과 이전 평가에서 마이그레이션 문제가 없음이 확인되었으므로, **마이그레이션하기 전에 데이터베이스를 평가하시겠습니까?** 확인란 선택을 취소합니다.
1. **다음**을 클릭합니다.
1. 대상 서버에 연결 대화 상자에 이전 연습에서 만든 서버의 **서버 이름**을 입력합니다. dp-050-servername.database.windows.net과 같은 **정규화된 이름**을 기준으로 서버 목록을 정렬합니다.
1. 인증 유형을 **SQL Server 인증**으로 변경합니다.
1. 사용자 이름으로 **sqladmin**을 입력합니다.
1. 암호를 다음과 같이 입력합니다. **Pa55w.rd.123456789**
1. **연결**을 클릭합니다.
1. SQL Database 서버의 대상 데이터베이스로 이전 연습에서 만든 대상 데이터베이스를 선택합니다.
1. **다음**을 클릭합니다.
1. 모든 스키마 개체를 검토한 결과 문제가 확인되지 않았으므로 SQL 스크립트 생성을 클릭합니다.

다음과 같은 Data Migration Assistant 창이 표시됩니다.
![Data Migration Assistant 창](/images/dbmigrate.png)

15. 스키마 배포를 클릭합니다.
1. Data Migration Assistant를 닫습니다.

**참고:** 이제 모든 데이터베이스 개체가 Azure SQL Database로 마이그레이션됩니다. 이 마이그레이션을 완료하려면 최대 10분이 걸릴 수 있습니다. 

### 태스크 3: 데이터 마이그레이션용 테이블 선택

이전 태스크에서 스키마 마이그레이션이 완료되면 다음 작업을 수행합니다. 

## 연습 3: Azure SQL Database로 온-프레미스 데이터베이스 마이그레이션(온라인)

이 연습에서는 Azure Data Migration Service를 사용하여 데이터베이스의 온라인 마이그레이션을 수행합니다.

구체적으로는 다음 작업을 수행합니다.

- 복제용으로 원본 인스턴스를 구성하고 마이그레이션을 위한 필수 구성 요소가 충족되었는지 확인
- Azure Data Migration Service를 사용하여 라이브 마이그레이션 수행

**예상 소요 시간:** 20분

### 태스크 1: SQL Server를 복제 배포자로 구성

SQL 2008 R2 가상 머신에 설치되어 있으며 **SQL Server 2017 인스턴스**에 연결된 SQL Management Studio를 사용하여 이 태스크를 완료합니다.

**참고:** 호스트되는 SQL 2008 R2 VM과 Azure 간에 VPN 액세스를 구성할 필요가 없도록 하려면 SQL Server 2017 인스턴스에 연결된 상태에서 이러한 태스크를 수행하세요. 랩 이외의 환경에서는 보통 VPN 또는 Expressroute를 구성합니다.

1. SQL Management Studio 개체 탐색기에서 랩 3에서 만든 SQL Server 2017 인스턴스에 연결합니다.
    SQL Server 2017 인스턴스가 등록되지 않은 경우 다음 단계를 수행합니다.
        **연결 | 데이터베이스 엔진**을 선택한 다음 **서버에 연결** 대화 상자에서 다음 정보를 입력합니다.

            서버 이름: **<SQL 2017 VM의 정규화된 도메인 이름>**

            예를 들면 sql2017vmxxxx.centralus.cloudapp.azure.com과 같이 입력할 수 있습니다. 

2. **복제**를 **마우스 오른쪽 단추로 클릭**합니다.
1. **배포 구성**을 선택합니다.
1. **배포 구성 마법사**를 실행합니다. 
1. 각 배포 구성 페이지에서 기본 설정을 적용합니다.
    1. 마법사에서 **SQL Server 에이전트를 자동으로 시작**할 것인지를 묻는 메시지가 표시되면 **예**를 클릭합니다.
1. **다음**을 클릭합니다.
1. **스냅숏** 폴더에서 **다음**을 클릭합니다.
1. **배포** 데이터베이스 이름에서 **다음**을 클릭합니다.
1. **게시자** 대화 상자 창에서 **다음**을 클릭합니다.
1. **마법사 작업** 대화 상자 창에서 **다음**을 클릭합니다.
1. **마침**을 클릭합니다.
1. **SQL Server 에이전트가 시작되지 않으면** SQL Management Studio에서 **SQL Server 에이전트 | 개체 탐색기 | SQL 서버 에이전트**를 마우스 오른쪽 단추로 클릭한 다음 서비스 시작을 클릭하여 에이전트를 수동으로 시작합니다.
1. 새 쿼리 창에서 다음 쿼리를 실행하여 **AdventureworksLT2008R2** 데이터베이스용 데이터베이스 복구 모델을 FULL로 설정합니다.

```sql 
ALTER DATABASE AdventureworksLT2008R2 SET RECOVERY FULL WITH NO_WAIT
```

14. 다음 쿼리를 실행하여 데이터베이스의 전체 백업을 수행합니다.  

```sql
BACKUP DATABASE AdventureworksLT2008R2 TO DISK = ‘d:\awlt2008r2backup.bak’
```

15. 백업이 완료되면 **SQL Management Studio**를 닫습니다.

### 태스크 2: Azure Database Migration Service를 사용하여 온라인 마이그레이션 수행

이 태스크에서는 SQL Database와 VM 내에서 실행 중인 데이터베이스 간에 라이브 마이그레이션을 수행할 수 있도록 Azure Database Migration Service를 구성합니다.

**참고:** 모든 태스크는 Azure Portal에서 완료합니다.

1. Azure Portal에서 **리소스 만들기**를 클릭합니다.
1. **Azure Database Migration Service**를 입력합니다.
1. **만들기**를 클릭합니다.
1. 서비스 이름에서 서비스 이름으로 **dp050dmsxxxxxx** 를 입력합니다. 여기서 xxxxxx는 고유한 값입니다.
1. 사용자의 **Azure 구독**을 선택합니다.
1. 이전에 만든 **리소스 그룹**을 선택합니다.
1. **위치**를 선택합니다.
1. **새 가상 네트워크**를 만들고 이름을 **OnPremGateway** 로 지정합니다.
1. 가격 책정 계층을 **프리미엄**으로 변경합니다.
1. **확인**을 클릭합니다.
1. **만들기**를 클릭합니다.

**참고:** 이 배포를 완료하려면 최대 10분이 걸릴 수 있습니다.

12.배포가 완료되면 리소스로 이동을 클릭합니다.

    필요한 경우 Azure Portal의 왼쪽 블레이드에 Azure Data Migration Service를 추가할 수 있습니다.

    a) Azure Portal의 모든 서비스에서 Azure Data Migration Service를 찾습니다. 
    b) 별표 아이콘을 선택하여 Azure Portal의 왼쪽 블레이드에 Azure Data Migration Service를 추가합니다.
    c) Azure Data Migration Service가 표시되도록 왼쪽 블레이드를 확장합니다.
    d) Data Migration Service를 선택합니다. 

13. 새 마이그레이션 프로젝트를 클릭합니다.
1. 새 마이그레이션 프로젝트 대화 상자에 다음 정보를 입력합니다.

    프로젝트 이름: **DP050OnlineMigration**

    원본 서버 유형: **SQL Server**

    대상 서버 유형: **Azure SQL Database**

    활동 유형 선택: **온라인 데이터 마이그레이션으로 변경**

15. 저장을 클릭합니다.
1. 활동 만들기 및 실행을 클릭합니다.
1. 마이그레이션 원본 세부 정보에서 다음 정보를 입력합니다.

    원본 SQL Server 인스턴스 이름: &lt;SQL 2017 VM의 정규화된 도메인 이름&gt;

    인증 유형: **SQL 인증**

    사용자 이름: **sqladmin**

    암호: **Pa55w.rd.123456789**

18. 연결 암호화 선택을 취소합니다.
1. 저장을 클릭합니다.
1. SQL 마이그레이션의 대상 선택하고 마이그레이션 대상 세부 정보에서 다음 세부 정보를 입력합니다.

    대상 서버 이름: &lt;Azure SQL Database Server의 정규화된 도메인&gt;

    인증 유형: **SQL 인증**

    사용자 이름: **sqladmin**

    암호: **Pa55w.rd.123456789**

21. **저장**을 클릭합니다.
1. **대상 데이터베이스에 매핑** 창에서 스키마를 마이그레이션한 데이터베이스를 선택하고 대상 데이터베이스도 선택합니다.
1. **저장**을 클릭합니다.
1. **마이그레이션 설정 구성** 창에서 드롭다운 목록을 클릭한 후 dbo 테이블에 대해 변경 데이터 캡처를 사용하도록 설정하지 않았으므로 테이블 하나가 복제되지 않음을 확인합니다.
1. **저장**을 클릭합니다.
1. **마이그레이션 프로젝트** 활동 이름으로 **dp050lab4activity**를 입력합니다.

**참고:** 프로덕션 환경에서는 데이터를 더 빠르게 로드할 수 있도록 SQL 데이터베이스의 데이터베이스 계층을 늘립니다.

27. **마이그레이션 실행**을 클릭합니다.
1. 마이그레이션 프로세스가 실행되는 동안 **활동** 창을 **새로 고칩니다**.
1. 상태가 **중단 준비 완료**로 표시될 때까지 상태를 검토합니다.

    **참고:** 데이터베이스가 변경된 경우에는 증분 데이터 로드를 검토할 수 있습니다.

    원본 데이터베이스로 설정한 SQL 인스턴스에서 다음 명령을 사용하여 원본 데이터베이스의 **Productcategory** 테이블에 레코드를 몇 개 삽입할 수도 있습니다.

```sql
USE [AdventureWorksLT2008R2]
GO

INSERT INTO [Sales LT].[ProductCategory]
 ([ParentProductCategoryID]
 ,[Name]
 ,[ModifiedDate])
VALUES
 (1, ' Myproduct', getdate())

```

    그런 다음 증분 데이터 동기화 탭을 검토하고 삽입한 레코드를 살펴봅니다.

30. **중단 시작**을 클릭합니다.
1. **확인**을 선택합니다.
1. **데이터베이스 유효성 검사**를 선택합니다.
1. 모든 유효성 검사 옵션을 선택합니다.
1. **적용**을 클릭합니다.

축하합니다! Azure SQL Database로의 온라인 마이그레이션을 정상적으로 완료했습니다.
