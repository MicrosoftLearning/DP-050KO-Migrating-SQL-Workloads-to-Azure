# DP 050 - Azure로 SQL 워크로드 마이그레이션

## 랩 3 - Azure Virtual Machine의 SQL Server로 SQL 워크로드 마이그레이션

**예상 소요 시간:** 60분

**전제 조건:** 이 랩에서는 수행해야 할 필수 단계가 없습니다.

**랩 파일:** 이 랩용 랩 파일은 없습니다.

## 랩 개요

이 랩에서는 먼저 온-프레미스 SQL Server 2008 R2 인스턴스를 가상 머신에서 실행되는 SQL Server 2017로 마이그레이션하는 데 사용할 마이그레이션 프로세스를 평가합니다. 그런 다음 Data Migration Assistant를 사용하여 마이그레이션을 수행해 데이터베이스를 이동합니다. 그리고 마지막으로 마이그레이션이 정상적으로 완료되었는지를 평가합니다.

## 랩 목표

이 랩을 완료하면 다음과 같은 작업을 수행할 수 있습니다.

- Azure에서 SQL Server 2017을 실행하는 새 가상 머신 만들기
- Azure 스토리지 계정 및 파일 공유 만들기
- SQL Server 2008 R2 데이터베이스를 Azure VM의 SQL Server로 마이그레이션하는 작업 수행

## 시나리오

여러분은 데이터 현대화 프로젝트 실행을 준비 중인 AdventureWorks의 선임 데이터베이스 관리 책임자입니다. 데이터베이스 집합을 Azure Virtual Machine의 SQL Server로 마이그레이션하는 데 필요한 환경을 준비하고, Data Migration Assistant를 사용하여 테스트 마이그레이션을 수행할 예정입니다.

## 연습 1: Azure에서 SQL Server 2017을 실행하는 새 가상 머신 만들기

이 연습에서는 Azure Portal을 사용하여 Azure에서 새 가상 머신을 만듭니다.

**예상 소요 시간:** 20분

이 연습에서 수행하는 작업은 다음과 같습니다.

1. Azure Portal에서 새 가상 머신 만들기

### SQL Server 2017 가상 머신 프로비전

> [!참고]
> 호스트형 랩 환경에서 이 랩을 실행 중이라면 해당 환경 내에서 다음 단계를 수행하세요.

1. [Azure Portal](https://portal.azure.com)에서 **리소스 만들기**를 선택합니다.
1. Marketplace 검색 상자에 **Windows Server 2019의 SQL Server 2017**을 입력하고 Enter 키를 누릅니다. **모든 결과 표시** 아래에서 **Windows Server 2019의 SQL Server 2017**을 선택합니다.
1. **플랜 선택** 드롭다운 목록에서 **무료 SQL Server 라이선스: Windows Server 2019의 SQL Server 2017 Developer** 를 선택하고 **만들기**를 선택합니다.
1. **가상 머신 만들기** 마법사의 **기본 사항** 페이지에서 다음 값을 입력한 후에 **다음: 디스크 \>** 를 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 구독 | 보유한 구독 선택 |
    | 리소스 그룹 | 새 리소스 그룹 **DP-050-Training** 만들기 |
    | 가상 머신 이름 | sql2017vm |
    | 지역 | 가까운 지역 선택 |
    | 가용성 옵션 | 인프라 중복이 필요하지 않습니다. |
    | 이미지 | 무료 SQL Server 라이선스: Windows Server 2019의 SQL Server 2017 Developer - Gen1 |
    | Azure Spot 인스턴스 | 아니요 |
    | 크기 | Standard_D2_v2 |
    | 사용자 이름 | sqladmin |
    | 암호 | Pa55w.rdPa55w.rd |
    | 암호 확인 | Pa55w.rdPa55w.rd |
    | 공용 인바운드 포트 | 선택한 포트 허용 |
    | 인바운드 포트 선택 | RDP(3389) |
    | 기존 Windows 라이선스를 사용하시겠습니까? | 아니요 |
    
1. **디스크** 페이지에서 기본 설정을 그대로 적용하고 **다음: 네트워킹 \>** 을 선택합니다.
1. **네트워킹** 페이지에서 기본 설정을 그대로 적용하고 **다음: 관리 \>** 를 선택합니다.
1. **관리** 페이지의 **부팅 진단** 목록에서 **사용 안 함**을 선택하고 **다음: 고급 \>** 을 선택합니다.
1. **고급** 페이지에서 기본 설정을 적용하고 **다음: SQL Server 설정 \>** 을 선택합니다.
1. **SQL Server 설정** 페이지에서 다음 값을 입력한 후에 **검토 + 만들기** 를 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | SQL 연결 | 공용(인터넷) |
    | 포트 | 1433 |
    | SQL 인증 | 사용 |
    | 로그인 이름 | sqladmin |
    | 암호 | Pa55w.rdPa55w.rd |
    | Azure Key Vault 통합 | 사용 안 함 |

1. **검토 + 만들기** 페이지에서 **만들기**를 선택합니다.

    > [!참고]
    > 이 단계를 완료하려면 약 10분 정도 걸릴 수 있습니다.

1. 배포가 완료되면 **리소스로 이동**을 선택합니다.
1. VM의 **공용 IP 주소**를 찾아서 적어 둡니다. 나중에 이 주소를 사용해야 합니다.
1. **DNS 이름** 옆의 **구성**을 선택합니다.
1. **DNS 이름 레이블(옵션)** 텍스트 상자에 고유한 DNS 이름을 입력하고 해당 이름을 적어 둡니다.

    sql2017vmxxxx.centralus.cloudapp.azure.com과 같이 입력할 수 있습니다.

1. **저장**을 선택합니다.

결과: 이 연습에서는 Azure Virtual Machine에서 실행되는 SQL Server 2017 인스턴스를 만들었습니다.

## 연습 2: Azure 스토리지 계정 및 파일 공유 만들기

이 연습에서는 Azure Portal을 사용하여 Azure에서 새 스토리지 계정을 만듭니다.

**예상 소요 시간:** 15분

이 연습의 주요 작업은 다음과 같습니다.

1. Azure 스토리지 계정 만들기
1. Azure 스토리지 계정에 파일 공유 만들기

### Azure Storage 계정 만들기

1. [Azure Portal](https://portal.azure.com)에서 **리소스 만들기**를 선택합니다.
1. **Marketplace 검색** 텍스트 상자에 **스토리지 계정**을 입력하고 **Enter** 키를 누릅니다.
1. **모든 결과 표시** 아래에서 **스토리지 계정**을 선택하고 **만들기**를 선택합니다.
1. **스토리지 계정 만들기** 마법사의 **기본 사항** 페이지에서 다음 값을 입력합니다.

    | 속성 | 값 |
    | --- | --- |
    | 구독 | 보유한 구독 선택 |
    | 리소스 그룹 | DP-050-Training |
    | 스토리지 계정 이름 | **dp050storagexxxx**. 여기서 xxxx는 임의의 문자 시퀀스입니다. |
    | 위치 | 가상 머신에 사용한 것과 같은 위치 선택 |
    | 성능 | 표준 |
    | 계정 종류 | StorageV2(범용 v2) |
    | 복제 | LRS(로컬 중복 스토리지) |

    > [!참고]
    > 사용할 스토리지 계정 이름을 정확하게 적어 두세요. 이 랩 뒷부분에서 해당 이름이 필요합니다.

1. **검토 + 만들기** 를 선택합니다.
1. **검토 + 만들기** 페이지에서 **만들기**를 선택합니다.

    > [!참고]
    > 이 배포는 몇 분 정도 걸릴 수 있습니다.

1. 배포가 완료되면 **리소스로 이동**을 선택합니다.
1. **설정** 아래에서 **액세스 키**를 선택합니다.
1. **액세스 키** 페이지에서 **키 표시**를 선택하고 **key1** 아래 **키** 텍스트 상자의 내용을 적어 둡니다.

### 파일 공유 만들기

1. 스토리지 계정 페이지 왼쪽 메뉴의 **파일 서비스** 아래에서 **파일 공유**를 선택합니다.
1. **파일 공유** 페이지에서 **+ 파일 공유** 를 선택합니다.
1. **새 파일 공유** 페이지에서 다음 값을 입력합니다.

    | 속성 | 값 |
    | --- | --- |
    | 이름 | backupshare |
    | 할당량 | 200GiB |

1. **만들기**를 선택합니다.

결과: SQL Server 데이터베이스 백업 파일용 공유 액세스 위치로 사용할 Azure 파일 공유를 만들었습니다. 다음 연습에서는 공유 위치에 액세스하도록 SQL 인스턴스를 구성합니다.

## 연습 3: Azure 파일 공유 연결을 위해 SQL Server 인스턴스용 연결 만들기

이 연습에서는 온-프레미스 서버와 새 Azure VM에서 모두 Azure 파일 공유에 액세스하도록 SQL Server 환경을 구성합니다.

**예상 소요 시간:** 10분

이 연습의 주요 작업은 다음과 같습니다.

1. 네트워크 드라이브를 매핑하여 SQL Management Studio를 통해 파일 공유 등록
1. 파일 공유에 연결

### SQL Management Studio에서 서버 인스턴스 등록

1. 수업용으로 실행 중인 **LON-DEV-01** 가상 머신에 로그인합니다. 사용자 이름은 **administrator**이고 암호는 **Pa55w.rd**입니다.
1. **SQL Management Studio**를 시작하고 로컬 인스턴스(LONDON)에 연결합니다.
1. SQL Management Studio 개체 탐색기에서 **연결**, **데이터베이스 엔진**을 차례로 선택합니다.
1. **서버에 연결** 대화 상자에서 다음 값을 입력한 후에 **연결**을 선택합니다.

    | 속성 | 값 |
    | --- | --- |
    | 서버 이름 | Azure 내 SQL 2017 VM의 정규화된 도메인 이름 또는 IP 주소 입력 예제: **sql2017vmxxx.centralus.cloudapp.azure.com** |
    | 인증 | SQL Server |
    | 로그인 | sqladmin |
    | 암호 | Pa55w.rdPa55w.rd |

### 파일 공유에 온-프레미스 SQL 인스턴스 연결

> [!참고]
> SQL Server가 파일 공유의 드라이브 문자에 연결할 수 있도록 하려면 SQL Server Management Studio에서 `xp_cmdshell`을 실행하여 네트워크 드라이브를 매핑해야 합니다. 그래야 SQL 서비스 계정이 공유에 액세스할 수 있습니다. Data Migration Assistant는 SQL 서비스 계정을 사용하여 데이터베이스를 백업합니다. 보안상 SQL Server 서비스 계정만 명령줄에 액세스하도록 제한해야 합니다. 기본적으로 SQL 명령줄은 사용할 수 없습니다.

1. 온-프레미스 SQL Server에서 연결을 구성하려면 SQL Management Studio의 **개체 탐색기**에서 **LONDON** 서버를 마우스 오른쪽 단추로 클릭하고 **새 쿼리**를 선택합니다.
1. 다음 Transact-SQL 코드를 입력합니다.

    ```sql
    EXECUTE sp_configure 'show advanced options', 1;
    RECONFIGURE;
    EXECUTE sp_configure 'xp_cmdshell', 1;
    RECONFIGURE;
    GO
    EXECUTE xp_cmdshell 'net use U: \\<storageaccountname>.file.core.windows.net\backupshare /persistent:Yes /u:Azure\<storageaccountname> <storageaccountkey>';
    EXECUTE xp_cmdshell 'dir U:';
    ```

1. 쿼리 텍스트에서 `<storageaccountname>`은 앞에서 만든 스토리지 계정 이름으로 바꿉니다. 두 곳에 이름을 입력해야 합니다.
1. `<storageaccountkey>`는 앞에서 적어 둔 스토리지 계정용 기본 액세스 키로 바꿉니다.
1. 쿼리를 실행하고 결과에 오류 메시지가 없음을 확인합니다.

    > [!참고]
    > 이 SQL 코드는 네트워크 드라이브 U:를 Azure의 스토리지 계정에 매핑합니다. 그러나 이 매핑은 SQL 서비스 계정 컨텍스트에서 수행되므로 `net use` 명령을 사용할 때 파일 탐색기에는 U: 드라이브가 표시되지 않습니다.

1. Labfiles 폴더에 쿼리를 **MapNetworkDrive.sql**로 저장합니다.
1. 새 쿼리 창을 시작하고 다음 쿼리를 실행하여 LONDON SQL Server에서 `xp_cmdshell`을 사용하지 않도록 설정합니다.

    ```sql
    EXECUTE sp_configure 'xp_cmdshell', 0;
    RECONFIGURE;
    ```

1. 파일을 저장하지 않고 쿼리 창을 모두 닫습니다.

### 파일 공유에 Azure VM SQL 인스턴스 연결

1. Azure VM SQL Server에서 연결을 구성하려면 **개체 탐색기**에서 Azure의 SQL Server를 마우스 오른쪽 단추로 클릭하고 **연결**을 선택합니다.
1. **서버에 연결** 대화 상자의 **암호** 텍스트 상자에 **Pa55w.rdPa55w.rd**를 입력한 후에 **연결**을 선택합니다.
1. **파일** 메뉴에서 **파일 열기**를 선택한 다음 앞에서 저장한 **MapNetworkDrive.sql** 파일을 엽니다.
1. 쿼리 창 아래쪽의 상태 표시줄에서 Azure VM에 연결되어 있는지 확인합니다.
1. Azure VM에서 U: 드라이브를 매핑하려면 쿼리를 실행하고 결과에 오류 메시지가 없음을 확인합니다.
1. 새 쿼리 창을 시작하고 다음 쿼리를 실행하여 `xp_cmdshell`을 사용하지 않도록 설정합니다.

    ```sql
    EXECUTE sp_configure 'xp_cmdshell', 0;
    RECONFIGURE;
    ```

1. SQL Server Management Studio를 닫습니다.

## 연습 4: SQL Server Data Migration Assistant를 사용하여 데이터베이스 마이그레이션 수행

이 연습에서는 온-프레미스 SQL Server에서 Azure의 VM으로 데이터를 마이그레이션합니다.

**예상 소요 시간:** 10분

이 연습의 주요 작업은 다음과 같습니다.

1. Data Migration Assistant를 사용하여 데이터베이스 마이그레이션
1. 데이터베이스가 정상적으로 마이그레이션되었는지 유효성 검사

### Data Migration Assistant를 사용하여 SQL 데이터베이스 마이그레이션

1. **LON-DEV-01** 가상 머신에서 **Microsoft Data Migration Assistant**를 열고 **+**를 선택합니다.
1. **새로 만들기** 페이지에서 다음 값을 입력합니다.

    | 속성 | 값 |
    | --- | --- |
    | 프로젝트 형식 | 마이그레이션 |
    | 프로젝트 이름 | Azure VM으로 마이그레이션 |
    | 원본 서버 유형 | SQL Server |
    | 대상 서버 유형 | Azure Virtual Machines의 SQL Server |

1. **만들기**를 선택합니다.
1. **원본 및 대상 지정** 페이지의 **원본 서버 정보** 아래에 다음 값을 입력합니다.

    | 속성 | 값 |
    | --- | --- |
    | 서버 이름 | localhost를 입력합니다. |
    | 인증 유형 | Windows 인증 |
    | 연결 암호화 | 아니요 |
    | 서버 인증서 신뢰 | 예 |

1. **대상 서버 정보** 아래에 다음 값을 입력합니다.

    | 속성 | 값 |
    | --- | --- |
    | 서버 이름 | Azure 내 가상 머신의 IP 주소 또는 DNS 이름 입력 |
    | 인증 유형 | SQL Server 인증 |
    | 사용자 이름 | sqladmin |
    | 암호 | Pa55w.rdPa55w.rd |
    | 연결 암호화 | 예 |
    | 서버 인증서 신뢰 | 예 |

1. **다음**을 선택합니다.
1. **데이터베이스 추가** 페이지에서 **AdventureWorks** 및 **AdventureWorksLT2008TR2**를 제외한 모든 데이터베이스의 선택을 취소합니다.
1. **공유 위치** 텍스트 상자에 **U:\\** 를 입력하고 **다음**을 선택합니다.
1. **로그인 선택** 창을 검토하여 마이그레이션할 로그인이 없음을 확인합니다. **마이그레이션 시작**을 선택합니다.

    > [!참고]
    > 모든 데이터베이스가 Azure Storage 파일 공유의 공유 네트워크 드라이브에 백업됩니다.

1. 마이그레이션 프로세스를 모니터링합니다.
1. 마이그레이션이 완료되면 Data Migration Assistant를 닫습니다.

### 마이그레이션이 정상 완료되었는지 유효성 검사

1. **LON-DEV-01** 가상 머신에서 **SQL Management Studio**를 엽니다.
1. **서버에 연결** 대화 상자의 **서버 이름** 목록에서 Azure VM의 IP 주소 또는 DNS 이름을 선택합니다.
1. **암호** 텍스트 상자에 **Pa55w.rdPa55w.rd**를 입력하고 **연결**을 선택합니다.
1. 개체 탐색기에서 **데이터베이스** 목록을 확장합니다.
1. **AdventureWorks** 데이터베이스가 정상적으로 마이그레이션되었는지 확인합니다.
1. **파일** 메뉴에서 **새로 만들기/현재 연결로 쿼리**를 선택합니다.
1. 아래 쿼리를 입력한 다음 실행하여 각 데이터베이스의 데이터베이스 호환성 수준 유효성을 검사합니다.

    ```sql
    SELECT name, compatibility_level FROM sys.databases
    ```

1. 아래 쿼리를 사용하여 **AdventureWorks** 데이터베이스의 데이터베이스 호환성 수준을 변경합니다.

    ```sql
    ALTER DATABASE AdventureWorks
    SET COMPATIBILITY_LEVEL = 110;
    GO
    ```

1. 아래 쿼리를 사용하여 AdventureWorks 데이터베이스를 백업합니다.

    ```sql
    BACKUP DATABASE Adventureworks  
    TO DISK = 'U:\Adventureworks'  
       WITH FORMAT,  
          MEDIANAME = 'Adventureworks',  
          NAME = 'Full Backup of Adventureworks';  
    ```

1. 백업이 정상적으로 완료되면 **SQL Management Studio**를 닫습니다.

> [!중요]
> 이 랩을 마친 후 Azure에서 실행 중인 SQL Server VM을 삭제하지 마세요. 랩 4에서 Azure Database로 마이그레이션할 때 해당 VM을 원본으로 사용합니다.

결과: Azure VM에서 실행되는 SQL Server 2017로 SQL Server 데이터베이스를 마이그레이션하는 과정을 완료했습니다.