---
title: "클라우드 환경 구성 - 계정 생성, IAM/MFA설정, VPC, Bastion host/NAT instance"
categories :	
 - AWS
tags : 
 - AWS
 - 클라우드
---

# 클라우드 환경 구성

## 1. AWS Free Tier 생성

클라우드 서비스 사용을 위해서는 AWS 계정 생성이 필요하다. 이메일 주소, 암호, 계정 이름을 입력하고 다음 페이지에서 주소와 전화번호 등의 개인정보를 입력한다.

![image-20210117122847783](2021-01-17-aws_environment.assets/image-20210117122847783.png)

사용량만큼 요금을 부과하는 클라우드 특성상, 미리 카드 정보를 받아두어 혹여 Free Tier로 요금 발생 시 카드로 결제되게끔 설정해두었다. 

![image-20210117123104537](2021-01-17-aws_environment.assets/image-20210117123104537.png)

Free Tier로 이용할 계획이므로 기본 플랜 (무료)을 선택한다. 그러면 계정 생성은 모두 완료된다! 

![image-20210117123457847](2021-01-17-aws_environment.assets/image-20210117123457847.png)

처음 회원가입한 계정은 AWS root 계정으로 간주되어, 로그인 시 '루트 사용자'로 로그인하면 된다.

![image-20210117123706769](2021-01-17-aws_environment.assets/image-20210117123706769.png)



## 2. AWS IAM 계정 생성 및 MFA 설정

### IAM 계정 생성

1. 서비스 : 'IAM' 검색 후 사용자 -> 사용자 추가 탭으로 이동한다.

2. 사용자 세부 정보 설정

   - 사용자 이름 : admin
   - 액세스 유형 : AWS Management Console 액세스 (콘솔 대시보드에 액세스 허용)
   - 콘솔 비밀번호 : 사용자 지정 비밀번호 (원하는 비밀번호 입력)
   - 비밀번호 재설정 필요 : 활성화 (만든 계정으로 처음 로그인하면 비밀번호 재설정 필요)

   ![image-20210117124353343](2021-01-17-aws_environment.assets/image-20210117124353343.png)

3. 권한 설정 : 기본 정책 직접 연결 -> 'AdministratorAccess' 선택

4. 태그 : 태그 미설정 -> 사용자 생성 완료 (ID : 641489732843 ) -> IAM user 로 로그인 가능

![image-20210117155351917](2021-01-17-aws_environment.assets/image-20210117155351917.png)

### MFA 설정

1. 핸드폰으로 Google OTP 앱 다운받기
2. IAM 계정으로 로그인 -> [내 보안 자격 증명] 메뉴 선택 -> [MFA 디바이스 할당] 탭 선택

![image-20210117160513966](2021-01-17-aws_environment.assets/image-20210117160513966.png)

3. [가상 MFA 디바이스] 유형 선택 -> 위 화면에서 QR 코드 표시 선택한 후, Google OTP 앱으로 QR코드 스캔하여 MFA 코드 입력하기 -> MFA 할당 완료

![image-20210117231146919](2021-01-17-aws_environment.assets/image-20210117231146919.png)

## 3. VPC 구축

### NAT 인스턴스

> VPC 생성하면서 자동으로 생성된다.
>
> Private subnet이 인터넷과 통신하기 위한 아웃바운드 인스턴스
>
> Private subnet에서 요청하는 아웃바운드 트래픽을 받아서 Internet Gateway와 연결한다. 

1. [서울 region]  	EC2 서비스 -> [네트워크 및 보안] 의 키 페어 메뉴 선택  -> 키 페어 생성
   - 이름 : mission
   - 파일 형식 : ppk (PuTTY와 함께 사용)

![image-20210117161629656](2021-01-17-aws_environment.assets/image-20210117161629656.png)

### VPC 구축

> 논리적으로 격리된 공간을 프로비저닝
>
> 하나의 계정에서 생성하는 리소스들만의 격리된 네트워크 환경 구성 가능
>
> 논리적인 독립 네트워크를 구성하는 리소스

1. VPC 서비스 -> [VPC 마법사 시작] 선택 -> **퍼블릭 및 프라이빗 서브넷이 있는 VPC** 선택

![image-20210117161841273](2021-01-17-aws_environment.assets/image-20210117161841273.png)

2. [서울 region]     IPv4 CIDR 블록 : 10.0.0.0/16 , VPC 이름 : mission-vpc
   - <Subnet> 
     - 퍼블릭 서브넷의 IPv4 CIDR : 10.0.0.0/24  
     - 가용영역 : ap-northeast-2a
     - 퍼블릭 서브넷 이름: Public subnet A
     - 프라이빗 서브넷의 IPv4 CIDR : 10.0.2.0/24 
     - 가용영역 :  ap-northeast-2a
     - 프라이빗 서브넷 이름 : Private subnet A 
   - <NAT instance>
     - 인스턴스 유형 : t2.micro
     - 키 페어 이름 : mission (위에서 설정한 키 페어)
     - DNS 호스트 이름 편집 : DNS 호스트 이름 활성화 (VPC 내부의 인스턴스에 Public DNS 호스트 이름 할당)

![image-20210117162638872](2021-01-17-aws_environment.assets/image-20210117162638872.png)

### Subnet 생성

> VPC를 CIDR 블록을 가지는 단위로 나누어 더 많은 네트워크 망을 만들 수 있음
>
> 실제 리소스가 생성되는 물리적인 공간
>
> - Public subnet : 인터넷과 연결되어 있음
> - Private subnet : 인터넷과 연결되어 있지 않음

1. VPC 서비스 -> [가상 프라이빗 클라우드] 메뉴 -> [서브넷] -> 서브넷 생성 선택
2. Public subnet B 생성
   - 가용영역 : ap-northeast-2c
   - IPv4 CIDR 블록 : 10.0.1.0/24

![image-20210117163840077](2021-01-17-aws_environment.assets/image-20210117163840077.png)

3. Private subnet B 생성 (위 단계와 동일)

   - 가용영역 : ap-northeast-2c

   - IPv4 CIDR 블록 : 10.0.3.0/24

4. 생성된 subnet 확인 -> 생성한 총 4개의 subnet 확인 가능

![image-20210117163944008](2021-01-17-aws_environment.assets/image-20210117163944008.png)

### 라우팅 테이블

> 데이터 요청 -> 라우터 -> 라우팅 테이블에서 정의한 범위 내에서 목적지를 찾는다. 
>
> - External-rt : Public subnet의 라우팅 테이블
> - Internal-rt :  Private subnet의 라우팅 테이블
>   - 위 2개의 라우팅 테이블은 VPC를 생성하면 자동으로 생성된다.

1. VPC 서비스 -> [가상 프라이빗 클라우드] 메뉴 -> [라우팅 테이블] 

2. [Public subnet] 의 라우팅 테이블 

   - 이름 : External-rt
   - 서브넷 연결 : 생성한 Public subnet A, B 추가
   - 라우팅 편집 : 0.0.0.0/0 - internet gateway

3. [Private subnet] 의 라우팅 테이블

   - 이름 : Internal-rt
   - 서브넷 연결 : 생성한 Private subnet A, B 추가
   - 라우팅 편집 : 0.0.0.0/0 - NAT instance 

   ![image-20210117194018597](2021-01-17-aws_environment.assets/image-20210117194018597.png)

### Internet Gateway

> VPC 생성하면서 자동으로 생성된다.
>
> VPC는 격리된 네트워크 환경이므로 VPC 에서 생성된 리소스들은 인터넷 사용 불가
>
> Internet Gateway는 VPC와 인터넷을 연결해준다. 

1. External-rt 라우팅 테이블을 통해 인터넷 게이트웨이가 생성되었음을 확인한다.
2. 인터넷 게이트웨이의 이름을 mission-igw로 수정한다.

![image-20210117194503764](2021-01-17-aws_environment.assets/image-20210117194503764.png)



## 4. Bastion host

> 외부와 내부 네트워크 사이에서 게이트 역할을 하는 Host
>
> 외부에서 접근이 가능하도록 Public IP 부여

### 인스턴스 보안 그룹 설정

1. EC2 서비스 -> [인스턴스->인스턴스] 탭으로 이동 -> 해당 인스턴스의 [보안] 탭 -> 보안그룹 선택 -> 인바운드 규칙 편집 선택

![image-20210117201526234](2021-01-17-aws_environment.assets/image-20210117201526234.png)

2. SSH 접속을 위해 22번 포트를 열어 놓는다. (규칙 추가 -> 2번째 유형으로 추가)

![image-20210117201851680](2021-01-17-aws_environment.assets/image-20210117201851680.png)

### Elastic IP 부여

1. EIP를 인스턴스에 부여하여 외부에서 접속 가능하게 만든다.
2. VPC를 생성하며 NAT instance에 자동으로 EIP 부여된다.

### OS 계정 생성과 비밀번호 로그인 허용

> PuTTY - 퍼블릭 IPv4 주소를 Host Name에 입력하고 Connection->SSH->Auth의 Private key file for authentication에 private key file을 넣는다. (ec2-user로 로그인)

1. 계정 생성

```linux
# sudo -i
# useradd admin
# passwd admin 	// 비밀번호 입력
# cat /etc/passwd	// 생성된 계정 확인
```

2. sudo 권한 부여
   - 생성한 admin 계정에 sudo 권한 부여하고 비밀번호 입력 없이 명령어 수행 가능하게끔 설정

```linux
# vi /etc/sudoers.d/cloud-init 	
```

![image-20210117230301290](2021-01-17-aws_environment.assets/image-20210117230301290.png)

- 위 사진처럼 vi 편집기에서 마지막 줄을 추가 -> :wq! 입력하여 저장 후 vi 창을 빠져나온다.

3. 비밀번호 로그인 설정

```linux
# vi etc/ssh/sshd_config	// 루트로 계정 변경 (아래 사진처럼 수정 후 저장)
```

![image-20210117230902117](2021-01-17-aws_environment.assets/image-20210117230902117.png)

```
# service sshd restart	// sshd 데몬 재시작
```

