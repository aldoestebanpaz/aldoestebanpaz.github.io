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
