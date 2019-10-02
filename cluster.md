## sample-spring-cloud-netflix-cluster
Setup one gateway, one client and one service discovery. with different zone setting. we could use --- to separate the config files. This 
``` CMD
java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=peer1 target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=peer2 target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-gateway-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-gateway-service-1.0-SNAPSHOT.jar
```

http://localhost:8761/    service discovery site

http://localhost:8765/api/client/ping     test URL

## sample-spring-cloud-config-master
get the config from the config server, the client only need one bootstrap config, not application config.
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

```
docker pull rabbitmq:management
```
```
docker run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:management
```

* MQ address http://192.168.99.100:15672


``` rabbit
docker star
docker stopt rabbit
```
## sample-spring-cloud-config-bus-amqp

```
java -jar -Xmx384m sample-config-service/target/sample-config-service-1.0-SNAPSHOT.jar
java -jar  -Dspring.profiles.active=zone1 sample-discovery-service/target/sample-discovery-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m -Dspring.profiles.active=zone1 sample-client-service/target/sample-client-service-1.0-SNAPSHOT.jar
```

* update the git and post to 
http://localhost:8889/monitor

* X-GitHub-Event: push
Content-Type: application/json
 {"commits": [{"modified": ["client-service-zone1.yml"]}]}

 * Then the config service will put one event to the MQ and the clients will get the event and refresh the config.

## sample-spring-cloud-comm-feign

mvn package -DskipTests

run all the applications in the consoles

```
start java -jar  discovery-service/target/discovery-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m  -Dspring.profiles.active=zone1 account-service/target/account-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m  -Dspring.profiles.active=zone1 -Dserver.port=18091 account-service/target/account-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m  -Dspring.profiles.active=zone1 customer-service/target/customer-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m  -Dspring.profiles.active=zone1 product-service/target/product-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m  -Dspring.profiles.active=zone1 order-service/target/order-service-1.0-SNAPSHOT.jar
java -jar -Xmx192m  gateway-service/target/gateway-service-1.0-SNAPSHOT.jar
```

http://localhost:8092//withAccounts/1

Run test case in the eclipse

1. import both projects order service and gateway service in the Eclipse.
2. run the test case in both projects order service and gateway service.
3. sample post {"id":null,"status":null,"price":0,"customerId":3,"accountId":null,"productIds":[5,7]}


## sample-spring-cloud-comm-weighted-lb
Import 4 projects into eclipse
Run the test case in the customer service.
you could change the the setting add one more 10091 and run test
```
listOfServers: account-service:8091, account-service:9091, account-service:10091
```
The result is TEST RESULT: 8091=34, 9091=39, 10091=27

Modify the class *RibbonConfiguration*, change it from WeightedResponseTimeRule to AvailabilityFilteringRule or BestAvailableRule

AvailabilityFilteringRule
TEST RESULT: 8091=48, 9091=48, 10091=4

BestAvailableRule
TEST RESULT: 8091=100, 9091=0, 10091=0


## Logging distributed.
sample-spring-cloud-comm-feign
```
git clone https://github.com/piomin/sample-spring-microservices.git
```
* Follow the instruction in 
https://piotrminkowski.wordpress.com/2017/04/05/part-2-creating-microservices-monitoring-with-spring-cloud-sleuth-elk-and-zipkin/

https://piotrminkowski.wordpress.com/2019/05/07/logging-with-spring-boot-and-elastic-stack/

```
docker pull logstash:6.7.2
docker pull kibana:6.7.2

docker network create es

docker run -d --rm --name elasticsearch --net es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:6.7.2

**bin\logstash.bat -f logstash.conf**

docker run -d --rm --name kibana --net es -e "ELASTICSEARCH_URL=http://elasticsearch:9200" -p 5601:5601 kibana:6.7.2

docker run -d --rm --name  logstash --net es -p 5000:5000 -v /c/Users/Administrator/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf logstash:6.7.2

docker start elasticsearch
docker start logstash
docker start kibana

docker stop elasticsearch
docker stop logstash
docker stop kibana

docker rm kibana
docker rm elasticsearch
docker container ls
docker logs -f logstash

docker exec -it logstash ls config-dir
docker exec -it logstash ls /usr/share/logstash/pipeline

**bin\logstash.bat -f logstash.conf**
```

* Start account service, because I used local logstash, not docker, need to change the logstash socket to localhost:5000
* Kibana : http://192.168.99.100:5601

http://localhost:8091/customer/1

* Start customer and gateway service.
http://localhost:8080/api/customer/withAccounts/1. Then you could search traceID in the ELK.

## MQ with ELK
1. Modify the produce service in sample-spring-cloud-comm-feign to use MQ
2. start mq in docker 
```
docker pull rabbitmq:management
docker run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:management
docker start rabbit
docker stop rabbit
```
* MQ address http://192.168.99.100:15672, guest, guest;
Create one queue q_logstash, create one exchange called ex_logstash, create the binding between them;

3. start the ELK, like previous one, but we need to update the logstash config.
```
**bin\logstash.bat -f logstash-mq.conf**
```
4. Start product service application with zone1 and visit http://localhost:8093/1 and search in the ELK.

## Consul as service discovery

```
docker run -d --name consul -p 8500:8500 consul
docker start consul
```
http://192.168.99.100:8500
1. use the repo sample-spring-cloud-consul-master


 

