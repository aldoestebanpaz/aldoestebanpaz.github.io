- [Spring Boot basics](#spring-boot-basics)
  - [How to build and run](#how-to-build-and-run)
    - [Example steps of building and running a maven project](#example-steps-of-building-and-running-a-maven-project)
  - [How to debug](#how-to-debug)
  - [Features](#features)
    - [Console apps](#console-apps)
    - [Field Injection: @Autowired vs @Inject](#field-injection-autowired-vs-inject)
    - [Field Injection vs Constructor Injection](#field-injection-vs-constructor-injection)
  - [Reference](#reference)

# Spring Boot basics

## How to build and run

It is possible to run the application (console or web) from the command line with Gradle or Maven.

The alternative is first building the single executable JAR file that contains all the necessary dependencies, classes, and resources and then run that. Building an executable jar makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

**Using Gradle**

If you use Gradle, you can run the application by using `./gradlew bootRun`. Alternatively, you can build the JAR file by using ./gradlew build and then run the JAR file, as follows:

```sh
java -jar build/libs/gs-consuming-rest-0.1.0.jar
```

**Using maven**

If you use Maven, you can run the application by using `./mvnw spring-boot:run`. Alternatively, you can build the JAR file with ./mvnw clean package and then run the JAR file, as follows:

```sh
java -jar target/gs-consuming-rest-0.1.0.jar
```

### Example steps of building and running a maven project

1. Open a terminal and `cd` into the project directory.
2. Update 'src/main/resources/application.properties' and all the necessary .properties files before to run the applications.
3. Make sure you are pointing to the correct target version of Java:

- On git-bash shell:

```sh
export JAVA_HOME="C:\\Program Files\\Java\\jdk-11.0.4"
export PATH="$JAVA_HOME\\bin":$PATH
```

- On Windows Powershell:

```powershell
$env:JAVA_HOME = "C:\Program Files\Java\jdk-11.0.4"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
```

4. Build and run the project:

- Using the Spring Boot plugin:

```sh
mvn clean spring-boot:run
```

- Packaging and running the .jar file manually:

```sh
mvn clean package; java -jar target/<generated-file>.jar
```

## How to debug

By default, the `spring-boot:run` Maven goal runs the application in a forked process and setting properties on the command-line will not affect the application.

If you need to debug it, you should add the necessary JVM arguments to enable remote debugging. The following command suspend the process until a debugger has joined on port 5005:

```sh
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"
```

Another simple approach is just diabling the `fork` flag:

```sh
mvn spring-boot:run spring-boot:run -Dspring-boot.run.fork=false
```

NOTE: Disabling forking will disable some features such as an agent, custom JVM arguments, devtools or specifying the working directory to use.

There is also explicit support for [system properties](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#run.examples.system-properties) and [environment variables](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#run.examples.environment-variables).

## Features

### Console apps

To create a console app using Spring Boot, three configuration steps are required:
* The @SpringBootApplication annotation in the Application class.
* At least one CommandLineRunner implementation bean should be present.
* The `spring.main.web-application-type=NONE` property to not run as a web application.

Spring's @SpringBootApplication annotation is added in the main class to enable auto-configuration.

CommandLineRunner is a simple Spring Boot interface with a run method. Spring Boot will automatically call the run method of all beans implementing this interface after the application context has been loaded.

Finally, to disable running the app as a web application, we have to configure it with one of the following snippets:

Application Properties (application.properties)

```
spring.main.web-application-type=NONE # REACTIVE, SERVLET
```

or SpringApplicationBuilder

```java
  @SpringBootApplication
  public class MyApplication {
      public static void main(String[] args) {
          new SpringApplicationBuilder(MyApplication.class)
              .web(WebApplicationType.NONE) // .REACTIVE, .SERVLET
              .run(args);
     }
  }
```

### Field Injection: @Autowired vs @Inject

@Autowired is Spring's own annotation. @Inject (javax.inject.Inject) is part of a Java technology called CDI that defines a standard for dependency injection similar to Spring. In a Spring application, the two annotations works the same way as Spring has decided to support some JSR-299 annotations in addition to their own.

### Field Injection vs Constructor Injection

Constructor Injection is actually recommended over Field Injection, and has several advantages:

* The dependencies are clearly identified. There is no way to forget one when testing, or instantiating the object in any other circumstance (like creating the bean instance explicitly in a config class).
* The dependencies can be final, which helps with robustness and thread-safety.
* You don't need reflection to set the dependencies. InjectMocks is still usable, but not necessary. You can just create mocks by yourself and inject them by simply calling the constructor.

See [Why field injection is evil, by Oliver Drotbohm](https://odrotbohm.de/2013/11/why-field-injection-is-evil/) for details.

## Reference

- [Spring Boot Maven Plugin Documentation](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)
