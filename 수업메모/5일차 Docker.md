curl -method GET http://localhost:8080/todo?status=TODO

#### 5일차 docker

- 4일차 내용 복습 

```powershell
docker service ps todo_mysql_master  //master 가  뭔지 알 수있음
docker service ps todo_mysql_slave  //slave 가 누구 인지 알수 있음 

docker exec -it worker01 sh -># docker ps ==  # docker container ls 같은 명령어 
-> todo_mysql_master.1
docker exec -it 8b10f4409c04 bash //node 1번 접속 
mysql -ugihyo -p tododb //mysql 접속하기 
mysql> select * from todo;
```

- API 서비스 구축(p148)

 slave(read ) 와 master (update) -단방향이라고 볼 수있음 

 도메인 -  쓸 수있는 범위 (업무의 로직) 

 cmd/main.go 실행으로 환경변수를 얻어옴 / app_api 는 gateway 역할을 함 /http요청시 (api_ngin) 생성및 앤드 포인트(사용자가 사용하기 직전의 단계 ) 등록, 서버 실행 핸들러-get,post,put 

 html 없기 때문에 웹브라우저 x -> curl 은 test용으로 사용 웹브라우저가 없어도 사용가능

 

 ```powershell
docker image build -tlocalhost:5000/ch04/todoapi . 
docker push localhost:5000/ch04/todoapi:latest 
 ```

다시 설정해보기

```
$ docker-compose up 
$ docker ps (registry, manager, worker01, worker02, worker03)
$ docker exec -it manager sh
	(M) $ docker swarm init  (-> join token 생성 됨)
$ docker exec -it worker01 sh  (worker02, worker03에서도 실행)
	(W1) $ docker swarm join (with join token) 
$ docker exec -t manager sh	
	(M) $ docker node ls (worker01~worker03, manager 확인)
	(M)	$ docker network create --driver=overlay --attachable todoapp
	(M) $ docker stack deploy -c /stack/visualizer.yml visualizer
	(M) $ docker stack deploy -c /stack/todo-mysql.yml todo_mysql
$ docker exec -t worker01 (or worker02, worker03) sh -> MASTER DB 접속
	ex, W1) $ docker exec -it [MASTER DB Container] bash
	ex, W1, Master Container) $ init-data.sh 
	ex, W1, Master Container) $ mysql -uroot -p tododb
	ex, W1, Master Container) mysql> select * from todo;
$ docker push localhost:5000/ch04/todoapi:latest 
```





  docker swarm join --token SWMTKN-1-3hutd2qe7u0l5z7chm5gs1bdw09d2ozv3wxnvd4ikpq94pjcoj-dwu1u4dtho52tt8jpaz0gtn3z 172.18.0.3:23771de98ca4fea5 

 01e1435305f1

5aec5efa7af5

99b38318519e -worker 1 192.168.0.5  / 10.0.2.4

DESKTOP-94S90OJ

- 각각의 worker01,02,03 manager의 ip를 확인하고  api 중 하나에 접속해서 확인 root@8df8cc1b879b:

```powershell
 apt update
 apt-get install -y net-tools
 curl -XGET  http://localhost:8080/todo?status=TODO
 curl -XGET  http://localhost:8080/todo?status=PROGRESS
 netstat -ntpl
 echo "current: todo_app_api.2"
 
 PS C:\Users\HPE\docker\day03\swarm> docker exec -it manager sh
/ # docker service logs -f todo_app_api //log파일 확인 하기

```

- Nginx구축하기 

proxy 단일 접근으로 log파일 집중 /캐시제어 / 라우팅 설정 

```powershell
>docker stack services todo_app
```

```powershell
docker build -t localhost:5000/ch04/nginx:latest .
docker image push localhost:5000/ch04/nginx:latest
docker exec -it workder01,02,03 sh
# docker service ls
```

- nginx  

```powershell
#docker exec -it eb8816af12d7 bash //worker 02 번 들어가기
#docker ps 

#apt update
#apt-get install -y curl //curl 주소 사용 
#apt-get install -y net-tools
#netstat -ntpl
#curl -XGET http://localhost:8000/todo?status=TODO //8000번으로 접속해서 8080 의 내용을 볼수 있음 
```

클라이언트 - nginx (8000)- api서비스 (8080) 

- 쿠버네티스 

docker container 운영을 자동화 하기 위한 컨테이너 오케스트레이션 툴 

swarm 과 동일한 기능을 수행하지만 더 많은 기능을 구현하는 것이 가능 

쿠베네티스 -- swarm 보다 충실한 기능을 갖춘 컨테이너 오케스트레이션 시스템

swarm --여러대의 호스트를 묶어서 기초적인 컨테이너 기능을 제공 / 간단한 멀티 컨테이너 구축 

POD -service단위로 공개/단위는 deployment 

파드 replica set *3 

service  이전과 다른 의미 - cluster name 을 노출 시키기 위한 단위  / 파드를 서비스에 묶어서 배포

kubetctl apply  -f  https://raw.githubsercontent.com/kubernetes/dashboard/v1.8.3/src/deploy/recommended/kubernetes--dashboard.yaml

kubetctl get pod --namespace=kube-system -l k8s--app=kubernetes-dashboard



```powershell
kubectl proxy
```

````powershell
kubectl get pods
kubectl get services
````

pod

컨테이너가 모인 집합체로 하나이상의 컨테이너로 구성 

중복되지 않는 ip,port 제공  // pod 문제가 생길시 삭제후 다시 생성 

