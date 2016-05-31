# Layer Android Releases

This repository contains binary distributions of Android products released by [Layer](http://layer.com).

If you have any questions, comments, or issues related to any products distributed via this repository then please contact the team by emailing [support@layer.com](mailto:support@layer.com). Questions about pricing or product roadmap can be directed to [growth@layer.com](mailto:growth@layer.com).

## Layer SDK

This repository contains releases of the Android SDK for interacting with the Layer communications cloud. It provides a simple, object oriented interface to the rich messaging capabilities provided by the platform.  See the [changelog](releases/com/layer/sdk/layer-sdk/CHANGELOG.md) for the latest version and updates.

In order to use the SDK you must be a registered developer with a provisioned application identifier and have configured a backend system to act as an identity provider for your client applications. All aspects of this setup are covered in detail in the [Layer Integration Guide](https://preview.layer.com/docs/integration).

### Installation

The Layer Android SDK can be installed directly into your application by importing a JAR from the filesystem or an AAR via Maven. Quick installation instructions are provided below for reference, but please refer to the [Layer Integration Guide](https://preview.layer.com/docs/integration) for full details and troubleshooting.

#### Maven Installation

The recommended path for installation on Android is via [Maven](http://maven.apache.org/). Maven is a software project management and comprehension tool. It is capable of managing a project's build configuration and dependencies from a central location. You can install the Layer SDK into your project via Maven by adding the following code to your project's `build.gradle` file:

```groovy
apply plugin: 'maven'

dependencies {
    compile 'com.layer.sdk:layer-sdk:0.16.1'
    compile 'org.slf4j:slf4j-nop:1.5.8'
}
```

Once you have successfully built your project, proceed to the [Verifying SDK Configuration](#verifying-sdk-configuration) section below.

#### JAR Installation

If you wish to install the Layer Android SDK directly from a JAR binary, then download the latest JAR release from the [Github project](https://github.com/layerhq/releases-android/tree/master/releases/com/layer/sdk/layer-sdk) and complete the following steps:

1. Drag the JAR file into the /libs directory of your Android Studio application
Navigate to the JAR file in Android Studio navigatior, right click and select "Add As A Library..."
2. Navigate to your build.gradle file and ensure that you include the following:

```groovy
apply plugin: 'maven'

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.google.android.gms:play-services-gcm:7.8.0'
    compile 'org.slf4j:slf4j-nop:1.5.8'
}
```
Once you have successfully built your project, proceed to the [Verifying SDK Configuration](#verifying-sdk-configuration) section below.

### Verifying SDK Configuration

Once you have finished installing the Layer SDK, you can test your configuration by connecting a client to the Layer cloud. To do so, edit your `Application` object's `onCreate()` method to include the code below (note that you must substitute the app ID placeholder text with your actual app identifier):

```java
// Instatiates a LayerClient object
LayerClient.Options options = new LayerClient.Options().googleCloudMessagingSenderId("INSERT-GCM-SENDER-ID-HERE");
LayerClient client = LayerClient.newInstance(this, "INSERT-APP-ID-HERE", options);

// Asks the LayerSDK to establish a network connection with the Layer service
client.connect();
```

Launch your application and verify that the connection is successful. You are now ready to begin authenticating clients and sending messages. Please refer to the [Layer Integration Guide](https://preview.layer.com/docs/integration) for details.

## Atlas SDK

This repository contains releases of the open source Atlas SDK which contains customizable UI components for use with the Layer SDK. See the [changelog](releases/com/layer/sdk/layer-atlas/CHANGELOG.md) for the latest version and updates. The project is at https://github.com/layerhq/Atlas-Android.

### Installation
The Atlas SDK can be installed directly into your application by importing an AAR from the filesystem or via Maven. It may also be included by cloning the repository directly.

#### Maven Installation

The recommended path for installation on Android is via [Maven](http://maven.apache.org/). Maven is a software project management and comprehension tool. It is capable of managing a project's build configuration and dependencies from a central location. You can install the Atlas SDK into your project via Maven by adding the following code to your project's `build.gradle` file:

```groovy
repositories {
    maven { url "https://raw.githubusercontent.com/layerhq/releases-android/master/releases/" }
}

dependencies {
    compile 'com.layer.atlas:layer-atlas:0.2.10'
}
```

#### Local AAR Installation

If you wish to install the Layer Android SDK directly from an AAR file, then download the latest AAR release from the [Github project](https://github.com/layerhq/releases-android/tree/master/releases/com/layer/atlas/layer-atlas) and complete the following steps:

1. In Android Studio, go to File -> New -> New Module... Choose "Import .JAR/.AAR Package".
2. Follow the wizard to import the downloaded AAR file.
3. Navigate to your build.gradle file and ensure that you include the following:

```groovy
dependencies {
    compile project(':layer-atlas-0.2.10')
}
```

Note that additional dependencies may need to be included in the `build.gradle` file when using this method.

#### Cloning 

1. Clone https://github.com/layerhq/Atlas-Android into the project folder
    - Alternatively, add Atlas as a submodule by typing  `git submodule add https://github.com/layerhq/Atlas-Android` followed by `git submodule update --init`.
2. Include the project in the `settings.gradle` file by adding `':layer-atlas'` to the include list.
3. Navigate to your build.gradle file and ensure that you include the following:

```groovy
dependencies {
    compile project(':layer-atlas')
}
```

## Contact

You can reach the Layer team at any time by emailing [support@layer.com](mailto:support@layer.com).

## License

Layer SDK for Android is licensed under the [Layer SDK License](LICENSE.md).
