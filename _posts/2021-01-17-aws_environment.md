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

![104845950-621d2b80-591b-11eb-89fc-b1db8472faa2](https://user-images.githubusercontent.com/77096463/104947414-15128580-59ff-11eb-961a-5617d4977a46.png) <br/>
<br/>



사용량만큼 요금을 부과하는 클라우드 특성상, 미리 카드 정보를 받아두어 혹여 Free Tier로 요금 발생 시 카드로 결제되게끔 설정해두었다. 

![104845961-7103de00-591b-11eb-93b5-8312a2d34d9e](https://user-images.githubusercontent.com/77096463/104947828-ad106f00-59ff-11eb-883d-b0f9a27b585a.png) <br/>
<br/>



Free Tier로 이용할 계획이므로 기본 플랜 (무료)을 선택한다. 그러면 계정 생성은 모두 완료된다! 

![104845982-8f69d980-591b-11eb-9142-e69b713ff051](https://user-images.githubusercontent.com/77096463/104947871-bc8fb800-59ff-11eb-9b5f-7085c205cfa6.png) <br/>
<br/>



처음 회원가입한 계정은 AWS root 계정으로 간주되어, 로그인 시 '루트 사용자'로 로그인하면 된다.

![104845989-9bee3200-591b-11eb-97ac-4a6fcd510469](https://user-images.githubusercontent.com/77096463/104947905-c87b7a00-59ff-11eb-9e59-8851259af03c.png) <br/>
<br/>
<br/>









## 2. AWS IAM 계정 생성 및 MFA 설정

### IAM 개념

- AWS 리소스에 대한 액세스를 제어할 수 있는 AWS 서비스
- AWS 리소스 사용 권한 부여와 제어
- 보안상의 이유로 운영 환경에서 root 계정 (처음 회원가입한 계정) 사용 금지
- 1인 1계정 생성하여 장애 발생 시 책임 소재를 명확히 함
- IAM 생성할 때만 root 계정 사용 (생성한 계정에 AdministratorAccess 권한 부여)

### MFA 개념

- Mulfi-Factor Authentication : 사용자 이름과 암호 +OTP 인증 추가하여 로그인하는 서비스
- 보안상 이유로 운영환경에서 IAM 개별 계정에 MFA 설정할 것을 권고 




### IAM 계정 생성

1. 서비스 : 'IAM' 검색 후 사용자 -> 사용자 추가 탭으로 이동한다.
2. 사용자 세부 정보 설정
   - 사용자 이름 : admin
   - 액세스 유형 : AWS Management Console 액세스 (콘솔 대시보드에 액세스 허용)
   - 콘솔 비밀번호 : 사용자 지정 비밀번호 (원하는 비밀번호 입력)
   - 비밀번호 재설정 필요 : 활성화 (만든 계정으로 처음 로그인하면 비밀번호 재설정 필요)

   ![104846004-b0cac580-591b-11eb-9cd8-eb9167bb4c97](https://user-images.githubusercontent.com/77096463/104947976-dcbf7700-59ff-11eb-8823-97633a49d572.png)
<br/>
3. 권한 설정 : 기본 정책 직접 연결 -> 'AdministratorAccess' 선택
4. 태그 : 태그 미설정 -> 사용자 생성 완료 (ID : 641489732843 ) -> IAM user 로 로그인 가능

![104846017-be804b00-591b-11eb-8c11-ea2f9dfcfd18](https://user-images.githubusercontent.com/77096463/104948029-f2cd3780-59ff-11eb-9860-f426e11ae2de.png) <br/>
<br/>
<br/>



### MFA 설정

1. 핸드폰으로 Google OTP 앱 다운받기
2. IAM 계정으로 로그인 -> [내 보안 자격 증명] 메뉴 선택 -> [MFA 디바이스 할당] 탭 선택

![104846031-cd66fd80-591b-11eb-809b-0d9254ccb706](https://user-images.githubusercontent.com/77096463/104948045-fbbe0900-59ff-11eb-8123-3e55b9e2dc82.png)

3. [가상 MFA 디바이스] 유형 선택 -> 위 화면에서 QR 코드 표시 선택한 후, Google OTP 앱으로 QR코드 스캔하여 MFA 코드 입력하기 -> MFA 할당 완료
![104846126-5bdb7f00-591c-11eb-944d-444d16384498](https://user-images.githubusercontent.com/77096463/104948269-61aa9080-5a00-11eb-970e-59470a3b848a.png)
<br/>
<br/>






## 3. VPC 구축

### VPC 개념

- 가상 네트워크 환경으로 논리적으로 격리된 공간을 프로비저닝한다.
- 논리적인 독립 네트워크를 구성하는 리소스
- 하나의 계정에서 생성하는 리소스들만의 격리된 네트워크 환경 구성 가능
- CIDR 범위는 사설 ip 대역으로 설정 (사설망 대역 : 10.0.0./8, 172.16.0.0/12, 192.168.0.0/16)
	- CIDR 블록 : IP 범위를 지정하는 방식 (VPC 내의 자원들은 VPC의 CIDR 범위 안에서 ip를 할당받는다)
	- route table의 기본 규칙이 VPC CIDR 블록에서 찾기 때문에 사설망이 아닌 CIDR를 사용하면 인터넷과 연결되는 라우트 규칙을 정의하더라도 통신 불가
	
	  
### VPC 구성 요소

- VPC
- subnet
- route table
- internet gateway

### Subnet 개념

- VPC를 CIDR 블록을 가지는 단위로 나누어 더 많은 네트워크 망을 만들 수 있음
- 실제 리소스가 생성되는 물리적인 공간
- VPC CIDR 블록 범위 안에서 지정 가능
- 인터넷과 연결되어 있으면 public subnet, 그렇지 않다면 private subnet

### Internet Gateway 개념

- VPC는 격리된 네트워크 환경이기 때문에 VPC에서 생성된 리소스들은 인터넷에 연결되어 있지 않음
- VPC와 인터넷을 연결해주는 관문
- VPC 생성 시 자동 생성됨

### Route Table 인스턴스 개념

- 데이터를 요청하면 -> 라우터가 라우터 테이블에서 정의한 범위 내에서 목적지를 찾아준다.

### NAT Gateway 개념

- private subnet이 인터넷과 통신하기 위한 아웃바운드 인스턴스
- private subnet은 외부에서 요청하는 인바운드는 차단하더라도 아웃바운드 트래픽을 허용할 필요 있음
- private subnet에서 외부로 요청하는 아웃바운드 트래픽을 받아 internet gateway와 연결

### Bastion Host 개념

- 내부와 외부 네트워크 사이에서 게이트 역할을 하는 host
- 외부에서 접근이 가능하도록 Public IP 부여

![aws_network](https://user-images.githubusercontent.com/77096463/109381939-a63f1b00-7920-11eb-926d-4d57c351d0ed.png)

### NAT 인스턴스

1. [서울 region]  	EC2 서비스 -> [네트워크 및 보안] 의 키 페어 메뉴 선택  -> 키 페어 생성
   - 이름 : mission
   - 파일 형식 : ppk (PuTTY와 함께 사용)

![104846051-e7084500-591b-11eb-9388-891c6a341bfd](https://user-images.githubusercontent.com/77096463/104948069-0a0c2500-5a00-11eb-9a12-cdfad490e601.png)<br/>
<br/>
<br/>



### VPC 구축

> 논리적으로 격리된 공간을 프로비저닝
> 하나의 계정에서 생성하는 리소스들만의 격리된 네트워크 환경 구성 가능
> 논리적인 독립 네트워크를 구성하는 리소스

1. VPC 서비스 -> [VPC 마법사 시작] 선택 -> **퍼블릭 및 프라이빗 서브넷이 있는 VPC** 선택
![104846063-f4253400-591b-11eb-9259-ab7fe2e7966d](https://user-images.githubusercontent.com/77096463/104948083-11333300-5a00-11eb-8aef-6325960d9b44.png)

<br/>
2. [서울 region]     IPv4 CIDR 블록 : 10.0.0.0/16 , VPC 이름 : mission-vpc
   - **Subnet** 
     - 퍼블릭 서브넷의 IPv4 CIDR : 10.0.0.0/24  
     - 가용영역 : ap-northeast-2a
     - 퍼블릭 서브넷 이름: Public subnet A
     - 프라이빗 서브넷의 IPv4 CIDR : 10.0.2.0/24 
     - 가용영역 :  ap-northeast-2a
     - 프라이빗 서브넷 이름 : Private subnet A 
   - **NAT instance**
     - 인스턴스 유형 : t2.micro
     - 키 페어 이름 : mission (위에서 설정한 키 페어)
     - DNS 호스트 이름 편집 : DNS 호스트 이름 활성화 (VPC 내부의 인스턴스에 Public DNS 호스트 이름 할당)

![104846072-ff785f80-591b-11eb-9cc1-dcf7c5773abd](https://user-images.githubusercontent.com/77096463/104948106-1bedc800-5a00-11eb-94f1-92459bf0eb9c.png) <br/> 
<br/>
<br/>


### Subnet 생성

> VPC를 CIDR 블록을 가지는 단위로 나누어 더 많은 네트워크 망을 만들 수 있음  
> 실제 리소스가 생성되는 물리적인 공간
> - Public subnet : 인터넷과 연결되어 있음
> - Private subnet : 인터넷과 연결되어 있지 않음

1. VPC 서비스 -> [가상 프라이빗 클라우드] 메뉴 -> [서브넷] -> 서브넷 생성 선택
2. Public subnet B 생성
   - 가용영역 : ap-northeast-2c
   - IPv4 CIDR 블록 : 10.0.1.0/24
![104846081-0b642180-591c-11eb-8dda-d54ce2da2b69](https://user-images.githubusercontent.com/77096463/104948118-23ad6c80-5a00-11eb-9717-d8d1382d2214.png)<br/>

3.  Private subnet B 생성 (위 단계와 동일)
   - 가용영역 : ap-northeast-2c
   - IPv4 CIDR 블록 : 10.0.3.0/24
4. 생성된 subnet 확인 -> 생성한 총 4개의 subnet 확인 가능

![104846083-11f29900-591c-11eb-9076-f1812b4ee633](https://user-images.githubusercontent.com/77096463/104948136-2ad47a80-5a00-11eb-9022-8dafaa23666d.png) 
<br/>
<br/>


### 라우팅 테이블

> 데이터 요청 -> 라우터 -> 라우팅 테이블에서 정의한 범위 내에서 목적지를 찾는다. 
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

   ![104846087-1b7c0100-591c-11eb-839d-b7461b31642b](https://user-images.githubusercontent.com/77096463/104948150-3162f200-5a00-11eb-9ff5-fd0114eb6674.png)  <br/>
<br/>
<br/>



### Internet Gateway

> VPC 생성하면서 자동으로 생성된다.
> VPC는 격리된 네트워크 환경이므로 VPC 에서 생성된 리소스들은 인터넷 사용 불가
> Internet Gateway는 VPC와 인터넷을 연결해준다. 

1. External-rt 라우팅 테이블을 통해 인터넷 게이트웨이가 생성되었음을 확인한다.
2. 인터넷 게이트웨이의 이름을 mission-igw로 수정한다.  

![104846094-26369600-591c-11eb-8cd5-a32de6f8fc53](https://user-images.githubusercontent.com/77096463/104948166-36c03c80-5a00-11eb-8488-6d86e3462fb3.png)  <br/>   
<br/>
<br/>




## 4. Bastion host

> 외부와 내부 네트워크 사이에서 게이트 역할을 하는 Host
> 외부에서 접근이 가능하도록 Public IP 부여

### 인스턴스 보안 그룹 설정

1. EC2 서비스 -> [인스턴스->인스턴스] 탭으로 이동 -> 해당 인스턴스의 [보안] 탭 -> 보안그룹 선택 -> 인바운드 규칙 편집 선택
![104846100-30f12b00-591c-11eb-85f2-56df7962178e](https://user-images.githubusercontent.com/77096463/104948182-3f187780-5a00-11eb-84c7-507555d6af77.png)<br/>

2. SSH 접속을 위해 22번 포트를 열어 놓는다. (규칙 추가 -> 2번째 유형으로 추가)
![104846105-377fa280-591c-11eb-84ca-bd7dc4e3709f](https://user-images.githubusercontent.com/77096463/104948203-450e5880-5a00-11eb-9e95-d460bfcf9f6a.png)  <br/>
<br/>



### Elastic IP 부여

1. EIP를 인스턴스에 부여하여 외부에서 접속 가능하게 만든다.
2. VPC를 생성하며 NAT instance에 자동으로 EIP 부여된다.  
<br/>
<br/>



### OS 계정 생성과 비밀번호 로그인 허용

> PuTTY - 퍼블릭 IPv4 주소를 Host Name에 입력하고 Connection->SSH->Auth의 Private key file for authentication에 private key file을 넣는다. (ec2-user로 로그인)

1. 계정 생성
```
# sudo -i
# useradd admin
# passwd admin 	// 비밀번호 입력
# cat /etc/passwd	// 생성된 계정 확인
```
<br/>

2. sudo 권한 부여
   - 생성한 admin 계정에 sudo 권한 부여하고 비밀번호 입력 없이 명령어 수행 가능하게끔 설정
```
# vi /etc/sudoers.d/cloud-init 	
```
![104846112-4403fb00-591c-11eb-8c56-a568a71f13de](https://user-images.githubusercontent.com/77096463/104948222-4e97c080-5a00-11eb-82b0-a2f44af2395f.png)

- 위 사진처럼 vi 편집기에서 마지막 줄을 추가 -> :wq! 입력하여 저장 후 vi 창을 빠져나온다.
<br/>

3. 비밀번호 로그인 설정
```
# vi etc/ssh/sshd_config	// 루트로 계정 변경 (아래 사진처럼 수정 후 저장)
```
![104846114-4cf4cc80-591c-11eb-8ccc-8267fd105fc8](https://user-images.githubusercontent.com/77096463/104948233-548da180-5a00-11eb-988f-52c8348d9082.png)
```
# service sshd restart	// sshd 데몬 재시작
```



<br/>
<br/>