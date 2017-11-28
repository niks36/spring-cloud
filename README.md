# Spring Cloud Config

Spring cloud config is client/server approach for storing & managing configuration across multiple application & env.

With spring cloud it is possible to manage central config server for all application & environment. and if any properties are changed, it will pick up the changes and reflect them without an application rebuild or restart.

# Architecture

### Repository Area:
The config server stores all the microservice properties in version control like Git, SVN or even on a file system as well.

The config server stores each microservice property based on serviceID. We can configure serviceID by setting spring.application.name in properties.
Properties name file format should be {serviceId}-{profile}.yml


### REST endpoint
Every microservice needs to communicate with config server so it can resolve property value. The config server publishes a REST endpoint through which microservices communicate, or we can see the properties in a browser.

### Actuator
If any properties has been changed, our microservice would get updated property without restarting application.

### Cloud bus
Its optional but useful. Its not good idea to refresh rest endpoint for every microservice, cloud bus helps to push the properties to all subscribed microservices.

### Load Balancer
Ideally, it should be load balanced so that we can run multiple instances of config servers, and the load balancer pool should have one public address where every microservice communicates.


# Config Server

Add following dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
    <version>1.1.2.RELEASE</version>
</dependency>
```

For config server, we need to annotate main class with @EnableConfigServer
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {

    public static void main(String[] arguments) {
        SpringApplication.run(ConfigServer.class, arguments);
    }
}
```

Add config server properties in application.properties.

```
server.port=8888
spring.cloud.config.server.git.uri=ssh://localhost/config-repo
spring.cloud.config.server.git.clone-on-start=true
```
For file system based repo add this
```
spring.cloud.config.server.native.searchLocations=file://path/to/repo
```

## Repo
Here we are adding repo which is based on file system.
CentalRepo has two files config.properties
```
welcome.message=Hello Spring Cloud
new.message=New Message
```
& config-prod.properties
```
welcome.message=Hello Spring Cloud For Production
prod.specific=Prod Specific
```

## Querying the Configuration
Now we are able to start our server, we can query by following path:
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

## Config Client
Add following maven dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>1.1.2.RELEASE</version>
</dependency>
```
Add following content in bootstrap.properties
```
spring.cloud.config.uri=http://localhost:8888
spring.cloud.config.username=root
spring.cloud.config.password=s3cr3t

```


### Example
Start Config Server & then hit below rest endpoint to verify

1) http://localhost:8888/config/default/master
```json
{
  "name": "config",
  "profiles": [
    "default"
  ],
  "label": "master",
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "file:///home/niket.shah/project/extra/repo/spring-cloud/CentralRepo/config.properties",
      "source": {
        welcome.message: "Hello Spring Cloud",
        new.message: "New Message"
      }
    }
  ]
}
```

2) http://localhost:8888/config-default.yml
```json
new:
  message: New Message
welcome:
  message: Hello Spring Cloud
```

3) http://localhost:8888/config-default.properties
```
new.message: New Message
welcome.message: Hello Spring Cloud
```

4) For Production profile
http://localhost:8888/config-prod.properties
```
new.message: New Message
prod.specific: Prod Specific
welcome.message: Hello Spring Cloud For Production
```

Here "welcome.message" is overridden by prod profile & "prod.specific" is new property added.