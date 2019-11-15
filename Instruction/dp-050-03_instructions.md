---
lab:
    title: '랩 3 - Azure Virtual Machine의 SQL Server로 SQL 워크로드 마이그레이션'
    module: '모듈 3: Azure Virtual Machines로 SQL 워크로드 마이그레이션'
---

# DP-050 - Azure로 SQL 워크로드 마이그레이션

## 랩 3 - Azure Virtual Machine의 SQL Server로 SQL 워크로드 마이그레이션

**예상 소요 시간:** 60분
**필수 구성 요소:** 이 랩에서는 수행해야 할 필수 단계가 없습니다.
**랩 파일:** 이 랩용 랩 파일은 없습니다.

## 랩 개요

이 랩에서는 먼저 온-프레미스 SQL Server 2008 R2 인스턴스를 가상 머신에서 실행되는 SQL Server 2017로 마이그레이션하는 데 사용할 마이그레이션 프로세스를 평가합니다. 그런 다음 Data Migration Assistant를 사용하여 마이그레이션을 수행해 Data Migration Assistant를 사용하는 데이터베이스를 이동합니다.  그리고 마지막으로 마이그레이션이 정상적으로 완료되었는지를 평가합니다.

## 랩 목표

이 랩을 마치면 다음 작업을 수행하는 방법을 배울 수 있습니다.

1. Azure에서 새 SQL Server 2017 가상 머신 만들기
2. Azure 스토리지 계정 및 파일 공유 만들기
3. SQL Server 2008 R2 데이터베이스를 Azure VM의 SQL Server로 마이그레이션하는 작업 수행

## 시나리오

여러분은 데이터 현대화 프로젝트 실행을 준비 중인 AdventureWorks의 선임 데이터 관리 책임자입니다. Azure Virtual Machine의 SQL Server로 데이터베이스 집합을 마이그레이션하는 데 필요한 환경을 준비하고 Data Migration Assistant를 사용하여 테스트 마이그레이션을 수행하려고 합니다.

## 연습 1: Azure에서 새 SQL Server 2017 가상 머신 만들기

이 연습에서는 Azure Portal을 사용하여 Azure에서 새 가상 머신을 만듭니다.

**예상 소요 시간:** 20분

이 연습의 주요 태스크는 다음과 같습니다.

1. Azure Portal에서 새 가상 머신 만들기

### 태스크 1: SQL Server 2017 가상 머신 프로비전

**참고:** 호스트형 랩 환경에서 이 랩을 실행하는 경우 프로비전된 랩 환경 내에서 이러한 단계를 수행하세요.

1. [Azure Portal](https://portal.azure.com)에서 **리소스 만들기** 를 클릭한 다음 마켓플레이스 검색 상자에 **Windows 2016의 SQL Server 2017** 을 입력합니다.
1. **소프트웨어 플랜 선택** 드롭다운에서 **무료 SQL Server 라이선스: Windows Server 2016의 SQL Server 2017 Developer** 를 선택합니다.
1. 미리 설정된 구성으로 시작을 선택합니다.
1. **워크로드 환경 선택** 에서 **개발/테스트** 를 선택합니다.
1. **워크로드 유형 선택** 에서 기본값인 **범용(D-시리즈)** 를 그대로 적용합니다.
1. **VM 계속 만들기** 를 선택합니다.
1. 프로젝트 세부 정보 창에서 **새로 만들기** 를 선택하여 새 리소스 그룹을 만듭니다.
1. 리소스 그룹 이름으로 **DP-050-Training** 을 지정합니다.
1. 다음 세부 정보를 입력하여 인스턴스 세부 정보를 제공합니다.

    1. 가상 머신 이름: **sql2017vm**
    1. 지역: **실제 위치에 가장 가까운 지역 선택**
    1. 이미지: **무료 SQL Server 라이선스: Windows Server 2016의 SQL Server 2017 Developer**
    1. 사용자 이름: **sqladmin**
    1. 암호: **Pa55w.rd.123456789**
    1. 암호 확인: **Pa55w.rd.123456789**

    **방화벽 설정:**

      a. 공용 인바운드 포트: 선택한 포트 허용 선택

      b. 인바운드 포트 선택 드롭다운에서 RDP 선택

10. **다음: 디스크를 선택합니다.**

    디스크 설정을 검토하되 변경하지는 말고 그대로 적용합니다.

1. **다음: 네트워킹을 선택합니다.**
1. **다음: 관리를 선택합니다.**

    관리 설정을 검토하고 부트 진단을 다음과 같이 변경합니다.

    a. 부트 진단을 **해제**로 설정합니다.

13. **다음: 고급을 선택합니다.**
1. 고급 탭을 건너뛰고 **SQL Server 설정**을 선택합니다.

    SQL Server 설정 탭에서 다음 정보를 입력합니다.

    a. SQL 연결: **공용(인터넷)** 선택
    b. SQL 인증: **사용 선택**
    c. 암호 상자에 **Pa55w.rd.123456789** 입력

15. **검토 + 만들기**를 선택합니다.

    검토 + 만들기 탭의 설정을 검토합니다.

    a. **만들기**를 선택하여 VM(가상 머신) 만들기를 시작합니다.

**참고:** 이 단계를 완료하려면 10분 정도 걸릴 수 있습니다.

16.	VM 생성이 완료되면 가상 머신 블레이드를 엽니다.
17.	 생성된 **sql2017vm**을 선택합니다.
18.	공용 IP 주소를 찾은 다음 나중에 연결할 때 사용할 수 있도록 적어 둡니다.
19.	**DNS 이름 구성**을 선택합니다.
20.	고유하게 식별할 수 있는 DNS 이름을 입력한 다음 지정한 전체 DNS 이름을 적어 둡니다.

    예를 들어 다음과 같은 이름을 입력할 수 있습니다. SQL2017VMxxxx.centralus.cloudapp.azure.com

21.	저장을 클릭합니다.

``` shell
결과: Azure Virtual Machine에서 실행되는 SQL Server 2017 인스턴스를 생성하여 이 연습을 완료해야 합니다.
```

## 연습 2: Azure 스토리지 계정 및 파일 공유 만들기

이 연습에서는 Azure Portal을 사용하여 Azure에서 새 가상 머신을 만듭니다.

**예상 소요 시간:** 15분

이 연습의 주요 태스크는 다음과 같습니다.

1. Azure 스토리지 계정 만들기
2. Azure 스토리지 계정에 파일 공유 만들기

### 태스크 1: Azure 스토리지 계정 만들기

1. [Azure Portal](https://portal.azure.com)에서 **스토리지 계정** 블레이드를 선택합니다.
2. **추가**를 선택합니다.
3. 스토리지 계정 만들기 창에서 다음 세부 정보를 입력합니다.
    a. 리소스 그룹: **기존**을 선택합니다.
    b. 이전 연습에서 만든 **DP-050-Training** 리소스 그룹을 선택합니다.
    c. 스토리지 계정 이름: **dp050storagexxxx**를 입력합니다. 여기서 xxxx는 임의의 문자 수입니다.
    d. 위치: 이전 연습에서 가상 머신을 만든 위치에 가장 가까운 위치를 선택합니다.
    e. 다른 기본 설정은 그대로 유지합니다.
4. **검토 + 만들기**를 클릭하여 고급 및 태그 섹션을 건너뜁니다.
5. **만들기**를 선택합니다

    **참고:**  이 배포를 완료하려면 몇 분 정도 걸릴 수 있습니다.

6. 배포가 완료되면 **리소스로 이동**을 클릭합니다.

### 태스크 2: Azure 파일 공유 만들기

1. 스토리지 계정 페이지에서 **파일**을 선택합니다.
2. 파일 페이지에서 **+ 파일 공유**를 선택합니다.
3. 다음 정보를 입력합니다.
    a. 이름: **backupshare**
    b. 할당량: **200Gib**
4. **만들기**를 선택합니다
5. 파일 공유를 만든 후 생성된 파일 공유 오른쪽에서 ...를 선택합니다.
6. 드롭다운 목록에서 **연결**을 선택합니다.

![드롭다운 목록](https://github.com/MicrosoftLearning/DP-050-Migrating-SQL-Workloads-to-Azure/blob/master/images/dropdown.png)

7. 연결 블레이드에서 드라이브 문자 **U:**를 선택합니다.
8. 아래에 나와 있는 텍스트에서 연결 명령 구문을 복사합니다. 키가 슬래시로 시작하지 않는 경우에는 이 명령을 실행할 수도 있습니다.

텍스트는 아래 명령 구문과 같습니다.

```shell
cmdkey /add:dp050storagexxxx.file.core.windows.net /user:Azure\dp050storagexxxx /pass:Hcty8OXn7jcON/ePk3OvswD7eRusfWWAw9lakhX9P9m4MnuqZEt2I8pDYAEiyAZJx4Na39UZEwAyH8PlnVa36Q==

```

9. **메모장**을 엽니다.
10. 위 명령의 내용을 붙여넣은 다음 **Labfiles** 폴더에 **MapNetworkdrive.txt**로 파일을 저장합니다.
11. **메모장**을 닫습니다.
12.	이제 Azure Portal 메모장을 닫아도 됩니다.

```shell
결과: SQL Server 데이터베이스 백업 파일용 공유 액세스 위치로 사용할 Azure 파일 공유를 만들었습니다. 다음 연습에서는 공유 위치에 액세스할 수 있도록 SQL 인스턴스를 구성합니다.
```

## 연습 3: Azure 파일 공유에 연결하기 위해 SQL Server 인스턴스로의 연결 생성

이 연습에서는 Azure 파일 공유에 액세스할 수 있도록 SQL Server 환경을 구성합니다.
**예상 소요 시간:** 10분

이 연습의 주요 태스크는 다음과 같습니다.

1. 네트워크 드라이브를 매핑하여 SQL Management Studio를 통해 파일 공유 등록
2. Azure 스토리지 계정에 파일 공유 만들기

### 태스크 1: SQL Management Studio에서 서버 인스턴스 등록

1. SQL Server 2008 R2 랩 환경에서 SQL Management Studio를 시작합니다.
2. 로컬 인스턴스(LONDON)에 연결합니다.
3. SQL Management Studio 개체 탐색기에서 **연결**을 선택합니다. **| 데이터베이스 엔진**을 선택한 다음 **서버에 연결** 대화 상자에서 다음 정보를 입력합니다.
    서버 이름: 'SQL 2017 VM의 정규화된 도메인 이름'
    예를 들면 sql2017vmxxxx.centralus.cloudapp.azure.com과 같이 입력할 수 있습니다.
4. SQL Server 인증을 선택하고 다음 사용자 및 암호를 입력합니다.
    로그인: **sqladmin**
    암호: **Pa55w.rd.123456789**

### 태스크 2: 파일 공유에 SQL 인스턴스 연결

**참고:** 파일 공유에 있는 드라이브 문자에 SQL Server가 연결할 수 있도록 하려면 xp_cmdshell을 실행하여 네트워크 드라이브를 매핑해야 합니다. 보안상 SQL Server 서비스 계정만 명령줄에 액세스하도록 제한해야 합니다. 기본적으로 SQL 명령줄은 사용할 수 없습니다.

1. 연습의 이전 부분에서 만든 MapNetworkdrive.txt를 메모장에서 엽니다.
2.	텍스트를 복사합니다.
3.	SQL Management Studio에서 로컬 서버(LONDON)에 연결된 상태로 새 쿼리를 만듭니다.
4.	MapNetworkdrive의 텍스트를 쿼리 창에 붙여넣습니다.
5.	다음 스크린샷을 반영하여 쿼리를 변경한 다음 xp_cmhdshell 저장 프로시저를 통해 명령줄을 실행합니다.
 ![sp_configure 옵션](https://github.com/MicrosoftLearning/DP-050-Migrating-SQL-Workloads-to-Azure/blob/master/images/cmdshell.png)
6.	쿼리를 실행하고 네트워크 드라이브에 액세스할 수 있는지 유효성을 검사합니다.
7.	Labfiles 폴더에 쿼리를 **MapNetworkdrive.sql**로 저장합니다.
8.	새 쿼리 창을 시작한 후 다음 쿼리를 실행하여 xp_cmdshell을 사용하지 않도록 설정합니다.

```sql
    SP_CONFIGURE ‘xp_cmdshell’,1
```

9.	개체 탐색기에서 Azure VM SQL Server 인스턴스를 선택하고 새 쿼리를 만듭니다.
10.	저장한 Mapnetworkdrive.sql 파일을 열고 SQL Server 2017 인스턴스에서 스크립트를 실행합니다.
11.	SQL Server 2017 인스턴스에서 xp_cmdshell을 사용하지 않도록 설정하는 단계를 반복합니다.


## 연습 4: SQL Server 데이터를 사용하여 데이터베이스 마이그레이션 수행 

이 연습에서는 Azure Portal을 사용하여 Azure에서 새 가상 머신을 만듭니다.
**예상 소요 시간:** 10분

이 연습의 주요 태스크는 다음과 같습니다.

1. Data Migration Assistant를 사용하여 데이터베이스 마이그레이션
2. 데이터베이스가 정상적으로 마이그레이션되었는지 유효성 검사

### 태스크 1: Data Migration Assistant를 사용하여 SQL 데이터베이스 마이그레이션

1.	SQL Server 2008 R2 랩 환경에서 **"Microsoft Data Migration Assistant"** 애플리케이션을 엽니다.
2.	**+**를 선택하면 새 프로젝트 대화 상자가 열립니다. 이 대화 상자에 다음 정보를 입력합니다.
    a. 프로젝트 유형: **마이그레이션**
    b.	프로젝트 이름: **SQL VM으로 마이그레이션**
    c.	원본 서버 유형: **SQL Server**
    d.	대상 서버 유형: **Azure Virtual Machines의 SQL 서버**
3.	**만들기**를 클릭합니다.
4.	**다음**을 클릭합니다.
5.	원본 서버 세부 정보 서버 이름 대화 상자에 **localhost의 서버 이름**을 입력합니다.
6.	대상 서버 세부 정보 서버 이름 대화 상자에 Azure SQL Virtual Machine의 서버 이름(정규화된 이름)을 입력합니다.
7.	대상 서버 인증 유형 대화 상자에서 **SQL Server 인증**을 선택합니다.
8.	대상 서버 세부 정보 SQL 인증 자격 증명에 **sqladmin**을 입력하고 암호로는 **Pa55.w.rd.123456789**를 입력합니다.
9.	연결 속성에서 **연결 암호화** 속성의 선택을 취소합니다.
10.	**다음**을 클릭합니다.
11.	다음 데이터베이스의 선택을 취소합니다.
    •	AdventureworksDW2008_4M
    •	Reportserver
    •	ReportServerTempDB
12.	공유 위치 대화 상자에 **U:\*를 입력합니다.*
13.	로그인 선택 창의 내용을 검토합니다.
14.	**마이그레이션 시작**을 클릭합니다.

**참고:** 모든 데이터베이스는 공유 네트워크 드라이브(Azure 파일 공유)에 백업됩니다.

15.	마이그레이션 프로세스를 모니터링합니다.
16.	마이그레이션이 완료되면 Data Migration Assistant를 닫습니다.

### 태스크 2: 마이그레이션이 정상적으로 완료되었는지 유효성 검사

1.	SQL Server 2008 R2 랩 환경에서 **“SQL Management Studio”**를 엽니다.
2.	개체 탐색기에서 **SQL Server 2017 인스턴스**의 **데이터베이스** 목록을 확장합니다.
3.	데이터베이스가 정상적으로 마이그레이션되었는지 유효성을 검사합니다.
4.	**새 쿼리**를 만듭니다.
5.	다음 쿼리를 입력하고 실행하여 각 데이터베이스의 데이터베이스 호환성 수준을 확인합니다.

```sql
SELECT name, compatibility_level FROM sys.databases
```

6.	쿼리 결과를 검토합니다.
7.	다음 쿼리를 사용하여 **AdventureworksLT2008R2** 데이터베이스의 데이터베이스 호환성 수준을 변경합니다.

```sql
ALTER DATABASE AdventureWorksLT2008R2
SET COMPATIBILITY_LEVEL = 110;
GO
```

8.	다음 쿼리를 사용하여 AdventureworksLT2008R2 데이터베이스를 백업합니다.
  
```sql
BACKUP DATABASE AdventureworksLT2008R2  
TO DISK = 'U:\AdventureworksLT2008R2'  
   WITH FORMAT,  
      MEDIANAME = 'AdventureworksLT2008R2',  
      NAME = 'Full Backup of AdventureworksLT2008R2;  
```


9.	백업이 정상적으로 완료되면 **SQL Management Studio**를 닫습니다.

```shell
결과: Azure VM에서 실행되는 SQL Server 2017로 SQL 2008 R2 데이터베이스를 마이그레이션하는 작업을 정상적으로 완료했습니다.
```
