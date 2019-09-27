# ProximiPro Engage SDK User Guide

Welcome to Engage SDK user guide. Here, you can find all the code snippets and required to use each and every feature of the Engage SDK. You can also view SDK documentation [here](https://verlos.github.io/engage-android-docs/index.html). Let's get started.



## Add Engage to your Project

It's very simple to add engage SDK to your android project. open your project level build.gradle file and add following code into it.

```groovy
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```

Now, open app level build.gradle file found at `app/build.gradle` and add this dependency:

```groovy
dependencies {
    implementation 'com.proximipro:engage-android:0.0.3'
}
```

Sync your project and you're ready to go.

You can find documentation of the SDK [here](https://github.com/verlos/engage-sdk-docs/). Follow this user guide to get started with the SDK.



## Initialization

The SDK needs to be initialized before one can use it. Please note that in order to use this SDK, it must be initialized before using it, otherwise it will throw `SdkNotInitializedException`. To initialize this sdk, following things are required:

* **ProximiPro API Key:** Provided by Engage Platform
* **Client ID:**  Provided by Engage Platform
* **Region Identifier:** [User defined]
* **UUID:** Provided by Engage Platform
* **Project Name/ Application Name:** Used as Notification title for all the notifications displayed from the SDK.



> In order to initialize this SDK for the first time, internet connection is required as it checks and verifies the API key with the server. SDK cannot be intialized without internet connect. Once the API key is verified through internet, it no longer requires internet connection for subsequent runs/initializations.



```kotlin
// Create Initialization request
private val request: InitializationRequest by lazy {
        InitializationRequest(
            apiKey = "API_KEY",
            appName = getString(R.string.app_name),
            clientId = "CLIENT_ID",
            regionId = "REGION_IDENTIFIER",
          	uuid = "UUID"
        )
    }

// Method:1 Kotlin friendly top level function [initializeEngage]
initializeEngage(
application = application, // application instance
request = request, // initialization request object
callback = object : InitializationCallback() {
    override fun onSuccess() {
        // SDK initialized successfully, proceed further
    }

    override fun onError(e: Throwable) {
        // Opps, error initializing the SDK, cannot continue.
    }
})

// Method:2 Use static method to initialize [Java friendly way]
Engage.initialize(
application = application, // application instance
request = request, // initialization request object
callback = object : InitializationCallback() {
    override fun onSuccess() {
        // SDK initialized successfully, proceed further
    }

    override fun onError(e: Throwable) {
        // Opps, error initializing the SDK, cannot continue.
    }
})
```



#### Check if SDK is initialised or not

Once the api key is verified, the SDK will be auto initialized on app start. No need to reinitialize it every time. Here's how you can check whether the SDK is initialized or not

```
if(Engage.isInitialized) { /* SDK is initialized */ }
```



## Usage

To use SDK, there is one single entry point for all kind of operation. The entry point is `Engage` class. This how the singleton instance of the SDK can be retrieved:

```kotlin
// throws SdkNotInitializedException when accessed before initializing
val engage:Engage = getEngage()

// throws SdkNotInitializedException when accessed before initializing
val engage:Engage by lazy{ getEngage() }

// returns sdk instance if it's initialized, null otherwise
val engage:Engage? = getEngageOrNull()

// returns sdk instance if it's initialized, throws SdkNotInitializedException otherwise
val engage:Engage by lazy{ getEngage() }
```



## Register User

To register the user with the SDK, 2 things are needed:

* **BirthDate** of the user `birthDate: Date` A valid Java Date object
* **Gender** of the user `gender:Gender`: 1. `Gender.Male` 2. `Gender.Female`

```kotlin
engage.registerUser(birthDate = birthDate, gender = Gender.Male) onSuccess {
    // User Registered successfully
} onFailure { error ->
    // Unable to register User
}
```



## Tags

Once the user is registered, tags related to the account can retrieved like this:

#### Get User Default Tags

```kotlin
val tags:List<Tag> = engage.config().userConfig.getTags()
```



#### Update User Tags

These tags are selectible so the user can make selection of his interest. Once selection is done, it needs to be synced with the server. For that, `updateUser` api can be used like this:

```kotlin
engage.updateUser(tags = adapter.tags) onSuccess {
    navigateToHomeScreen()
} onFailure {
    showSnackBar(root, "Something went wrong!")
    enableViews(true)
}
```



## Get/Update User Information

Once the user is registered successfully with the SDK, you can retrieve all the user related information from engage config object like this:

```kotlin
val birthDate = engage.config().userInfo.birthDate
val gender = engage.config().userConfig.gender
val tags = engage.config().userConfig.getTags()
```



`updateUser` api can be used to update **Birth Date, Gender and Tags**. 

```kotlin
engage.updateUser(
    gender = gender,
    birthDate = selectedDate,
    tags = adapter.tags
) onSuccess {
    // user info updated successfully
} onFailure {
    // Something went wrong!
}
```



## Starting and Stopping Scan

#### Start Scan

This SDK supports lifecycle aware scans which means that scan results will only be delivered if the is in foreground. Also, upon scan results, SDK shows notifications based on the rules triggered.

> Note that the SDK uses foreground service for beacon scanning. This service will display a notification whenever the scan is started and will remove the notification whenever the scan is stopped. see [***Configure Service Notification***](#configure-service-notification) section to modify default notification settings.



`engage.startScan()` method call requires Location permission to start scanning as underlying native module requires this permission for scanning beacons. Make sure that the app has location permission before starting the scan, otherwise the scanning process won't get started and it won't give any scan results.

```kotlin
// start scan only if it is not already on going
if (!engage.isScanOnGoing()) { 
    // this: LifecycleOwner [Activity | Fragment]
    engage.startScan(this, object : BeaconScanResultListener() {
    override fun onRuleTriggered(rule: Rule) {
        // called when a rule is triggered upon beacon detection
    }

    override fun onBeaconExit(beacon: ProBeacon) {
        // called when a beacon exit event is triggered
    }

    override fun onBeaconCamped(beacon: ProBeacon) {
        // called when a beacon camping event is triggered
    }

    override fun onScanStopped() {
        // called when the beacon scan is stopped
    }

})
}
```



#### Stop Scan

Stopping an ongoing scan is pretty easy. It will remove all the background and forground scan listeners.

```kotlin
engage.stopScan()
```



## Configure Service Notification

Engage uses foreground service for scan process. Now that due to platform restrictions, foreground service needs to display a notification that the scanning process is on-going. This notification uses default configuration which can be modified in the following way.

To modify default settings for scan service notification,

```kotlin
with(engage.config().serviceNotificationInfo){
    content = "Scan is ongoing"
    channelName = "Scan Channel"
    priority = NotificationManager.IMPORTANCE_DEFAULT
    smallIconId = R.drawable.ic_scan_colored
    actionText = "Exit"
}
```

* **`Content:`** Service Notification content.
* **`channelName:`** Notification channel name that will be used to create notification channel for devices running Android 8+.
* **`Priority:`** Importance of the notification.
* **`smallIconId:`** Notification small icon that is displayed in the left top corner of the notification.
* **`actionText:`** Text displayed on Notification action that stops scanning process.



The service notification has a `Stop Scan` button that stops the ongoing scan process. As this notification is shown as a part of foreground service, it cannot be cancelled until the service is stopped.



## Background Mode

This SDK supports background scan mode to keep scanning even when the app is not in foreground. It displays notifications on scan results and those notifications leads back to main app. Settings background mode enabled, it will also start scan on device boot.

#### Enable Background Mode

```kotlin
engage.config().isBackgroundModeEnabled = true
engage.config().isNotificationEnabled = true
```

> In order to enable background scan, both `isBackground `and `isNotificationEnabled` needs to be set to true. If one of them is not set to true, background scan won't start.
>
> Note that in order to apply these changes, scan process doesn't need to be restarted as it is needed for other settings.



## Location Based Content

Location based content enables the SDK to use device location and fetch data based on the location. To enabled location based content:

```kotlin
engage.config().isLocationBasedContentEnabled = true
```



## Configuring SDK

The SDK is highly customizable and there are lots of settings that can be applied to the SDK. All those configurations can be accessed/modified through `EngageConfig` like this:

```kotlin
val config: EngageConfig = engage.config()
```



#### Example

Changing this configuration is very easy. Just assign a new value to it. However there are some properties that cannot be changed directly. Please refer to the docs for more information.

```kotlin
engage.config().appName = "Engage Demo"
```

Please refer to the docs for more information on available customizations [here](https://verlos.github.io/engage-android-docs/com.proximipro.engage.android.core/-engage-config/index.html). 



## Using ContentListView

The SDK provides a list view to load all the supported content when the app is in foreground. Using this `ContentListView` is very simple.

Start by adding the view in your `Activity`/`Fragment`'s xml file:

```xml
<com.proximipro.engage.android.view.ContentListView
    android:id="@+id/contentList"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```



#### Setting content for the view

Once you receive the rule from `onRuleTriggered` method, you can create instances of `Content` class and create a list of it. Then all you need to do is pass the content list to the view and it will load data from the server.

```kotlin
contentList.setContent(contents) // where contents is List<Content>
```



#### Setting Click listener on items

Setting item click listener on `ContentListView` is pretty straight forward. The itemClickListener provides the content object of the clicked item which be used to load content in a separate screen like showing details view.

```kotlin
list.setItemClickListener { content->
    // here you get item clicks from ContentListView
}
```



## Using ContentDetailView

As the SDK provides `ContentListView` to load list of content, it also provides a detailed view of the content. Using `ContentDetailView` is pretty simple. Add the following xml to your detailed `Activity`/`Fragment`'s xml layout file:

```xml
<com.proximipro.engage.android.view.ContentDetailView
    android:id="@+id/contentDetailView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```



#### Loading content in `ContentDetailView`

You can pass the `Content` instance from one component to another as it implements `Parcelable` interface. Once you get the instance of the `Content` class, you can directly set it to the `ContentDetailView`:

```kotlin
contentDetailView.setContent(content)
```

   

## Handle Notifications

The SDK shows notifications based on the rule triggered once the scanning process is started. These notifications when clicked, launches the launcher activity of the application by default.

**Example:** Suppose your app is having this flow: `SplashActivity` > `HomeActivity`. When notification is tapped, it will launch SplashActivity as It is the default launcher activity for the application. 

To handle notification taps in this way, required data will be passed through intent. So in above case, the intent will be received by the `SplashActivity`. then to handle that intent in `HomeActivity`, the intent is needed to be passed to the `HomeActivity`.

#### Custom PendingIntent

A better option could be setting custom `PendingIntent` to handle notification taps. This can be done very easily.

```kotlin
engage.config().pendingIntentClassName = MainActivity::class.java.name
```

or

```kotlin
engage.config().pendingIntentClassName = "com.example.app.MainActivity"
```

> Note that a fully qualified Activity name must be passed in order to get it resolved properly at run time.

This will set `PendingIntent` for the notification taps to be redirected to the provided Activity class.



### Setting Beacon UUID

The sdk uses default beacon UUID provided by ProximiPro Engage platform. However if any requirement arises to change beacon UUID, it can be done very easily:

> Please make sure that the UUID is in right format and is not incorrect.  Also, in order for SDK to apply changes, the app needs to be restarted.

```kotlin
// returns true if the uuid is updated successfully, false otherwise
engage.updateBeaconUUID(uuid) 
```



### Change SDK API key

The SDK provides a way to change the API key used by the SDK. 

> Once the API key is changed, the SDK re-verifies it on the next run to make sure that it is a valid API key. If it fails to verify the new API key then it won't initialize the SDK. Also, for this change to be effective, the app needs to be restarted.



```kotlin
engage.updateApiKey(apiKey) { isVerified ->
    if (isVerified) {
        // api key updated successfully, restart the application
    } else {
        // failed to update api key
    }
}
```



### Log out from SDK

Logging out from SDK stops ongoing scan and resets all the settings to default.

```kotlin
engage.logout()
```