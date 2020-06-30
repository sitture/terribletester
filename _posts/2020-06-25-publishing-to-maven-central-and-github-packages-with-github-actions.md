---
layout: post
title:  Publishing to maven central and github packages with github actions
date:   2020-06-25 12:48:30 +0100
categories: [Git, Guide]
tags: [howto, git, continuous-integration]
---

I recently decided to move my [CircleCI](https://circleci.com/) configuration to [Github Actions](https://github.com/features/actions). There was nothing wrong with my CircleCI setup, I just wouldn’t be a terribletester if I didn’t experiment new tools. There are loads of blogs out there laying out the comparison of both CI platforms. In this particular post, I will show how I've switched my CI platform from CircleCI to Github Actions and how I'm now publishing jars to both maven central and [Github packages](https://github.com/features/packages).

<!--more-->

Before deciding this move to Actions, I just wanted to make a list of requirements that my current CircleCI solution was performing and what I would like to achieve in Github Actions.

## Requirements for a CI solution?

1. It should run tests on every push on master.
2. It should run tests on every commit push on a PR only (Not for pushes without an open pull request).
3. It should update the version of jar on github release with a release version.
4. It should publish the jars to maven and github packages simultaneously on creating the release.
5. It should cache and restore the maven dependencies based on the project's pom.xml

## Workflows in Github Actions

This concept of workflows exists both in CircleCI and Actions. Both let you define custom routines and run one or more jobs within each workflow. Workflows must have at least one job, and jobs contain a set of steps that perform individual tasks. Steps can run commands or use an action.

I would be creating a couple of workflows to cover all of the outlined requirements, one for running the tests (both on master and PR commits) and second for publishing the jars. All of the workflows are stored under the `.github/workflows` directory in the root of your repository.

### Creating a workflow for running tests

Create a new file `.github/workflows/maven-build.yml`.

```yml
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
```

One of the benefits of using Github actions is that you can trigger workflows based on any git action performed (e.g. push, issue creation, releases, etc). Adding the above will make sure this workflow is only run `on:push` to master and` on:pull_request` event to master branch.

The next step is to create a job under this workflow which would run the maven unit tests.

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - name: Run Tests
        run: mvn test
```

The above creates a job named `test` to run on using the `ubuntu-latest` image. Then we define a list of steps to perfom using the pre-defined actions to setup the environment and a step to the actual tests.

1. `actions/checkout@v2` just checks out the commit
2. `actions/setup-java@v1` - Sets up JDK environment (Java 8 in this case)
3. Runs the tests using the `mvn test` command

Doing the above will achieve the requirements 1 and 2.


{% raw %}
```yml
- name: Cache
    uses: actions/cache@v1.0.0
    with:
        path: ~/.m2/repository
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-maven-
```
{% endraw %}

For the caching of dependencies requirement, adding the pre-defined [cache action](https://github.com/actions/cache) from github lets us cache the `.m2` dependencies and restore based on the pom.xml. More reading on configuration can be found here [caching dependencies to speedup workflows](https://help.github.com/en/actions/configuring-and-managing-workflows/caching-dependencies-to-speed-up-workflows).

You can find the full workflow here: [sitture/env-config/tree/master/.github/workflows](https://github.com/sitture/env-config/tree/master/.github/workflows)

### Creating a workflow for publishing

Once we're happy, we're able to validate/test our changes using the workflow, it's now time to start publishing to maven central and github packages.

One of the really differentiating features of Github actions is that it lets you define a workflow based `on:release` event. Having used Travis CI and Circle CI, there are different ways to triggering these jobs, either by looking at if git reference is tag push. On release event makes it a really clean solution when triggering specific publish related tasks.

I decided to create a separate workflows for publishing to maven and github packages, so that if one of them fails, it can be retried independently. Create the 2 new workflow files, `.github/workflows/maven-publish.yml` and `.github/workflows/github-publish.yml`.

1. Add distribution repositories to pom.xml

Add two new profiles to your project's pom.xml to allow publishing to maven central or github packages. E.g.

```xml
<profiles>
	<profile>
		<id>ossrh</id>
		<distributionManagement>
			<snapshotRepository>
				<id>ossrh</id>
				<url>https://oss.sonatype.org/content/repositories/snapshots</url>
			</snapshotRepository>
			<repository>
				<id>ossrh</id>
				<url>https://oss.sonatype.org/service/local/staging/deploy/maven2</url>
			</repository>
		</distributionManagement>
	</profile>
	<profile>
		<id>github</id>
		<distributionManagement>
			<repository>
				<id>github</id>
				<name>GitHub Packages</name>
				<url>https://maven.pkg.github.com/sitture/env-config</url>
			</repository>
		</distributionManagement>
	</profile>
</profiles>
```

2. Add the following to both workflows, so that these are triggered only when new releases are created.

```yml
on:
  release:
    types: [created]
```

3. Add the deploy job to your maven-publish workflow:

```yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
          server-id: ossrh
          server-username: MAVEN_USERNAME
          server-password: MAVEN_PASSWORD
```

Similar to the build workflow, the above steps does a checkout of the commit and sets up the JDK environment with the addition of a couple `server` variable sets which generates the maven `settings.xml` on the fly for us.

* `server-id` - the id defined in your distribution repository in pom.xml'
* `server-username` and `server-password` - is setup to pick up the values from the environment variable that will be set during the publish step.

4. Add a step under the deploy job to publish

#### Maven Central `maven-publish.yml`

{% raw %}
```yml
- name: Publish package
    run: |
        mvn -B org.codehaus.mojo:versions-maven-plugin:2.5:set -DnewVersion=${GITHUB_REF##*/}
        mvn -B -Possrh deploy
    env:
        MAVEN_USERNAME: ${{ secrets.OSSRH_USER }}
        MAVEN_PASSWORD: ${{ secrets.OSSRH_PASS }}
```
{% endraw %}

#### Github Packages `github-publish.yml`

{% raw %}
```yml
- name: Publish package
    run: |
        mvn -B org.codehaus.mojo:versions-maven-plugin:2.5:set -DnewVersion=${GITHUB_REF##*/}
        mvn -B -Pgithub deploy
    env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
{% endraw %}

For publishing into maven central, you would need to add the `OSSRH_USER`and `OSSRH_PASS` as secrets. These secrets are then mapped to `MAVEN_USERNAME` and `MAVEN_PASSWORD` variables. You can read more about how to add secrets to Github actions here: [creating-and-storing-encrypted-secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets).

For publishing into Github packages, you only need to grab `GITHUB_TOKEN` from secrets which is available by default.

`mvn -B org.codehaus.mojo:versions-maven-plugin:2.5:set -DnewVersion=${GITHUB_REF##*/}` - Before performing a deploy, we need to grab the release version and set our pom.xml to that tag version. Github actions has a set of [default environment variables](https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables#default-environment-variables) and `GITHUB_REF` gives us the branch or the tag name.

However, the value returned from the variable looks something like this `refs/heads/1.0.0` for release with tag `1.0.0`. And we need to only grab the tag from the reference, for that we can just use bash parameter expansion `${GITHUB_REF##*/}` to get the tag name only which would be set in pom.xml before publishing the jar.

That's it. We have now finished the migration from CircleCI to Github Packages within an hour, we should now be publishing our jars to maven central and github packages simultaneously. :)

You can find the all the above workflows here: [sitture/env-config/tree/master/.github/workflows](https://github.com/sitture/env-config/tree/master/.github/workflows)

Please leave your feedback around what you think of Github Actions.
