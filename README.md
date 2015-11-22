# Android-SDK
This document provides an overview of Notify.io’s SDK integration
#1. Quick Overview
Integration with our platform has two components:

An ingestion endpoint that you use to send us information about your items (such as your videos, articles, or products). This is the universe of items from which we recommend

Native SDKs that you use to send us user activity and that we use to deliver personalized contextual notifications.  We use activity data(including screen views, engagement, conversion, and tap activity) to build models of user behavior.  
#4. Android SDK
##4.1. Installation

1. Download the SDK

2. Include the SDK directory in the project’s build.gradle file:
    ```gradle
    repositories {
        jcenter()
        flatDir {
            dirs '/[SDK Directory Here]/'
        }
    }
    ```

3. Add the following dependencies in your project’s build.gradle file: 
    ```gradle
    compile 'com.google.android.gms:play-services:7.3.0'
    compile(name:'notifysdk-release', ext:'aar')
    compile 'com.android.support:support-v4:23.0.0'
    compile 'com.google.code.gson:gson:2.3.0'
    compile 'com.squareup.retrofit:retrofit:1.9.0'
    compile 'com.squareup.okhttp:okhttp:2.5.0'
    compile 'com.google.android.gms:play-services:7.3.0'
    compile 'com.squareup:tape:1.2.3'
    compile 'com.squareup:otto:1.3.8'
    compile 'javax.inject:javax.inject:1'
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.squareup.dagger:dagger:1.2.+'
    provided 'com.squareup.dagger:dagger-compiler:1.2.+'
    ```
 
4. Define following permission in the application manifest file of your application:
    ```xml
    <permission android:name="[your app package].permission.C2D_MESSAGE"
    ```

5. Add following permissions:
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.ACCESS_NOTIFICATION_POLICY" />
    <uses-permission android:name="android.permission.INTERACT_ACROSS_USERS" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="[your app package].permission.C2D_MESSAGE" />
    ```

##4.2 Basic Setup (Configuration) 
The SDK uses APIConfig class for configuration. Call static getInstance() method of APIConfig class to initialize APIConfig. It takes apiKey as parameter.

```java
APIConfig config = APIConfig.getInstance(this.getApplicationContext(),"[Your API Key]");
```

Initialize the SDK by by adding the following in onCreate method of your MainActivity class or your Application class:

```java
Notify.initialize(config, this.getApplicationContext());
```

##4.3 Get the Current Push Permissions
To aid you in optimising your push opt-in workflow, we provide a simple method of getting the user’s current permission state. 

This method will return one of the following states.
* `Enabled` Push notifications are enabled for the application.
* `Disabled` The user has disabled push notification for the application.

```java
Notify.getInstance().isNotificationEnabled();
```

##4.4 Track Action on an Item
The track action API is used to send us actions a user takes on an item (including screen views, engagement, conversion, and tap activity). We use activity data to build models of user behavior.

We support the following standard action types:
* __Viewed__ for a user viewing an item. This is the most standard event type. 
* __Engaged__ this action is used to indicate that the user has engaged with the item.  For a video this might mean the user has vided more than 50% of it.  For content, it can mean the user has scrolled through more than 50% of the page.  
* __Ended__ indicates that the user has completed viewing or reading the item.  For example it would indicate that the user has reached the end of the video.  
* __Purchased__ when the user has completed the purchase of an item.  If the user purchases multiple items you’ll have to make an individual call for each purchase. 
* __AddedToCart__ after a user adds an item to their cart.  

You can also use any custom action type you would like.  However, you will need to contact us to discuss how you would like the action to effect recommendations.  

Actions are always associated with items that we have ingested.  The item parameter(itemID) specifies the unique identifier for the item and should match the unique_id specified in the ingestion API.

The item does not need to be ingested before making an action call.  If the item has not been ingested yet, we will buffer the action on our backend end till the item is ingested.  

For example when a user engages with an item

```java
Notify.getInstance().trackAction("shared", item.id)
```

##4.5 Deep linking
The default deep link action assumes that the application is setup to handle deep links through Android implicit intents. Implicit intents are intents used to launch components by specifying an action, category, and data instead of providing the component’s class. This allows any application to invoke another application’s component to perform an action. Only application components that are configured to receive the intent, by specifying an intent filter, will be considered.

In this example, an activity will be declared to handle all deep links for the app with the scheme `notify` and host `notify.io`.

Declare the activity in the AndroidManifest.xml file with the intent filter. The intent filter should accept the default intent action `android.intent.action.VIEW` and either `android.intent.category.DEFAULT` or `android.intent.category.BROWSABLE` category. It should also filter out any intents that do not contain a data URI with the scheme `notify` and host `notify.io`.

```xml
<intent-filter >
   <action android:name="android.intent.action.VIEW" />

   <category android:name="android.intent.category.DEFAULT" />
   <category android:name="android.intent.category.BROWSABLE" />

   <data
       android:host="notify.io"
       android:scheme="notify" />
</intent-filter>
```

The following is the `onCreate` method of activity that handles the deeplinking intent:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_content);

   if (getIntent().getAction() == Intent.ACTION_VIEW) {
       Uri data = getIntent().getData();

       if (data != null) {

           // show an alert with the "custom" param
           new AlertDialog.Builder(this)
                   .setTitle("Welcome to New Content!")
                   .setMessage("Found custom param: " +data.getQueryParameter("custom"))
                   .setPositiveButton(android.R.string.yes, new DialogInterface.OnClickListener() {
                       public void onClick(DialogInterface dialog, int which) {
                           dialog.dismiss();
                       }
                   })
                   .setIcon(android.R.drawable.ic_dialog_alert)
                   .show();
       }
   }
}
```

##4.6 Login
You should call `loginUser` as soon as the user is identified (generally after logging in). 

Logging in a user, allow you to track users across devices and platforms, improving the quality of recommendations. 

Before calling `loginUser`, we assign a unique identifier to your users and classify them as anonymous users.  Upon calling `loginUser`, the anonymous user’s activity history is transferred to the logged in user’s profile.  

Your `userId` should be unique and unchanging. 

```java
Notify.getInstance().loginUser(userId);
```

##4.7 Logout
You will need to tell us when the user logs out of your app.  After `logoutUser`, our platform defaults to an anonymous user. 

You can do that with following: 

```java
Notify.getInstance().logoutUser();
```

##4.8 Adding User Attributes 
Our platform gives you the ability to target users based on attributes and uses attributes to improve recommendations.  It is important to keep attributes up-to-date.  

Attributes include, but are not limited to email, gender, telephone number and other demographic information.  User attributes can be set using the `addUserAttribute` API. 

Attributes you should supply, include the following:
* Email
* Username
* Gender
* TwitterID
* FacebookID

Note: We currently do not merge user attributes on `loginUser`.  If you set attributes for an anonymous user, you will have to set them again after calling `loginUser`. 

Here is an example of setting the user’s email address and gender. 

```java
Notify.getInstance().addUserAttribute(“email“,“somename@somedomain.com“);
Notify.getInstance().addUserAttribute(“gender“,“male“);
```

##4.9 The user id of the Currently Logged in User

You can get the userID of the currently logged in user by making the following call:

```java
Notify.getInstance().getUserId();
```

##4.10 How to get your push token

You can retrieve the device’s current push token by making the following call:

```java
Notify.getInstance().getPushToken();
```

##4.11 How to get the device id
We generate a `DeviceId` to uniquely identify each device. To the extent possible, we keep this ID the same through application upgrades. 
```java
Notify.getInstance().getDeviceId()
```

##4.12 Notification Subscription Management 
We provide advanced subscription management for users allowing them to opt-in or opt-out of individual types of notifications.

Each user’s subscription state is synced to our backend.  To get a list of notification types and the user’s current subscription state you register handlers with the `registerSubscriptionListHandlers`.  The use of these handlers is explained below. 

`SubsListHandler` defines three handlers as methods. 

* `onSuccess` is called when the list of notification types has been successfully synced to the device.  The list of notification types is passed to the handler as an array.  Each entry in the array is a dictionary that defines a particular notification type.  The following keys that are defined in the dictionary :
  * `subscription_id` The ID of this notification type
  * `display` The display name of this notification type
  * `desc` The display description of this notification type
  * `value`  A `String` which represents the user’s subscription state for this notification. "true" = subscribed, "false" = unsubscribed.
  * `type` This is for future use, we currently only one type ‘sub’

    For example a notification type called “Recommended Items” that the user is currently subscribed to would be represented by:
    ```
    {
        “subscription_id” = “recommended_items”;
        “display” = “Recommended Items”;
        “desc” = "Personalized recommendations for you";
        “value” = true;
        “type” = “sub”;
    }
    ```
    Note: We sync notifications during SDK initialization which will result in the invocation of this handler.  If there was a network error during initialization we’ll continue retrying and this handler will be called as soon as the list is ready. 

* `onError` is called when due to network issue or if the notification types have not completed syncing yet.  If this is called we recommend displaying an error message to the user.  We will continue retrying to sync will call the OnSuccess once the sync is complete.  
* `onUpdate` is called when list is updated for example when a user unsubscribes from a notification type. 

```java
public interface SubsListHandler {
    public void onSuccess(ArrayList<HashMap<String,String>> su);
    public void onUpdate(ArrayList<HashMap<String,String>> su);
    public void onError(String error);
}

public void registerSubscriptionHandler(SubsListHandler handler)
```

You can update an individual subscription setting by calling the `setSubscription` method, which will also result in the invocation of your onupdate handler.

```java
Notify.getInstance().setSubscription("recommended_items", "true)
```

##4.13 Preload Params
Our platform provides you ability to download/update content from a url before showing it in response to notification. Your application needs to implement an intentservice that handles following intent.

```xml
<service>
   android:name=".MyTestService"
   android:exported="false" >
   <intent-filter>
       <action android:name="com.notify.preloadservice" />
   </intent-filter>
</service>
```

The service can access preload data as follows:

```java
@Override
protected void onHandleIntent(Intent intent) {
   if (intent != null) {
       final String action = intent.getAction();
       if(action.equals("com.notify.preloadservice")) {

           Random r = new Random();
           Notification notification = intent.getParcelableExtra("notification");
           final NotificationManager notificationManager = (NotificationManager) this.getApplicationContext().getSystemService(Context.NOTIFICATION_SERVICE);
           notificationManager.notify(r.nextInt(10000), notification);
       }
       if (ACTION_FOO.equals(action)) {
           final String param1 = intent.getStringExtra(EXTRA_PARAM1);
           final String param2 = intent.getStringExtra(EXTRA_PARAM2);
           handleActionFoo(param1, param2);
       } else if (ACTION_BAZ.equals(action)) {
           final String param1 = intent.getStringExtra(EXTRA_PARAM1);
           final String param2 = intent.getStringExtra(EXTRA_PARAM2);
           handleActionBaz(param1, param2);
       }
   }
}
```
