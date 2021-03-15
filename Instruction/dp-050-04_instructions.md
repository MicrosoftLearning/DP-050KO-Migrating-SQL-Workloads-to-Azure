# DP 050 - Azure로 SQL 워크로드 마이그레이션

## 랩 4 - Azure SQL Database로 SQL 워크로드 마이그레이션

**예상 소요 시간:** 60분

**전제 조건:** 이 랩에서는 수행해야 할 필수 단계가 없습니다.

**랩 파일:** 이 랩용 랩 파일은 없습니다.

## 랩 개요

이 랩에서는 Azure SQL Database로의 마이그레이션을 수행합니다. 먼저 스키마 마이그레이션을 수행하고 랩 필수 구성 요소로 대상 데이터베이스를 만듭니다. 그런 다음 온-프레미스 SQL 인스턴스에서 실행되는 데이터베이스를 Azure SQL Database로 마이그레이션합니다. 그 후에는 새 데이터베이스로의 단독형 마이그레이션을 수행할 때까지 원본 데이터베이스와 대상 데이터베이스 간에 데이터가 동기화 상태로 유지되도록 DMS(Database Migration Service)를 사용하여 온라인 마이그레이션을 수행합니다.

## 랩 목표

이 랩을 마치면 다음 작업을 수행할 수 있습니다.

- Azure Cloud Shell을 사용하여 Azure SQL Database에서 데이터베이스 만들기
- Azure Data Migration Service 구성
- Azure SQL Database로 데이터베이스 스키마 마이그레이션
- Data Migration Service를 사용하여 온라인 마이그레이션 수행

## 시나리오

여러분은 데이터 현대화 프로젝트 실행을 준비 중인 AdventureWorks의 선임 데이터베이스 관리 책임자입니다. 데이터베이스 집합을 Azure VM(Virtual Machine)의 SQL Server로 마이그레이션하는 데 필요한 환경을 준비하고, Data Migration Assistant를 사용하여 테스트 마이그레이션을 수행할 예정입니다.

## 연습 1: Azure Cloud Shell을 사용하여 Azure SQL Database에서 데이터베이스 만들기

이 연습에서는 Azure Cloud Shell을 사용하여 다음 작업을 수행합니다.

- 새 리소스 그룹 만들기
- 새 Azure SQL Database 서버 인스턴스 만들기
- Azure SQL Database 서버 방화벽 구성
- 새 범용 데이터베이스 만들기

**예상 소요 시간:** 20분

Azure에는 서비스 설치, 관리 및 배포를 자동화할 수 있는 여러 가지 옵션이 포함되어 있습니다. 이러한 옵션 중 하나는 간단한 플랫폼 간 명령줄 도구인 Azure CLI 명령줄 도구를 설치하는 것입니다. Azure Cloud Shell을 사용하여 해당 작업을 자동화하고 스크립트로 생성하는 옵션도 있습니다.

Azure Cloud Shell은 Azure에서 호스트되며 브라우저를 통해 관리할 수 있는 대화형 셸 환경입니다. Cloud Shell에서는 Bash 또는 PowerShell을 사용하여 Azure 서비스 관련 작업을 수행할 수 있습니다. Cloud Shell의 미리 설치된 명령을 사용하는 경우 로컬 환경에 별도의 도구를 설치하지 않고도 이 문서에 나와 있는 코드를 실행할 수 있습니다.

### 작업 1: Azure Cloud Shell 로그온 및 구성

> [!참고]
> 이 작업은 Azure Cloud Shell만 사용하여 완료할 수 있습니다.

1. [Azure Shell](https://shell.azure.com)로 이동합니다.
1. 이 교육에 사용하는 자격 증명으로 Azure 구독에 로그온합니다.
1. 스크립팅 환경으로 **Bash**를 선택합니다.

### 작업 2: Azure Cloud Shell용 새 스토리지 계정 및 공유 만들기

1. Azure Cloud Shell용으로 작성된 스토리지 계정이 없다는 메시지가 표시되면 **고급 설정 표시**를 선택하고 아래 값을 입력한 후에 **스토리지 만들기**를 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 구독 | 이 랩에 사용하는 Azure 구독 선택 |
    | Cloud Shell 지역 | 현재 위치에 가까운 사용 가능한 Cloud Shell 지역 선택 |
    | 리소스 그룹 | 새 리소스 그룹 **dp050lab4rg** 만들기 |
    | 스토리지 계정 | 새 스토리지 계정 **dp050sa&lt;youridentifier&gt;** 만들기. 여기서 **&lt;youridentifier&gt;** 는 고유한 문자열입니다. |
    | 파일 공유 | 새 파일 공유 **dp050share&lt;youridentifier&gt;** 만들기. 여기서 **&lt;youridentifier&gt;** 는 고유한 이름입니다. |

1. 스토리지 계정을 만들면 Azure Cloud Shell이 열립니다.

### 작업 3: Azure SQL Database 서버 인스턴스 및 데이터베이스 만들기

1. **{}** 아이콘을 선택하여 코드 편집기를 엽니다.
1. 코드 편집기에 다음 스크립트를 붙여넣습니다.

    ```bash
    #bash script to create a new sql db server instance and sql db  
    
    #Disclaimer: this script is a sample script on how to create an Azure database but uses least restrictive firewall settings for lab purposes. Do not use this script  
    
    #defining a name for the resource group
    resourcegroup=dp-050-labresourcegroup
    
    #edit the variable below to provide a unique server name
    servername=dp-050-servername
    
    #edit the variable below to provide the location – azure locations can be listed by typing az account list-locations -o table in the shell command interface
    location=eastus
    
    adminuser=sqladmin
    password=Pa55w.rdPa55w.rd
    firewallrule=dp-050-access
    
    #edit the script to provide a unique database name
    labdatabase=dp-050-Adventureworksxxx
    
    #creates a resource group
    az group create --name $resourcegroup --location $location
    
    #creates a sql db servername
    az sql server create --name $servername --resource-group $resourcegroup --admin-user $adminuser --admin-password $password --location $location
    
    #shows current firewall list
    az sql server firewall-rule list --resource-group $resourcegroup --server $servername
    
    #creates a server-based firewall – note – you should restrict the start/end ip range based on your environment
    az sql server firewall-rule create --resource-group $resourcegroup --server $servername --name $firewallrule --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
    
    #creates a general-purpose SQL database
    az sql db create --name $labdatabase --resource-group $resourcegroup --server $servername -e GeneralPurpose
    ```

1. 스크립트에서 `dp-050-servername` 값을 `dp-050-serverxxx` 등의 고유한 서버 이름으로 변경합니다. 이 이름에는 대문자를 사용하지 마세요.
1. 스크립트에서 `eastus` 값을 가까운 Azure 위치로 변경합니다.
1. 스크립트에서 `dp-050-Adventureworksxxx` 값을 고유한 데이터베이스 이름으로 변경합니다.
1. 스크립트를 저장하려면 <kbd>Ctrl+S</kbd>를 누르고 **CreateSQLDBandServer.sh**를 입력한 후에 **저장**을 선택합니다.
1. 코드 편집기를 끝내려면 <kbd>Ctrl+Q</kbd>를 누릅니다.
1. 스크립트를 실행하려면 **sh CreateSQLDBandServer.sh**를 입력하고 <kbd>Enter</kbd> 키를 누릅니다.

    스크립트 실행이 정상적으로 완료되면 Azure 계정 구독에 리소스 그룹, 서버 및 데이터베이스가 작성되었는지 유효성을 검사할 수 있습니다.

## 연습 2: Azure SQL Database로 데이터베이스 마이그레이션

이 연습에서는 온-프레미스에서 실행되는 SQL Server 인스턴스에 포함된 데이터베이스의 데이터베이스 스키마를 마이그레이션한 다음 이전 연습에서 만든 SQL Database에 해당 스키마를 로드합니다. 데이터베이스 스키마의 원본 SQL Server 인스턴스는 LON-DEV-01 VM에서 실행됩니다.

이 랩을 마치면 다음 작업을 수행할 수 있습니다.

- Data Migration Assistant를 사용하여 새 마이그레이션 프로젝트 만들기
- 데이터베이스 스키마 마이그레이션

**예상 소요 시간:** 20분

> [!참고]
> LON-DEV-01 VM에서 실행되는 Data Migration Assistant를 사용하여 이 연습을 완료합니다.

### 작업 1: Data Migration Assistant를 사용하여 마이그레이션 프로젝트 만들기

1. 수업용으로 실행 중인 **LON-DEV-01** 가상 머신에 로그인합니다. 사용자 이름은 **administrator**이고 암호는 **Pa55w.rd**입니다.
1. 새 데이터 마이그레이션 프로젝트를 만들려면 **Microsoft Data Migration Assistant**를 열고 **+**를 클릭합니다.
1. **새로 만들기** 페이지에서 다음 값을 입력한 후에 **만들기**를 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 프로젝트 형식 | 마이그레이션 |
    | 프로젝트 이름 | SQLSchemaMigration |
    | 원본 서버 유형 | SQL Server |
    | 대상 서버 유형 | Azure SQL Database |
    | 마이그레이션 범위 | 스키마만 |

### 작업 2: 스키마 마이그레이션 수행

1. **원본 선택** 페이지의 **원본 서버에 연결** 아래에 다음 값을 입력한 후에 **연결**을 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 서버 이름 | localhost를 입력합니다. |
    | 인증 유형 | Windows 인증 |
    | 연결 암호화 | 아니요 |
    | 서버 인증서 신뢰 | 예 |

1. 데이터베이스 목록에서 **AdventureworksLT2008R2**를 선택하고 **마이그레이션하기 전에 데이터베이스 평가** 체크박스 선택을 취소합니다.
1. **다음**을 선택합니다.
1. **대상 선택** 페이지의 **대상 서버에 연결** 아래에 다음 값을 입력한 후에 **연결**을 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 서버 이름 | 이전 연습에서 만든 서버의 **서버 이름** 입력. 서버가 dp-050-servername.database.windows.net과 같은 **정규화된 이름**을 기준으로 나열되도록 입력해야 합니다. |
    | 인증 유형 | SQL Server 인증 |
    | 사용자 이름 | sqladmin |
    | 암호 | Pas55w.rdPa55w.rd |
    | 연결 암호화 | 아니요 |
    | 서버 인증서 신뢰 | 예 |

1. 데이터베이스 목록에서 이전 연습에서 만든 대상 데이터베이스를 선택하고 **다음**을 선택합니다.
1. 모든 스키마 개체를 검토하고 **SQL 스크립트 생성**을 선택합니다.

    다음과 같은 Data Migration Assistant 창이 표시됩니다.

    ![Data Migration Assistant 창](/images/dbmigrate.png)

1. **스키마 배포**를 선택합니다.
1. 배포가 완료되면 Data Migration Assistant를 닫습니다.

## 연습 3: 온-프레미스 데이터베이스를 Azure SQL Database로 마이그레이션

이 연습에서는 Azure Data Migration Service를 사용하여 데이터베이스 온라인 마이그레이션을 수행합니다.

이 연습을 마치면 다음 작업을 수행할 수 있습니다.

- 복제용으로 원본 인스턴스를 구성하고 마이그레이션 필수 구성 요소 충족 여부 확인
- Azure Data Migration Service를 사용하여 실시간 마이그레이션 수행

**예상 소요 시간:** 20분

### 작업 1: SQL Server를 복제 배포자로 구성

이 작업은 **SQL Server 2017 인스턴스**에 연결한 상태로 SQL 2008 R2 가상 머신의 SQL Management Studio를 사용하여 완료합니다.

> [!참고]
> 호스트되는 SQL 2008 R2 VM과 Azure 간의 VPN 액세스를 구성하지 않아도 되도록 Azure의 SQL Server 2017 인스턴스에 연결한 상태로 LON-DEV-01에서 아래 작업을 수행하세요. 랩 이외의 환경에는 대개 VPN이나 ExpressRoute를 구성합니다.

1. SQL Management Studio 개체 탐색기에서 랩 3에서 만든 SQL Server 2017 인스턴스에 연결합니다.

    SQL Server 2017 인스턴스가 등록되지 않았으면 **연결/데이터베이스 엔진** 을 선택하고 다음 값을 사용합니다.

    | 속성 | 값 |
    | --- | --- |
    | 서버 이름 | SQL 2017 VM의 IP 주소 또는 정규화된 도메인 이름(예: sql2017vmxxxx.centralus.cloudapp.azure.com) |
    | 인증 | SQL Server 인증 |
    | 로그인 | sqladmin |
    | 암호 | Pa55w.rdPa55w.rd |

1. **개체 탐색기**에서 **복제**를 마우스 오른쪽 단추로 클릭하고 **배포 구성**을 선택합니다.
1. **배포 구성 마법사**에서 각 배포 구성 페이지의 기본 설정을 적용합니다. 마법사에서 **SQL Server 에이전트를 자동으로 시작하시겠습니까?** 메시지가 표시되면 **예**를 선택합니다.
1. **마법사 완료** 페이지에서 **마침**을 클릭합니다.
1. **SQL Server 에이전트가 시작되지 않으면** **SQL Management Studio에서 SQL Server 에이전트**를 마우스 오른쪽 단추로 클릭하여 에이전트를 수동으로 시작합니다. **| 개체 탐색기 | SQL Server 에이전트**를 마우스 오른쪽 단추로 클릭한 다음 서비스 시작을 클릭하여 에이전트를 수동으로 시작합니다.
1. **AdventureworksLT2008R2** 데이터베이스의 데이터베이스 복구 모델을 FULL로 설정하려면 새 쿼리 창에서 다음 쿼리를 실행합니다.

    ```sql
    ALTER DATABASE AdventureworksLT2008R2 SET RECOVERY FULL WITH NO_WAIT
    ```

1. 다음 쿼리를 실행하여 데이터베이스의 전체 백업을 수행합니다.  

    ```sql
    BACKUP DATABASE AdventureworksLT2008R2 TO DISK = 'd:\awlt2008r2backup.bak'
    ```

1. 백업이 완료되면 **SQL Management Studio**를 닫습니다.

### 작업 2: Azure Database Migration Service를 사용하여 온라인 마이그레이션 수행

이 작업에서는 Azure SQL Database와 VM에서 실행 중인 데이터베이스 간의 실시간 마이그레이션이 가능하도록 Azure Database Migration Service를 구성합니다.

> [!참고]
> 모든 작업은 Azure Portal에서 완료합니다.

1. Azure Portal에서 **리소스 만들기**를 선택합니다.
1. **Marketplace 검색** 텍스트 상자에 **Azure Database Migration Service**를 입력하고 <kbd>Enter</kbd> 키를 누릅니다.
1. **만들기**를 선택합니다.
1. **마이그레이션 서비스 만들기** 마법사의 **기본 사항** 페이지에서 다음 값을 입력한 후에 **다음: 네트워킹 \>\>** 을 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 구독 | 구독 선택. |
    | 리소스 그룹 | dp050lab4rg |
    | 마이그레이션 서비스 이름 | **dp050dmsxxx**. 여기서 **xxx**는 고유한 값입니다. |
    | 위치 | 가까운 위치를 선택합니다. |
    | 서비스 모드 | Azure |
    | 가격 책정 계층 | 프리미엄 |

1. **네트워킹** 페이지의 **가상 네트워크 이름** 텍스트 상자에 **OnPremGateway**를 입력하고 **검토 + 만들기** 를 선택합니다.
1. **검토 + 만들기** 페이지에서 **만들기**를 선택합니다.

    > [!참고]
    > 이 배포는 최대 10분이 걸릴 수 있습니다.

1. 배포가 완료되면 **리소스로 이동**을 선택합니다.
1. **+ 새 마이그레이션 프로젝트**를 선택합니다.
1. **새 마이그레이션 프로젝트** 페이지에서 다음 값을 입력한 후에 **활동 만들기 및 실행**을 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 프로젝트 이름 | DP050OnlineMigration |
    | 원본 서버 유형 | SQL Server |
    | 대상 서버 유형 | Azure SQL Database |
    | 활동 유형 선택 | 온라인 데이터 마이그레이션 |

1. **SQL Server에서 Azure SQL Database로의 온라인 마이그레이션 마법사**의 **원본 선택** 페이지에서 다음 값을 입력한 후에 **다음: 대상 선택 \>\>** 을 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 원본 SQL Server 인스턴스 이름 | Azure 내 SQL 7 VM의 정규화된 도메인 이름 |
    | 인증 유형 | SQL 인증 |
    | 사용자 이름 | sqladmin |
    | 암호 | Pa55w.rdPa55w.rd |
    | 연결 암호화 | 선택 취소 |
    | 서버 인증서 신뢰 | 선택 |

1. **대상 선택** 페이지에서 다음 값을 입력한 후에 **대상 데이터베이스에 매핑 \>\>** 를 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 대상 서버 이름 | Azure SQL Database 서버의 정규화된 도메인 |
    | 인증 유형 | SQL 인증 |
    | 사용자 이름 | sqladmin |
    | 암호 | Pa55w.rdPa55w.rd |
    | 연결 암호화 | 선택 취소 |
    | 서버 인증서 신뢰 | 선택 |

1. **대상 데이터베이스에 매핑** 페이지에서 스키마 마이그레이션을 수행했던 데이터베이스를 선택하고 대상 데이터베이스를 선택한 다음 **마이그레이션 설정 구성 \>\>** 을 선택합니다.
1. **마이그레이션 설정 구성** 페이지에서 각 표의 참고 사항을 확인하고 **요약 \>\>** 을 선택합니다.
1. **활동 이름** 텍스트 상자에 **dp050lab4activity**를 입력하고 **마이그레이션 실행**을 선택합니다.

    > [!참고]
    프로덕션 환경에서는 데이터를 더 빠르게 로드하기 위해 SQL 데이터베이스의 데이터베이스 계층을 늘려야 합니다.

1. 마이그레이션 프로세스가 진행되는 동안 **새로 고침**을 가끔씩 클릭하여 **상태** 열을 모니터링합니다. **중단 준비 완료**가 표시되면 마이그레이션이 완료된 것입니다.

    > [!참고]
    > 마이그레이션 중에 원본 데이터베이스의 데이터에 적용하는 변경 내용은 대상 데이터베이스에 동기화됩니다.

    시간이 있으면 다음 SQL 명령을 사용하여 원본 데이터베이스의 **ProductCategory** 테이블에 레코드를 몇 개 삽입해 볼 수 있습니다. 이 명령은 원본 VM에서 실행해야 합니다.

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

1. 데이터 마이그레이션이 완료되면 데이터베이스를 선택하고 **중단 시작**을 선택합니다.
1. **확인**, **적용**을 차례로 선택합니다.

결과: Azure SQL Database로의 온라인 마이그레이션을 정상적으로 완료했습니다.