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
implementation 'io.getplaybook:SDK:1.0.2'
```

`1.1` is current version.

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
```

#### 5. Starting Activities

```kotlin
    // Start main activity
    PlaybookSDK.startMainActivity(context)
    // Start updates activity
    PlaybookSDK.startMainActivity(context)
    // Start academy activity
    PlaybookSDK.startMainActivity(context)
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

    val channelId = "fcm_default_channel"
    val defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
    val notificationBuilder = NotificationCompat.Builder(this, channelId)
        .setSmallIcon(R.drawable.ic_launcher_foreground)
        .setContentTitle("Acme Academy")
        .setContentText(remoteData["body"]) // Your notification content body
        .setAutoCancel(true)
        .setSound(defaultSoundUri)
        .setContentIntent(pendingIntent)

    val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

    // Since android Oreo notification channel is needed.
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val channel = NotificationChannel(channelId, "Playbook Notification Channel", NotificationManager.IMPORTANCE_DEFAULT)
        notificationManager.createNotificationChannel(channel)
    }

    notificationManager.notify(0 /* ID of notification */, notificationBuilder.build())
}
```
