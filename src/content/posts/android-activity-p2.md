---
title: Android Activity Security P2
published: 2022-02-25
description: "How Android activities work and where they go wrong: the activity lifecycle, launch modes, intent filters and the exported and protectionLevel attributes, followed by a Drozer walkthrough against InjuredAndroid and a hardening checklist."
image: ''
tags: [Android]
category: Android
draft: false
lang: ''
---

## What is Activity?
An Android app is built from four component types: Activity, Service, Content Provider and Broadcast Receiver. Of these, Activity is the one the user actually sees — it is the app's user interface, and it is usually where you will spend most of your testing time.

An activity is a single screen of user interface. An SMS app, for example, might have one activity listing contacts, another for composing a message to a selected contact, and a third for scrolling through previous messages and changing settings. Together they form a coherent app, but each activity is independent of the others, and each is implemented as a subclass of the base `Activity` class.

An app can have a single activity or many. One of them is normally marked as the entry point the user sees at launch — the `LAUNCHER` activity. Moving from one activity to another means starting the next one from the current one.


## Background

Reference: http://developer.android.com/guide/components/activities.html

### The life cycle

![](https://i.imgur.com/4Ilj5v9.png)

The callbacks you will see in code and in logs:

- `onCreate()` — the activity is created; views inflated, saved state restored
- `onStart()` — becoming visible
- `onResume()` — in the foreground and interactive
- `onPause()` — losing focus, still partly visible
- `onStop()` — no longer visible
- `onDestroy()` — being torn down

For security work the interesting part is rarely the callbacks themselves but what happens to data as the activity moves between them — what gets written to disk or to logs on `onPause`/`onStop`, and whether sensitive values survive in saved instance state.

### Starting an activity

An activity can be started **explicitly** (you name the target component) or **implicitly** (you describe an action and let the system resolve it).

Explicit start — register the component in `AndroidManifest.xml`:
```xml
<activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

Directly use the intent object to specify application and activity startup

```java
Intent intent = new Intent(this, ExampleActivity.class);
startActivity(intent);
```

implicit start

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
startActivity(intent);
```
### launch mode

Activity has four loading modes:

standard 
: Default behavior. Every time an activity is started, the system creates a new instance in the target task.

singleTop 
: If an instance of the target activity already exists on the top of the stack of the target task, the system will use the instance directly and call the activity's onNewIntent() (it will not be recreated)

singleTask 
: The activity becomes the root of its own task, and only one instance ever exists. If it already exists, the system reuses it and calls onNewIntent() instead of recreating it.

singleInstance 
: Similar to "singleTask", but no other activities will run in the target activity's task, and there will always be only one activity in that task.<br/>
The location of the setting is in the android:launchMode attribute of the activity element in the `AndroidManifest.xml`.  file:

```xml
<activity android:name="ActB" android:launchMode="singleTask"></activity>
```
Launch mode controls how tasks and activity instances are created; the default is `standard`. When a new task is created, the contents of the intent may be readable by other applications, so unless there is a specific requirement, leave `launchMode` at its default — it can also be overridden at runtime by intent flags.
 
### Intent Selector

When multiple activities have the same action, a selector will pop up for the user to choose when this action is called.

### Permission
`android:exported`

Whether an Activity component can be started by an external application depends on this property. When set to true, the Activity can be started by an external application. When set to false, it cannot. At this time, the Activity can only be started by its own app. (The same user id or root can also be started)

The action attribute exported with no intent-filter configured defaults to false (without a filter, the activity can only be started through a clear class name, so it is equivalent to only the program itself can be started), and the action attribute exported with an intent-filter configured defaults to true.

The exported attribute is only used to limit whether the activity is exposed to other apps, and the external startup of the activity can also be restricted through the permission declaration in the configuration file.

**Since Android 12 (API 31) the `exported` value is no longer merely a default.** Any component that declares an `<intent-filter>` — activity, activity-alias, service or receiver — must set `android:exported` explicitly, or the app will not install at all. The manifest merger fails the build and the store rejects the upload, with:

```
Manifest merger failed : Apps targeting Android 12 and higher are required to
specify an explicit value for android:exported when the corresponding
component has an intent filter defined.
```

If a client's app refuses to install for no obvious reason, this is a common cause.

`android:protectionLevel`

http://developer.android.com/intl/zh-cn/guide/topics/manifest/permission-element.html
![](https://i.imgur.com/IFlK4iJ.png)
![](https://i.imgur.com/IbIBmXO.png)

normal: Default value. Low-risk permissions can be used as long as they are applied for, and user confirmation is not required during installation.

dangerous: Permissions like WRITE_SETTING and SEND_SMS are risky because these permissions can be used to reconfigure the device or cause phone bills. Use this protectionLevel to identify some permissions that users may be concerned about. Android will warn the user about the need for these permissions when installing the program, and the exact behavior may vary depending on the Android version or the installed mobile device.

signature: These permissions are only granted to programs signed with the same key as this program.

signatureOrSystem: Similar to `signature`, but system apps are also eligible to access. This allows custom system applications to gain the permission, and helps integrate the system build process. **Deprecated since API 23** — the modern equivalent is `signature|privileged`.

```xml

<!-- *** POINT 1 *** Define a permission with protectionLevel="signature" -->
<permission
android:name="com.xenion.android.permission.protectedapp.MY_PERMISSION"
android:protectionLevel="signature" />
<application
android:icon="@drawable/ic_launcher"
android:label="@string/app_name" >
<!-- *** POINT 2 *** For a component, enforce the permission with its permission attribute -->
<activity
android:name=".ProtectedActivity"
android:exported="true"
android:label="@string/app_name"
android:permission="com.xenion.android.permission.protectedapp.MY_PERMISSION" >
<!-- *** POINT 3 *** If the component is an activity, you must define no intent-filter -->
</activity>

```

## Activity classification

Activity type and usage determine its risks and defence methods. The classification used here comes from the OWASP MASTG: Private, Public, Partner, In-house.

![](https://i.imgur.com/oN9l2PY.png)

### Security advice

1-The private Activity used in the app should not be configured with intent-filter. If the intent-filter is configured, the exported property needs to be set to false.  <br/>
2-Use default taskAffinity <br/>
3-Use default launchMode <br/>
4-Do not set the FLAG_ACTIVITY_NEW_TASK label of the intent when starting the Activity <br/>
5-Be careful with the intents you receive and the information they carry <br/>
6-Signature verification in-house app <br/>
7-When Activity returns data, pay attention to whether the target Activity has the risk of leaking information <br/>
8-Use display startup when the purpose Activity is very clear <br/>
9-Handle the data returned by the activity with care. The data returned by the target activity may be forged by a malicious application. <br/>
10-Verify whether the target Activity is a malicious app, so as not to be deceived by intent, which can be verified by hash signature <br/>
11-When passing an asset on to another component, protect it at the same level as at its origin<br/>
12-Do not send sensitive information as much as possible, and consider the risk that the information of the intent in the public Activity may be stolen by malicious applications

## Vuln Example

Download InjuredAndroid from its official releases page: **[B3nac/InjuredAndroid releases](https://github.com/B3nac/InjuredAndroid/releases)**

then install apk with adb tool with 
![](https://i.imgur.com/PVYtWzK.png)

### Testing with Drozer

first we need to know package name with drozer 
```bash
dz> run app.package.list -f  injured
b3nac.injuredandroid (InjuredAndroid)
```

Identifying the attack surface
![](https://i.imgur.com/jCHQD1q.png)

retrieve info about each activity in the app and list the exported and hidden ones
![](https://i.imgur.com/hCAT552.png)

Start activity with Drozer 
![](https://i.imgur.com/6rkgTxB.png)

```bash
run app.activity.start --component <package name> <component name>

run app.activity.start --component b3nac.injuredandroid   b3nac.injuredandroid.b25lActivity

```
### With the ADB tool
In this approach you open the APK with jadx-gui and search for `android:exported="true"`

![](https://i.imgur.com/lBVj0Mp.png)

`adb shell am start -n b3nac.injuredandroid/.b25lActivity`

### With a POC app
![](https://i.imgur.com/wpi3fbu.png)
create an intent that specifies first the package name and then the component name
and then start the Intent will open the target activity

![](https://i.imgur.com/N05HSk6.png)![](https://i.imgur.com/2GWvSTK.png)

![](https://i.imgur.com/6inTjBY.png)



## Test method

### View activity

* Decompile and view the activity component in the configuration file AndroidManifest.xml (focus on those with intent-filter configured and those without export="false")
* Open the installed app directly with RE to view the configuration file
* Drozer scan: run app.activity.info -a packagename
* Dynamic viewing: logcat sets the filter tag to ActivityManager

### Start the activity

* adb shell：am start -a action -n package/component
* drozer: run app.activity.start --action android.action.intent.VIEW ...
* Write your own app to call startActivity() or startActivityForResult()


## References
[Android activity developer ](https://developer.android.com/reference/android/app/Activity)

report example <br/>
https://hackerone.com/reports/537670

Book Android Secure Code <br/>
![](https://i.imgur.com/UsvXUER.png)



**At the end** <br/>
![](https://i.imgur.com/uuSwt1q.png)

