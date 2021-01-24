---
title: "쿠버네티스 환경 기반의 EKS 클러스터 구축, Nginx 서비스 배포 "
categories :	
 - AWS
tags : 
 - AWS
 - 클라우드
 - EKS
 - Nginx
---

1. 쿠버네티스 환경 구축
2. EKS Cluster 생성
3. Nginx 서비스 배포
<br/>
<br/>

# 1. 쿠버네티스 환경 구축
`yum update`를 진행한다. 

```
$ sudo su - ec2-user
$ sudo yum update -y
```
<br/>
`aws cli version`을 확인한다.

```
$ aws --version
```
<br/>
IAM Role을 생성한다.
1. [IAM] -> [역할] ->[역할 만들기] 메뉴를 선택한 뒤, 역할 만들기 개체와 사례를 선택한다.
![역할1](https://user-images.githubusercontent.com/77096463/105619774-31685500-5e39-11eb-84c6-fd76d38afb06.PNG)
<br/>

2. 권한 정책 연결로는 'AdministratorAccess'를 선택하고 넘어간다. (태그는 생략) 
![역할2](https://user-images.githubusercontent.com/77096463/105619787-447b2500-5e39-11eb-90ea-3505bd06b8c1.PNG)
<br/>

3. IAM 역할 이름과 설명을 'mission-admin-Role'이라고 입력한 뒤 넘어간다.
![역할3](https://user-images.githubusercontent.com/77096463/105619790-4c3ac980-5e39-11eb-8a30-3b98a45926ec.PNG)
<br/>
<br/>


생성한 역할을 EC2에 부여한다.
1. 이미 생성된 인스턴스에 이름을 부여하고, 해당 인스턴스를 우클릭하여 [보안] -> [IAM 역할 수정]로 이동한다.
![인스턴스 이름부여](https://user-images.githubusercontent.com/77096463/105619811-83a97600-5e39-11eb-8b97-e6582416613a.PNG)
<br/>

2. IAM 역할 수정 페이지에서 IAM 역할로 방금 생성한 역할을 선택한다.
![image](https://user-images.githubusercontent.com/77096463/105619868-295ce500-5e3a-11eb-9392-4b7b9655948f.png)
<br/>
<br/>

kubectl을 설치한다.
```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl	// kubectl 설치파일 다운로드
$ chmod +x ./kubectl	// 권한 변경
$ sudo mv ./kubectl /usr/local/bin/kubectl		// 디렉토리 변경
$ kubectl version --client		// 설치 확인
```
![image](https://user-images.githubusercontent.com/77096463/105620043-18ad6e80-5e3c-11eb-86ae-b625ecc4ff16.png)

<br/>

eksctl을 설치한다.
```
$ curl --silent location "https://github.com/ weaveworks eksctl /releases/latest/ eksctl uname -s)_amd64.tar.gz" | tar xz C / tmp 	
// eksctl 설치파일 다운로드
$ sudo mv /tmp/eksctl /usr/local/bin		// 디렉토리 변경
$ eksctl version	// 설치 확인
```
![image](https://user-images.githubusercontent.com/77096463/105620067-53170b80-5e3c-11eb-9d42-72a53b205e3e.png)

<br/>

Nodegroup용 ssh키를 생성한다.
 - `ssh-keygen` 명령어 입력 후, 차례대로 나오는 작업은 모두 enter를 입력한다.
 - `ssh-keygen` 를 통해 키를 생성하면 'id_rsa', 'id_rsa.pub' 이름의 키가 만들어짐을 확인한다.

```
$ aws configure set region ap-northeast-2			// region 설정
$ cd .ssh 											// .ssh 폴더로 이동
$ ssh-keygen										// 키 생성
$ ls												// 키 확인
```
![image](https://user-images.githubusercontent.com/77096463/105620147-0b44b400-5e3d-11eb-8817-f3ffe8b03ab3.png)
![image](https://user-images.githubusercontent.com/77096463/105620192-755d5900-5e3d-11eb-8b2f-cc4f502efaa7.png)
<br/>

방금 생성된 공개키를 사용중인 EC2 리전으로 업로드한다.

- 명령어 수행 후, [EC2] -> [키 페어] 로 이동하여 키 페어 목록에 workernode-key가 추가되었는지 확인한다.

```
$ aws ec2 import-key-pair --key-name "workernode-key" --public-key-material file://~/.ssh/id_rsa.pub
```
![image](https://user-images.githubusercontent.com/77096463/105620246-3085f200-5e3e-11eb-8668-736f18c46be2.png)
<br/>
<br/>
<br/>

# 2. EKS 클러스터 생성

eksctl 명령어를 사용하여 eks 클러스터와 workerNode를 생성한다.
 - 15~20분 정도 소요된다.
 - EKS 클러스터의 이름 : mission-cluster
 - NodeGroup의 이름 : mission-wn
 - NodeGroup의 키페어 : 기본 ssh-key 사용
 - [컴퓨팅 구성 설정] 인스턴스 유형 : t3.medium
 - [조정 구성 설정] 최소, 최대, 원하는 크기 : 1 (비용 문제 때문에 일단 작게 설정한다)

```
$ eksctl create cluster --name mission-cluster --version 1.17 --region ap-northeast-2 --nodegroup-name mission-wn --node-type t3.medium --nodes 1 --nodes-min 1 --nodes-max 1 --ssh-access --ssh-public-key workernode-key --managed
```
![Untitled Diagram](https://user-images.githubusercontent.com/77096463/105620632-439ac100-5e42-11eb-99ae-6dfaae4b2dec.png)
![image](https://user-images.githubusercontent.com/77096463/105620533-71cbd100-5e41-11eb-85ef-3ad80298e29a.png)
<br/>

아래의 화면의 마지막 문구를 통해 eks 클러스터가 성공적으로 생성되었음을 확인한다.
![image](https://user-images.githubusercontent.com/77096463/105620762-97f27080-5e43-11eb-9bfa-fe9d2dacdeb8.png)
<br/>

생성된 노드를 확인한다.
```
$ kubectl get node 
```
![image](https://user-images.githubusercontent.com/77096463/105620805-d12ae080-5e43-11eb-9a31-f577b10e7e68.png)
<br/>
<br/>
<br/>

# 3. Nginx 서비스 배포

Nginx-Deployment.yaml 파일을 생성한다.
```
$ cat <<EOF > ~/nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: my-nginx
spec:
 selector:
  matchLabels:
   run: my-nginx
 replicas: 2
 template:
  metadata:
   labels:
    run: my-nginx
  spec:
   containers:
   - name: my-nginx
     image: nginx
     ports:
     - containerPort: 80
EOF
```
<br/>

Nginx 파드를 배포한다.
```
$ kubectl apply -f nginx-deployment.yaml	// nginx-deployment.yaml을 배포한다.
$ kubectl get pod						// 생성된 파드 확인
```
![image](https://user-images.githubusercontent.com/77096463/105621448-c58ee800-5e4a-11eb-878f-8abb0f820480.png)

<br/>

Nginx-service.yaml 파일을 생성한다.
```
$ cat <<EOF > ~/nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
 name: my-nginx
 labels:
  run: my-nginx
spec:
 ports:
 - port: 80
   protocol: TCP
  selector:
   run: my-nginx
  type: LoadBalancer
EOF
```
<br/>

Nginx service를 배포한다.
 - Load Balancer 타입으로 생성하였기에 CLB타입의 ELB가 생성된다.
 - [EC2] ->[로드 밸런서] 메뉴로 이동하면 새로운 로드밸런서가 생성되었음을 확인할 수 있다.

```
$ kubectl apply -f nginx-service.yaml		// nginx-service.yaml을 배포한다.
$ kubectl get service	 	// 배포된 서비스를 확인한다.		
```
![image](https://user-images.githubusercontent.com/77096463/105621591-85306980-5e4c-11eb-8c41-7e3bb8362c10.png)

<br/>

로드 밸런서의 DNS를 통해 Nginx 접속이 가능하다.
![image](https://user-images.githubusercontent.com/77096463/105621641-156eae80-5e4d-11eb-9898-46615d43b8f4.png)
<br/>
<br/>
<br/>

# 4. Cluster와 NodeGroup 삭제

> 비용 문제때문에 일시적으로 EKS cluster와 NodeGroup을 생성한 뒤, 다시 삭제한다.

Nginx-service.yaml, Nginx-deployment.yaml을 삭제한다.
 - 확실하게 삭제하기 위해 콘솔에서 CLB가 삭제되었는지 확인한다.
 - [EC2] -> 로드밸런서 -> CLB 삭제 확인

```
$ kubectl delete -f nginx-service.yaml
$ kubectl delete -f nginx-deployment.yaml
```
![image](https://user-images.githubusercontent.com/77096463/105621713-da20af80-5e4d-11eb-8aeb-b6f9931d5ba0.png)

<br/>

EKS 클러스터와 Node Group을 삭제한다.
 - 확실한 삭제를 위해 콘솔에서 EKS 클러스터와 Node Group 가 삭제되었는지 확인한다.
 - EKS Cluster
 - [EC2] -> Autoscaling group -> Autoscaling group 삭제 확인
```
$ eksctl delete cluster --region ap-northeast-2 --name=mission-cluster
```
![image](https://user-images.githubusercontent.com/77096463/105621783-b6aa3480-5e4e-11eb-94e4-7971621dfa3f.png)

<br/>
<br/>

