# iProov Android SDK (v3.4.0)

## 🤖 Introduction

The iProov Android SDK provides a programmatic interface for embedding the iProov technology within a 3rd party Android application (“host app”).

The iProov SDK supports Android API Level 16 (Android 4.1) and above, which as of May 2017 encompasses ~98% of active Android devices.

Within this repository you can find the Waterloo Bank sample Android app, which illustrates an examples iProov integration.

## 🛠 Upgrade Guide

### Upgrading from SDK v3.3

All changes from 3.3 are non-breaking. There are now some additional options available to customize the interface presented by the SDK:

```java
IProov.UIOptions options = new IProov.UIOptions()
        .setBackgroundTint(Color.BLACK)         //background colour shown after the flashing stops. Default Color.BLACK
        .setShowIndeterminateSpinner(true)      //when true, shows an indeterminate upload progress instead of a progress bar. Default false
        .setSpinnerTint(Color.WHITE)            //only has an effect when setShowIndeterminateSpinner is true. Default Color.WHITE
        .setTextTint(Color.WHITE)               //only has an effect when setShowIndeterminateSpinner is true. Default Color.WHITE
        .setAutostart(true);                    //instead of requiring a user tap, auto-countdown from 3 when face is detected. Default false
        .setLocaleOverride("");                 //overrides the device locale setting for the iProov SDK. Must be a 2-letter ISO 639-1 code: http://www.loc.gov/standards/iso639-2/php/code_list.php


Intent i = new IProov.NativeClaim.Builder(this)
        .setMode(IProov.Mode.Verify)
        .setServiceProvider("{{api-key}}")
        .setUsername("{{username}}")
        .setUIOptions(options)
        .getIntent();
startActivityForResult(i, 0);
```


### Upgrading from SDK v3.2

In SDK v3.1.0, we moved to explicit Intents with methods to build the Intent for you. We had feedback from users that they wanted more customisation options when creating an iProov request, so we have now moved to a builder-style approach for building the intent which allows a much greater degree of flexibility.

#### Old method (v3.2):

```java
Intent i = IProov.newVerifyUsernameIntent(this, "{{api-key}}", "{{username}}");
startActivityForResult(i, 0);
```

#### New method (v3.3):

```java
Intent i = new IProov.NativeClaim.Builder(this)
        .setMode(IProov.Mode.Verify)
        .setServiceProvider("{{api-key}}")
        .setUsername("{{username}}")
        .getIntent();
startActivityForResult(i, 0);
```

Consult the documentation for the full set of builders and available options that they can accept.

### Upgrading from SDK v3.1 and earlier

Previously, iProov was launched by starting an Intent with the action `IProov.INTENT_IPROOV`. We have now moved to using explicit Intents.

We have also taken this opportunity to simplify the process of launching iProov, so we provide a series of `IProov.newIntent()` methods that builds the Intent for you, via a simple, clean API.

For further information, see the Launch Modes section. An example of the new approach versus the old one is as follows:

#### Old method (pre-v3.2):

```java
Intent i = new Intent(IProov.INTENT_IPROOV);
i.putExtra(IProov.EXTRA_MODE, IProov.Mode.Verify.ordinal());
i.putExtra(IProov.EXTRA_SERVICE_PROVIDER, "{{api-key}}");
i.putExtra(IProov.EXTRA_USERNAME, "{{username}}");
startActivityForResult(i, 0);
```

#### New method (v3.2):

```java
Intent i = IProov.newVerifyUsernameIntent(this, "{{api-key}}", "{{username}}");
startActivityForResult(i, 0);

```

The delivery of the iProov result to your Activity via `onActivityResult()` is unchanged.

### Upgrading from SDK v2.6.5 and earlier

You are no longer required to explicitly initialise iProov with a call `IProov.init(context)` before using iProov in your application, which was introduced in SDK v2.6.4.

You should remove any calls to `IProov.init(context)` from your application.
## 📲 Installation

The Android SDK is provided in AAR format (Android Library Project) as a Maven dependency.

>NOTE FOR ECLIPSE USERS: The SDK is packaged in AAR format. Android Studio (which is now the main Android development environment being promoted by Google) has excellent support for AAR dependencies. For legacy projects still using Eclipse, support for AARs is limited, but can be achieved using various plugins/workarounds. For more information on using AAR dependencies within Eclipse, please see [here](https://commonsware.com/blog/2014/07/03/consuming-aars-eclipse.html) (However, please note that this is unsupported.)

The installation guide assumes use of Android Studio.

1. Open the build.gradle file corresponding to your new or existing Android Studio project with which you wish to integrate (commonly, this is the build.gradle file for the `app` module).

2. Add the repositories section at the top level of your build file:

	```gradle
	repositories {
	   maven { url 'https://raw.githubusercontent.com/iProov/android/master/maven/' }
	}
	```

3. Add the dependencies section at the top level of your build file:

	```gradle
	dependencies {
	   compile('com.iproov.sdk:iproov:3.4.0@aar') {
	       transitive=true
	   }
	}
	```

You may now build your project!

## 🚀 Launch Modes

There are 2 primary ways iProov can be launched for verification or enrollment:

* By being called natively from within a host application.

* From a GCM (push) notification.

iProov is always launched via an Intent from your application.

When starting a new iProov session, the starting point is always to create a new Intent using one of the static Intent-creating methods, as shown below.

### 1. Verify (with Service Provider)

You would use this launch mode where you are a service provider who knows the username you want to authenticate against, but nothing else. iProov will handle the entire end-to-end process of generating a new token and authenticating the user.

```java
Intent i = new IProov.NativeClaim.Builder(this)
        .setMode(IProov.Mode.Verify)
        .setServiceProvider("{{api-key}}")
        .setUsername("{{username}}")
        .getIntent();
startActivityForResult(i, 0);
```

For an explanation of receiving the result from the Intent, see the “Intent Result” section below.

### 2. Verify (with Token)

You would use this launch mode where you already have the encrypted token for the user you wish to authenticate (you may have already generated this elsewhere and now wish to authenticate the user).

```java
Intent i = new IProov.NativeClaim.Builder(this)
        .setMode(IProov.Mode.Verify)
        .setServiceProvider("{{api-key}}")
        .setEncryptedToken("{{encrypted-token}}")
        .getIntent();
startActivityForResult(i, 0);
```

### 3. Enrol (with Service Provider)

You would launch this mode where you are a service provider who wishes to enrol a new user, with a given username.

```java
Intent i = new IProov.NativeClaim.Builder(this)
        .setMode(IProov.Mode.Enrol)
        .setServiceProvider("{{api-key}}")
        .setUsername("{{username}}")
        .getIntent();
startActivityForResult(i, 0);
```

### 4. Enrol (with Token)

You would launch this mode where you are a service provider who wishes to enrol a new user, where you already have the encrypted token for the user you wish to enrol (you may have already generated this elsewhere and now wish to authenticate the user).

```java
Intent i = new IProov.NativeClaim.Builder(this)
        .setMode(IProov.Mode.Enrol)
        .setServiceProvider("{{api-key}}")
        .setEncryptedToken("{{encrypted-token}}")
        .getIntent();
startActivityForResult(i, 0);
```

### 5. iProov with Notification

When the notification is received, you can use a `NotificationClaim.Builder` to attach the `Bundle` from the notification:

```java
Intent i = new IProov.NativeClaim.Builder(this)
        .setBundle(bundle)
        .getIntent();
```

In most cases rather than launch the `Intent` directly, you would then most likely wrap it into a `PendingIntent` and attach it to a Notification to be displayed to the user via the `NotificationManager`.

When launching the `Intent` from a `PendingIntent` (as opposed to `startActivityForResult()`), the iProov session is launched as a standalone session. When the iProov session completes, your application cannot handle the result.

Providing a full tutorial on integrating push with your app is beyond the scope of this documentation. Please see Google's [Cloud Messaging Documentation](https://developers.google.com/cloud-messaging/) for further information.

### Additional options

Starting in SDK v3.3, you can now set additional options on the builder to customise the iProov experience for your app.

The following additional option can be set for `NativeClaim` builder:

* `setPod(String)` - sets the pod your claim should use

These additional options can be set for both `NativeClaim` and `NotificationClaim` builders:

* `setPrivacyPolicyDisabled(boolean)` - disables the Privacy Policy

* `setInstructionsDialogDisabled(boolean)` - disables the instructions dialog

* `setMessageDisabled(boolean)` - disables the message shown during the canny preview

## 🎯 Intent Result

When launching iProov from an Intent, your application will in most cases (aside from GCM notifications) wish to handle the result.

In order to do so, make sure you always launch your Intent with `startActivityForResult()`. When the iProov session is complete, `onActivityResult()` will then be called in the Activity which launched the Intent.

The `resultCode` parameter will be one of 3 values:

#### `IProov.RESULT_SUCCESS`

The iProov session has completed and iProov has successfully verified or enrolled the user. You can obtain the encrypted token for this user from the returned Intent with:

```java
String encryptedToken = data.getStringExtra(IProov.EXTRA_ENCRYPTED_TOKEN)
```

> SECURITY WARNING: Never use iProov as a local authentication method. You cannot rely on the fact that a result was received to prove that the user was authenticated successfully (it is possible the iProov process could be manipulated locally by a malicious app). You can treat the verified result as a hint to your app to update the UI, etc. but must always independently validate the encrypted token server-side before performing any authenticated user actions.

#### `IProov.RESULT_FAILURE`

The iProov process has completed and iProov has failed to verify or enrol the user. The result Intent provides a reason that the authentication could not be confirmed. This could be a generic message, or could provide tips to the user to improve their chance of iProoving successfully (e.g. “lighting too dark”, etc.)

```java
String reason = data.getStringExtra(IProov.EXTRA_REASON)
```

You should present this to the user as it may provide an informative hint for the user to increase their chances of iProoving successfully next time.

#### `IProov.RESULT_ERROR`

The iProov process failed entirely (i.e. iProov was unable to verify or enrol the user due to a system or network issue). This could be for a number of reasons, for example the user cancelled the iProov process, or there was an unrecoverable network issue.
You can obtain an Exception relating to the cause of the failure as follows:

```java
Exception e = (Exception) data.getSerializableExtra(IProov.EXTRA_EXCEPTION);
```

You may wish to display the `localizedMessage` to the user.
## 🤷‍♂️ FAQs

### Why is the iProov AAR file so large?

Since release 3.3 we've put a lot of thought into reducing the footprint of the APK library. The two most major changes have been:

* we have rebuilt the OpenCV library from the ground up to strip out any unused parts
* iProov now only supports Arm processor architectures, plus x86 running under Arm emulation mode (Houdini). This should account for > 99% of Android devices out there; but if you have a specific requirement to target x86 or MIPS, you can download the OpenCV library (version 3.2 is required) from http://opencv.org/releases.html and copy the relevant libopencv_java3.so file(s) under the architecture you need into the jniLibs directory (in appropriate architecture-specific subdirectory). You won't benefit from our work on shrinking the OpenCV library, but can mitigate this using the abiFilter technique described below.

Including iProov will add ~7MB to the size of your app's APK (compared to ~36MB for version 3.3). You can reduce this size even further by using Gradle’s `abiFilter` feature to only build for certain architectures:

```gradle
productFlavors {
   arm {
       ndk {
           abiFilter "armeabi-v7a"
       }
   }
}
```

You can then deploy separate APKs to Google Play for each architecture (which is a supported feature of Google Play), or simply restrict your APK to common architectures if you prefer.

A full discussion of this feature is beyond the scope of this article, please see [the build system documentation](http://tools.android.com/tech-docs/new-build-system/tips) for further information.