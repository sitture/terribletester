---
layout: post
current: post
cover:  assets/images/cucumber-hooks.jpg
navigation: True
title: BeforeAll and AfterAll workaround for cucumber-java
date: 2019-09-15 09:00:00
tags: [guide]
class: post-template
subclass: 'post tag-guide'
author: haroon
---

As a cucumber-jvm user, I've always felt the need for a `@BeforeAll` and `@AfterAll` hooks. Although, cucumber still does not have a support this yet, there're many ways to workaround and achieve a solution.

There's a 5+ years old issue opened with cucumber `https://github.com/cucumber/cucumber-jvm/issues/515`, you can follow this thread for other workarounds mentioned.

One easiest and most optimal way how it could be done is by implementing a `CucumberHooks` base class which can then be used a cucumber plugin within your cucumber `@RunWith` class.

### Dependencies

Add the following cucumber dependencies to your project:

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>4.7.2</version>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>4.7.2</version>
</dependency>
```

### CucumberHooks implements ConcurrentEventListener

Create a new class `CucumberHooks` implementing cucumber's `ConcurrentEventListener` from `cucumber.api.event.ConcurrentEventListener`.

```java
import cucumber.api.event.ConcurrentEventListener;
import cucumber.api.event.EventHandler;
import cucumber.api.event.EventPublisher;
import cucumber.api.event.TestRunFinished;
import cucumber.api.event.TestRunStarted;

public class CucumberHooks implements ConcurrentEventListener {

    @Override
    public void setEventPublisher(EventPublisher eventPublisher) {
        eventPublisher.registerHandlerFor(TestRunStarted.class, beforeAll);
        eventPublisher.registerHandlerFor(TestRunFinished.class, afterAll);
    }

    private EventHandler<TestRunStarted> beforeAll = event -> {
        // something that needs doing before everything
    };

    private EventHandler<TestRunFinished> afterAll = event -> {
        // something that needs doing after everything
    };

}
```

### Use with @RunWith

Once you've created the above class, you can now start using this with your runner (`@RunWith`) class:

```java
import org.junit.runner.RunWith;
import io.cucumber.junit.Cucumber;
import io.cucumber.junit.CucumberOptions;

@RunWith(Cucumber.class)
@CucumberOptions(
        plugin = {"pretty","your.package.CucumberHooks"},
        features = "src/test/resources/features"
)
public class CucumberRunner {
}
```

And that's it, you should now be able to achieve the missing `@BeforeAll` and `@AfterAll` doing this. Hopefully, the above is helpful to you and others. Good Luck! :)
