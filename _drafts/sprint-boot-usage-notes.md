# Spring Boot basics

## How to build and run

1. Open a terminal and `cd` into the project directory.
2. Update 'src/main/resources/application.properties' and all the necessary .properties files before to run the applications.
2. Make sure you are pointing to the correct target version of Java:

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

3. Build and run the project:

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

## Reference

- [Spring Boot Maven Plugin Documentation](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)
