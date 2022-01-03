---
title: "Maven commands for analysis and troubleshooting"
tags: maven troubleshooting
---

# Maven commands for analysis and troubleshooting

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

This may be useful too in a continuous integration system where you want to know all parent poms of the project.

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

The following command shows basic information about a aven plugin along a description of the goals inside:

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
