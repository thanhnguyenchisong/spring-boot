https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.spring-application.command-line-runner

Quited easy so you can read on this ducumentation to get detail

# Spring Boot
Easy to build the application so you just focus to buinesss, don't need to care so much to config.

## 1. Feature of spring
- Create stand-alone Spring applications.
- Embed Tomcat, Jetty or Undertow directly (No need to deploy WAR file)
- Provide 'starter' dependencies to simplify your build configuration.
- Automatically configure Spring and 3rd party libs.
- Provide production-ready features such as metrics, help check and externalized config
- No code generation and no requirement for XML config

From that features we have advantages
- Easy to understand and develop spring applications
- Increases productivity and reduces development time
- Minimum configuration
- Write anotaion, no need xml config
- No need to deploy war file.

## 2. Spring Boot key components
- auto-configuration
- CLI
- starter POMs
- actuators
  
## 3. Spring over spring
- Starter POM
- Version Management : Each release of Spring Boot provides a list of dependencies that it supports, don't need to provide a version for any of these dependencies in your build configuration.
- Component Scanning: scan package or bean
- Auto-configuration : scanning the classpath components and registering the beans with anotations.
- Embedded server
- in-memory db : there's full support for it in the Spring Boot ecosystem
- Actuators

## 4. Starter dependency of Spring Boot
- Data JPA starter
- Test Starter
- Security starter
- Web starter
- Mail starter
- Thymeleaf starter

Dependencies are needed to start a particular functionality

## 5. How to Spring Boot works
Automatically config your applications based on depenedencies which was added by annotation. Entry point is class that contain `@SpringBootApplication` anotation and main method.
Can scan all component by usign `@ComponentScan`

## 6. @SpringBootApplication anotation do ?
`@SpingBootApplication` anotation is equivalent to using `@Configuration` `@EnnableAutoConfiguration` and `@ComponentScan` with default attribute.

## 7. @ComponentScan
Scan all bean and package

## 8. How to spring boot application get started
like other Java program, main method call `SpringApplication.run`

## 9. Spring Initializer
It's a web to help you create a spring boot application - just down load and import to your IDE

## 10. Spring Boot CLI - benefits
It is command-line interface that allows you create a spring-based java application'
Never use before.

Good knowleage : https://www.interviewbit.com/spring-boot-interview-questions/#pros-of-using-spring-boot

# 11. Spring Actuator
Brings production-ready features to our application : monitoring our app, metrics, understanding traffic, state of database
Main benefit: get production-grade tools without having to actually implement these features ourselves.
Main target: expose perational information about the running application, use HTTP endpoints or JMX bean to interact with it

#### Technology support
Version 1.x It was tied to MVC so to the Servlet API
Version 2.x It wasn't tied to any technology (technology-agnostic), It defines models as pluggable and extensible wihout relying on MVC for it.
Technologies could be added by implementing the right adapter
JMX supported to expose endpoints without any additional code

### Changes ver 1x and ver 2.x
Ver 2.x Actuator comes with most endpoints disable. thus, only 2 available default are /health  and /info
If we want to ennable all of them, can set management `endpoint.web.exposure.include=*`, alternatively, we can list endpoints that should be enable.

Actuator now shares the security config with regular App security rules.
Tweak Actuator security rules - add `/actuator/**`
```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .pathMatchers("/actuator/**").permitAll()
      .anyExchange().authenticated()
      .and().build();
}
```
all Actuator endpoints are now placed under the /actuator path

Or we can tweak path by config in properties file: `management.endpoints.web.base-path = ...`

### Enpoints
- `/auditevents` lists security audit-related events such as user login/logout. Also can filter by pricipal or type among ther fields
- `/beans` all available beans in our BeanFactory and doesn't support filtering
- `/conditions` formerly known as `/autoconfig`, builds a report of condition around autoconfiguration
- `/configprops` allows us to fetch all `@ConfigurationProperties` beans.
- `/env` return the current environment properties and can retrieve single properties
- `flyway` provides details about or Flyway databse migrations
- `/health` summarizes the health status of our application
- `/heapdump` build and return a heap dump from the JVM used by our application
- `/info` return genral information, it can be custom data, build information or detail about the lastest commit.
- `/liquibase` bahaves like `flyway` but for Liquibase
- `/logfile` return orfinary application logs.
- `/loggers` enable us to query an mofigy the logging level of our application
- `/metric` detail metric like the previous one, formatted to work with a Prometheus server
- `/scheduledtasks` provides detail about every scheduled task within our application
- `/sessions` lists HTTP sessions given we are using Spring Session
- `/shutdown` perform a graceful shutfown or the application
- `/threaddump` dumps the thread information of underlying JVM


To check how many enabled endpoints in your application we can call the base path  `/actuator` - in the case you congig another base path by `management.endpoints.web.base-path` you should request to `/{your base path}`
 ```json
 {
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    "features-arg0": {
      "href": "http://localhost:8080/actuator/features/{arg0}",
      "templated": true
    }, ...
 ```
 if `management.endpoints.web.base-path = /` then the discovery enpoind be disabled

 ### Health Indicator custom
 Custom the implementations of `ReactiveHealthIndicator` interface.

 ### Health Groups
 Organizing health indicators into `groups` and apply the same configuration to all group members.
 `management.endpoint.health.group.custom.include=diskSpace,ping` : custom goup contains diskSpace and ping health indicators
if we send a request to `/actuator/health/custom` the response `{"status":"UP"}`

To see detail:
`management.endpoint.health.group.custom.show-components=always`
`management.endpoint.health.group.custom.show-details=always`

or show these details only for authrized user
`management.endpoint.health.group.custom.show-components=when_authorized`
`management.endpoint.health.group.custom.show-details=when_authorized`

We also have a custom status mapping:
`management.endpoint.health.group.custom.status.http-mapping.up =200`

### Metrics in Spring Boot 2
`Micrometer` support
We interact with `Micrometer` directly, we get bean of MeterRegistry type which is autoconfigured for us.
`Mictomter` is a part of Actuator's dependencies.
Response from metric endpoint
```json
{
  "names": [
    "jvm.gc.pause",
    "jvm.buffer.memory.used",
    "jvm.memory.used",
    "jvm.buffer.count",
    // ...
  ]
}
```
To get the actual valie fo specific metric, we can now navigate to desired metric example `/actuator/metrics/jvm.gc.pause`, from that we can get detail response.

### Customizing the /info Endpoint
Add git details using the repective Maven dependency
```xml
<dependency>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
</dependency>
```
Add the name, group, version using Maven pluin
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>build-info</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Create custom Endpoint
```java
@Component
@Endpoint(id = "features")
public class FeaturesEndpoint {

    private Map<String, Feature> features = new ConcurrentHashMap<>();

    @ReadOperation
    public Map<String, Feature> features() {
        return features;
    }

    @ReadOperation
    public Feature feature(@Selector String name) {
        return features.get(name);
    }

    @WriteOperation
    public void configureFeature(@Selector String name, Feature feature) {
        features.put(name, feature);
    }

    @DeleteOperation
    public void deleteFeature(@Selector String name) {
        features.remove(name);
    }

    public static class Feature {
        private Boolean enabled;

        // [...] getters and setters 
    }

}
```
When call the discovery endpoint - It will be display as a feture of actuator.
`@ReadOperation`: It'll map to HTTP GET.
`@WriteOperation`: It'll map to HTTP POST.
`@DeleteOperation`: It'll map to HTTP DELETE.

### Extending Existing Endpoints
Extending behavior o a predefined endpoint using the `@EndpointExtension` annotations or more concrete specialization `@EndpointWebExtension` or `@EndpointJmxExtention`
