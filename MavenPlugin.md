# Maven Plugin #

The gcmutils-maven-plugin provides two main features:
  1. **[Manifest generator](#Manifest_generator.md)**
  1. **[Test server](#Test_server.md)**

To include the gcmutils-maven-plugin in your own project, add the following:
```
<repositories>
  <repository>
    <id>gcmutils-repo-releases</id>
    <url>http://gcmutils.googlecode.com/svn/trunk/repository/releases</url>
  </repository>
</repositories>
<pluginRepositories>
  <pluginRepository>
    <id>gcmutils-pluginrepo-releases</id>
    <url>http://gcmutils.googlecode.com/svn/trunk/repository/releases</url>
  </pluginRepository>
</pluginRepositories>

<plugin>
  <groupId>net.jarlehansen.android.gcm.plugins</groupId>
  <artifactId>gcmutils-maven-plugin</artifactId>
  <version>LATEST_VERSION</version>
  <configuration>
    <!-- Manifest generator config -->
    <manifestPath>my-folder/AndroidManifest.xml</manifestPath> <!-- Optional -->
    <skipBackup>true</skipBackup> <!-- Optional -->

    <!-- Test server config -->
    <port>8080</port> <!-- Optional -->
    <contextRoot>/myServer</contextRoot> <!-- Optional -->
    <!-- <apiKey></apiKey> Should be included as System property or in .m2/settings.xml -->
  </configuration>
</plugin>
```


---


## Manifest generator ##

One of the more error-prone and tedious tasks when creating a GCM project is to add the correct manifest entries. The GCMUtils `AndroidManifest` generator is created to make this process fast and simple.

The manifest generator automatically generates the required manifest entries for a GCM app. By running either the maven plugin or the Java application, it will scan your existing `AndroidManifest.xml` file and add the missing tags.

There are two main ways to use the manifest generator:
  * [Maven plugin](#Manifest_generator_maven_plugin.md)
  * [Java application](#Manifest_generator_Java_application.md)


### Manifest generator maven plugin ###
Execute the following command to run the plugin:
```
mvn gcmutils:gcmify
```
**Configuration values:**
| **manifestPath** | the path to your manifest-file. Default: `AndroidManifest.xml`. |
|:-----------------|:----------------------------------------------------------------|
| **skipBackup** | If set to true, no backup-file is created. Default: false.|

By default (the skipBackup configuration) the gcmutils-maven-plugin will automatically create a backup file, called `AndroidManifest-backup.xml`. If anything wrong happens, you will still have your original `AndroidManifest.xml`-file available. It is recommended to keep the backup-feature enabled, if everything is generated successfully you can delete the backup-file.

If you have a standard Android maven setup you do not need to add any configuration-options.

### Manifest generator Java application ###
  1. Download the gcmutils-maven.jar from Downloads.
  1. To generate the manifest tags, run the command:
```
java -jar gcmutils-main.jar AndroidManifest.xml my.package.name
```

If you want to skip the creation of the backup-file, add the system property `-DskipBackup=true`.


---


## Test server ##
The gcmutils-maven-plugins provides a very simple test server for GCM. This makes it easy to test your application locally with a fully functional server implementation. It is also created to integrate with the GCMUtils client library.

The test server offers registration/unregistration service (storing the regIds in memory) and provides a webpage that makes it easy to send test messages to the devices.

To start the test server execute the following command:
```
mvn gcmutils:run-server
```

**Configuration values:**
| **apiKey** | The API key is created in [Google APIs console](https://code.google.com/apis/console/) and is used when sending messages to the registered devices.<br />It is recommended to add the apiKey value to either a system property when running the command (_gcmutils:run-server `-DapiKey`=_) or in the _.m2/settings.xml_ file.<br />See the [GCM Getting Started guide](http://developer.android.com/google/gcm/gs.html#create-proj) for more information on how to generate the API key. |
|:-----------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **port** | The port number that the server will run on. Default: 9595 |
| **contextRoot** | The context root of the webapp. Default: "/" |

<br />
**Test server webpage:**
<a href='http://tinypic.com?ref=9lfnzc'><img src='http://i49.tinypic.com/9lfnzc.png' alt='Image and video hosting by TinyPic' border='1' />