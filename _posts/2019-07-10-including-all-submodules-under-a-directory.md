---
title: Including all sub-modules under a directory
summary: There are multiple ways to add sub-modules and it's submodules within the parent settings.gradle.
categories: [Gradle, Guide]
tags: [howto, gradle]
---

There are multiple ways to add sub-modules and it's submodules within the parent `settings.gradle`.

Given the following directory structure:

```sh
myproject/
├── gradle/
├── submodule/
│   └── submodule1/
│   └── submodule2/
├── build.gradle
├── gradle.properties
├── gradlew
├── gradlew.bat
└── settings.gradle
```

You can either add the following explicitly to `settings.gradle` to include all modules one by one:

```groovy
include "submodule"
include ":submodule:submodule1"
include ":submodule:submodule2"
```

The above works when you're not going to be adding or removing these submodules. However, you can automate the addition/removal of submodules under a module by adding the following to your `settings.gradle`:

```groovy
// include all modules under the 'submodule' directory to a multi-module project
include "submodule"
new File("$rootDir/submodule").listFiles().each {
  def moduleName = it.name
  include moduleName
  project(":${moduleName}").projectDir = file("${it.getParentFile().name}/${moduleName}")
}
```

Hopefully, the above is helpful to others. Good Luck! :)
