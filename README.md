# example-spring-boot

This is an example of Spring Boot [Getting Started](https://github.com/spring-guides/gs-spring-boot/tree/c42d4edfec8e704431380b884f5cfed78f17e876/initial) modified to work on OpenJDK CRaC.

Changes: https://github.com/CRaC/example-spring-boot/compare/base..crac

For more info, check:

* [What is CRaC?](https://docs.azul.com/core/crac/crac-introduction)
* [CRaC Usage Guidelines](https://docs.azul.com/core/crac/crac-guideline)

## Building

Use maven to build
```
mvn package
```

## Running

Please refer to [README](https://github.com/CRaC/docs#users-flow) for details.

### Preparing the image
1. Run the [JDK](README.md#JDK) in the checkpoint mode
```
$JAVA_HOME/bin/java -XX:CRaCCheckpointTo=cr -jar target/example-spring-boot-0.0.1-SNAPSHOT.jar
```
2. Warm-up the instance
```
siege -c 1 -r 100000 -b http://localhost:8080
```
3. Request checkpoint
```
jcmd target/example-spring-boot-0.0.1-SNAPSHOT.jar JDK.checkpoint
```

### Restoring

```
$JAVA_HOME/bin/java -XX:CRaCRestoreFrom=cr
```

## Running with docker

```
docker build -t zulu-crac .

./mvnw clean package
java -jar target/example-spring-boot-0.0.1-SNAPSHOT.jar
curl -i http://localhost:8080

docker run --privileged -it --rm -p 8080:8080 -v $PWD:/opt/mnt --name example-spring-boot-crac zulu-crac bash

echo 128 > /proc/sys/kernel/ns_last_pid
java -XX:CRaCCheckpointTo=/opt/mnt/crac -jar /opt/mnt/target/example-spring-boot-0.0.1-SNAPSHOT.jar

docker exec -it example-spring-boot-crac bash
jcmd 129 JDK.checkpoint

java -XX:CRaCRestoreFrom=/opt/mnt/crac

docker build -t example-spring-boot-crac-restore -f Dockerfile.restore .
docker run -it --rm -p 8080:8080 --name example-spring-boot-crac-restore example-spring-boot-crac-restore
```
