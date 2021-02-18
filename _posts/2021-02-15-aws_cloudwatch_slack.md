---
title: "AWS CloudWatch, Slack 알람 발생"
categories :	
 - AWS
tags : 
 - AWS
 - 클라우드
 - CloudWatch
 - Slack
 - SNS
 - Lambda
---

# 1. CloudWatch -> Slack 알람 전송 체계

> CloudWatch를 통해 실시간으로 AWS 자원을 모니터링<br/>
> CloudWatch에서 설정한 알람이 '경보' 상태로 변경되면 연결된 Slack 채널로 알람 전송 (24*365 모니터링)

![image](https://user-images.githubusercontent.com/77096463/107950968-e0342700-6fda-11eb-9325-1ba84533d906.png)
<br/>

**CloudWatch에서 알람 발생 -> SNS 푸시 서비스 호출 -> Lambda 함수 트리거 -> 연동된 Slack 으로 알람 전송**

1. CloudWatch : AWS 리소스 및 실행중인 애플리케이션 모니터링
2. SNS : Publisher<-> Subscriber 간 통신 채널. 지원되는 프로토콜을 이용해 클라이언트에 메시지 전송
3. Lambda : 서버리스 컴퓨팅 서비스. 서버를 관리하지 않아도 코드를 실행할 수 있게 하는 컴퓨팅 서비스
4. KMS : 데이터 암호화에서 사용하는 암호화 키
5. Slack : 채널 기반 메시징 플랫폼. Channel 고유의 WebHook url을 통해 알람 수신 

<br/>
<br/>
<br/>

# 2. Slack 설정

### 로그인 & 워크스페이스 생성

1. Slack 홈페이지 접속 -> 로그인 -> 워크 스페이스 생성 -> 채널 생성 (project)
<br/>
<br/>

### Webhook 설정 & 테스트

1. 워크 스페이스 -> 설정 및 관리 -> 앱 관리 -> 앱으로 이동
2. Web hook 검색하여 수신 웹후크를 Slack에 추가하기

![image](https://user-images.githubusercontent.com/77096463/108217291-c3345b00-7176-11eb-9e6d-9f2fa8de0b4d.png)

3. project 채널을 선택한 뒤 수신 웹후크 통합 앱 추가
4. 설정 저장 -> 웹후크 URL 복사 (추후 키 암호화에 사용할 예정)
5. cURL 요청 복사 ->EC2 인스턴스에 붙여넣은 뒤 Slack 메시지 수신된 결과 확인하기

![image](https://user-images.githubusercontent.com/77096463/108217400-e2cb8380-7176-11eb-89d4-35c5b0ee06c7.png)

![image](https://user-images.githubusercontent.com/77096463/107952055-96e4d700-6fdc-11eb-861f-bbb04be8b1f7.png)

<br/>
<br/>
<br/>

# 3. SNS 토픽 생성

1. SNS -> 주제 -> 주제 생성 

- 주제 이름 : mission-sns로 설정한 후, 주제 생성하기 버튼 클릭

![image](https://user-images.githubusercontent.com/77096463/107952393-0c50a780-6fdd-11eb-92a2-eccd279ddb80.png)

<br/>
<br/>
<br/>

# 4. Lambda 함수 생성

1. lambda -> 함수 생성
2. 블루프린트 사용 -> slack을 검색하여 Cloudwatch-alaram-to-slack-python 을 선택한다.

![image](https://user-images.githubusercontent.com/77096463/107952743-7ff2b480-6fdd-11eb-9ecc-85a588ca63e9.png)
<br/>
3. 기본 정보 

- 함수 이름 : mission-lambda
- 실행 역할 : 기본 lambda 권한을 가진 새 역할 생성

![image](https://user-images.githubusercontent.com/77096463/107953184-1cb55200-6fde-11eb-9a33-5eb087c1f604.png)
<br/>
4. SNS 트리거 -> 아까 SNS 에서 생성한 주제의 ARN을 입력한다.

![image](https://user-images.githubusercontent.com/77096463/107953209-2939aa80-6fde-11eb-8502-ff91dd99a320.png)
<br/>
5. 환경 변수 설정

- slackChannel: 값으로 slack 워크스페이스의 채널명인 project 입력
- kmsEncryptedHookUrl : KMS 키 암호화 전이기 때문에 임의의 문자열인 test 입력

![image](https://user-images.githubusercontent.com/77096463/107953220-2e96f500-6fde-11eb-9fd3-bd1c50b45bdf.png)
<br/>
<br/>
<br/>

# 5. KMS 키 설정

### KMS 키 생성
1. KMS 키 생성
2. KMS -> 고객 관리형 키 -> 키 생성 확인

```
$  aws kms create-key --region ap-northeast-2
```

<br/>

### KMS 키 별칭 생성

1. KMS -> 고객 관리형 키 -> 키 별칭 생성 확인

```
$ aws kms create-alias --alias-name alias/mission-kms-key --target-key-id [key_id] --region ap-northeast-2
```

![image](https://user-images.githubusercontent.com/77096463/107955052-a82fe280-6fe0-11eb-87ac-81850d8ce71a.png)

<br/>

### KMS 키 암호화

1. AWS CLI version 확인하여 명령어 수행

- 명령어 수행 후 CiphertextBlob 결과 값을 이전에 생성한 lambda 함수 환경변수인 kmsEncryptedHookUrl 의 value 값으로 입력하기

```
$ aws --version
$ aws kms encrypt --key-id alias/mission-kms-key --plaintext "[web_hook_url]" --region ap-northeast-2 --encryption-context LambdaFunctionName=mission-lambda
```

![image](https://user-images.githubusercontent.com/77096463/107955631-59367d00-6fe1-11eb-9ac0-907709aba14e.png)
<br/>

![image](https://user-images.githubusercontent.com/77096463/108217688-39d15880-7177-11eb-8bac-d5d46cf7c854.png)
<br/>
<br/>
<br/>

# 6. Lambda 함수 설정

### Lambda 환경변수 추가
1. lambda -> mission-lambda 함수 -> 환경변수 편집

- kmsEncryptedHookUrl 값을 CiphertextBlob 값으로 변경
- 암호화 구성: 고객 마스터키 사용
- 고객 마스터키 : mission-kms-key (위에서 생성한 KMS 키 사용)

![image](https://user-images.githubusercontent.com/77096463/107956059-f1346680-6fe1-11eb-8302-c6d7e93cc157.png)
<br/>

### Lambda Role 정책 추가 

1. IAM -> 정책 -> 정책 생성
2. JSON 탭 클릭 -> Policy 입력 (Resource : KMS키의 ARN 입력하기)

![image](https://user-images.githubusercontent.com/77096463/107963177-2abd9f80-6feb-11eb-8207-77d9cec6c2db.png)
<br/>

3. 정책 이름 : AwsLambdaKmsExecutionPolicy 설정하여 정책 생성하기
4. Lambda -> mission-lambda 함수 클릭 -> 권한 -> 역할 이름 클릭
5. 정책 연결 -> AwsLambdaKmsExecutionPolicy 정책 연결하기

![image](https://user-images.githubusercontent.com/77096463/107963588-b6373080-6feb-11eb-9893-1c5ad2afae4a.png)
<br/>
<br/>
<br/>

# 7. 테스트

### Test Event로 Slack 연동 확인

1. lambda 함수 -> 테스트 -> 새로운 테스트 이벤트 구성 -> 새로운 테스트 이벤트 생성 

- 이벤트 이름 : MissionSlackTest

```
# 테스트 케이스 코드
{
  "Records": [
    {
      "EventSource": "aws:sns",
      "EventVersion": "1.0",
      "EventSubscriptionArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "Sns": {
        "Type": "Notification",
        "MessageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "TopicArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms",
        "Subject": "ALARM: \"Example alarm name\" in EU - Ireland",
        "Message": "{\"AlarmName\":\"Example alarm name\",\"AlarmDescription\":\"Example alarm description.\",\"AWSAccountId\":\"000000000000\",\"NewStateValue\":\"ALARM\",\"NewStateReason\":\"Threshold Crossed: 1 datapoint (10.0) was greater than or equal to the threshold (1.0).\",\"StateChangeTime\":\"2017-01-12T16:30:42.236+0000\",\"Region\":\"EU - Ireland\",\"OldStateValue\":\"OK\",\"Trigger\":{\"MetricName\":\"DeliveryErrors\",\"Namespace\":\"ExampleNamespace\",\"Statistic\":\"SUM\",\"Unit\":null,\"Dimensions\":[],\"Period\":300,\"EvaluationPeriods\":1,\"ComparisonOperator\":\"GreaterThanOrEqualToThreshold\",\"Threshold\":1.0}}",
        "Timestamp": "2017-01-12T16:30:42.318Z",
        "SignatureVersion": "1",
        "Signature": "Cg==",
        "SigningCertUrl": "https://sns.eu-west-1.amazonaws.com/SimpleNotificationService-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.pem",
        "UnsubscribeUrl": "https://sns.eu-west-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "MessageAttributes": {}
      }
    }
  ]
}
```



![image](https://user-images.githubusercontent.com/77096463/107963984-2e055b00-6fec-11eb-9dd6-828e8ea1b91c.png)
<br/>
2. 테스트 버튼 클릭 -> Slack 테스트 메신저 수신 확인

![image](https://user-images.githubusercontent.com/77096463/107964071-483f3900-6fec-11eb-9af1-d3927953b40b.png)

<br/>
<br/>
<br/>

# 8. CloudWatch 알람 생성

### CloudWatch CPU 임계치 50%인 알람 생성

목표 : 1분동안 bastion instance의 CPU 사용률이 50%인 상황이 1회 이상 발생하면 알람 발생

1. CloudWatch -> 경보 -> 경보생성
2. 지표 선택 -> EC2 -> 인스턴스별 지표 -> CPU utilization 선택
3. 지표 및 조건 지정

- [지표] 통계 : 평균 / 기간 : 1분
- [조건] 임계값 : 50

![image](https://user-images.githubusercontent.com/77096463/107964878-22666400-6fed-11eb-99cc-4dfeaef4b591.png)
<br/>
4. 작업 구성

- [알림] 경보 상태 트리거: 경보 상태 / SNS 주제 선택 : 기존 SNS 주제 선택 (mission-sns)
- 이메일 (엔드포인트)가 계속 입력되지 않아 우여곡절을 겪었는데, SNS 트리거가 활성화되지 않았던 이유였다. lambda 함수 생성 후 **sns 서비스가 트리거되어 있는지 반드시 확인**하자

![image](https://user-images.githubusercontent.com/77096463/108070558-01af1480-70a8-11eb-9b37-352d9456eb42.png)
<br/>
5. 이름 및 설명 추가 -> mission-cpu-alarm

![image](https://user-images.githubusercontent.com/77096463/107965271-ac163180-6fed-11eb-8677-134c35d847d5.png)
<br/>

### CPU 부하 테스트로 알람 발생 유도

1. bastion instance에 stress 설치
2. stress 실행 -> 5분 동안 cpu 99.9% 사용

```
$ sudo yum install -y stress	//stress 설치
$ stress --cpu 1 --timeout 300	//5분동안 cpu 99.99% 사용
```

![image](https://user-images.githubusercontent.com/77096463/108070910-6cf8e680-70a8-11eb-9c48-c68b486762e0.png)

<br/>

### Slack 메신저 수신 확인

1. 알람 발생 확인 : CloudWatch -> 경보 -> mission-cpu-alarm
2. Slack 메신저 메시지 수신 확인

![경보상태](https://user-images.githubusercontent.com/77096463/108224285-178f0900-717e-11eb-9890-3be2075f0327.PNG)

![slack_alarm](https://user-images.githubusercontent.com/77096463/108224394-37263180-717e-11eb-826c-50c004f7eac0.PNG)

<br/>

<br/>

# 9. Slack 알람 문구 수정

### 메시지 포맷 개선

1. 알람 가독성 향상을 위한 메시지 포맷 개선

[기존]

![image](https://user-images.githubusercontent.com/77096463/108224577-676dd000-717e-11eb-86ae-697eea7e92b1.png)

<br/>

### Lambda 함수 코드 수정

1. CloudWatch 알람 발생 -> SNS를 통해 기록데이터의 "publishedMessage" 내용이 Lambda 함수로 전송됨
2. 따라서 메시지 내용을 Lambda 함수에서 가공해 Slack 메신저의 가독성 향상

- publishedMessage 내용이 slack 메신저에 출력됨을 확인

![image](https://user-images.githubusercontent.com/77096463/108225523-5d989c80-717f-11eb-8af2-d4faa6b080f4.png)

3. 테스트 이벤트 내용의 Message 내용을 파이썬 코드에서 parsing 및 편집

![image](https://user-images.githubusercontent.com/77096463/108226603-63db4880-7180-11eb-9d1b-49c421b63b2b.png)

4. 환경 변수 추가

- Project : Mission-Comento (서비스)
- Environment: dev (개발망)

![image](https://user-images.githubusercontent.com/77096463/108227100-e19f5400-7180-11eb-8047-12224475c8bf.png)

<br/>

모든 작업이 완료된 후, lambda 함수 테스트하면 slack 메신저는 다음과 같은 메시지를 출력한다.

![image](https://user-images.githubusercontent.com/77096463/108227331-22976880-7181-11eb-819c-8e17eaca4155.png)

<br/>

### 메시지 포맷 개선 후 CPU 부하 테스트

1. CPU 부하 테스트 stress 실행 -> 5분 동안 cpu 점유율 99.9% 
2. CloudWatch 알람 발생 확인
3. Slack 메신저 수신 확인

![image](https://user-images.githubusercontent.com/77096463/108236366-2c719980-718a-11eb-9d50-6f63318fac64.png)