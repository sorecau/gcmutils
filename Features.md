# GCMUtils features #
GCMUtils was created to make it easier to use GCM. It works in combination with the standard GCM classes, such as GCMRegistrar. If, for some reason, the utilities are not a match for parts of your project, simply use standard GCM in those cases.


**The main features are:**
  * Registration id handling (both for registration and unregistration), automatically sends a request to a configured GCM server containing the registration id. Supports features such as exponential backoff.
  * Extended verification of project with `GCMUtils.checkExtended(context)`. Verifies the GCM service created in the project.
  * Uses the _gcmutils.properties_ to make it easy to provide configuration options, such as _receiver-url_ and _sender-id_.
  * An alternative base intent service _GCMUtilsBaseIntentService_, that includes a few helper methods, such as automatic registrationId handling.

**Requirements:**
  * Android 2.2 or newer
  * gcm-client jar-file
  * Java 1.5 or newer


---


  * **[1 Setup](#1_Setup.md)**
  * **[2 Configuration](#2_Configuration.md)**
  * **[3 GCMUtils class](#3_GCMUtils_class.md)**
  * **[3.1 Extended check](#3.1_Extended_check.md)**
  * **[3.2 Registration id handling](#3.2_Registration_id_handling.md)**
  * **[3.2.1 Automatic registration id request](#3.2.1_Automatic_registration_id_request.md)**
  * **[3.2.2 Custom request parameters](#3.2.2_Custom_request_parameters.md)**
  * **[4 GCMUtilsBaseIntentService class](#4_GCMUtilsBaseIntentService_class.md)**
  * **[5 Receiving server](#5_Receiving_server.md)**
  * **[6 GCMUtils maven plugin](#6_GCMUtils_maven_plugin.md)**
  * **[7 Source code](#7_Source_code.md)**


---


## 1 Setup ##
Read the [GCM: Getting Started](http://developer.android.com/google/gcm/gs.html) guide. GCMUtils requires the same initial setup, with generation of the sender id (project id) and API key.

See the [project setup page](http://code.google.com/p/gcmutils/wiki/ProjectSetup) for more info on how to include GCMUtils in your project.

Also see the [android-gcm-quickstart archetype](https://github.com/akquinet/android-archetypes).

## 2 Configuration ##
By default GCMUtils uses a properties file called _gcmutils.properties_. This should be placed in the assets folder of the projects. When this file is available you can use the GCMUtils-methods that takes a Context object as input (for example: _GCMUtils.createRegSender(context, regId)_).

**Available properties:**
```
# Required
receiver-url=http://...
sender-id=

# optional
# values: enabled/disabled
check-extended=enabled
request-backoff-millis=2000
request-retries=5
```

## 3 GCMUtils class ##
The most important class is GCMUtils. It provides helper methods for the Android client. This section will provide more detailed information about these features, how they work and when they should be used.

### 3.1 Extended check ###
The standard GCMRegistrar class provides two very useful methods that are used to verify that your project setup is correct. These are:
```
GCMRegistrar.checkDevice(this);
GCMRegistrar.checkManifest(this);
```
While these methods are very useful, they do not verify your service implementation. GCMUtils extends this, by offering the _GCMUtils.checkExtended_. It is useful during development and will first start by calling the GCMRegistrar.checkManifest-method. It will continue by doing a more extensive verification of your service class, that it exists and that it is a subclass of _GCMBaseIntentService_. The code should be added like this in the onCreate-method:
```
GCMRegistrar.checkDevice(this);
GCMUtils.checkExtended(this);
```
There is no need to call GCMRegistrar.checkManifest again, since this is done by GCMUtils for you. When you are installing the app on real devices (not in development mode) you should only include the _GCMRegistrar.checkDevice_ method. Simply disable or remove the other checks.

You can enable or disable the checkExtended in  _gcmutils.properties_ with the property: _check-extended=enabled/disabled_. If GCMUtils does not find the check-extended property, it will use the _`ApplicationInfo.FLAG_DEBUGGABLE`_. If it is set to debuggable checkExtended() will be executed, if not it will be skipped. More info on _[ApplicationInfo.FLAG\_DEBUGGABLE](http://stackoverflow.com/questions/7022653/how-to-check-programmatically-whether-app-is-running-in-debug-mode-or-not)_.

### 3.2 Registration id handling ###
By default GCM does not provide support for handling the registration id, which needs to be sent from the Android client to the server. GCMUtils adds support for automatic registration id handling. This can be done in a number of ways, depending on your requirements.

#### 3.2.1 Automatic registration id request ####
```
GCMUtils.createRegSender(String receiverUrl, String regId).send();
```
or
```
GCMUtils.createRegSender(Context context, String regId).send();
```
Both of these methods will create a _GCMSender_ instance for you. With the first method (taking two Strings) it will send it to the receiverUrl you specify. When providing the _Context_ it will read the _gcmutils.properties_ (in the assets folder) where you need to specify the _receiver-url_ property.
The createRegSender-method, as shown above, should be used where you receive the registration id from GCM. Typically this will be in the onCreate method:
```
String regId = GCMRegistrar.getRegistrationId(this);
if (regId.equals(""))
  GCMRegistrar.register(this, SENDER_ID);

GCMUtils.createRegSender(Context context, String regId).send();
```
and in the onRegistered method:
```
@Override
protected void onRegistered(Context context, String regId) {
  GCMUtils.createRegSender(GCMUtilsProperties.GCMUTILS.getReceiverUrl(), regId).send();
}
```
If you only want to send the registration id and not any other request parameters, you can also use the _getAndSendRegId-method_. This will handle the registration id process for you (no need to manually call _GCMRegistrar.register()_), just add the following code:
```
GCMUtils.getAndSendRegId(context);
```
or
```
GCMUtils.getAndSendRegId(context, callback)
```
The callback variable (_GCMSenderCallback_) is an interface that will be called when the request has been sent. This is where you can add your own implementation to get the status of the request sent.

There is also a standard method available for sending the registration id and the configured e-mail account on the device. If no email address is registered on the device, the email parameter is not sent (in these cases it will only send the registration id).
```
GCMUtils.getAndSendRegIdAndEmail(context);
```

The same registration id handling methods are available for the unregister-process. For example:
```
GCMUtils.createUnregSender(context);
```


If the request fails for some reason, GCMUtils will retry with an exponential backoff. By default this is set to 2000ms and it will retry 5 times. You can changes these values in _gcmutils.properties_. After the request is finished, regardless of whether it was successful or not, a callback is sent to the registered class with the response (_GCMSenderResponse_).

The getAndSendRegId-method is especially useful combined with the _GCMUtilsBaseIntentService_ class, see section
[4 GCMUtilsBaseIntentService](#4_GCMUtilsBaseIntentService_class.md) for more detailed information.

#### 3.2.2 Custom request parameters ####

By default the methods _GCMUtils.createRegSender_ and _GCMUtils.createUnregSender_ will only send the registration id (with the key regId and unregId). If you need more request parameters you can use the GCMUtils.createSender method:
```
GCMUtils.createSender(receiverUrl, new BasicNameValuePair("key1", "value1"), new BasicNameValuePair("key2", "value2"));
```
or
```
GCMUtils.createSender(context, new BasicNameValuePair("key1", "value1"), new BasicNameValuePair("key2", "value2"));
```
You can send is as many _`BasicNameValuePair`_ instances as you need. These will then be sent to the specified server.

## 4 GCMUtilsBaseIntentService class ##
The GCMUtilsBaseIntentService adds a few helper methods to the standard _GCMBaseIntentService_ class offered by GCM.
GCMUtilsBaseIntentService includes a simplified method for receiving String messages, which will extract the message from intent for you:
```
protected void onMessage(Context context, String msg) {
}
```
By default this uses the key defined in the gcm-commons-utils:
```
GCMUtilsConstants.DATA_KEY_MSG
```

Additionally, the GCMUtilsBaseIntentService can handle the onRegistered and onUnregistered methods for you automatically. By extending the GCMUtilsBaseIntentService-class, it will automatically send regId and unregId to the configured server. This must be used in combination with the _gcmutils.properties_ file, where the _receiver-url_ (url to the server receiving the requests) is specified.

## 5 Receiving server ##
The request sent by the _GCMSender_ consists of a simple POST request containing regId=_registrationId_.
If you are writing your server in Java, you can add the gcm-commons-utils dependency to your project. This contains constants, which means you can read the regId and unregId by doing this:
```
String regIdParam = request.getParameter(GCMUtilsConstants.PARAM_KEY_REGID);
String unregIdParam = request.getParameter(GCMUtilsConstants.PARAM_KEY_UNREGID);
```

[Click here](https://code.google.com/p/gcmutils/wiki/MavenPlugin#Test_server) for information on the test server that is available in the GCMUtils maven plugin.

For more information on the general server implementation, see [the GCM documentation](http://developer.android.com/google/gcm/gs.html#server-app).

## 6 GCMUtils maven plugin ##
The gcmutils-maven-plugin provides two main features: 1) [manifest generator](https://code.google.com/p/gcmutils/wiki/MavenPlugin#Manifest_generator) and 2) [test server](https://code.google.com/p/gcmutils/wiki/MavenPlugin#Test_server). Click [here](https://code.google.com/p/gcmutils/wiki/MavenPlugin) for more information.

## 7 Source code ##
The source code is available at github: https://github.com/jarlehansen/gcmutils

Demo Android project: https://github.com/jarlehansen/gcmutils/tree/master/gcm-client-sample