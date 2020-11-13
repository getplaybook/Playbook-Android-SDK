# Playbook #

## Using Playbook ##

### Installation ###

#### 1. Add repositories ####

Add playbook repository in the `build.gradle` file in the root of your project:

```gradle
repositories {
    ...
    maven {
        url = "https://getplaybook.io/repo/"
    }
}
```

#### 2. Add PlaybookSDK module dependencies ####

```gradle
implementation 'io.getplaybook:SDK:1.2.1'
```

`1.2.1` is current version.

#### 3. Turn on Java 8 support ####

If not enabled already, you also need to turn on Java 8 support in all
`build.gradle` files depending on PlaybookSDK, by adding the following to the
`android` section:

```gradle
android {
	…
	defaultConfig {
    		…
    		multiDexEnabled true
	}

	…

	compileOptions {
    		sourceCompatibility JavaVersion.VERSION_1_8
    		targetCompatibility JavaVersion.VERSION_1_8
	}

	kotlinOptions {
    		jvmTarget = "1.8"
	}
}
```

#### 4. Initialize PlaybookSDK

Initialize Playbook SDK with your settings information and open the Playbook Activitiy:

```kotlin

import io.getplaybook.SDK.PlaybookSDK
import io.getplaybook.SDK.PBMainActivity


// Playbook Basic Setting
PlaybookSDK.set(
    "Acme Academy", // Company Name 
    "...", // Client Token
    "...", // User Id
    listOf("..", "..") // User Segment Ids
)

// Extra Settings
PlaybookSDK.set(this, PBExtraSettings(
    mainColor = Color.parseColor("#28ace8"),
    spinnerColor = Color.parseColor("#28ace8"),
    mainTitle = mapOf(
        "tr" to "Acme Akademi",
        "en" to "Acme Academy"
    )
))
```

#### Posible properties of extra settings
Prop | Description | Type | default
------ | ------ | ------ | ------
`spinnerColor` | Color of main loading spinner  | `Color` | Color.parseColor("#5AC8FA")
`mainColor` | Application main color | `Color` |  Color.parseColor("#5AC8FA")
`mainTitle` | Title of main screen | `Map<String,String>` | null
`mainDescriptionText` | Description of main screen | `Map<String,String>` | null
`categoryDescriptionText` | Description of main screen | `Map<String,String>` | null
`QRModule` | State of QRModule | `Boolean` | true
`updatesModule` | State of Update Module | `Boolean` | true
---

#### Locale Your SDK

First of all you should set available languages as shown code block below for the SDK.

```swift
    PlaybookSDK.set(availableLocales: [
        Locale(identifier: "EN"),
        Locale(identifier: "DE"),
        Locale(identifier: "TR")
    ])
```

Current version of the SDK has localizated for only English, Turkish and Arabic yet. But developers can create their localization file from current JSON template. The example template can be found the main dir of the repository.

Please follow the three basic step below to localize the sdk for a new language.

1. Set the available languages.
2. Copy the example json to main application assets.
3. Replace name of file with language short descriptor. For example `de.json`.

#### 5. Starting Activities

```kotlin
    // Start main activity
    PlaybookSDK.startMainActivity(context)
    // Start updates activity
    PlaybookSDK.startUpdatesActivity(context)
    // Start academy activity
    PlaybookSDK.startAcademyActivity(context)
```

#### 6. Remote notification for updates

In your `FirebaseMessagingService` class you have to make this changes to receive and show playbook update notifications:

```kotlin
import io.getplaybook.SDK.PlaybookSDK

...

override fun onMessageReceived(remoteMessage: RemoteMessage) {

    if(remoteMessage.data.containsKey("pb_update_id")) {
        sendPlaybookNotification(remoteMessage.data)
    }

}

...

private fun sendPlaybookNotification(remoteData: Map<String, String>) {

    // Create a pending intent with Playbook feed activitiy
    val pendingIntent = PlaybookSDK.createFeedPendingIntent(this, remoteData["pb_update_id"] as String)

    val channelId = "playbook_channel"
    val defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
    val notificationBuilder = NotificationCompat.Builder(this, channelId)
        .setSmallIcon(R.drawable.ic_launcher_foreground)
        .setContentTitle(remoteData["title"])
        .setContentText(remoteData["body"])
        .setAutoCancel(true)
        .setSound(defaultSoundUri)
        .setFullScreenIntent(pendingIntent, true)
        .setPriority(NotificationCompat.PRIORITY_HIGH)
        .setDefaults(Notification.DEFAULT_ALL)
        .setVibrate(longArrayOf(0L))

    val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

    // Since android Oreo notification channel is needed.
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val channel = NotificationChannel(channelId, "Playbook Notification Channels", NotificationManager.IMPORTANCE_HIGH)
        notificationManager.createNotificationChannel(channel)
        notificationBuilder.setChannelId(channelId)
    }

    notificationManager.notify(0 /* ID of notification */, notificationBuilder.build())
}
```
