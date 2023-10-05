---
layout: post
title:  Managing cucumber-java dependencies with Maven BOM
date:   2023-10-04 15:00:00 +0000
categories: [Testing, Guide]
tags: [testing, cucumber, automation]
---

As users of cucumber-java, it’s quite common to use multiple cucumber libraries within a project which sometimes results in a conflict when the cucumber dependency versions are not in sync.  These conflicts can eat a lot unnecessary time to investigate and resolve.
<br /><br />
Good news! This can now be resolved easily using Maven’s Bill of Materials (BOM). We can take advantage of Maven’s BOM concept for cucumber version 7.0.0 onwards.

<!--more-->

To reduce this pain of having to validate the compatibility of different versions, cucumber are now publishing a [cucumber-bom](https://mvnrepository.com/artifact/io.cucumber/cucumber-bom) project which is just grouping dependency versions that are known to be valid.



## What is Bill of Materials?

A [Bill of Materials](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#bill-of-materials-bom-poms) is a feature from Maven that offers easier and safer dependency management. It is a special POM file similar to a parent project POM that groups dependency versions that are known to be valid and tested to work together.

This allows us to consume a set of required dependencies, just by importing the BOM into own projects. We can also avoid specifying versions of each dependencies, instead allowing the BOM version to control that.

## How to use cucumber-bom?

__Note__: The cucumber-bom project is only applicable to cucumber version `7.x.x` and beyond.


### Maven

If you're using maven, we need to a add a [dependencyManagement](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Management) section into `pom.xml` for adding lookup references to the cucumber dependencies.

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>io.cucumber</groupId>
      <artifactId>cucumber-bom</artifactId>
      <version>7.14.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Adding into `dependencyManagement` does not add it to the dependency tree of the project, it is used just for lookup reference. The `import` scope set in the dependency section indicates that this dependency should be replaced with all effective dependencies declared in its POM.

We can now add the actual dependencies needed to our project's `dependencies` section. The version is mandatory in the `dependencyManagement` section but no longer required for when defining `dependencies`, it will automatically find the dependency versions from the list of dependencies provided in `dependencyManagement`.

```xml
<dependencies>
  <dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit-platform-engine</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

### Gradle

```groovy
dependencies {
    testImplementation(platform("io.cucumber:cucumber-bom:7.14.0"))

    testImplementation("io.cucumber:cucumber-java")
    testImplementation("io.cucumber:cucumber-junit-platform-engine")
}
```

This should help you keep the versions of cucumber dependencies aligned without worrying about compatibility with each other.

You can also checkout the [cucumber-java-skeleton](https://github.com/cucumber/cucumber-java-skeleton) project for examples of BOM usage.
