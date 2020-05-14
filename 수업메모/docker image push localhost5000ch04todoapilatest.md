docker image push localhost:5000/ch04/todoapi:latest 

curl -s -XGET http://localhost:8080/todo?status=TODO

curl -XPOST -D '{"title": "4장 집필하기", "content" : "내용 검토 중"}' http://loccalhost:8080/todo