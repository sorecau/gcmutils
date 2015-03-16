# Scenarios #
Common scenarios for using GCMUtils.
<br />
## Case 1: I only need to send registration id ##
The simple case. You only need to send the registration id (both for registration and unregistration).

  1. Add _gcm-client-utils.jar_ and _gcm-common-utils.jar_ to the project (see [project setup](https://code.google.com/p/gcmutils/wiki/ProjectSetup)).
  1. Update the `AndroidManifest.xml` with the required information (see [GCM Getting Started](http://developer.android.com/google/gcm/gs.html)).
  1. In you main activity onCreate-method, add the following code:
```
GCMRegistrar.checkDevice(this);
GCMUtils.checkExtended(this);
GCMUtils.getAndSendRegId(this);
```
  1. Add _gcmutils.properties_ in the assets folder and add the _receiver-url_, _sender-id_ and _check-extended_:
```
# Required
receiver-url=
sender-id=

# optional
# values: enabled/disabled
check-extended=enabled
```
  1. Create a class called _GCMIntentService_ and extend _GCMUtilsBaseIntentService_. Override the _onError_ and _onMessage_ methods:
```
public class GCMIntentService extends GCMUtilsBaseIntentService {
  @Override
  protected void onMessage(Context context, String msg) {
  }

  @Override
  protected void onError(Context context, String s) {
  }
}
```
<br />

---

## Case 2: I want to write the registration logic ##
Requires a bit more code than the simplest use case, but is still easy to set up.

  1. Add _gcm-client-utils.jar_ and _gcm-common-utils.jar_ to the project (see [project setup](https://code.google.com/p/gcmutils/wiki/ProjectSetup)).
  1. Update the `AndroidManifest.xml` with the required information (see [GCM Getting Started](http://developer.android.com/google/gcm/gs.html)).
  1. In you main activity onCreate-method, add the following code:
```
GCMRegistrar.checkDevice(this);
GCMUtils.checkExtended(this);

String regId = GCMRegistrar.getRegistrationId(this);
if ("".equals(regId))
  GCMRegistrar.register(this, GCMUtilsProperties.GCMUTILS.getSenderId());
else
  GCMUtils.createRegSender(this, regId).send();
```
  1. Add _gcmutils.properties_ in the assets folder and add the _receiver-url_, _sender-id_ and _check-extended_:
```
# Required
receiver-url=
sender-id=

# optional
# values: enabled/disabled
check-extended=enabled
```
  1. Create a class called _GCMIntentService_ and extend _GCMBaseIntentService_. Override the _onMessage_, _onError_, _onRegistered_ and _onUnregistered_ methods:
```
public class GCMIntentService extends GCMBaseIntentService {
  @Override
  protected void onMessage(Context context, Intent intent) {
  }

  @Override
  protected void onRegistered(Context context, String regId) {
    // Or write your custom logic
    GCMUtils.createRegSender(context, regId).send();
  }

  @Override
  protected void onUnregistered(Context context, String regId) {
    // Or write your custom logic
     GCMUtils.createUnregSender(context, regId).send();
  }

  @Override
  protected void onError(Context context, String errorMsg) {
  }
}
```
<br />

---

## Case 3: I need to customize the request and receive the response ##
A more advanced case where you want to customize the content of the request sent during the registration and/or unregistration process. Additionally, you can register your own class to get a callback with the result of the requests sent.

  1. Add _gcm-client-utils.jar_ and _gcm-common-utils.jar_ to the project (see [project setup](https://code.google.com/p/gcmutils/wiki/ProjectSetup)).
  1. Update the `AndroidManifest.xml` with the required information (see [GCM Getting Started](http://developer.android.com/google/gcm/gs.html)).
  1. In you main activity onCreate-method, add the following code:
```
GCMRegistrar.checkDevice(this);
GCMUtils.checkExtended(this);

String regId = GCMRegistrar.getRegistrationId(this);
if ("".equals(regId))
  GCMRegistrar.register(this, GCMUtilsProperties.GCMUTILS.getSenderId());
else {
  BasicNameValuePair regIdParam = new BasicNameValuePair(GCMUtilsConstants.PARAM_KEY_REGID, regId);
  BasicNameValuePair myCustomParam = new BasicNameValuePair("my-key", "my-value");

  GCMSender sender = GCMUtils.createSender(this, regIdParam, myCustomParam);
  sender.setCallback(new MyCallback());
  sender.send();
}
```
  1. Create your own callback class (in the above code its called _`MyCallback`_) and implement the _GCMSenderCallback_:
```
public class MyCallback implements GCMSenderCallback {
  @Override
  public void onRequestSent(GCMSenderResponse gcmSenderResponse) {
  }
}
```
  1. Add _gcmutils.properties_ in the assets folder and add the _receiver-url_, _sender-id_ and _check-extended_ (and optionally _request-backoff-millis_ and _request-retries_):
```
# Required
receiver-url=
sender-id=

# optional
# values: enabled/disabled
check-extended=enabled
request-backoff-millis=2000
request-retries=5
```
  1. Create a class called _GCMIntentService_ and extend _GCMBaseIntentService_. Override the _onMessage_, _onError_, _onRegistered_ and _onUnregistered_ methods (This is just example code, obviously you do not want to duplicate the createSender-code in your own projects).
```
public class GCMIntentService extends GCMBaseIntentService {
  @Override
  protected void onMessage(Context context, Intent intent) {
  }

  @Override
  protected void onRegistered(Context context, String regId) {
    BasicNameValuePair regIdParam = new BasicNameValuePair(GCMUtilsConstants.PARAM_KEY_REGID, regId);
    BasicNameValuePair myCustomParam = new BasicNameValuePair("my-reg-key", "my-value");

    GCMSender sender = GCMUtils.createSender(this, regIdParam, myCustomParam);
    sender.setCallback(new MyCallback());
    sender.send();
  }

  @Override
  protected void onUnregistered(Context context, String regId) {
    BasicNameValuePair regIdParam = new BasicNameValuePair(GCMUtilsConstants.PARAM_KEY_UNREGID, regId);
    BasicNameValuePair myCustomParam = new BasicNameValuePair("my-unreg-key", "my-value");

    GCMSender sender = GCMUtils.createSender(this, regIdParam, myCustomParam);
    sender.setCallback(new MyCallback());
    sender.send();
  }

  @Override
  protected void onError(Context context, String errorMsg) {
  }
}
```