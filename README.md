
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

## 구조
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
