# Project setup #

GCMUtils must be added to the Android app project as a dependency. There are two main alternatives:

**1. Direct project dependency**

This can be done by simply [downloading the jar-files](https://code.google.com/p/gcmutils/downloads/list) (_gcm-client-utils.jar_ and _gcm-common-utils.jar_) and adding them to your libs folder.

**2. Maven setup**

Include the Google code repositories in your pom.xml file:
```
<repositories>
  <repository>
    <id>gcmutils-repo-releases</id>
    <url>http://gcmutils.googlecode.com/svn/trunk/repository/releases</url>
  </repository>
</repositories>
<pluginRepositories>
  <pluginRepository>
    <id>gcmutils-repo-releases</id>
    <url>http://gcmutils.googlecode.com/svn/trunk/repository/releases</url>
  </pluginRepository>
</pluginRepositories>
```

Then add the required dependency:
```
<dependency>
  <groupId>net.jarlehansen.android.gcm</groupId>
  <artifactId>gcm-client-utils</artifactId>
  <version>LATEST_VERSION</version>
</dependency>
```

If you want to use the gcmutils-maven-plugin to generate the required tags in AndroidManifest.xml, go to the [ManifestGenerator-page](http://code.google.com/p/gcmutils/wiki/ManifestGenerator)