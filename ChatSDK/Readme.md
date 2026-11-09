# Tilism Android Chat SDK

A ready-made customer chat for Android apps: your users chat with your agents (and chatbot) from
inside your app, and get push notifications for new messages.

- Text, images, videos, voice notes, files, contacts and location messages
- Departments, chatbot flows, agent typing / read receipts, end-of-chat rating
- Look and features configured from the portal; colours can also be overridden in XML
- English and Urdu screens

Current version: **4.0.0**. Version 4 talks to the Chat SDK v2 API; broadcasts / announcements are
no longer supported. Upgrading from 3.x? See [Migrating from 3.x](#migrating-from-3x).

## Contents

1. [Requirements](#requirements)
2. [Installation](#installation)
3. [Initialize the SDK](#1-initialize-the-sdk)
4. [Register the user](#2-register-the-user)
5. [Open the chat](#3-open-the-chat)
6. [Push notifications](#4-push-notifications)
7. [Log out](#5-log-out)
8. [Unread message count](#unread-message-count)
9. [Chat screen options](#chat-screen-options)
10. [Other methods](#other-methods)
11. [Customizing colours](#customizing-colours)
12. [Migrating from 3.x](#migrating-from-3x)

## Requirements

- `minSdk` 26 or higher. The SDK is built with `compileSdk` 36 and Java 21.
- From the portal (**Channels → Chat SDK → App keys**): your channel's **Client ID** and
  **Channel Token**, and the **chat server URL** of your environment.
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/android/client) set
  up in your app, for push notifications. The SDK does not bring Firebase itself.
- Only if the channel has identity verification on: a backend endpoint that computes the
  `userHash` (see [Register the user](#2-register-the-user)).

## Installation

### Option 1: JitPack (recommended)

1. Add the access token to `$HOME/.gradle/gradle.properties`:

    ```properties
    authToken=jp_p6dmteat0vu8e805pm7dl1k5c0
    ```

2. Add the JitPack repository in `settings.gradle`:

    ```gradle
    dependencyResolutionManagement {
        repositories {
            google()
            mavenCentral()
            maven {
                url "https://jitpack.io"
                credentials { username authToken }
            }
        }
    }
    ```

   Older projects that declare repositories in the project-level `build.gradle` put the same
   `maven { ... }` block in `allprojects { repositories { ... } }` instead.

3. Add the dependency to your app's `build.gradle`:

    ```gradle
    dependencies {
        implementation 'com.github.tilismsdk:tilismsdk-android:4.0.0'
    }
    ```

   All the SDK's own dependencies come with it.

### Option 2: Local AAR

1. Download the 4.0.0 AAR (and its sources jar) from
   [TilismTechnologies/AndroidSDKs](https://github.com/TilismTechnologies/AndroidSDKs/tree/main/ChatSDK) and copy
   them into your app's `libs` folder.
2. A local AAR does not bring its dependencies, so add them along with it in your app's
   `build.gradle`:

    ```gradle
    dependencies {
        implementation files('libs/tilismsdk.aar')

        implementation 'androidx.core:core-ktx:1.17.0'
        implementation 'androidx.appcompat:appcompat:1.7.1'
        implementation 'androidx.constraintlayout:constraintlayout:2.2.1'
        implementation 'androidx.cardview:cardview:1.0.0'
        implementation 'androidx.recyclerview:recyclerview:1.4.0'
        implementation 'androidx.drawerlayout:drawerlayout:1.2.0'
        implementation 'androidx.exifinterface:exifinterface:1.4.1'
        implementation 'androidx.browser:browser:1.9.0'
        implementation 'androidx.lifecycle:lifecycle-process:2.10.0'
        implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.10.0'
        implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.10.0'
        implementation 'androidx.fragment:fragment-ktx:1.8.9'
        implementation 'androidx.activity:activity-ktx:1.12.4'
        implementation 'androidx.room:room-runtime:2.8.4'
        implementation 'androidx.media3:media3-transformer:1.8.0'
        implementation 'androidx.media3:media3-effect:1.8.0'
        implementation 'androidx.media3:media3-common:1.8.0'
        implementation 'com.google.android.material:material:1.13.0'
        implementation 'com.google.code.gson:gson:2.13.2'
        implementation 'com.squareup.retrofit2:retrofit:3.0.0'
        implementation 'com.squareup.retrofit2:converter-gson:3.0.0'
        implementation 'com.squareup.okhttp3:okhttp:5.3.2'
        implementation 'com.squareup.okhttp3:logging-interceptor:5.3.2'
        implementation 'pub.devrel:easypermissions:3.0.0'
        implementation 'com.burhanrashid52:photoeditor:3.0.2'
        implementation 'com.vanniktech:android-image-cropper:4.6.0'
        implementation 'com.facebook.fresco:fresco:3.6.0'
        implementation 'com.facebook.fresco:vito:3.6.0'
        implementation 'com.facebook.fresco:vito-view:3.6.0'
        implementation ('io.socket:socket.io-client:2.1.2') {
            exclude group: 'org.json', module: 'json'
        }
    }
    ```

   `tilismsdk/build.gradle` is the source of truth for this list; it also lists the Fresco
   native modules the SDK leaves out to stay small.

### R8 / ProGuard

Nothing to add: the SDK ships its consumer rules, which R8 applies to your app automatically.

## Integration

### 1. Initialize the SDK

Build the client once, in your `Application.onCreate`:

```kotlin
class MyApplication : Application() {
    var telloApiClient: TelloApiClient? = null

    override fun onCreate() {
        super.onCreate()
        telloApiClient = TelloApiClient.Builder()
            .clientId("<YOUR_CLIENT_ID>")
            .channelToken("<YOUR_CHANNEL_TOKEN>")
            .serverUrl("<YOUR_SERVER_URL>")   // defaults to the staging server
            .context(this)
            .notificationIcon(R.drawable.ic_notification)
            .showMessageSnackbar(false)       // optional: in-app banner for new messages
            .build()

        telloApiClient?.setHomeActivity(HomeActivity::class.java)
        telloApiClient?.setGoogleApiKey(getString(R.string.maps_api_key)) // optional
    }
}
```

| Builder method | |
| --- | --- |
| `clientId(String)` | **Required.** From the portal's App keys. |
| `channelToken(String)` | **Required.** From the portal's App keys. |
| `context(Context)` | **Required.** Any context; the application context is kept. |
| `serverUrl(String)` | Chat server of your environment. Defaults to `TelloApiClient.DEFAULT_SERVER_URL` (**staging**), so set it for production. |
| `notificationIcon(Int)` | Small icon of chat notifications (white on transparent). Defaults to the app icon. |
| `showMessageSnackbar(Boolean)` | Shows an in-app banner for new messages while your app is open but the chat screen is not. Off by default. |

`build()` throws `IllegalStateException` when a required value is missing. The client is also
available anywhere through `TelloApiClient.getInstance()`.

`setGoogleApiKey` takes a Google Maps **Static API** key, used for the map preview of location
messages. Without it those messages show no map.

### 2. Register the user

Call `registerUser` when your user logs in, **and on every app start** while they are logged in.
It is cheap when the user already has a session. The chat cannot be opened before it succeeds.

```kotlin
telloApiClient?.registerUser(
    profileId = user.id,              // required: your unique, stable id for the user, 1-50 chars
    name = user.name,
    mobileNumber = user.phone,
    email = user.email,
    fcmToken = fcmToken,              // optional, same as setPushToken()
    userHash = hashFromYourBackend,   // only with identity verification
    fields = mapOf("city" to "Lahore") // answers to pre-chat form fields, by fieldName
) { status ->
    if (status == TelloApiClient.STATUS_OK) {
        // ready: the chat can be opened
    } else {
        Log.w("Chat", "Chat SDK registration failed: $status")
    }
}
```

- `profileId` must be the same every time for the same user, or their chat history is not found.
  Registering a different `profileId` switches the SDK to that user.
- `name`, `mobileNumber` and `email` left `null` keep the values already saved on the server.
- The listener gets `"200"` (`TelloApiClient.STATUS_OK`) on success, otherwise an error message.
  Callbacks run on the main thread.
- `customerType` is ignored and only kept for compatibility; send extra data with `fields`.

**Identity verification.** When it is on for the channel, `registerUser` must get
`userHash = hex(HMAC_SHA256(identitySecret, profileId))`. Compute it **on your backend**; never put
the identity secret in the app.

**Pre-chat form.** If the channel has a pre-chat form, pass the answers in `fields`, keyed by each
field's `fieldName` (read them with `getChannelConfig`, see [Other methods](#other-methods)). The
SDK's own pre-chat form screen is switched off in this version, so the chat opens directly.

**Captcha.** If the channel requires a captcha for new users, set a `CaptchaProvider` that shows
your captcha and returns its token; the SDK ships no captcha widget:

```kotlin
telloApiClient?.setCaptchaProvider { activity, captchaConfigJson, callback ->
    // show your captcha, then:
    callback(tokenOrNullIfCancelled)
}
```

### 3. Open the chat

```kotlin
telloApiClient?.openConversation(
    context = this,          // your activity
    initialMsg = null,       // optional text to start a new chat with
    customData = null,       // optional string (e.g. JSON) sent with the user's first message
    department = null        // optional Department
)
```

What opens:

- the user's current chat, if they have one that has not ended;
- otherwise a new chat with the given `department`;
- with no `department`: the only department or, when `showDepartmentListDialog` is `true`, the
  SDK's department picker.

If the channel has several departments, either turn on the SDK's picker:

```kotlin
telloApiClient?.showDepartmentListDialog = true
telloApiClient?.openConversation(this, null, null)
```

or show your own list and pass the chosen department:

```kotlin
telloApiClient?.getDepartments { departments ->
    // show departments (dptName, name_u for Urdu, dptImage), then:
    telloApiClient?.openConversation(this, null, null, departments[selected])
}
```

When the channel is switched off in the portal, a message is shown instead of the chat. If the user
is not registered, a "chat not available" message is shown.

### 4. Push notifications

While your app is in the foreground the SDK receives messages over its real-time connection. When
the app goes to the background the connection is closed and new messages arrive as FCM pushes.

**a. Forward FCM tokens and messages** from your `FirebaseMessagingService`:

```kotlin
class MyFcmService : FirebaseMessagingService() {

    override fun onNewToken(token: String) {
        TelloApiClient.getInstance()?.setPushToken(token)
    }

    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        val handled = TelloApiClient.getInstance()?.showFcmNotification(
            remoteMessage.data, this,
            remoteMessage.notification?.title, remoteMessage.notification?.body
        ) ?: false
        if (!handled) {
            // not a chat message: handle your own pushes here
        }
    }
}
```

`setPushToken` can be called before `registerUser`; the token is sent once the user is logged in.
To pass the current token on login, get it with `FirebaseMessaging.getInstance().token`.

- While the app is in the background or closed, Android shows chat pushes by itself and
  `onMessageReceived` is not called for them. The SDK registers their notification channel.
- While the chat screen is open, no notification is shown.
- On Android 13+ ask for the `POST_NOTIFICATIONS` permission; without it no notification appears.

**b. Open the chat when a notification is tapped.** Notifications open the activity set with
`setHomeActivity` (your launcher activity if none is set). There:

```kotlin
class HomeActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // ...
        openChatIfFromNotification(intent)
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        openChatIfFromNotification(intent)
    }

    private fun openChatIfFromNotification(intent: Intent?) {
        val sdk = TelloApiClient.getInstance() ?: return
        if (sdk.isTelloNotificationIntent(intent)) {
            sdk.openConversationFromNotification(this, loggedInProfileId, sdk.notificationChatId(intent))
        }
    }
}
```

If the app was closed, `openConversationFromNotification` registers the user first (with a progress
dialog) and then opens the chat.

### 5. Log out

When your user logs out, log them out of the chat too. This ends the session and removes the
device's push token:

```kotlin
telloApiClient?.logout { /* done */ }
```

Their messages stay on the device and are shown again when the same user signs back in. Messages
are deleted only when the user taps **Clear previous chat** in the chat screen, or when a
different user is registered on the device.

## Unread message count

To show a badge on your chat button, set a `MessageCounterListener`. It is called right away with
the current counts and again whenever they change, on the main thread:

```kotlin
telloApiClient?.setMessageCounterListener(object : MessageCounterListener {
    override fun onAllMessageCountUpdate(count: Int) {
        badge.isVisible = count > 0
        badge.text = count.toString()
    }

    override fun onConversationMessageCountUpdate(conversations: List<DepartmentConversations?>?) {
        // per department: it.department, it.conversation?.unreadCount
    }
})
```

The listener is held in memory: set it in your activity and remove it (`setMessageCounterListener(null)`)
when the activity is destroyed.

## Chat screen options

Set these on the client before calling `openConversation`:

| Property | Default | |
| --- | --- | --- |
| `showDepartmentListDialog` | `false` | Show the SDK's department picker when no department is passed. |
| `showLargeSendButton` | `true` | A new or ended chat shows a large "send us a message" button; tapping it shows the input. `false` shows the input directly. |
| `showPreviousMessagesOnly` | `false` | Read-only chat: shows the history, hides the input. |
| `showBundledMsg` | `false` | How `initialMsg` starts a new chat: `true` puts it in the input for the user to edit and send; `false` sends it hidden in front of the user's first message. Only for channels whose chats start with the user's first message. |

And these methods:

```kotlin
telloApiClient?.setLanguage(TelloApiClient.LANGUAGE_URDU)        // or LANGUAGE_ENGLISH (default)
telloApiClient?.setHomeActivity(HomeActivity::class.java)        // opened by notifications
```

## Other methods

```kotlin
// Channel configuration from the portal: settings, pre-chat form fields, departments, availability
telloApiClient?.getChannelConfig { config ->
    config?.preChatForm?.forEach { field -> Log.d("Chat", "${field.fieldName} required=${field.isRequired}") }
    val online = config?.availability?.agentsOnline
}

// The user's profile on the chat server
telloApiClient?.getProfile { profile -> }
telloApiClient?.updateProfile(name = "New name", email = null, mobileNumber = null) { profile -> }

// Departments stored on the device, with their conversations (call off the main thread)
val list = telloApiClient?.departmentConversations

// Told when a message is opened from an SDK notification
telloApiClient?.setNotificationClickListener { messageId, from, messageType, campaignId -> }
```

All callbacks run on the main thread. The full API reference is in the KDoc of `TelloApiClient`
(also bundled as the Javadoc jar).

## Customizing colours

Colours, text sizes and features (attachments, voice notes, edit / delete, location) are set per
channel in the portal and applied by the SDK when the chat opens.

The SDK's default colours can also be overridden by declaring colours of the same name in your
app's `res/values/colors.xml`:

```xml
<color name="toolbar_color">#E7EDF1</color>
<color name="toolbar_title_text_color">#000000</color>
<color name="outgoingMessageBubbleColor">#FBF39A</color>
<color name="outgoingMessageBubbleTextColor">#000000</color>
<color name="incomingMessageBubbleColor">#FDFDFC</color>
<color name="incomingMessageBubbleTextColor">#000000</color>
<color name="timeIncomingColor">#A9ACB6</color>
<color name="send_btn_color_sdk">#00CCAE</color>
<color name="indicator">#009688</color>
<color name="float_buttons">#009688</color>
```

Colours set in the portal take precedence over these.

## Migrating from 3.x

Version 4.0.0 renamed several members:

| 3.x | 4.0.0 |
| --- | --- |
| `Builder.setApplicationContext(context)` | `Builder.context(context)` |
| `Builder.showSnack(Boolean?)` | `Builder.showMessageSnackbar(Boolean)` |
| `Builder.accessKey(...)`, `projectToken(...)`, `CRYPTO_LIB_KEY(...)`, `CRYPTO_LIB_IV(...)` | `Builder.clientId(...)`, `channelToken(...)`, `serverUrl(...)`: new Chat SDK app keys from the portal |
| `updateFcmToken(token)` | `setPushToken(token)` |
| `setLocality(lang)` | `setLanguage(language)` with `LANGUAGE_ENGLISH` / `LANGUAGE_URDU` |
| `returnActivityName(String)` | `setHomeActivity(Class<out Activity>)` |
| `launchConversationFromNotification(...)` | `openConversationFromNotification(...)` |
| `getDepartmentList(listener)` | `getDepartments(listener)` |
| `setNotificationCLickedListener(onNotificationClickListener)` | `setNotificationClickListener(NotificationClickListener)` |
| `isInteractive` | removed |

Broadcasts / announcements are no longer part of the Chat SDK. The Maven coordinates changed to
`com.github.tilismsdk:tilismsdk-android`.
