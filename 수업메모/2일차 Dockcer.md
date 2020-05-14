#### 2일차 Dockcer 

- docker 내용 수정하기 -1

```powershell
>docker build -t suhaw0325/simpleweb:modified .
>docker stop 774a4e801c37
>docker rm 774a4e801c37
>docker run -d -p 8080:8080 suhaw0325/simpleweb:modified //수정된 결과를 출력 권장x 
```

- docker 내용 수정하기 -2 (container 내용 수정 )

```powershell
docker exec -it 164f1b913c85 sh
vi index.js
docker restart 164f1b913c85
```

- docker 내용 수정하기 -3 (1+2)

```powershell
>docker run -v C:\Users\HPE\docker\day01\simpleweb:/home/node -d -p 8080:8080 
suhaw0325/simpleweb:modified //-v 호스트 디렉토리: 컨테이너 폴더 
>docker ps
>docker exec -it b907f7d5fdbe sh
>cat index.js
>docker restart b907f7d5fdbe sh //반영이 됨 
```

- 데이터 볼륨에 mysql 데이터 저장하기 

```powershell
>docker run -d --rm --name mysql  `
-e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" `
-e "MYSQL_DATABASE=volume_test" `
-e "MYSQL_USER=example" `
-e "MYSQL_PASSWORD=example" `
--volumes-from mysql-data `
mysql:5.7
```

```powershell
>docker exec -it mysql mysql -uroot -p volume_test
mysql> 데이터 베이스 접속 
create table user (id int primary key auto_increment, name varchar(20))
insert into user (name) values ('ss'),('aa'),('dd');
>docker container stop mysql 
>docker run -d --rm --name mysql  `
-e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" `
-e "MYSQL_DATABASE=volume_test" `
-e "MYSQL_USER=example" `
-e "MYSQL_PASSWORD=example" `
--volumes-from mysql-data `
mysql:5.7
docker exec -it mysql mysql -uroot -p volume_test //다시 접속 
==docker exec -it mysql bash # mysql -uroot -p volume_test
mysql>select * from user; //지워도 그대로 존재 
```

- docker-compose 파일 (여러개의 서비스를 만들수 있음 )

```yml
version: "3"
services: 
    my-mysql1:
        image: mysql:5.7
        ports: 
            - "13306:3306"
        environment: 
            - MYSQL_ALLOW_EMPTY_PASSWORD=true
    my-mysql2:
        image: mysql:5.7
        ports: 
            - "23306:3306"
        environment: 
            - MYSQL_ALLOW_EMPTY_PASSWORD=true
            
```

- docker-compose up  

- docker-compose down  //폴더 이동해서 가능 

```yaml
version: '3'
services: 
    simpleweb:
        image: suhaw0325/simpleweb:latest
        ports:
            - 8080:8080
```

- docker-compose up 으로 켜기 



mongodb 를 docker container  싱행

 docker file 작성

docker image buid 

​	-suhaw0325/mymongodb:latest

container 생성 -> 실행 

client 에서 mongodb테스트  (day2에서 mkdir mongo 만들기)

- Dockerfile 

```yaml
FROM mongo
CMD ["mongod"] 
```
```powershell

>docker build -t  suhaw0325/mongodb:latest .
>docker run -p 27017 suhaw0325/mongodb:latest //mongodb는 이미지명 
>docker exec -it eb584b82fca0 bash
root@eb584b82fca0:/# mongo
>db.books.save({'title':'docker compose sample'});
>db.books.find();
```


primary ,x1,x2 

monod ---replSet myapp --dbpath /폴더지정 --port 27017 --bind_ip_all 

rs.initiate()

cls 

````yaml
FROM mongo
CMD ["mongod", "--replSet", "myapp"] 
````

mongo 접속 

```powershell
>docker build -t suhaw0325/mymongo:latest .
>docker run -p 27017:27017 suhaw0325/mymongo:latest
>docker exec -it f762f1315f8f bash
root@f762f1315f8f:/# mongo

> rs.initiate();
>cls -> secondary -> primary 로 
>  rs.initiate();
```

