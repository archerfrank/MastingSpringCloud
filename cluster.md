``` CMD
java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=peer1 target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=peer2 target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-gateway-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-gateway-service-1.0-SNAPSHOT.jar
```

http://localhost:8761/

http://localhost:8765/api/client/ping

## sample-spring-cloud-config-master

```
java -jar -Xmx384m -Dspring.profiles.active=native target/sample-config-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=zone1 target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-client-service-1.0-SNAPSHOT.jar
```

http://localhost:8889/client-service-zone1.yml


## sample-spring-cloud-config-bus

```
java -jar -Xmx384m target/sample-config-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=zone1 target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-client-service-1.0-SNAPSHOT.jar
```

- update config 

https://github.com/archerfrank/sample-spring-cloud-config-repo/blob/master/client-service-zone1.yml
- check the content in 

 http://localhost:8889/client-service-zone1.yml

- check the application

http://localhost:8081/ping

- post to below to refresh

http://localhost:8081/refresh

## create rabbitmq

docker pull rabbitmq:management

docker run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:management

http://192.168.99.100:15672

docker stop rabbit
docker start rabbit

## sample-spring-cloud-config-bus-amqp

```
java -jar -Xmx384m sample-config-service/target/sample-config-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=zone1 sample-discovery-service/target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone1 sample-client-service/target/sample-client-service-1.0-SNAPSHOT.jar
```

update the git and post to 
http://localhost:8889/monitor

X-GitHub-Event: push
Content-Type: application/json

{"commits": [{"modified": ["client-service-zone1.yml"]}]}
