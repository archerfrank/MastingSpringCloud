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


## MQ Services inter service call.

* In the master repo
    1. Start MQ
    ```
    docker run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:management
    docker start rabbit
    docker stopt rabbit
    ```

    MQ address http://192.168.99.100:15672

    2. Start product, account and order service, and run the test case in order service. 
    3. add a listener function in the order application to read the output from account service.

* In Pubsub repo.
    1. check the application.xml, it support the consumer group and partition. Without partition, you don't know which one in the group will handle the message.
    2. One message from order will be consumed by both account and product service.

    3. Both service will send back the message to order.

```
-Dspring.profiles.active=zone1 
```

##  Docker

1. Common command
```
docker rmi $(docker images -q -f dangling=true)   ----Remove all the dangling images.

docker rm -f containerName

mvn package -DskipTests

docker build -t archerfrank/order-service:1.0 .
docker build -t archerfrank/customer-service:1.0 .
docker build -t archerfrank/account-service:1.1 .
docker build -t archerfrank/product-service:1.0 .
docker build -t archerfrank/discovery-service:1.0 .
docker login --username=archerfrank
docker push archerfrank/discovery-service:1.0
docker push archerfrank/account-service:1.1
docker push archerfrank/product-service:1.0
docker push archerfrank/customer-service:1.0
```

2. To run the containers
```
docker network create sample-cloud

docker run -d --rm --name account -p 8091:8091 -e EUREKA_DEFAULT_ZONE=http://discovery:8761/eureka -m 256M  --network sample-cloud archerfrank/account-service:1.0

docker run -d --rm --name customer -p 8092:8092 -e EUREKA_DEFAULT_ZONE=http://discovery:8761/eureka -m 256M --network sample-cloud archerfrank/customer-service:1.0

docker run -d --rm --name order -p 8090:8090 -e EUREKA_DEFAULT_ZONE=http://discovery:8761/eureka -m 256M --network sample-cloud archerfrank/order-service:1.0

docker run -d --rm --name product -p 8093:8093 -e EUREKA_DEFAULT_ZONE=http://discovery:8761/eureka -m 256M --network sample-cloud archerfrank/product-service:1.0

docker run -d --rm --name discovery -p 8761:8761 --network sample-cloud archerfrank/discovery-service:1.0
```
3. Visit http://192.168.99.100:8761

 * Visit http://192.168.99.100:8092/withAccounts/1

```
docker stats --format "table {{.Name}}\t{{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

4. Build with Maven

Go into each folder, you could run this command.

```
mvn clean install docker:build -DskipTests

```

# Jenkins


# Kubernate

1. install minikube https://yq.aliyun.com/articles/221687 or https://blog.csdn.net/yyqq188/article/details/88019467 

* Start the minikube

        ```
        minikube start
        minikube start --registry-mirror=https://docker.mirrors.ustc.edu.cn
        minikube dashboard
        minikube ip
        minikube stop

        kubectl version
        kubectl get nodes
        kubectl cluster-info
        ```

2. Run the applications in the minikube.

* Run single application

    ```
    kubectl run discovery --image=archerfrank/discovery-service:1.0 --port=8761
    kubectl get pods -o wide
    ```
* Useful command
    * create deployment to deploy account and customer and discovery.
    * check the logs.
    * pull the image, if it is to slow.
    * Use IP to get the cloud service, instead of the DNS name.

    ```
    kubectl get svc 
    kubectl logs account-service-6465c6dc6d-2597l
    kubectl logs customer-service-5f769dc486-jxgj6
    kubectl exec customer-service-5f769dc486-slrb4 -c customer-service -- curl http://172.17.0.2:8091/customer/1
    kubectl delete pod discovery-56cc6f84f5-rfbdq
    kubectl describe pod discovery-56cc6f84f5-rfbdq
    kubectl expose deployment discovery --type=NodePort

    kubectl apply -f discovery.yaml
    kubectl apply -f nodeport.yaml
    kubectl apply -f product.yaml
    kubectl apply -f account-service.yaml
    kubectl port-forward account-service-6465c6dc6d-2597l 8091:8091
    kubectl port-forward customer-service-5f769dc486-jxgj6 8092:8092

    kubectl delete deployment account-service
    kubectl delete pod customer-service-5f769dc486-slrb4
    docker pull archerfrank/product-service:1.0
    docker pull archerfrank/account-service:1.1      1.1 version is the preferIpAddress Version.

    curl http://10.100.247.206:8091/customer/1
    curl http://account-service:8091/customer/1

    kubectl apply -f account.yaml
    kubectl apply -f customer.yaml
    ```

3. Visit http://192.168.99.101:32000/ to find the discovery service.

4. After port forward, visit http://127.0.0.1:8091/1, http://127.0.0.1:8092/withAccounts/1 or we could create a nodeport service. **Caution**, the port forward is very slow.

        ```
        kubectl apply -f nodeport-customer.yaml
        ```
Then visit http://192.168.99.100:32001/withAccounts/1.

5.  Use ingress servcie as gateway.

    ```
    kubectl get pods -o wide
    kubectl get svc 
    minikube addons list
    minikube addons enable ingress
    kubectl apply -f gateway.yaml
    kubectl get endpoints
    kubectl get ing
    kubectl get pods -n kube-system

    kubectl describe ing  gateway-ingress

    ```
    * Adjust the memory to 3.5 GB for the virtual machine and visit http://archerfrank.com/customer/withAccounts/1. 
    
    * Please also check https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/rewrite/README.md , the rewrite logic.


    ## sample-spring-boot-web
Run the application, use the dev tool, which could hot load the changes without restart the application. 

http://localhost:2222/person 
http://localhost:2222/swagger-ui.html#/ 


## sample-spring-cloud-netflix-master

Start the server first.
Check app at 
http://localhost:8761/ 

The start the client using cmd.

```
java -jar -DPORT=8083 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar -DPORT=8082 target/sample-client-service-1.0-SNAPSHOT.jar

java -jar -DSEQUENCE_NO=2 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar -DSEQUENCE_NO=3 target/sample-client-service-1.0-SNAPSHOT.jar
```
we could use port or sequence no.
POST to http://localhost:8083/shutdown 

## sample-spring-cloud-netflix-cluster

Start the server first.
Check app at 
http://localhost:8761/ 

```
java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-client-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-client-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=zone3 target/sample-client-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=peer1 target/sample-discovery-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=peer2 target/sample-discovery-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=peer3 target/sample-discovery-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=zone1 target/sample-gateway-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=zone2 target/sample-gateway-service-1.0-SNAPSHOT.jar

java -jar -Xmx192m -Dspring.profiles.active=zone3 target/sample-gateway-service-1.0-SNAPSHOT.jar
```

localhost:8765/api/client/ping


## sample-spring-cloud-comm-master

```
java -jar -Xmx384m account-service/target/account-service-1.0-SNAPSHOT.jar

java -jar -Xmx384m customer-service/target/customer-service-1.0-SNAPSHOT.jar

java -jar -Xmx384m product-service/target/product-service-1.0-SNAPSHOT.jar

java -jar -Xmx384m order-service/target/order-service-1.0-SNAPSHOT.jar

java -jar -Xmx384m gateway-service/target/gateway-service-1.0-SNAPSHOT.jar
```

* account service  http://localhost:8091/customer/1

* customer-service http://localhost:8092/withAccounts/1

* product-service http://localhost:8093/3

* order-service http://localhost:8090/   only post

* apigateway 
    * http://localhost:8080/account/customer/1
    * http://localhost:8080/customer/withAccounts/2
    * http://localhost:8080/product/4
    * http://localhost:8080/order/


## sample-spring-cloud-comm-hystrix

0. **spring-boot-starter-actuator** need to add to all the applicatons
1. import the projects into the eclipse;
2. Run the applications. The account server is config at both 8091 and 9091, but it only start at 8091. Product service only start at 8093
    * account service  http://localhost:8091/customer/1

    * customer-service http://localhost:8092/withAccounts/1

    * product-service http://localhost:8093/3

    * order-service http://localhost:8090/

3. Run the costomer service test case, you could get if remote service fails, the data is from fallback. If the circuit breaker is open, the data is all from cache. 

4. Use BestAvailableRule(Always use available client) or RoundRobinRule, the result is different

5. Monitor the stream of the remote call. Start the Hystrix Dashboard with all other service. Visit http://localhost:9000/hystrix. Input the endpoint, for example http://localhost:8090/hystrix.stream. Run the order service test case, and we can see the result.


## sample-spring-cloud-comm-turbine
1. The same as before start all the applications including the service discovery.
2. Check all the service at http://localhost:8761/
3. start hystrix-dashboard, and check http://localhost:9000/turbine.stream
4. add the http://localhost:9000/turbine.stream to the http://localhost:9000/hystrix
5. run the test case in the order service and check the result. We can also terminate account service and see the change in the report.



## GateWay
1. start up all the service, and could check the routes http://localhost:8080/routes or http://localhost:8080/routes?format=details 

```
in application.yml
management:
  security:
    enabled: false

in pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. http://localhost:8080/filters
Filter is like the filter in the servlet, it could modify the request header or extra functions.


## Log with Zipkin

Use the project in Chapter 6 sample-spring-cloud-comm-feign.
Below dependency is required to generate the traceID
```
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```
The zipkin service could register in the discovery service.

1. start discovery 
2. start zipkin and check url http://localhost:8761/ .
3. start other applications, product, account, customer and order.
4. visit http://localhost:9411/ to check the traces, please click the Find Traces button.
5. you could see the depenencies and you could click them.
6. sample is using the default port and address of zipkin, this could be reset by spring.zipkin.baseUrl in application XML.

* Zipkin could be integrated by MQ as well. Client needs to use spring-cloud-starter-zipkin and amqp.

## Security
### sample-spring-cloud-security-master

1. Create keystore for discovery and other applications.
2. All other applications .cer files need to be imported into discovery.jks keystore.
3. discovery.cer need to be imported into other applications. Below are some command to generate the keys.
```
keytool -genkey -alias order -storetype JKS -keyalg RSA -keysize 2048 -keystore order.jks -validity 3650
You need to input the password.

keytool -exportcert -alias order -keystore order.jks -file order.cer
keytool -importcert -alias order -keystore discovery.jks -file order.cer
keytool -importcert -alias discovery -keystore order.jks -file discovery.cer

keytool -importkeystore -srckeystore discovery.jks -srcstoretype JKS -deststoretype PKCS12 -destkeystore discovery.p12

keytool -export -alias discovery -file discovery.cer -keystore discovery.jks

List all the keys
keytool -v -list -keystore discovery.jks
```
4. Current setting is that the discovery service should verify all the applications, but applications don't verify each other requests.  JKS is keystore, which contains both public key of others and the private key of the application its own. If we add **_client-auth: need_** into the account application, then we can't using our browser to access the service. 

5. To check https://localhost:8761/ and avoid the error, you have to create one new key to identify your browser. 

```
keytool -genkey -alias browser -storetype JKS -keyalg RSA -keysize 2048 -keystore browser.jks -validity 3650
keytool -exportcert -alias browser -keystore browser.jks -file browser.cer
keytool -importcert -alias browser -keystore discovery.jks -file browser.cer

keytool -importkeystore -srckeystore browser.jks -srcstoretype JKS -deststoretype PKCS12 -destkeystore browser.p12
```

Import browser.cer into the discovery.jks, then the discovery service will trust this public key. Import browser.p12 to browser, this is the private key, because the service has trust the public key, so you could visit the https://localhost:8761/.

6. To access the account application again, we need to import browser.cer into account.jks.

```
keytool -importcert -alias browser -keystore account.jks -file browser.cer
```

7. Refer to this article https://piotrminkowski.wordpress.com/2018/05/21/secure-discovery-with-spring-cloud-netflix-eureka/

## Security Config
### sample-spring-cloud-security-secureconfig
1. After import the cer into discovery.jks in config-service, please visit https://localhost:8888/account-service.yml

2. Use fiddler to post
```
https://localhost:8888/encrypt

User-Agent: Fiddler
Host: localhost:8888
Content-Length: 6
Content-Type: text/html; charset=utf-8

Body
123456

This is to use the discovery public key to encrypt.
encrypt:
  keyStore:
    location: classpath:/discovery.jks
    password: 123456
    alias: discovery
    secret: 123456
```
https://localhost:8888/decrypt

3. put the discovery.jks in account resource and use below to decrypt the cipher. Need to add them into VM arguement

```
-Dencrypt.keyStore.location=classpath:/discovery.jks -Dencrypt.keyStore.password=123456 -Dencrypt.keyStore.alias=discovery -Dencrypt.keyStore.secret=123456
```

## OAuth2

```
security:
  basic:
    enabled: false
  oauth2:
    client:
      clientId: piotr.minkowski
      clientSecret: 123456
      accessTokenUri: http://localhost:8088/oauth/token
      userAuthorizationUri: http://localhost:8088/oauth/authorize
    resource:
      userInfoUri: http://localhost:8088/user
```

1. Start both auth service and account service
2. Visit http://localhost:8091/customer/1 
3. You need to input user name and password.
http://localhost:9999/oauth/authorize?response_type=token&client_id=piotr.minkowski&redirect_uri=http://abc.com&scope=read

4. Start TCP monitor in eclipse to mapping 8088 to 9999, then to check the request from account service to auth service.
    - visit http://localhost:8091/customer/1 should redirect to the _/login_, then to auth service _/oauth/authorize_
    ```
    http://localhost:8088/oauth/authorize?client_id=piotr.minkowski&redirect_uri=http://localhost:8091/login&response_type=code&state=h6EzzJ
    ```
    - with the code from auth service, the browser redirect to the account login url. The state is the link to the original url.
    ```
    http://localhost:8091/login?code=nZKlKG&state=h6EzzJ
    ```
    - After login the account app should verfiy the code with auth using /oauth/token with the code nZKlKG 
    - If the code is correct, account service then request resource by /user to get the user information.

## SSO with Gateway

1. Don't use account service, please start product, discovery, gateway and auth service

2. Make sure gateway and product can registed in discovery. 

3. The Gateway main function add, this is required to connect to discovery.

```
	@Bean
	public DiscoveryClient.DiscoveryClientOptionalArgs discoveryClientOptionalArgs() throws NoSuchAlgorithmException {
		DiscoveryClient.DiscoveryClientOptionalArgs args = new DiscoveryClient.DiscoveryClientOptionalArgs();
		System.setProperty("javax.net.ssl.keyStore", "src/main/resources/account.jks");
		System.setProperty("javax.net.ssl.keyStorePassword", "123456");
		System.setProperty("javax.net.ssl.trustStore", "src/main/resources/account.jks");
		System.setProperty("javax.net.ssl.trustStorePassword", "123456");
		EurekaJerseyClientBuilder builder = new EurekaJerseyClientBuilder();
		builder.withClientName("account-client");
		builder.withSystemSSLConfiguration();
		builder.withMaxTotalConnections(10);
		builder.withMaxConnectionsPerHost(10);
		args.setEurekaJerseyClient(builder.build());
		return args;
	}
```

4. Start TCP monitor 8088->9999, and comments out some uncessary security code in product configration file. Only use **@EnableResourceServer** in the product service. Only _userInfoUri_ should config in the application.xml
```
security:
  user:
    name: piotr.minkowski
    password: 123456
  oauth2:
    resource:
#      loadBalanced: true
      userInfoUri: http://localhost:8088/user
```

5. Test with below URL.
http://localhost:8093/3
http://localhost:8080/api/product/1

6. Reference https://github.com/piomin/sample-spring-oauth2-microservices/tree/advanced 






