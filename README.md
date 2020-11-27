
# 1. Docker

## 개요  
Container 기반의 오픈소스 가상화 플랫폼

## 구조  
Docker는 하나의 실행 파일이지만 실제로는 Client와 Server로 구성됨.  
Docker Command를 입력하면 Docker Client가 Docker Server로 명령을 전송하고 결과를 받아 터미널에 출력해줌.  
이러한 구조로 OS에 관계없이 Docker Container가 동작할 수 있음.  
아래 Command 입력시 Client와 Server 정보가 출력됨.  
> docker version  
※ permission 문제로 Server 정보 미출력시 아래 Command 입력  
sudo docker version

## Docker Image  
Container 실행에 필요한 파일과 설정값 등을 포함하고 있음.  
Container는 Docker Image를 실행한 상태라고 볼 수 있고, Container에서 동작하는 Application에 추가되거나 변경된 값은 Container 내부에 저장됨.

## Layer 저장 방식  
유니온 파일 시스템을 이용하여 여러개의 레이어를 하나의 파일 시스템으로 사용할 수 있게 해줌.  
이미지는 여러개의 읽기 전용 레이어로 구성되고 파일이 추가되거나 수정되면 새로운 레이어가 생성됨.  

## 명령어
   * Container 실행  
	docker run [**option**] [**image**]  
	ex) docker run **ubuntu:16.04**

   * Container 중지  
	docker stop [**container**]  
	ex) docker stop **mysql**

   * Container 제거  
	docker rm [**container**]  
	ex) docker rm **mysql**
		
   * Image 목록  
	docker **images**
		
   * Image 삭제  
	docker rmi [**image**]  
	ex) docker rmi **ubuntu:16.04**

## ETC  
* Container에서 실행되는 Application의 Log 및 Data File은 Container 내부에 저장되기 때문에 Container가 삭제되면 해당 File도 영구적으로 삭제됨.  
Volume 설정을 통해 Host의 Directory 및 File을 Container에서 사용.  
Container가 삭제 되도 수정 및 추가 된 File은 Volume으로 설정된 Directory에 저장되어 재사용이 가능함.  
	>docker run -v [**host path**]:[**container path**] [**image**]  
	ex) docker run -d **-v /my/own/datadir:/var/lib/mysql** mysql
	
* 외부에서 Container 내부에 실행되는 Application에 접근하기 위해서 Port Forwarding 설정.  
	>docker run -p [**host port**]:[**container port**] [**image**]  
	ex) docker run -d **-p 3306:3306** mysql

# 2. Docker Compose

## 개요
다중 Container Application을 정의하고 공유하 수 있도록 개발된 도구

## 설정 (docker-compose.yml)
> 해당 설정은 version 2.2 버전임.   
> 버전마다 작성 방법이 다를 수 있음.
```yaml
version: '2.2'   ## compose 파일 버전

services:
  tomcat:  ## service name
    container_name: tomcat  ## container name
    image: tomcat   ## image name
    ports:
      - 8080:8080   ## port forwarding
    volumes:
      - /test:/test   ## volume setting
    environment:      ## 환경 설정
     - TZ=Asia/Seoul
  mysql:
    container_name: mysql
    image: mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_USER=user
      - MYSQL_PASSWORD=password
      - TZ=Asia/Seoul
    command: mysqld --character-set-server=utf8, --sql_mode=""  ## 실행 command 설정
```

## 명령어
> ※ 설정 파일 (yml)이 있는 곳에서 명령어 실행

* 모든 Container 생성 및 실행  
docker-compose -d **up**  

* 모든 Container 중지 및 삭제  
docker-compose **down** 

* 모든 Container 실행  
docker-compose **start**  
		
* 모든 Container 중지  
docker-compose **stop**
		
* 단일 Container 조작  
docker-compose [**up** | **down** | **start** | **stop**] [**service**]  
ex) docker-compose **start tomcat**
	
* Service 목록 조회  
docker-compose **ps**

* 설정 파일 지정  
docker-compose **-f** [**file path**] [**up** | **down** | **start** | **stop**]  
ex) docker-compose **-f /tmp/docker-compose.yml start**

# 3. Kubernetes

## 개요

멀티 호스트로 구성된 클러스터 구성에서 컨테이너의 시작 및 정지와 같은 조작뿐만 아니라 호스트 간의 네트워크 연결이나 스토리지 관리, 컨테이너를 어떤 호스트에서 가동시킬지와 같은 스케줄링 기능 및 모니터링 기능 필요함.  
Kubernetes는 이러한 기능을 갖추고 있으면서 컨테이너를 통합 관리할 수 있는 컨테이너 오케스트레이션 툴

## Kubernetes Component
kubernetes cluster는 컨테이너화된 애플리케이션을 실행하는 노드라고 하는 워커 머신의 집합.  
모든 cluster는 최소 한 개의 워커 노드를 가진다.  
워커 노드는 애플리케이션의 구성요소인 Pod를 호스트한다.  
컨트롤 플레인은 워커 노드와 cluster 내 pod를 관리한다.  
프로덕션 환경에서는 일반적으로 컨트롤 플레인이 여러 컴퓨터에 걸쳐 실행되고, cluster는 일반적으로 여러 노드를 실행하므로 내결함성과 고가용성이 제공된다.

## 컨트롤 플레인 컴포넌트
컨트롤 플레인은 cluster에 관한 전반적인 결정(스케줄링 등...)을 수행하고 클러스터 이벤트를 감지하고 반응하고 cluster 내 어떠한 머신에서든지 동작할 수 있다.  
그러나 간결성을 위하여, 구성 스크립트는 보통 동일 머신 상에 모든 컨트롤 플레인을 구동시키고, 사용자 컨테이너는 해당 머신 상에 동작시키지 않는다.

* **kube-apiserver**  
api 서버는 kubernetes api를 노출하는 kubernetes 컨트롤 플레인 컴포넌트이다.  
kube-apiserver는 수평으로 확장되도록 디자인되었다.  
즉, 더 많은 인스턴스를 배포해서 확장할 수 있다.  
여러 kube-apiserver 인스턴스를 실행하고, 인스턴스간의 트래픽을 균형있게 조절할 수 있다.

* **etcd**  
모든 cluster 데이터를 담는 kubernetes 저장소로 사용되는 일관성, 고가용성 키-값 저장소.  
kubernetes cluster에서 etcd를 저장소로 사용한다면, 이 데이터를 백업하는 계획은 필수이다.

* **kube-scheduler**  
노드가 배정되지 않은 새로 생성된 Pod를 감지하고, 실행할 노드를 선택하는 컨트롤 플레인 컴포넌트.  
스케줄링 결정을 위해서 고려되는 요소는 리소스에 대한 개별 및 총체적 요구 사항, 하드웨어/소프트웨어/정책적 제약, affinity 및 anti-affinity 명세, 데이터 지역성, 워크로드 간 간섭, 데드다인을 포함한다.

* **kube-controller-manager**  
컨트롤러를 구동하는 마스터 상의 컴포넌트.  
논리적으로, 각 컨트롤러는 개별 프로세스이지만, 복잡성을 낮추기 위해 모두 단일 바이너리로 컴파일되고 단일 프로세스 내에서 실행된다.  
	* **노드 컨트롤러**: 노드가 다운되었을 때 통지와 대응에 관한 책임을 가진다.
	* **레플리케이션 컨트롤러**: 시스템의 모든 레플리케이션 컨트롤러 오브젝트에 대해 알맞은 수의 Pod들을 유지시켜 주는 책임을 가진다.
	* **엔드포인트 컨트롤러**: 엔드포인트 오브젝트를 채운다. (Service와 Pod를 연결시킨다.)
	* **서비스 어카운트 & 토큰 컨트롤러**: 새로운 네임스페이스에 대한 기본 계정과 api 접근 토큰을 생성한다.

* **cloud-controller-manager**  
클라우드별 컨트롤 로직을 포한하는 kubernetes 컨트롤러 플레인 컴포넌트이다.  
클라우드 컨트롤러 매니저를 통해 cluster를 클라우드 공급자의 api에 연결하고, 해당 클라우드 플랫폼과 상호 작용하는 컴포넌트와 cluster와 상호 작용하는 컴포넌트를 분리할 수 있다.  
cloud-controller-manager는 클라우드 제공자 전용 컨트롤러만 실행한다.  
사내 또는 PC 내부의 학습 환경에서 kubernetes를 실행 중인 경우 cluster에는 cloud-controller-manager가 없다.

## 노드 컴포넌트
노드 컴포넌트는 동작 중인 Pod를 유지시키고 kubernetes 런타임 환경을 제공하며, 모든 노드 상에서 동작한다.

* **kubelet**  
cluster의 각 노드에서 실행되는 에이전트.  
kubelet은 Pod에서 컨테이너가 확실하게 동작하도록 관리하고 다양한 메커니즘을 통해 제공된 pod spec의 집합을 받아서 컨테이너가 해당 pod spec에 따라 정상 동작하는 것을 확실히 한다.  
kubelet은 kubernetes를 통해 생성되지 않은 컨테이너는 관리하지 않는다.

* **kube-proxy**  
kube-proxy는 cluster의 각 노드에서 실행되는 네트워크 proxy로, kubernetes의 service 개념의 구현부이다.  
노드의 네트워크 규칙을 유지 관리하고, 이 네트워크 규칙이 내부 네트워크 세션이나 cluster 바깥에서 pod로 네트워크 통신을 할 수 있도록 해준다.  
kube-proxy는 운영 체제에 가용한 패킷 필터링 계층이 있는 경우, 이를 사용하며 그렇지 않으면, kube-proxy는 트래픽 자체를 forward 한다.

* **컨테이너 런타임 (docker)**  
컨테이너 런타임은 컨테이너 실행을 담당하는 소프트웨어다.

## Resource 종류

 1. **Pod**
 * kubernetes에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위
 * 하나 이상의 컨테이너 그룹
 * 그룹은 스토리지/네트워크를 공유하고, 해당 컨테이너를 구동하는 방식에 대한 명세를 갖음
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-nginx
  labels:
    app: test
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
```

 2. **ReplicaSet**
 ```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # 케이스에 따라 레플리카를 수정한다.
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

 3. **Deployment**
 ```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

 4. **Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
