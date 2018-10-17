




# Tutorial

This tutorial is a mix of those two links:

Ref: https://www.javainuse.com/spring/cloud-data-flow
Ref: https://www.e4developer.com/2018/02/18/getting-started-with-spring-cloud-data-flow/

Is also inspired massively by [this presentation from the SCDF lead developer](https://www.slideshare.net/Pivotal/orchestrating-data-microservices-with-spring-cloud-data-flow).

My working directory where this repo has been cloned into:
    cd ~/datashare/dev/java/training/spring-cloud-data-flow-example/spring

## Database install

Install MySQL docker
    docker run --name dataflow-mysql -e MYSQL_ROOT_PASSWORD=dataflow -e MYSQL_DATABASE=scdf -p 3306:3306 -d mysql:5.7

## Message Bus install

Install RabbitMQ with docker
    docker run --name dataflow-rabbit -p 15672:15672 -p 5672:5672 -d rabbitmq:3-management

Access RabbitMQ UI from my laptop:
http://localhost:15672/#/

from my VM:
http://172.16.115.1:15672/#/


## SDCF server & Dashboard Spring Cloud UI

### Get the binaries

Download SCDF server binaries:

    wget https://repo.spring.io/libs-release/org/springframework/cloud/spring-cloud-dataflow-server-local/1.3.0.RELEASE/spring-cloud-dataflow-server-local-1.3.0.RELEASE.jar

### how to launch SDCF

    java -jar spring-cloud-dataflow-server-local-1.3.0.RELEASE.jar --spring.datasource.url=jdbc:mysql://localhost:3306/scdf --spring.datasource.username=root --spring.datasource.password=dataflow --spring.datasource.driver-class-name=org.mariadb.jdbc.Driver --spring.rabbitmq.host=127.0.0.1 --spring.rabbitmq.port=5672 --spring.rabbitmq.username=guest --spring.rabbitmq.password=guest

### How to access the Dashboard Spring Cloud UI

Now that the previous Java commmand was started, the Spring Cloud UI is available on port 9393:

from my laptop
http://localhost:9393/dashboard

from my VM
http://172.16.115.1:9393/dashboard



## Spring Shell

Download the Spring Shell
    wget http://repo.spring.io/milestone/org/springframework/cloud/spring-cloud-dataflow-shell/1.3.0.M1/spring-cloud-dataflow-shell-1.3.0.M1.jar

Start the Spring Shell
    java -jar spring-cloud-dataflow-shell-1.3.0.M1.jar

## Compile and generate project artifacts

```
cd source
mvn clean install
cd ../processor
mvn clean install
cd ../sink
mvn clean install
```

Your artifacts should now be in the local maven repo
i.e.
* maven://com.javainuse:source:jar:0.0.1-SNAPSHOT
* maven://com.javainuse:processor:jar:0.0.1-SNAPSHOT
* maven://com.javainuse:sink:jar:0.0.1-SNAPSHOT

This means they will be ready to be used by the SCDF server.

## Register project Apps and deploy on the SCDF server

```
app register --name source-app --type source --uri maven://com.javainuse:source:jar:0.0.1-SNAPSHOT
app register --name processor-app --type processor --uri maven://com.javainuse:processor:jar:0.0.1-SNAPSHOT
app register --name sink-app --type sink --uri maven://com.javainuse:sink:jar:0.0.1-SNAPSHOT
stream create --name log-data --definition 'source-app | processor-app | sink-app'
stream deploy --name log-data
```

## How to import SCDF starters provided by Spring

 Spring Cloud Stream App Starters is a project that provides a multitude of ready-to-go starter apps for building Streams. You can read from FTP, HTTP, JDBC, twitter and more, process and save to a multitude of sources.

 The following link can be used to bulk import the RabbitMQ + Maven flavour for the starters into SCDF:

    http://repo.spring.io/libs-release/org/springframework/cloud/stream/app/spring-cloud-stream-app-descriptor/Celsius.SR1/spring-cloud-stream-app-descriptor-Celsius.SR1.stream-apps-rabbit-maven


Notes:  the equivalent link for Kafka can be found [here](http://repo.spring.io/libs-release/org/springframework/cloud/stream/app/spring-cloud-stream-app-descriptor/Celsius.SR1/spring-cloud-stream-app-descriptor-Celsius.SR1.stream-apps-kafka-10-maven)


## Example of a SCDF starter example

```
stream create --name http-file --definition 'http --port=7171 | transform --expression=payload.toUpperCase() | file --directory=/home/mathieu/datashare/dev/java/training/spring-cloud-data-flow-example/tmp'
stream deploy --name http-file
```

### REST call to test the example

Post to http://localhost:7171 (REST call)
body:
```
Hello world
```

tail file:
    less /home/mathieu/datashare/dev/java/training/spring-cloud-data-flow-example/tmp/file-sink
