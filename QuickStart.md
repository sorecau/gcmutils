# Quick start #

For more detailed information about standard GCM, see the

  1. Complete step 1, 2 and 3 in the [GCM:Getting Started](http://developer.android.com/google/gcm/gs.html) guide.
  1. To automatically generate the required xml-tags in the `AndroidManifest.xml` file, either use the [gcmutils-maven-plugin](https://code.google.com/p/gcmutils/wiki/MavenPlugin#Manifest_generator_maven_plugin) or [download and run the Java application](https://code.google.com/p/gcmutils/wiki/MavenPlugin#Manifest_generator_Java_application). Also, see the [android-gcm-quickstart](https://github.com/akquinet/android-archetypes) archetype.
  1. [Download jar-files](https://code.google.com/p/gcmutils/downloads/list) (gcm-client-utils and gcm-common-utils) or [add them to your pom.xml](https://code.google.com/p/gcmutils/wiki/ProjectSetup).
  1. In your main activity, add the following code:
```
GCMRegistrar.checkDevice(this);
GCMUtils.checkExtended(this);
GCMUtils.getAndSendRegId(this);
```
  1. Add _gcmutils.properties_ in the assets folder and add the _receiver-url_, sender-id and check-extended:
```
# Required
receiver-url=
sender-id=

# optional
# values: enabled/disabled
check-extended=enabled
```
  1. Create a GCMIntentService class and extend GCMUtilsBaseIntentService. Implement the onMessage and onError methods (if you need custom behavior you can also override the onRegistered method).
```
public class GCMIntentService extends GCMUtilsBaseIntentService {
  @Override
  protected void onMessage(Context context, String msg) {
    Log.i(TAG, "Message received: " + msg);
  }

  @Override
  protected void onError(Context context, String error) {
    Log.e(TAG, "onError: " + error);
  }
}
```
  1. Run the [GCMUtils test server](https://code.google.com/p/gcmutils/wiki/MavenPlugin#Test_server):
```
mvn gcmutils:run-server
```

Or implement a server that can receive registration ids and send messages (see [GCM: Getting Started guide](http://developer.android.com/google/gcm/gs.html#server-app) for more info).
The default request parameters have the following format (key=value):
  * regId=_`[`registration id`]`_
  * unregId=_`[`registration id`]`_
  * email=_`[`email`]`_ (optional)
<br />
**(Server code) receive registration id example:**
```
@Override
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
  String regIdParam = request.getParameter(GCMUtilsConstants.PARAM_KEY_REGID);
  String unregIdParam = request.getParameter(GCMUtilsConstants.PARAM_KEY_UNREGID);
  String email = request.getParameter(GCMUtilsConstants.PARAM_KEY_EMAIL);
  if (regIdParam != null) {
    logger.info("Received registration id: {}", regIdParam);
  }
}
```

**(Server code) send msg example:**
```
Sender sender = new Sender(apiKey);
Message message = new Message.Builder().addData(GCMUtilsConstants.DATA_KEY_MSG, msg).build();
Result result = sender.send(message, regId, 0);
logger.info("Push message result: {}", result);
```

For more detailed information, see the [features page](https://code.google.com/p/gcmutils/wiki/Features).