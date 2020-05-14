```
docker image rm  -f c4449df885e1 //강제 삭제 

docker run -d -p 27017:27017 suhaw0325/mymongodb
docker run -d -p 27018:27017 suhaw0325/mymongodb
```

- Dockerfile 

```dockerfile
FROM mongo

RUN mkdir /usr/src/configs

WORKDIR /usr/src/configs

COPY  replicaSet.js . 
#COPY setup.sh .

CMD ["mongo", "mongodb://mongo1:27017", "replicaSet.js" ]
```

- docker-compose.yml 

```yaml
version: "3"
services: 
    mongo1:
        image: "mongo"
        ports: 
            - "27017:27017"
        volumes: 
            - $HOME/mongoRepl/mongo1:/data/db
        networks: 
            - mongo-networks
        command: mongod --replSet myapp

    mongo2:
        image: "mongo"
        ports: 
         - "27018:27017"
        volumes: 
         - $HOME/mongoRepl/mongo2:/data/db
        depends_on: 
            - mongo1
        networks: 
         - mongo-networks
        command: mongod --replSet myapp

    mongo3:
        image: "mongo"
        ports: 
          - "27019:27017"
        volumes: 
          - $HOME/mongoRepl/mongo3:/data/db
        depends_on: 
            - mongo2
        networks: 
          - mongo-networks
        command: mongod --replSet myapp --bind_ip_all

    mongo_setup:
        #build:  .  
        image: "mongo_repl_setup:latest"
        depends_on: 
            - mongo3
        
        networks: 
            - mongo-networks

networks: 
    mongo-networks:
        driver: bridge             
```

- replicaSet.js

```js
config = {
    _id: "myapp",
    members:[
        {_id:0, host: "mongo1:27017" },
        {_id:1, host: "mongo2:27017" },
        {_id:2, host: "mongo3:27017" },
    ]
}

rs.initiate(config);
rs.conf();
```

- power shell((Secondary) -> (Primary))

```powershell
docker build --no-cache -t mongo_repl_setup:latest .
docker-compose up
docker exec -it mongo 1번 bash -> mongo -> primary 로 
myapp.primary > db.books.save({"title":"docker file"})
docker exec -it mongo 1번 bash -> mongo mongodb://mongo2:27017 -> secondary 로 
myapp.secondary > rs.slaveOk();
myapp:SECONDARY> use bookstore;
myapp:SECONDARY> db.books.find();

docker stop mongo1 
docker start mongo1  다시 접속을 하면 secondary  변경되고 다른곳이 primary 로 승격
```



overlay network -host간의 네트워크 공유 // 다른 호스트라할지라도 같은 네트워크에 있어 작업을 하는데 용이

->swarm

docker inspect  host ip 주소를 알수 있음 

docker network ls

swarm : 여러개의 호스트를 묶는 것이 목적

docker compose 주로 단일 호스트를 관리 

docker swarm  클러스터 구축 및 관리 

docker service (컴포턴트 ,스웜 ,쿠버네티스)

docker stack  스웜에서 여러개의 서비스를 합한 전체 어플리케이션을 관리 

docker file모여서 compose -> service -> stack 

- swarm 설치하기

```powershell
docker-compose.yml  작성하기 -> docker--compose up 
docker exec -it manager sh //manager 로 접근 -> 
docker swarm join --token SWMTKN-1-50zbzypn292igefos3efy2qcj446su1q077tbcabx4y9ttxzaz-0rlcisrjqkp76t4z217ciwa8c 172.30.0.3:23

```

호스트 네임으로 연동이 가능 worker01 ,manager ,,,

2377:cluster managerment 통신에 사용 

- manager에 이미지 넣기 

```powershell
docker tag busybox:latest localhost:5000/busybox:latest 

docker pull localhost:5000/busybox:latest -> 오류가 발생

대신 docker pull registry:5000/busybox:latest -> image 를 확인 할 수있음 (docker images )

윈도우에서는 registry:5000 으로 하면 오류가 발생 
```

- swarm 확인하기 

```powershell
docker exec -it manager docker node ls // manager :leader /worker1,2,3 active 인것을 확인 
```

```
manager에 들어가서 docker node ls 로 확인가능 
```



