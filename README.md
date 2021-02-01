# From Google Play to Huawei AppGallery

If you landed here, I'm confident you:

- know what are the [Huawei Mobile Services (HMS)](https://developer.huawei.com/consumer/en/hms) , Huawei's alternative to [Google Play Services (GMS)](https://developers.google.com/android/guides/overview)
- are aware that - because of [US-China trade war](https://en.wikipedia.org/wiki/China–United_States_trade_war) - the new Huawei Android powered devices (P40 and Mate 30 series upwards) will not have GMS preinstalled anymore, therefore users cannot access Google Play (GP) to download your apps and instead they can do it via [Huawei's AppGallery](https://appgallery.huawei.com/).
- have the desire to onboard AppGallery (AG) with your apps already published on GP.



### Step 1: Create an AppGallery account

Visit: https://developer.huawei.com/consumer/en/appgallery/ and look for the “Sign up” in the upper-right corner of the page. The process will guide you trough the following steps:

1. first, create a Huawei ID (your identity in the Huawei universe, same as the Google account): this ID will be the (future) AG account holder (further explanations [here](https://developer.huawei.com/consumer/en/doc/start/registration-and-verification-0000001053628148) )
2. next, verify your identity as a developer or enterprise; a comparison between the benefits of both roles can be found [here](https://developer.huawei.com/consumer/en/doc/help/developerrights-0000001053174003) and the different requirements to successfully verify the identity, [here](https://developer.huawei.com/consumer/en/doc/start/ibca-0000001062388135) .
3. it takes between 1 to 2 working days to be approved (but the good part is that the account creation is FREE)
4. now the AG account is ready to be used and you can access the portal at the same link used for registration: https://developer.huawei.com/consumer/en/appgallery/ 



### Step 2: Create listings for each app

Similar to GP, you'll basically create a new app in AG -> *My apps* and customize its appearance on the store: upload an icon, descriptions, screenshots and so on. Explanations are provided directly in the portal, but if you ever get stuck you can find the full documentation [here]( https://developer.huawei.com/consumer/en/doc/distribution/app/agc-create_app).

##### What's an AG project and how is it different to an app?

A [project](https://developer.huawei.com/consumer/en/doc/development/AppGallery-connect-Guides/agc-project-introduction) is a collection of resources and settings such as: API management, user and apps data, data storage location, authenticated signing certificates, etc. A project can contain multiple apps that share the same configuration. On AG each app is identified by a unique integer named *appId* . A further requirement - same as on GP - is that the app needs to have an unique package name = [applicationId](https://developer.android.com/studio/build/application-id).

Projects are needed *only* if you plan to integrate HMS at some point.



### Step 3: Publish the apps - as they are - on AppGallery

This step is optional as you can directly skip to *step_4* if you wish to integrate HMS in your apps now.

But there could be 2 reasons why this step would still make sense.



#### Reason 1: could it be that you have no Google Play Services dependencies at all in your apps?

Just a quick reminder: all that's pure Android still works on HMS devices, only the GMS don't.

Generally speaking, any dependency starting with `com.google.android.gms:play-services-` is an indication you rely on GMS; a more or less complete list here: https://developers.google.com/android/guides/setup#split 

Furthermore, parts of Firebase are counting on GMS to function properly: https://firebase.google.com/docs/android/android-play-services (as you can see, many services were recently decoupled from GMS)



#### Reason 2: it's ok for now to distribute your apps only to those Huawei devices still having Google Play Services

There are plenty of users on Huawei devices that still have GMS, devices released before the US trade ban was enforced. Those devices are still quite powerful - which means they will most likely stick around for the time being. Although it's true that users can go to GP to find and download your apps, sometimes they don't - simply because they might be more familiar with AG that benefits from better exposure on those devices.

##### How to distribute the app only on GMS powered devices

Before submitting a new version, specify in the *Review information* text field: `only GMS`. Having trouble locating the field? Instructions [here](https://developer.huawei.com/consumer/en/doc/distribution/app/agc-release_app#h1-1605230887778) , under *Completing other version information*.

Later, when a newer version doesn’t need anymore to be distributed only on GMS (because it supports HMS as well now), change the *Review information* text field to `GMS and HMS`. This is needed only if there was a `only GMS` mention previously, otherwise apps are by default distributed on GMS and HMS devices.

It’s not possible to distribute only on HMS devices. The choices are only 2: `only GMS` and `GMS and HMS` (the default)

##### Signing considerations

Anticipating the next step and how to grow the apps further inside AG it's maybe a good time now to discuss different choices when it comes to app signing.

If you use [Google Play App Signing](https://developer.android.com/studio/publish/app-signing#app-signing-google-play) then it's obvious you will have to use a different signing key for AG as Google controls now the signing key and you have no access to it anymore (you only have the upload key). 

1. Sign with the same key for both GP and AG:

   This makes sense especially if at *step_4* you plan to opt for a GMS&HMS integration (ffw there to read more). Since there is only one build (therefore an unique *applicationId*), it's convenient to have also only one binary - that can be uploaded at the same time to both GP and AG. Furthermore, on Huawei devices still having GMS, both app stores exist for now, but there is no guarantee that GP will still be available in the future. What you want in that case is to have a seamless upgrade experience where both GP and AG - because both stores can update your app since the signature is the same.

   **Don't use the same key** if you know for sure you want to have a different build for HMS, but you wish to keep the same *applicationId* - because competing updates from GP and AG (on devices having both) can make the user experience weird: i.e. version `v` (coming from GP) has Google Maps, the next one `v+1` happens to be from AG, so it uses HMS Maps and maybe later GP updates to `v+2` and users are back on Google Maps.

2. Sign with different keys:

   Opting for this option on purpose makes sense in the situation you wish to distinguish updates coming via AG from the ones from GP for the same *applicationId*. Different signatures means that all updates will come only from the app store used to download the app in the first place. The other store will silently fail to update the app because signatures don't match.
   
   **Warning**: [Google Play Protect](https://developers.google.com/android/play-protect) might block the initial app installation from AG since it detects that the version on GP is signed with a different key. You could try to file an appeal with Google [here](https://support.google.com/googleplay/android-developer/answer/2992033) , but their answers are not always satisfying: the approval process is as obscure and random as the initial app blocking.  To generalize from some data points: apps with lots of downloads on GP have a higher chance of a positive outcome of the appeal.

**To summarize this step**, all you need to do for now is to to complete the app listing at *step_2* by creating a new release and uploading an apk/bundle. More details [here](https://developer.huawei.com/consumer/en/doc/distribution/app/agc-release_app).



### Step 4: Preparations for the HMS integration into your apps

Before using any of the HMS SDKs  (see full offering here: https://developer.huawei.com/consumer/en/hms) there are several configuration steps.

They are detailed [here](https://developer.huawei.com/consumer/en/doc/development/AppGallery-connect-Guides/agc-get-started) and again in this [codelab](https://developer.huawei.com/consumer/en/codelab/HMSPreparation/index.html#0), but let's do a short summary of every step.

##### Create a project in AG

It can have any name (not visible to users) and you need to link to it the app(s) that will share the project's configuration. Create a new project from AG -> *My projects*.

##### Generate a agconnect-services.json

Your app's Android project has to contain a special configuration file - same as for Google Play and the *google-services.json*. This file can be easily generated from AG immediately after creating a new app on the portal or at any time from *App Information* under *General information*. Later changes in the used AG APIs could add additional information to the file, so you might need to re-generate it.

Back to your Android project, place the file under the module that uses HMS SDKs (usually that's the main module  */app*). 

*What if you have multiple build variants and/or flavors that alter the applicationId?*

Most of the time you will have one production flavor (that needs to go live on the store) and other flavors for different testing purposes, each one with a different *applicationId*. For AG they count as different apps (since the *applicationId* is different), therefore, you will need to create in AG, under the same project, a new app entry for every *applicationId*. You *only* have to configure the package name information (equal to the *applicationId*) - no need to fill anything additional. Then generate a different *agconnect-services.json* file for each app and place them under the respective source sets. See an example [here](https://developer.huawei.com/consumer/en/doc/development/AppGallery-connect-Guides/agc-config-flavor).

##### Manage APIs

Go to the current project in AG and under *Manage APIs* you can toggle on/off various APIs. Some will require to generate a new *agconnect-services.json*.

##### Authenticate your app

AG will reject any API requests if your app's signing configuration is unknown to AG. To add (any number of) signing certificate SHA-256 fingerprints, simply go the current project in AG and look under *General information* for *SHA-256 certificate fingerprint* : press the add icon and paste the fingerprints you wish to authenticate. As guessed, the debug variant needs to authenticate as well.

[This](https://developers.google.com/android/guides/client-auth) is a quick guide on how to retrieve the fingerprint for all the signing configurations. Just remember AG accepts only [SHA-256](https://en.wikipedia.org/wiki/SHA-2).

##### Configure the build.gradle files

Into your Android project's <u>root</u> *build.gradle*:

```groovy
allprojects {
    repositories {  
		... 
		maven {url 'https://developer.huawei.com/repo/'}  
	}  
}
...
buildscript {  
	repositories {  
		...
		maven {url 'https://developer.huawei.com/repo/'}  
    }
    ...
    dependencies {  
		classpath 'com.huawei.agconnect:agcp:1.4.2.301' 
	}  
}
```

Into your project's module (i.e. *app/build.gradle*)

```groovy
apply plugin: 'com.huawei.agconnect'
...
dependencies {       
	implementation 'com.huawei.agconnect:agconnect-core:1.4.2.301'     
}
```

Check the newest version of any dependency [here](https://developer.huawei.com/consumer/en/doc/development/HMSCore-Guides/hmssdk-kit-0000001050042513#EN-US_TOPIC_0000001050042513__section19904112219278).



### Step 5: Integrate HMS SDKs in your app

TO BE CONTINUED ...