---
title: "쿠버네티스 환경의 로그수집 - EFK (ElasticSearch, Fluentd, Kibana) Stack 구축 "
categories :	
 - AWS
tags : 
 - AWS
 - 클라우드
 - EFK
 - ElasticSearch
 - Fluentd
 - Kibana
---

1. ElasticSearch 구축

2. Fluentd 구축

3. Kibana 구축

  <br/>
### EFK Stack  구축을 통한 로그 수집
- 컨테이너 환경에서의 로그 수집을 위해 EFK (ElasticSearch + Fluentd + Kibana) Stack을 구축
- 로그 저장소인 ElasticSearch, 로그 수집기인 Fluentd, 로그 시각화 툴인 Kibana를 EKS 환경에서 구축
- 쿠버네티스는 파드가 정상상태가 아니라면 새로 생성 -> **로그 저장소**는 죽은 파드에 있는 컨테이너가 남긴 로그를 수집

### EFK 개념
- ElasticSearch : 로그를 저장하기 위한 대용량 저장소
- Fluentd : 컨테이너의 스트림 로그를 수집하는 로그 수집기. 모든 노드마다 동일하게 배포되어야 함 -> Daemonset
- Kibana : ElasticSearch와 연동하여 로그 시각화 -> 로그 시각화를 통한 문제 해결 및 예방 가능

![image](https://user-images.githubusercontent.com/77096463/109522240-68452100-7af1-11eb-8695-e757a15986ef.png)

### Daemonset 개념
- 쿠버네티스 컨트롤러 (deployment, replicaset) 중 하나 
	- 컨트롤러 : 쿠버네티스 기본 오브젝트를 생성하고 관리하는 역할
- Daemonset은 파드가 각각의 노드에 하나씩만 배포되게 하는 파드 관리 컨트롤러 

<br>

# 0. EKS 구성 및 Nginx 실행

저번 포스트에서 비용 문제때문에 삭제하였던 EKS Cluster와 NodeGroup을 다시 생성한다.

```
$ eksctl create cluster --name mission-cluster --version 1.17 --region ap-northeast-2 --nodegroup-name mission-wn --node-type t3.medium --nodes 1 --nodes-min 1 --nodes-max 1 --ssh-access --ssh-public-key workernode-key --managed
```
<br/>

생성된 노드를 확인한다.

```
$ kubectl get node 
```

![image-20210131230724134](https://user-images.githubusercontent.com/77096463/106752840-3ebde480-666e-11eb-9d79-bd95cae6ec95.png)

<br/>

Nginx 서비스를 배포하기 위해 우선 Nginx-Deployment.yaml 파일을 생성한 다음, Nginx 파드를 배포한다.

```
$ kubectl apply -f nginx-deployment.yaml
$ kubectl get pod				
```

![image-20210131230903147](https://user-images.githubusercontent.com/77096463/106752880-48dfe300-666e-11eb-8f6a-62fe13a9b91f.png)

<br/>

다음 단계로, Nginx-Service.yaml 파일을 생성한 뒤, Nginx 서비스를 배포한다.

```
$ kubectl apply -f nginx-service.yaml		
$ kubectl get service	 	
```

![image-20210131233835030](https://user-images.githubusercontent.com/77096463/106753105-90666f00-666e-11eb-9411-711900d46a28.png)

<br/>

로드 밸런서의 DNS를 통해 Nginx 서버 접속이 가능한지 확인한다.

![image-20210131233946149](https://user-images.githubusercontent.com/77096463/106753128-99574080-666e-11eb-8a31-c17f0e28e878.png)
<br/>
<br/>
<br/>

# 1. ElasticSearch 구축

> elasticSearch 란 모든 유형의 데이터를 위한 분산형 오픈 소스 검색 및 분석 엔진으로, 많은 양의 데이터를 보관하고 실시간으로 분석하는 엔진이다.

쿠버네티스 환경에서는 elasticSearch.yaml 파일을 작성하여 elasticSearch를 배포할 수 있다.
먼저 elasticSearch.yaml 파일을 작성한다.

```
$ cat<<EOF > ~/elasticSearch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  labels:
    app: elasticsearch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: elastic/elasticsearch:6.4.0
        env:
        - name: discovery.type
          value: "single-node"
        ports:
        - containerPort: 9200
        - containerPort: 9300
        
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch-svc
  namespace: default
spec:
  ports:
  - name: elasticsearch-rest
    nodePort: 30920	//(2)Node의 port
    port: 9200	//(3)Service의 port
    protocol: TCP
    targetPort: 9200	//(4)Pod의 port
  - name: elasticsearch-nodecom
    nodePort: 30930
    port: 9300
    protocol: TCP
    targetPort: 9300
  selector:
    app: elasticsearch
  type: NodePort
EOF
```
![image](https://user-images.githubusercontent.com/77096463/109523763-0c7b9780-7af3-11eb-8932-57ac83ab12f9.png)

1) LB의 보안 그룹 9200 포트 오픈 -> 인터넷을 통해 9200번 포트로 접근
2) 9200번 포트로 들어온 트래픽을 Instance (workernode)의 30920번 포트로 전달
3) workernode로 전달된 트래픽은 9200번 포트를 통해 서비스로 전달 -> yaml파일에 정의된 대로 9200번 포트를 통해 서비스는 클러스터 안에서 내부적으로 노출됨
4) 9200번 포트로 보내진 요청은 서비스에 의해 선택된 Pod의 9200번 포트로 전달됨  (targetPort)


<br/>

elasticSearch.yaml파일로 NodePortType에 따라 elasticSearch를 배포한다.

```
$ kubectl apply -f elasticSearch.yaml	# 배포
$ kubectl get pod | grep elastic	# pod 확인
$ kubectl get svc | grep elastic	# service 확인
```

![image-20210201004701321](https://user-images.githubusercontent.com/77096463/106753158-a5430280-666e-11eb-9a37-795402bec2ea.png)
<br/> <br/>



> 리스너란 정의한 프로토콜과 포트를 사용하여 연결요청을 확인하는 프로세스다.
> 리스너에 정의한 규칙에 따라 로드밸런서가 등록된 대상으로 요청을 라우팅하는 방법을 결정한다.

LoadBalancer에 리스너를 추가하기 위해 elasticSearch.yaml파일에서 정의한 9200 포트를 로드밸런서의 리스너 탭에 추가한다. 

![image-20210201005612258](https://user-images.githubusercontent.com/77096463/106753180-aecc6a80-666e-11eb-99cf-54e43bdb6e79.png)
<br/>

또한, 로드밸런서의 9200번 포트로 외부 접근 허용을 위해 로드밸런서의 보안 그룹 인바운드 규칙을 편집해야 한다.

![image-20210201005412491](https://user-images.githubusercontent.com/77096463/106753225-bee44a00-666e-11eb-9d40-ef4e544b0986.png)
<br/>

모든 과정이 완료된다면 **로드밸런서DNS:9200** 접속을 통해 ElasticSearch를 NodePort type의 서비스로 배포하여 Nginx의 로드밸런서에 연결되어 통신이 이루어짐을 확인할 수 있다.

<br/>

<br/>



# 2. Fluentd 구축

> Fluentd는 로그 수집기로 다양한 데이터 소스(http, tcp 등)로부터 원하는 형태로 가공하여 목적지(s3, ElasticSearch)로 전달 가능하다.

fluentd.yaml 파일을 작성하여 fluentd를 배포할 수 있다. 먼저 fluentd.yaml 파일을 작성한다.

```
$ cat<<EOF > ~/fluentd.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-system

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
      version: v1
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.4.2-debian-elasticsearch-1.1
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch-svc.default.svc.cluster.local"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "http"
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
EOF
```
<br/>

fluentd.yaml 파일로 fluentd Daemonset pod를 배포한 후 서비스가 배포되었는지 확인한다.

![image-20210201012457525](https://user-images.githubusercontent.com/77096463/106753270-cd326600-666e-11eb-939c-1376c2098718.png)
<br/>

아래의 kibana 구축 단계를 거쳤다면 kibana에서 로그를 수집할 인덱스 패턴을 생성할 수 있다. 
(먼저 kibana 구축 끝내고 이번 단계 진행)

- 인덱스 패턴 생성 : kibana -> [management] -> [index patterns] -> Index Pattern: logstash* -> next step -> @timestamp -> create index pattern

![image-20210201013017631](https://user-images.githubusercontent.com/77096463/106753329-e3d8bd00-666e-11eb-9536-aa5eb87fed1b.png)

![image-20210201013034114](https://user-images.githubusercontent.com/77096463/106753356-eaffcb00-666e-11eb-8cdf-6effd3fabd46.png)
<br/>

로그 시각화 마지막 단계이다. 검색 필터를 설정하여 nginx의 로그를 확인할 예정이다.
kibana -> [discover] -> 검색필터 설정 -> kubernetes.labels.run is [nginx 배포 시 사용한 label]

![image-20210201013236572](https://user-images.githubusercontent.com/77096463/106753392-f3580600-666e-11eb-99f4-250c347d10b4.png)
<br/>
<br/> 
<br/>

# 3. Kibana 구축

> kibana는 **오픈 소스 기반의 분석 및 시각화 플랫폼**이다. Elastic stack을 기반으로 구축된 오픈 소스 프론트엔드 애플리케이션으로 ElasticSearch에서 색인된 데이터를 검색하고 시각화하는 기능을 제공한다.

kibana.yaml 파일을 작성하여 kibana를 배포할 수 있다. 먼저 kibana.yaml 파일을 작성한다.

```
$ cat<<EOF > ~/kibana.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  labels:
    app: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: elastic/kibana:6.4.0
        env:
        - name: SERVER_NAME
          value: "kibana.kubenetes.example.com"
        - name: ELASTICSEARCH_URL
          value: "http://elasticsearch-svc.default.svc.cluster.local:9200"
        ports:
        - containerPort: 5601

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kibana
  name: kibana-svc
  namespace: default
spec:
  ports:
  - nodePort: 30561
    port: 5601
    protocol: TCP
    targetPort: 5601
  selector:
    app: kibana
  type: NodePort
EOF
```
<br/>

작성한 kibana.yaml로 NodePortType의 서비스로 kibana를 배포한다. 
```
$ kubectl apply -f kibana.yaml	# 배포
$ kubectl get pod	# pod 확인
$ kubectl get service	# 서비스 확인
```

![image-20210201010727073](https://user-images.githubusercontent.com/77096463/106753460-0965c680-666f-11eb-91b4-697da2363887.png)
<br/>

LoadBalancer에 리스너를 추가하기 위해 kibana.yaml파일에서 정의한 5601포트를 로드밸런서의 리스너 탭에 추가한다. 

![image-20210201011325301](https://user-images.githubusercontent.com/77096463/106753502-184c7900-666f-11eb-98c0-62636aea78b6.png)
<br/>

또한, 로드밸런서의 5601번 포트로 외부 접근 허용을 위해 로드밸런서의 보안 그룹 인바운드 규칙을 편집해야 한다.

![image-20210201011429082](https://user-images.githubusercontent.com/77096463/106753536-213d4a80-666f-11eb-8b6a-5265ba90451f.png)
<br/>

모든 과정이 완료된다면 **로드밸런서DNS:5601** 접속을 통해 kibana를 NodePort type의 서비스로 배포하여 Nginx의 로드밸런서에 연결되어 통신이 이루어짐을 확인할 수 있다.

![image-20210201011800197](https://user-images.githubusercontent.com/77096463/106753558-27cbc200-666f-11eb-855b-90e5cf09b80d.png)

<br/>

<br/>



# 4. 자원 삭제

1. kubernetes 자원 삭제

```
$ kubectl delete -f elasticSearch.yaml
$ kubectl delete -f kibana.yaml
$ kubectl delete -f fluentd.yaml
$ kubectl delete -f nginx-service.yaml
$ kubectl delete -f nginx-deployment.yaml
```

![image-20210201013708055](https://user-images.githubusercontent.com/77096463/106753597-374b0b00-666f-11eb-8ef6-456a29d80ee4.png)
<br/>

2. EKS 클러스터와 Node Group을 삭제한다.

- 콘솔에서 eks cluster와 nodegroup이 삭제되었는지 확인한다.

```
$ eksctl delete cluster --region ap-northeast-2 --name=mission-cluster
```

![image-20210201013800623](https://user-images.githubusercontent.com/77096463/106753616-3d40ec00-666f-11eb-999f-a26407c6c3f0.png)

