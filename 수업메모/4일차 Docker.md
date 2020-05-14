#### 4일차 Docker 

registy 제외 manage, worker1 -3  컨테이터로 또다른 컨테이너가 올수 있다

- manger 접속하기 

```powershell
>docker exec -it manager sh

# docker node ls 
# docker info 
# docker stack ls // swarm 이 수행되어야 가능한 명령어 

```

- 서비스 - 어플리케이션을 구성하는 일부컨테이너를 제어하기 위한 단위

docker service 는 swarm 상태여야지만 가능하다 

```powershell
>docker service create --replicas 1 --publish 8000:8080 --name mybusybox >registry:5000/busybox:latest //registry:5000번의 busy box사용 

```

```powershell
>docker tag gihyodocker/echo:latest localhost:5000/example/echo:latest
>docker images
>docker push localhost:5000/example/echo:latest
#docker exec -it manager sh
#docker service ls
```



w-8000->manger80 -> 8080 2번의 port forwording  

```powershell
 # docker service create --replicas 1 --publish 8000:80 --name echo registry:5000/example/echo:latest //8000요청이면 80으로 nager ->삭제 
 # docker service create --replicas 1 --publish 80:8080 --name echo registry:5000/example/echo:latest //localhost:8000 으로 접속
 
 # docker service ps echo 
 #docker service scale echo=3 // 3개로 구성 
 
```

- stack - 하나이상의 **서비스를 그룹**으로 묶는 단위 /  여러서비스를 함께 다룰 수 있음 /스택을 사용한 배포된 

서비스 그룹은 overlay 네트워크에 속함 

worker 1,worker2 모두 echo service  worker 서비스는 또다른 서비를 가짐 -> 같은 네트워크에 묶기 위해 overlay  

docker stack depploy -c "file path" [stack name]

```yaml
version: "3"
services:
    api:
        image: registry:5000/example/echo:latest
        deploy:
            replicas: 3
            placement:
                constraints: [node.role != manager]
        networks:
            - ch03

    nginx:
        image: gihyodocker/nginx-proxy:latest
        deploy:
            replicas: 3
            placement: 
                constraints: [node.role != manager]
        environment: 
        		SERVICE_PORTS: 80
                BACKEND_HOST: echo_api:8080
        networks:
                - ch03
        depends_on:
                - api
networks:
    ch03:
        external: true
```

> $docker stack deploy -c /stack/my-webapi.yml echo -> localhost:8000 번 접속

```yaml
version: "3"
services:
  app:
    image: dockersamples/visualizer
    ports:
      - "9000:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      mode: global
      placement:
        constraints: [node.role == manager]
```

> $docker stack deploy -c /stack/visualizer.yml  visualizer -> localhost:9000번 접속 

```yaml
version: "3"
services:
    haproxy:
        image: dockercloud/haproxy
        networks: 
            - ch03
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
        deploy:
            mode: global
            placement: 
                constraints:
                    - node.role == manager
        ports:
            - 80:80
            - 1936:1936 # for stats page (basic auth. stats:stats)
networks: 
    ch03:
        external: true
```

- HAProxy

> $docker stack deploy -c /stack/my-webapi.yml  echo // 스택 echo 로 다시 배포

> $docker stack deploy -c /stack/ch03-ingress.yml ingress //스택 ingress 로 배포 

> $docker service ls 

-> localhost:8000 -> hello docker 의 내용이 뜨게 됨 

- MYSQL 서비스 구축 

mysql -master /slave 이미지 생성

```powershell
>docker build -t localhost:5000/ch04/tododb:latest . // tag 명령어까지 실행
>docker push localhost:5000/ch04/tododb:latest // registry 의 이미지 목록  http://localhost:5000/v2/_catalog 확인

```

replicas master ->1 replicas slave ->2개 생성 

- docker networkd  -manager

```powershell
 #docker network create --driver=overlay --attachable todoapp
 #stack deploy -c /stack/todo-mysql.yml todo_mysql
 
```

```powershell
docker exec -it worker01 sh
hostname -i
172.30.0.5
docker ps
docker exec -it 8b10f4409c04 bash
```

- MYSQL 접근하기 

```powershell
docker exec -it manager `
    docker service ps todo_mysql_master --no-trunc `
    --filter "desired-state=running" `ex
    --format "docker exec -it {{.Node}} docker exec -it {{.Name}} 
    .{{.ID}} bash"

   docker exec -it manager `
    docker service ps todo_mysql_slave --no-trunc `
    --filter "desired-state=running" `
    --format "docker exec -it {{.Node}} docker exec -it {{.Name}} 
    .{{.ID}} bash"
 master (password -jihyo)
>docker exec -it 1de98ca4fea5 docker exec -it todo_mysql_master.1.uk5kj8sstyjgqbaha4uvgbrxm bash
 slave1
>docker exec -it 6ceb60fc0bb6 docker exec -it todo_mysql_slave.1
    .nr3x47asu3w3cfephqhtqoscn bash
 slave2 
>docker exec -it 6ceb60fc0bb6 docker exec -it todo_mysql_slave.2
    .ehjl1komkmj4dphlarmzjf0cy bash
```

- MYSQL 데이터 확인 

```powershell
docker exec -it 537327e57681 docker exec -it todo_mysql_master.1.p97u13qv89uwjdm4zrf0rh0xt bash // 접속
init-data.sh
mysql -uroot -p tododb
mysql> select * from todo; 
```

 slave 가서 확인하면  mysql> select * from todo;  확인 가능