Angular Webjar
==============

This project is a skeleton of an Angular project that builds a WebJar.
It is designed to be used with a Java servlet container, such as
[jersey2-guice](https://github.com/otasyn/jersey2-guice), but it should
be compatible with other web projects that accept WebJars.

Angular
-------

This project was generated with
[Angular CLI](https://github.com/angular/angular-cli) version 16.0.4.

This project uses Angular CLI to generate project files.  Only a few
changes have been made from the generated project.  Notably,
`angular.json` can be used to specify to output directory for the
generated files that will be bundled into a WebJar.

### Generated index

In `angular.json`, the `index` file can be specified.  By default,
this is `src/index.html`.  In this project, that has been changed
to `src/includes.jsp`.  This is so that the generated `<script>`
tags can be included into another JSP file in the servlet container
project.

Also, the build is executed with the `--deploy-url` argument so
that the paths of those files can be set to reflect the path
used in the WebJar.

Gradle
------

Gradle is used to build and publish the WebJar.  It is configured to
publish both snapshots and production builds based on the version.

### Building Angular project

There are several ways to execute an Angular CLI build from Gradle,
but this project uses the plugin `com.moowork.node`.  It generates
several standard npm tasks that can be used.  For this project,
the `npm_run_build` task is most relevant.  Basically, it is the
same as typing `npm run build` into the command line.  The `build`
script is defined in `package.json` unders `scripts`.

```bash
$ ./gradlew npm_run_build
```

In `package.json`, you could define additional scripts and run
them in the same way.

```bash
$ ./gradlew npm_run_flyKite
```

This plugin also lets you define the versions of Node and npm
to use.  This project is set to use whichever version is
available, and the version is set using Node Version Manager
and `.nvmrc`.

_Source: https://plugins.gradle.org/plugin/com.moowork.node_

### Creating a JAR

Creating a JAR file depends upon the plugin `java`.  The project
does not necessarily need to include Java code for using this
plugin.  It's only needed for bundling the files into a JAR.

#### Bundling the generated files

In the `jar` block, specify where to get the generated project files
using `from` and where to put them inside the JAR using `into`.  In
this project, Angular CLI is configured to put them into different
directories depending upon the version name, but that is not
required.  For a WebJar, the contents must be put into
`META-INF/resources/webjars/${archivesBaseName}/${project.version}`.

```groovy
jar {
  from `dist/angular-webjar/`
  into `META-INF/resources/webjars/${archivesBaseName}/${project.version}`
}
```

_Source: https://www.webjars.org/contributing_

### Publishing

Publishing depends upon the plugin `maven-publish`.

#### Local Repo

To publish to a local repo, such as `$HOME/.m2/repository`:

```bash
$ ./gradlew publishToMavenLocal
```

#### Remote Repo

To publish to a remote repo, make sure that you have the repo URL
and credentials in a `gradle.properties` file, such as
`$USER_HOME/.gradle/gradle.properties`.  These properties should
look something like:

```bash
# Angular Webjar Repo
repoUrl=http://repo.com/repository
repoUsername=otasyn
repoPassword=password
```

Then, to publish it:

```bash
$ ./gradlew publishAngularWebjarPublicationToTemplateRepoRepository
```

In `publishAngularWebjarPublicationToTemplateRepoRepository`, the
_AngularWebjar_ portion is taken from the name of the publication
in the `publications` block of `publishing`.  This can be renamed
to whatever you like.

```groovy
publishing {
  publications {
    angularWebjar(MavenPublications) {
      from components.java
    }
  }
  ...
}
```

The _TemplateRepo_ portion is taken from the name of the repo that
is defined in the `repositories` block of `publishing`.  Inside
the `maven` block, set the `name` property to whatever you like.

```groovy
publishing {
  ...
  repositories {
    maven {
      name 'TemplateRepo'
      url 'http://repo.com/repository'
      credentials {
        username 'otasyn'
        password 'password'
      }
    }
  }
}
```
