---
title: "Maven commands for analysis and troubleshooting"
tags: maven troubleshooting
---

# Maven commands for analysis and troubleshooting

## A basic exlanation about how to call plugin goals from command line

'Plugin Prefix Resolution' is a feature which enables to invoke goals of Maven plugins in the terminal by using its prefix (e.g. `mvn prefix:goal`) instead of using the fully-featured form `mvn groupId:artifactId:version:goal` or `mvn groupId:artifactId:goal`.

By default, Maven will make a guess at the plugin-prefix to be used, by removing any instances of "maven" or "plugin" surrounded by hyphens in the plugin's artifact ID. The conventional artifact ID formats to use are:

- `maven-${prefix}-plugin` - for official plugins maintained by the Apache Maven team itself (you must NOT use this naming pattern for your plugins).
- `${prefix}-maven-plugin` - for plugins from other sources

If a plugin artifactId fits this pattern, Maven will automatically map your plugin to the correct prefix in the metadata stored within the specified plugin groupId path on the repository.

By default, if not groupId is specified in the command, in the pom.xml file, or in a Maven settings file (per-user: `${user.home}/.m2/settings.xml`; global: `${maven.home}/conf/settings.xml`), Maven will search the groupId 'org.apache.maven.plugins' for prefix-to-artifactId mappings for the plugins it needs to execute the command.

Examples:

- When you directly invoke `mvn clean`, resolves the 'clean' plugin prefix to the 'maven-clean-plugin' artifactId, and
it resolves to the full name 'org.apache.maven.plugins:maven-clean-plugin:clean'.

- Spring Boot projects by default includes the 'org.springframework.boot:spring-boot-starter-parent' parent POM and the
'org.springframework.boot:spring-boot-maven-plugin' plugin in the pom.xml file. Having that plugin specified in the POM
file, you can make calls like `mvn spring-boot:run` for running the application, `mvn spring-boot:help -Ddetail=true -Dgoal=<goal-name>` to display help information, or create the .jar file with the standard `mvn package` that implicitly calls the 'spring-boot:repackage' because it is already configured to be executed with the 'package' phase in the parent POM. Reference: https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals.

- Without necessarily having anything configured in pom.xml, `mvn io.github.git-commit-id:git-commit-id-maven-plugin:revision` generates the priperties file 'target\classes\git.properties' with values from the git repository that could be useful in pom.xml for other configurations. Reference: https://github.com/git-commit-id/git-commit-id-maven-plugin.

Reference: https://maven.apache.org/guides/introduction/introduction-to-plugin-prefix-mapping.html

## Create a project from project templates (archetypes)

An archetype is a Maven project templating toolkit that allows to create and initialize a project form a template.

You can search archetype templates by running `mvn archetype:generate` and interacting with the prompt.

Reference:
- https://maven.apache.org/archetype/maven-archetype-plugin/
- https://maven.apache.org/archetype/index.html

### Creating a simple JAR project

The following creates a basic project using the 'maven-archetype-quickstart' and initializes the pom.xml file with the values 'com.aldo.core.tools' for groupId and 'javatool' for artifactId.

```sh
mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.aldo.core.tools -DartifactId=javatool -DinteractiveMode=false
```

This project can be compiled using `mvn clean package` and you can see the content of the archive running for example `jar -tvf target/javatool.jar`, but to be able to execute the program you will need to specify the classpath and the mainclass with a command like `java -cp target/javatool.jar <file-path-to-dependencies> com.aldo.core.tools.javatool.App <args>`.

To be able to use a shorter command like `java -jar target/javatool.jar <args>` for execution the project, you need to include a manifest file and include all the third-party dependencies inside the same JAR. This type of archive with every information and dependencies inside is commonly known as an 'assembly'.

### Creating assemblies

The 'maven-assembly-plugin' plugin allows you to combine project output into a single distributable archive that also contains dependencies, modules, site documentation, and other files.

A framework that creates this type of self-contained JARs is Spring Boot. By default, Spring Boot repackages your JAR into an executable JAR, and it does that by putting all of your classes inside BOOT-INF/classes, and all of the dependent libraries inside BOOT-INF/lib.

Finally I've found also an archetype that includes this plugin in the template and you can create it with:

```sh
mvn archetype:generate -DarchetypeGroupId=com.rudolfschmidt -DarchetypeArtifactId=javase8-assembly-archetype
```

and compile it with `mvn clen package`.

References:
- https://maven.apache.org/plugins/maven-assembly-plugin/
- https://github.com/rudolfschmidt/javase8-assembly-archetype
- https://github.com/netyjq/spring-boot-archetype

## Show system properties and environment variables

```sh
cd myproject

mvn help:system
```

## Interactive console for evaluating expressions

With the following command you can evaluate the value of expressions you use inside your pom.xml commonly for configuring plugin configuration parameters.

```sh
cd myproject

mvn help:evaluate
# using the full name:
# mvn org.apache.maven.plugins:maven-help-plugin:3.2.0:evaluate
```

Reference: https://maven.apache.org/plugins/maven-help-plugin/evaluate-mojo.html

## Show all repositories used by the build

The following command uses the 'list-repositories' goal that resolves all project dependencies and then lists the repositories used by the build and by the transitive dependencies:

```sh
cd myproject

mvn dependency:list-repositories
# using the full name:
# mvn org.apache.maven.plugins:maven-dependency-plugin:3.2.0:list-repositories
```

Reference: https://maven.apache.org/plugins/maven-dependency-plugin/list-repositories-mojo.html

## Show all dependencies of the project

This is helful to answer the following questions:
- What is the effective version of a dependency that will be used?
- Who is determining the final version of a dependency?

**Show a simple dependency list**

```sh
cd myproject

mvn dependency:resolve -DoutputFile=./dependency-list.txt
# using the alias:
# mvn dependency:list -DoutputFile=./dependency-list.txt
# using the full name:
# mvn org.apache.maven.plugins:maven-dependency-plugin:3.2.0:resolve -DoutputFile=./dependency-list.txt
```

Reference: https://maven.apache.org/plugins/maven-dependency-plugin/resolve-mojo.html

**Show dependency list with absolute artifact filenames**

```sh
cd myproject

mvn dependency:resolve -DoutputAbsoluteArtifactFilename=true -DoutputFile=./dependency-list.txt
# using the alias:
# mvn dependency:list -DoutputAbsoluteArtifactFilename=true -DoutputFile=./dependency-list.txt
# using the full name:
# mvn org.apache.maven.plugins:maven-dependency-plugin:3.2.0:resolve -DoutputAbsoluteArtifactFilename=true -DoutputFile=./dependency-list.txt
```

**Show a dependency list as a tree**

```sh
cd myproject

mvn dependency:tree -DoutputFile=./dependency-tree.txt
# using the full name:
# mvn org.apache.maven.plugins:maven-dependency-plugin:3.2.0:tree -DoutputFile=./dependency-tree.txt
```

Reference: https://maven.apache.org/plugins/maven-dependency-plugin/tree-mojo.html

## Show all ancestor POMs of the project

This is helful to answer the following questions:
- What is my inheritance chain of parent POMs?

This may be useful too in a continuous integration system where you want to know all parent POMs of the project.

```sh
cd myproject

mvn dependency:display-ancestors
# using the full name:
# mvn org.apache.maven.plugins:maven-dependency-plugin:3.2.0:display-ancestors
```

## Search parent POM

1. Run `mvn help:effective-settings` and copy the value inside the 'localReposity' tag.
2. `cd` into the value copied before.
3. Search the .pom file for the parent POM specified in your pom.xml file. E.g. If the parent POM has a groupId 'com.foo.bar', artifactId 'my-parent-pom' and vertion '1.2.3' then the file will be inside '<localRepository>/com/foo/bar/my-parent-pom/1.2.3/my-parent-pom-1.2.3.pom'.

## Show the effective POM

This is helful to answer the following questions:
- What does my pom.xml really look like when Maven is done combining everything?
- What is the result of my inheritance chain of parent POMs?
- How are my pom.xml properties affected by an active profile?
- What is the final value of some plugin configurations using property values in my pom.xml?

The effective POM is the POM that Maven uses to build your project, and that results from the application of:
- interpolation,
- inheritance
- and active profiles.

Displaying the effective POM is useful for two reasons:
- The effective POM is what Maven really uses to do its work.
- Because the parent POM inheritance chain (the chain of parent POMs and the Maven super POM from the installation directory), along with active profiles, and your settings.xml file is already resolved and combined before the dependency plugin (or any other plugin) is even activated.

The following command, using the 'effective-pom' goal, will iterate over all projects in the current build session, printing the effective POM for each.

```sh
cd myproject

mvn help:effective-pom -Doutput=./effective-pom.xml
# using the full name:
# mvn org.apache.maven.plugins:maven-help-plugin:3.2.0:effective-pom -Doutput=./effective-pom.xml
```

Reference: https://maven.apache.org/plugins/maven-help-plugin/effective-pom-mojo.html

## Show plugin information

In the following examples, you have to specify a Maven Plugin to describe in one of three ways:
- plugin-prefix, i.e. 'help'.
- groupId:artifactId, i.e. 'org.apache.maven.plugins:maven-help-plugin'.
- groupId:artifactId:version, i.e. 'org.apache.maven.plugins:maven-help-plugin:2.0'.

The following command shows basic information about a Maven plugin along a description of the goals inside:

```sh
cd myproject

mvn help:describe -Dplugin=help
```

The following command shows a more detailed information including configurable parameters for each goal:

```sh
cd myproject

mvn help:describe -Ddetail -Dplugin=help
```

Reference: https://maven.apache.org/plugins/maven-help-plugin/describe-mojo.html
