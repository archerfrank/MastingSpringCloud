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