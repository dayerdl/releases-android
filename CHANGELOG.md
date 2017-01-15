# Android SDK Change Log

## 0.23.5

### Features
   * Add support for imported data

## 0.23.4

### Bug fixes
  * Not reporting errors that occurred in past sync iterations because they have already been reported (APPS-2671)
  * Allowing de-authentication when in an authentication challenged state (APPS-2672)
  * Fixed crash when syncing rich content under the 2KB lower limit (APPS-2667)
  
## 0.23.3

### Bug fixes
  * Switched to ConcurrentHashMap to store Conversation metadata (APPS-2639)
  * De-authenticating with a `DeauthenticationAction` runs on a separate thread (APPS-2635)

## 0.23.2

### Bug fixes
  * Fixed certificate clearing on de-authentication causing sync to get stuck (APPS-2596)
  * Improved logging during authentication and de-authentication
  * Fixing redundant TLS credential creation

## 0.23.1

### Bug fixes
  * Support for multiple FCM senders (APPS-2606)
  * Fixed crash when receiving an FCM token refresh before new LayerClient.Options had been saved (APPS-2607)
  * Sending FCM token to Layer server on same thread that received the token refresh then closing the LayerClient
  
## 0.23.0

### Features
  * Support for Firebase cloud messaging

### Migration Guide
  * Both Firebase and Google cloud messaging are now supported. You no longer need to pass the sender ID to the Layer SDK. This is now read from the `google-services.json` file in your app. You can generate this file from either the Firebase or Google developer console. This may change your API key so be sure to update/add this in your Layer developer console.
  * Add the `google-services` dependency to your root level `build.gradle` file:

        buildscript {
            // ...
            dependencies {
                // ...
                classpath 'com.google.gms:google-services:3.0.0'
            }
        }

  * Apply the `google-services` plugin at the bottom of your module's `build.gradle` file:

  	apply plugin: 'com.google.gms.google-services'
    
  * Add "com.google.firebase:firebase-messaging:9.6.1" into your app/build.gradle

  * Note that it may be necessary to update the `play-services-gcm` library.
  * The `GcmBroadcastReceiver` and `GcmIntentService` have been removed. Remove the `com.layer.sdk.services.GcmBroadcastReceiver` and `com.layer.sdk.services.GcmIntentService` components in your `AndroidManifest.xml`.
  * Add the following to your `AndroidManifest.xml`:

        <service
            android:name="com.layer.sdk.services.LayerFcmService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
        <service
            android:name="com.layer.sdk.services.LayerFcmInstanceIdService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
            </intent-filter>
        </service>

  * If you need to handle GCM/FCM messages you can extend `LayerFcmService`. In `onMessageReceived()`, the following code should execute first to ensure Layer push messages get handled:

        if (isLayerMessage(remoteMessage)) {
            // Handle message in Layer SDK
            super.onMessageReceived(remoteMessage);
            return;
        }

  * You should be able to continue using existing `GcmReceiver` and `GcmListenerService` as they still receive intents. Thorough testing should be done to verify this behavior.
  * `LayerClient.Options.googleCloudMessagingSenderId(String)` has been replaced with `LayerClient.Options.useFirebaseCloudMessaging(boolean)`. This should be set to true if using FCM or GCM.
  * `LayerClient.setGcmRegistrationId(String, String)` has been removed since sender IDs are read from the JSON file and registration is now handled by the Firebase SDK.
   * If using multiple sender IDs, manually merge the `google-services.json` files by comma separating the `project_number` and adding the additional client(s) to the `client` JSON array.
  * Change `PushNotificationPayload.fromGcmIntentExtras()` to `PushNotificationPayload.fromLayerPushExtras()` when extracting data from a `Layer.PUSH` action.

## 0.22.0

### Features
  * Support for Identities

### Bug Fixes
  * Updates to logs
  * Fixed a crash when receiving a push message with a null extras bundle (APPS-2571)

### Migration Guide
  `Identity` objects have mostly replaced the use of user ID strings. If a user ID is needed in a place where an `Identity` object is returned, call `Identity.getUserId()`.

#### `LayerClient`
  * `newConversation(...)` methods now take `Identity` objects. Replace these calls with `newConversationWithUserIds(...)` to keep using the user IDs. Otherwise, `Identity` objects will need to be used in the method arguments.
  * `getAuthenticatedUserId()` has been removed. Replace these calls with `getAuthenticatedUser()`. This new method returns an `Identity` object, which can be null in the case of an un-authenticated session.

#### `LayerTypingIndicatorListener`
  * The method signature of `onTypingIndicator()` has changed. The third parameter is now a full `Identity` object instead of just the user's ID.

#### `Conversation`
  * `addParticipants(...)` methods now take `Identity` objects. Replace these calls with `addParticipantsByIds(...)` to keep using the user IDs. Otherwise, `Identity` objects will need to be used in the method arguments.
   * Also note that the arguments for these methods are now `Set`s instead of `List`s. This more accurately reflects the particpants since there are no duplicates and order doesn't matter.
  * `removeParticipants(...)` methods now take `Identity` objects. There is no helper method that takes user IDs as it is always expected to use `Identity` objects when removing participants.
  * `getParticipants()` now returns a collection of `Identity`s instead of user IDs. Iterators over this collection will need to be updated.
   * Also note that a `Set` is returned instead of a `List` to more accurately reflect the collection.

#### `Message`
  * The `Actor` class has been removed. `getSender()` now returns an `Identity` object. This is nullable as the sender is not set until a message is sent or received (e.g. calling `getSender()` immediately after `LayerClient.newMessage()`).
  * `getRecipientStatus()` now returns a `Map` of `Identity` objects instead of user IDs. Any iterators on this `Map` will need to be updated.
  * Getting a specific `RecipientStatus` by a user (`getRecipientStatus(...)`) now takes an `Identity` object rather than a user ID.

#### `MessageOptions`
  * Setting and retrieving user `PushNotificationPayloads` are set and accessed by `Identity` objects rather than user IDs.
   * `userPushNotificationPayload()`, `userPushNotificationPayloads()` and `getUserPushNotificationPayload()` now take `Identity` objects in their method arguments.
   * `getUserPushNotificationPayloads()` now returns `Identity` objects as the `Map` keys.

## 0.21.3

### Features
  * Update to Google Play Services `9.2.0`

### Bug Fixes
  * Remove requirement to have extra participants while creating conversation (APPS-2486)
  * Fixed an issue in Sync, with log "Stream Id is null" (APPS-2496)
  * Fixed an issue where ListViewController.PreProcessCallback.onCache was missing for item 0 (APPS-2513)
  * Updates to logs
  
## 0.21.2

### Features
  * Support for Android N
  * Increased SSL timeouts to improve performance in poor networks
  
## 0.21.1

### Features
  * Publishing Javadoc.

## 0.21.0

### Features
  * Added support for advanced customization of push notification messages. Push notification
    options are now added by creating `PushNotificationPayload` objects and setting them through the
    appropriate `MessageOptions` methods. `PushNotificationPayload.fromGcmIntentExtras()` helps to
    avoid querying the GCM intent extras directly when receiving a push notification and allows
    working with a POJO instead. This is a breaking change and existing `MessageOptions` calls
    and push notification receivers will need to be updated.
  * Added `LayerClient.waitForContent` which waits for a particular Layer object to be synced to device.
  * Change `Message`'s `receivedAt` time. Sets `receivedAt` to reflect the time the message
    was received by this user (instead of the time it was received by the particular device). In
    case of messages sent from a device, it will have the time at which message was sent by user
    i.e., current behavior. (APPS-2405)
  * Added support for querying by conversation metadata.
  * Removed deprecated api `LayerClient.getUnreadMessageCount`.

### Bug Fixes
  * Fixed a `ClassCastException` warning in `LayerClient.sendLogs()`
  * Fixed a null pointer exception in CacheWindow (APPS-2449)
  * Queries for object IDs are now case insensitive (APPS-2106)

### Migration Guide
  * To support the new `PushNotificationPayload`, any instances of

        new MessageOptions().pushNotificationMessage(message).pushNotificationSound(sound);

    will need to be changed to

        PushNotificationPayload payload = new PushNotificationPayload.Builder()
                .text(message)
                .sound(sound)
                .build();
        new MessageOptions().defaultPushNotificationPayload(payload);

  * Similarly, `MessageOptions.pushNotificationMessage(userId, message)` should be replaced by
    `MessageOptions.userPushNotificationPayload(userId, payload)`.
  * Push notification receivers will also need to change. Use the following to extract the message
    and sound from the extras in the `Intent`:

        PushNotificationPayload payload = PushNotificationPayload.fromGcmIntentExtras(intent.getExtras());
        String message = payload.getText();
        String sound = payload.getSound();

  * Replace `LayerClient.getUnreadMessageCount` with either `executeQueryForCount` OR
    `Conversation.getTotalUnreadMessageCount()` based on requirements.

## 0.20.4

### Features
  * Added `deauthenticate(DeauthenticationAction)`, that allows keeping user's local data for
    resuming when they authenticate again.

### Bug Fixes
  * Fixed an issue with deleting conversation for `ALL_MY_DEVICES` (APPS-2404).
  * Changed `Conversation.syncAllMessages()` / `Conversation.syncFromEarliestUnreadMessage()` /
    `Conversation.syncMoreHistoricMessages()` to be ignored for new `Conversations`.


## 0.20.3

### Bug Fixes
 * Fixed an issue where conversations were deleted in local device due to transport errors (APPS-2389)
 * Fixed `LayerException: STREAM_DELETED` from being logged when stream is already deleted (APPS-2394)

## 0.20.2

### Bug Fixes
 * Fixed exception "column stream_id is not unique while persisting conversations" (APPS-2369).
 * Ensured distinct conversation check always includes the currently authenticated user (APPS-2373).
 * Fixed `IllegalArgumentException` exception "cannot delete a deleted changeable" (APPS-2379).


## 0.20.1

### Bug Fixes
 * Fixed exception spam in logs caused by connection issues (APPS-2362)
 * Fixed Stream has no database identifier exception (APPS-2366)

## 0.20.0

### Features
 * Added `DeletionMode.ALL_MY_DEVICES`, which replaces `DeletionMode.LOCAL`.  `ALL_MY_DEVICES` is a
   synchronized deletion of conversations and messages for the authenticated user.  This deletion
   mode will delete content across a user's devices and between logins.  When the user receives a
   new message for a conversation they applied `ALL_MY_DEVICES` to, the conversation resurrects from
   that message forward until deleted again.

### Bug Fixes
 * Infinite sync loop from pushed content from deleted conversation (APPS-2332).
 * Fixed `IllegalArgumentException` "Stream with no client_id set" (APPS-2327).

## 0.19.5

### Features

### Bug Fixes
 * Funnel `SyncMaster` exceptions through `LayerSyncListener.onSyncError()` (APPS-2298).
 * Clear listeners later when disposing `LayerClient` so `LayerObjectExceptionListeners` can handle
   errors while disposing (APPS-2298).

## 0.19.4

### Features
 * Android M: Changed `targetSdkVersion` and `compileSdkVersion` to 23, and removed `org.apache`
   dependency (APPS-2263).

### Bug Fixes
 * Corrected LayerClient configuration check with `android.permission.GET_ACCOUNTS` (APPS-2288).
 * Corrected from `android:maxSdkVersion="15"` to `android:maxSdkVersion="18"` since `maxSdkVersion`
   was introduced in Android API 19.
 * Fixed an issue where Historic sync was stuck. (APPS-2295)

## 0.19.3

### Features
 * Added `android:maxSdkVersion="15"` to `android.permission.GET_ACCOUNTS`.  `GET_ACCOUNTS` is not
   required by Google Cloud Messaging for API 16+.

### Bug Fixes
 * Fixed inability to recover runtime authentication challenges (e.g. `SESSION_NOT_FOUND`)
   introduced in `0.19.2` (APPS-2287).

## 0.19.2

### Features
 * Added verbose logging to TLS negotiation (APPS-2240).
 * Bumped GCM to `com.google.android.gms:play-services-gcm:8.4.0`.

### Bug Fixes
 * Fixed  `NullPointerException` in SyncRecon line 348 (APPS-2267).
 * Added `null` checks to `LayerClient.getConversationsWithParticipants()` (APPS-2268).
 * Fixed `NOT_AUTHENTICATED` exception in GcmIntentService (APPS-2282)

## 0.19.1

### Features
 * Reduced bandwidth usage during sync operations.

### Bug Fixes
 * Fixed `NullPointerException` in `AppStateMonitor` (APPS-2200).
 * Fixed `Invalid migration name` in `Migration` (APPS-2205).
 * Fixed stack overflow exception in database corruption handler (APPS-2207).
 * Fixed authentication challenge loop in poor network conditions (APPS-1964).
 * Fixed `event_not_found` error when synchronizing clients with a distinct conversation containing
   only themselves (APPS-2259).

## 0.19.0

### Features
 * All listeners have BackgroundThread versions for receiving callbacks on a background thread
   (APPS-2165).
 * LayerChangeEventListener.MainThread is removed; LayerChangeEventListener handles main thread
   callbacks.
 * LayerChangeEventListener signature changed from onEventAsync / onEventMainThread to
   onChangeEvent.
 * Background external content exceptions now reported through `LayerObjectExceptionListener`
   (APPS-1956).
 * Changed `LayerClient.enableLogging()` and `LayerClient.disableLogging()` to single
   `LayerClient.setLoggingEnabled()` setter and added `LayerClient.isLoggingEnabled()` for returning
   the current Layer SDK programmatic logging state.
 * Added `LayerClient.getVersion()` for returning the Layer SDK's version name.

### Bug Fixes
 * Fixed `FAILED_API_ACTION (9003) - LayerObject cannot be null` (APPS-2169).


## 0.18.2

### Features
 * Added `PARTICIPANT_COUNT` as a queryable property on Conversation (APPS-2159).

### Bug Fixes
 * Fixed `SQLiteDatabaseLockedException` in `DBPool` line 131 (APPS-2043).
 * Fixed `ArithmeticException` exception in `GetEventsTask.java` line 647 (APPS-2157).
 * Improved logging around database migration failures (APPS-2145).

## 0.18.1

### Features
 * Improved performance for sync operations by reducing memory and time consumed.

### Bug Fixes
 * Fixed divide-by-zero in GetEventsTask. (APPS-2130)
 * Fixed locally-generated Conversation totalMessageCount and totalUnreadMessageCount change events
   in response to marking messages read, deleting messages, and sending messages. (APPS-2133)
 * Fixed multi-deletion exceptions in Message and Conversation. (APPS-2138)
 * Added handling for `null` intent in `WakefulBroadcastReceiver`. (APPS-2124)

## 0.18.0

### Features
 * Improved `RecyclerViewController` and `ListViewController` scrolling performance.
   * Added `updateBoundPosition(int position)` to RecyclerViewController and ListViewController.
     When called from ListView and RecyclerView View-binding methods, this method pre-caches
     surrounding LayerObjects (the Conversations, Messages, and MessageParts associated with the
     controller's Query) into the LayerClient cache from a background thread.  LayerObjects are then
     readily available from the LayerClient cache when scrolling into view.
   * Added optional `PreProcessCallback` to RecyclerViewController and ListViewController, called on
     a background thread, for parsing and caching content (e.g. MessagePart data and Metadata) prior
     to binding.  This callback works in conjunction with `updateBoundPosition(int position)` to
     allow long-running content parsing to take place off the UI thread.
 * Updates to `LayerSyncListener` notifications
   * Introduced `SyncType`. With this change `LayerSyncListener` notifications are raised for all sync operations.

### Bug Fixes
  * Fixed ConcurrentModificationException in PolicyManager line 61 (APPS-2114).

## 0.17.1

### Bug Fixes
  * Fixed an issue where Metadata was not updated after first sync.
  * Fixed an issue where `Conversation.syncMoreData(...)` results in some messages missing.
  * Fixed a `LayerObjectException` that could happen when invoking API's when `LayerClient` is `deauthenticated`.
  * Fixed an issue with `Conversation.getHistoricSyncStatus()`. New `Conversation`s start with `INVALID`.

## 0.17.0

### Features
 * Historic synchronization
     * The LayerClient defaults to only synchronizing in each Conversation's Message history from
       the earliest unread message to present.
     * Other synchronization policies are available via LayerClient.Options.historicSyncPolicy().
     * For past behavior, where all history is synchronized, use `HistoricSyncPolicy.ALL_MESSAGES`.
     * Added methods to `Conversation` for synchronizing down more message history.
     * Updated `LayerSyncListener` to raise `onBeforeSync` and `onAfterSync` only for the sync
       operation when a client fetches data from server for first time after authentication (e.g.
       fetching history).
     * Added `onSyncProgress` to `LayerSyncListener` which provides a historic synchronization
       progress.
     * Added `getTotalMessageCount()` and `getTotalUnreadMessageCount()` to `Conversation`.
     * `Conversation` raises change events for new `totalMessageCount`, `totalUnreadMessageCount`,
       and `historicSyncStatus` attributes.
     * More details at: https://developer.layer.com/docs/android/integration#historic-synchronization

### Bug Fixes
  * Fixed an issue that caused delay in creating LayerClient after a push notification.
  * Fixed an issue with cursor for SQLite during migration (APPS-2016).
  * Fixed an issue with `null` metadata that caused sync to fail.

## 0.16.1

### Features
 * Added `proguard.txt` to AAR (APPS-1828).
 * Lowered thread priority of background operations (APPS-1962).
 * ListView and RecyclerView controllers allow Queries to change after construction (APPS-1982).
 * ListView and RecyclerView controllers have `getPosition(item, lastPosition)` for efficiency.
 * Bumped GCM to `com.google.android.gms:play-services-gcm:7.8.0`.

### Bug Fixes
 * Synchronize in response to remote Conversation Metadata changes (APPS-1977).
 * Allow external content less than 2k (APPS-1976).
 * Better transport channel management.

## 0.16.0

### Features
 * Added `ListViewController` designed to drive a `ListView` `Adapter` based on a `Query`.  This is
   in addition to the existing `RecyclerViewController`.
 * `LayerClient.newRecyclerViewController()` takes a `RecyclerViewController.Callback` instead of a
   `QueryController.Callback`.
 
### Bug Fixes
 * Fixed `MessagePart.getMessage()` returning `null`.
 * Fixed `MessagePart.download()` NPE.
 * Fixed memory leak with `LayerReceiver`.
 * Fixed memory leak with `ConnectionManager`.
 * Fixed `Changeable cannot be null` exception at ChangeableTransaction.java:64 (APPS-1883).
 * Fixed `Foreground changeable cannot be null` exception at ChangeableCacheReconciler.java:167 (APPS-1946).
 * Fixed NPE at ChangeableCacheReconciler.java:85 (APPS-1884). 
 
## 0.15.2

### Features
 * Added `close()` and `isClosed()` to LayerClient.  These are used to release resources when the
   client is no longer needed.
 * Renamed all `WeakReference` listener interfaces to `Weak`.

### Bug Fixes
 * Corrected LayerChangeEventListener.BackgroundThread.Weak not being on a background thread.
 * Fixed various database exceptions when resuming (APPS-1822, APPS-1878).
 * Fixed `Cannot process a receipt for EventType` exception (APPS-1931).
 * Fixed NPE in Transport.java line 398 (APPS-1867).
 * Fixed `IllegalStateException` race condition when creating distinct conversations.
 * Do not start synchronizing when deauthenticating and a network connection connects (APPS-1941).
 * Do not throw exception when unregistering a LayerProgressListener and no session exists.

## 0.15.1
 * Fixed "stuck" clients due to pre-0.15.0 race condition.
 * Logging respects e.g. `adb shell setprop log.tag.LayerSDK VERBOSE` again.

## 0.15.0

### Features
 * App ID format changed from UUID string to Uri string, in the forms:
    * `layer:///apps/staging/{UUID}`
    * `layer:///apps/production/{UUID}`
 * Added per-user custom push notifications to `MessageOptions` (APPS-1708).
 * Removed deprecated methods.
 * Added Message.getOptions(), and `MessageOptions` can be modified prior to sending (APPS-1840).
 * Added queryable Conversation `LAST_MESSAGE_SENT_AT` property (APPS-1553).
 * Changed Layer SDK log format to have a single `LayerSDK` tag (APPS-1756).
 * Created type-safe `Metadata`, added Conversation.putMetadata(Metadata, boolean), and deprecated
   Conversation.putMetadata(Map, boolean) (APPS-1810).
 * Removed timer-based polling.
 * Added LayerObjectExceptionListener and LayerClient.registerObjectExceptionListener() for handling
   exceptions with Conversation, Message, and MessagePart objects during background operations.  If
   no listeners are registered, such exceptions are thrown as before.  If a listener is registered,
   the exceptions are not thrown, and are passed to the listeners, where the object that caused the
   exception can be retrieved with LayerObjectException.getLayerObject().
 * All listeners have optional WeakReference sub-interfaces (for example,
   `LayerChangeEventListener.MainThread.WeakReference`) which the LayerClient registers as weak
   references.  These listeners get garbage collected - without explicitly unregistering from the
   LayerClient - when stronger external references to them drop out (APPS-1800).

### Bug Fixes
 * Fixed `SQLiteConstraintException: columns stream_database_identifier, seq are not unique`
   (APPS-1647).
 * Fixed setting message positions on conversations with no messages yet synchronized (APPS-1803).
 * Announcement recipient map contains the authenticated user (APPS-1792).
 * Announcement push Intents contain `layer-message-id` (APPS-1819).
 * Removing a Metadata key results in synchronization (APPS-806). 
 * Fixed NPE in Transport.java:492 when deleting Conversations.
 * Fixed NPE in Helper.java:122 when deleting Conversations.
 * Fixed NPE in OutboundRecon.java:590 when deleting Messages.
 
## 0.14.0
 * Added `Announcement` (extends `Message`) for user-specific announcements.  Event listeners now
   include LayerObject.Type.Announcement, and Queries work for `Announcement.class`.
 * Added support for `Distinct` conversations. New `Conversation` objects will be distinct by
   default.  Adding / removing participants will turn off distinct for the specific `Conversation`.
 * Removed Message.setMetadata().
 * Added MessageOptions as an optional parameter to LayerClient.newMessage(), with methods for
   setting push message and sound.
 * Bumped GCM to `com.google.android.gms:play-services-gcm:7.5.0`.
 * Removed log4j dependency (with `slf4j-nop`)
 * Fixed IndexOutOfBounds exception in Byte.valueOf() on certain devices

## 0.13.3
 * Removed requirement for registering the
   `<receiver android:name="com.layer.sdk.services.LayerReceiver">` in `AndroidManifest.xml`.
 * Added static `LayerClient.applicationCreated(Application)` method.  Call this method from your
   `Application.onCreate()` to improve foreground/background state recognition.

## 0.13.2
 * Allow MessagePart data access before sending for MessageParts not constructed with InputStreams.
 * Removed `allowBackup` reference from `AndroidManifest.xml`

## 0.13.1
 * Removed `android.permission.RECEIVE_BOOT_COMPLETED` permission requirement.
 * Bumped GCM to `com.google.android.gms:play-services-gcm:7.3.0`.
 * Fixed a bug in Message RecipientStatus.
 * Enforcing size limit per message part of 2GB, and count limit of 1000 MessageParts per message.
 * Preventing access to MessagePart data before the message is queued for sending.

## 0.13.0
 * Added `Actor` class for supporting system messages.
 * Changed `String Message.getSentByUserId()` to `Actor Message.getSender()`.
 * NOTE: `Actor` is an undocumented feature to support system messages sent via the Platform API,
   which is currently in closed beta. Documentation will be updated in the coming weeks. `String
   Message.getSender().getUserId()` can be used to replace `String Message.getSentByUserId()`.
 * Instantiating the `LayerClient` supports taking a String with the App ID directly:
   `LayerClient layerClient = LayerClient.newInstance(context, "App ID");`

## 0.11.6
 * Fixed a crash when conversations are deleted locally (APPS-1500).
 * Fixed default content settings in `LayerClient.Options`.

## 0.11.5
 * Set LayerClient.Options for `autoDownloadMimeTypes`, `autoDownloadSizeThreshold`, and
   `diskCapacity` without being authenticated.
 * Removed `android.permission.READ_PHONE_STATE` permission requirement.
 * Fixed `null` device ID.
 * Fixed `MessagePart.getData()` returning `null`.

## 0.11.4
 * Enforcing mime-types to have the format of `*/*`.
 * Fixed `getMessageParts()` on `Conversation.getLastMessage()` returning `null`.

## 0.11.3
 * Fixed remote metadata update bug.
 * Added LayerChangeEvent for MessagePart changes.
 * Added querying on MessageParts.
 * Fixed a bug where MessagePart content would not be deleted from disk on Message deletion.

## 0.11.2
 * Fixed `column object_identifier is not unique` exception in InboundRecon.
 * Pre-fetch data during push notifications while in background.
 * Bumped GCM to `com.google.android.gms:play-services-gcm:7.0.0`.
 * Fixed an issue that caused PERSISTENCE_CLOSED exception during Recon (APPS-1281).
 * Updated Logging. Changed SendLogs to include db and removed APIs that controlled log level and
   info.
 * Model operations now throw typed LayerExceptions rather than IllegalArgumentExceptions.
 * Fixed `Push event with no client seq encountered` exception.
 * Fixed bug with foreign keys that could disable cascade deletion in certain cases. Added a
   software migration for existing databases that have extra entries.

## 0.11.1
 * Improved synchronization performance.
 * Improved UI performance by de-prioritizing background tasks.
 * Added `RecyclerViewController.getPosition(Tquery queryable)`.

## 0.11.0
 * Removed Message `index` in favor of more efficient `position`.  Both `index` and `position` can
   order messages within a conversation, but `index` was an absolute index, while `position` is
   relative.  To maintain its 0-based, monotonically-increasing nature, `index` required periodic
   updates, where `position` requires no updates.
 * Added logged check for GcmIntentService.

## 0.10.2
 * Fixed "Could not create content directory" exception.

## 0.10.1
 * Null check for sending a large message without a LayerProgressListener attached.

## 0.10.0
 * Added external content: MessageParts less than or equal to 2KB are treated as before, while those
   greater than 2KB are uploaded to cloud storage.  LayerProgressListener monitors upload and
   download progress.  MessagePart status can be monitored with `isContentReady()`, and downloaded
   with MessagePart.download().  Various disk capacity and auto-download options are available
   through LayerClient.

## 0.9.9
 * Fixed remote metadata update bug.
 * Fixed `column object_identifier is not unique` exception in InboundRecon.
 * Fixed `Push event with no client seq encountered` exception.
 * Fixed bug with foreign keys that could disable cascade deletion in certain cases. Added a
   software migration for existing databases that have extra entries.

## 0.9.8
 * Message senders set the sent message's `receivedAt` immediately to a local timestamp (APPS-1222).
 * Block policy synchronization settles with more consistent callbacks.
 * LayerClient constructor accepts an new optional Options class which takes the place of the
   GCM sender parameter, and allows for a new option to broadcast PUSH while in the foreground.

## 0.9.7
 * Fixed metadata `Null URI` bug when processing metadata for deleted conversations.
 * Metadata keys validated as case insensitive alpha-numeric, dash and underscore.

## 0.9.6
 * Fixed metadata `Unknown value type` bug.

## 0.9.5
 * GET_TASKS permission is optional.  If the permission is available, pre-Lollipop devices use it to
   determine initial foreground or background state during initialization.  Otherwise, LayerClient
   assumes it is initialized in the foreground.
 * Synchronization efficiency improved.

## 0.9.4
 * Fixed retrying conversation metadata attempts that result in UNAUTHORIZED errors. (APPS-1157)
 * LayerClient sets its foreground/background state at instantiation time based on the current top
   task in Android SDK < 21, or by process in 21 and above (PLAT-595).  This requires a new
   permission in APIs below 21:
      <uses-permission android:name="android.permission.GET_TASKS"/>
 * Fixed a bug with Blocking policies when the server would incorrectly overwrite local policy list.

## 0.9.3
 * Added blocking policies to LayerClient.
 * Added logging to LayerClient.

## 0.9.2
 * Challenges are suppressed during deauthentication.
 * Corrected migrations to prevent SQLiteParser exceptions.

## 0.9.1
 * Improved session expiration and other authentication recovery scenarios.
 * Fixed participant count mismatch between devices when removing participants from devices that did
   not add said participants (APPS-1064).

## 0.9.0
 * Moved Conversation, Message, and MessagePart constructors to LayerClient.
 * Moved Conversation, Message, and MessagePart actions from LayerClient to their models.
 * Deprecated old LayerClient Conversation, Message, and MessagePart actions.
 * Added Query, Predicate, CompoundPredicate, SortDescriptor, etc (APPS-785).
 * Added LayerClient.setGcmRegistrationId(String senderId, String registrationId) for registering
   multiple GCM sender IDs externally.

## 0.8.20
 * Challenges are suppressed during deauthentication.
 * Corrected migrations to prevent SQLiteParser exceptions.

## 0.8.19
 * Improved session expiration and other authentication recovery scenarios.
 * Fixed participant count mismatch between devices when removing participants from devices that did
   not add said participants (APPS-1064).

## 0.8.18
 * Fixed NPE in SyncRecon line 307.

## 0.8.17
 * Fixed "stuck" messages not sending.
 * Improved authentication state management.
 * Fixed NPE in LayerClientImpl.isConnecting() (APPS-1021).

## 0.8.16
 * Database manager drops schema before re-creating if valid managed schema cannot be found
   (APPS-904).
 * Fixed intermittent failure to emit Conversation.lastMessage updates.
 * Added a lock around database creation/deletion so ensure that state remains consistent when
   authenticate/deauthenticate are called in quick succession
 * Mark messages as READ even when delivery receipts are disabled for a conversation.

## 0.8.15
 * Added `ConversationOptions` with the ability to disable delivery receipts.

## 0.8.14
 * Added `LayerClient.getUnreadMessageCount()` for returning unread message counts for a particular
   conversation or all conversations.

## 0.8.13
 * Added heartbeat to typing indicators to handle longer typing sessions (APPS-840).
 * Fixed intermittent NPE in PatchStreamMetadataTaskMaster line 65 (APPS-903, APPS-432).
 * Removed custom `com.layer.sdk.permission.C2D_MESSAGE` permission from SDK to avoid duplicate
   permission issues in Android 5.  A package-specific C2D_MESSAGE permission must be defined in
   your app's manifest, per the getting started guide (APPS-935).

## 0.8.12
 * Bumped compileSdkVersion to 21 (target and min are still 14).
 * Bumped buildToolsVersion to 21.1.2.
 * Prioritize sending messages during synchronization.
 * Removed irrelevant deauthentication errors.
 * Removed `No subscribers registered for event class` logs.
 * Fixed intermittent `PERSISTENCE_CLOSED` exceptions during deauthentication.

## 0.8.11
 * Reverted deauthentication action from 0.8.10 to clearing local data.

## 0.8.10
 * Improved LayerClient initialization time.
 * Deauthenticating no longer clears local cache.  Unique app IDs and user IDs create persistent
   caches.
 * Added check for downgrading database schema (and clearing contents).
 * Fixed intermittent failure to alert change events during sync.

## 0.8.9
 * Removed temporary object IDs.
 * Changed permanent object ID URI format to:
   * layer:///messages/[uuid]
   * layer:///conversations/[uuid]
 * Improved push channel management.
 * Added validation for null Conversation in creating and sending Messages, and sending typing
   indicators.
 * Added validation for empty and oversized participants for Conversations before sending.
 * Added validation for incremental participant additions.

## 0.8.8
 * Added metadata to conversations.
 * Added typing indicators.
 * Reduced database memory footprint.
 * Prevent concurrent database migration errors (SUPP-133).
 * Improved management of foreground/background state (SUPP-128).
 * Minor efficiency improvements.

## 0.8.7
 * LayerClient can be instantiated on a background thread or isolated process.
 * Fixed `No schemas in DataSource set` error when instantiating LayerClient on certain devices
   (SUPP-116).
 * Minor efficiency improvements.

## 0.8.6
 * Fixed intermittent exception in GcmIntentService line 147 (SUPP-110).
 * Corrected issues with large numeric participant IDs.

## 0.8.5
 * Corrected getConversationsWithParticipants() bug introduced in 0.8.4.

## 0.8.4
 * Improved connection management.
 * Improved sync performance.
 * Made looking LayerObjects up by ID case-insensitive.

## 0.8.3
 * More forgiving connection / authentication state.
 * LayerClient connects again if it was connected when the app was killed.

## 0.8.2
 * Added DeletionMode to deleteMessage() and deleteConversation().
 * Improved sync performance.
 * Improved error message for TaskMaster 3->2 error.
 * Fixed marking a message delivered and read after getting re-added to a conversation.
 * Fixed intermittent IllegalArgumentException in Persist line 288.
 * Fixed nested-cursor memory errors.

## 0.8.1
 * Fixed intermittent SQLiteConstraintException.
 * Fixed intermittent NPE in Transport line 297.
 * Fixed intermittent NPE in Transport line 437.

## 0.8.0
 * Updated endpoints to production.

## 0.7.21
 * API actions (sendMessage, deleteConversation, etc.) are asynchronous.  Listen for
   LayerChangeEvents for completion.
 * Improved synchronization efficiency.
 * Added `category` to Layer PUSH broadcast intents to isolate push broadcasts to the current
   package.
 * LayerClient.authenticate() attempts to connect first if not connected.
 * LayerClient.getAppId() returns UUID instead of String.
 * Improved local storage performance.
 * Decreased push overhead.
 * Fixed `layer-message-id` in push notification.

## 0.7.20
 * Fixed skipped first push notification after backgrounding.

## 0.7.19
 * Fixed intermittent crash when resuming closed app.
 * Fixed intermittent crash when synchronizing with poor connectivity.

## 0.7.18
 * Close Layer push channel (not GCM push) after a few seconds of being in the background; resume
   in foreground.

## 0.7.17
 * Fixed re-adding participant to conversation not alerting participant changes.

## 0.7.16
 * Created LayerException in place of error alerts with `int code, String message`.
 * LayerSyncListener has onSyncError for reporting LayerExceptions encountered during
   synchronization.
 * Corrected Conversation `lastMessage` bug.
 * Corrected alerting 'SESSION_NOT_FOUND' as an authentication error.

## 0.7.15
 * Deleting conversations and messages synchronizes faster.
 * Fixed error on synchronizing conversations from scratch that include deleted messages.

## 0.7.14
 * Reduced synchronization time.
 * Send PUSH broadcasts from both GCM and Layer connection.
 * Corrected intermittent NullPointerException on launch.

## 0.7.13
 * Removed LayerNotificationCallback; added new Intent broadcast with `com.layer.sdk.PUSH` action.

## 0.7.12
 * LayerClient.markMessageAsRead().
 * Message.getRecipientStatus() map acts like Message.getRecipientStatus(userId).

## 0.7.11
 * Message.sentAt and Message.receivedAt populated.
 * Recipient status map contains known recipients, conversation participants, and sender.
 * Do not return locally-deleted Conversations, Participants, and Messages prior to synchronization.

## 0.7.10
 * LayerClient takes a UUID instead of a String for its Layer App ID.
 * Listeners moved to new `com.layer.sdk.listeners` package.
 * Services and Receivers moved to new `com.layer.sdk.services` package.
 * LayerChange and LayerChangeEvent moved to new `com.layer.sdk.changes` package.

## 0.7.9
 * Fixed bug in synchronizing a conversation from scratch which had a member deleted.
 * Fixed proguard error on Session.
 * Bumped Google Cloud Messaging to 5.+

## 0.7.8
 * Fixed LayerClient-not-instantiated-in-main-thread bug.

## 0.7.7
 * Managed connection and notification states.
 * Notification callback.
 * getConversationsWithParticipants() bug fix.

## 0.7.6
 * GCM.
 * Notifications.
 * LayerNotificationCallback.
 * Improved sync notifications.
 * Corrected uncached notification bug.
