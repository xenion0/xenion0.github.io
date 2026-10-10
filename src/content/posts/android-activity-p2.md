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
Each Android Application is composed of basic Android components such as Activity, Service, content Provider, and Broadcast Receiver. Among them, Activity is the main body of the application. It undertakes a lot of display and interaction work, and can even be understood as an "interface" "It's just an Activity.

Activity is a visual user interface displayed for user actions. For example, an activity can display a list of menu items for the user to choose from, or display some photos with descriptions. An SMS application can include an activity that displays a list of contacts to send to, an activity that writes a text message to a selected contact, and an activity that scrolls through previous text messages and changes settings. Although together they form a cohesive user interface, each activity remains independent of the others. Each is implemented as a subclass of the base class Activity class.

An application can have only one activity, or, like the SMS application just mentioned, contain many. The role of each activity, and its number, naturally depends on the application and its design. In general, there is always an application that is marked as the first one the user sees when the application starts. The way to move from one activity to another is to start the next one with the current activity.


## Knowledge points?

Reference: http://developer.android.com/guide/components/activities.html

### The life cycle

![](https://i.imgur.com/4Ilj5v9.png)

### Start method
show start
Register components in the configuration file AndroidManifest.xml
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
: Creates an instance of the activity on top of a new task's stack. If the instance already exists, the system will use the instance directly and call the activity's onNewIntent() (it will not be recreated)

singleInstance 
: Similar to "singleTask", but no other activities will run in the target activity's task, and there will always be only one activity in that task.<br/>
The location of the setting is in the android:launchMode attribute of the activity element in the `AndroidManifest.xml`.  file:

```xml
<activity android:name="ActB" android:launchMode="singleTask"></activity>
```
Activity launch mode is used to control the creation of task and Activity instances. Default "standard" mode. Standard mode will generate a new Activity instance once it is started and will not create a new task.
 When creating a new task, the content in the intent may be read by malicious applications, so it is recommended to use the default standard mode without configuring the launch mode attribute unless there are special requirements. launchMode can be overridden by Intent flags.
 
### Intent Selector

When multiple activities have the same action, a selector will pop up for the user to choose when this action is called.

### Permission
`android:exported`

Whether an Activity component can be started by an external application depends on this property. When set to true, the Activity can be started by an external application. When set to false, it cannot. At this time, the Activity can only be started by its own app. (The same user id or root can also be started)

The action attribute exported with no intent-filter configured defaults to false (without a filter, the activity can only be started through a clear class name, so it is equivalent to only the program itself can be started), and the action attribute exported with an intent-filter configured defaults to true.

The exported attribute is only used to limit whether the activity is exposed to other apps, and the external startup of the activity can also be restricted through the permission declaration in the configuration file.

`android:protectionLevel`

http://developer.android.com/intl/zh-cn/guide/topics/manifest/permission-element.html
![](https://i.imgur.com/IFlK4iJ.png)
![](https://i.imgur.com/IbIBmXO.png)

normal: Default value. Low-risk permissions can be used as long as they are applied for, and user confirmation is not required during installation.

dangerous: Permissions like WRITE_SETTING and SEND_SMS are risky because these permissions can be used to reconfigure the device or cause phone bills. Use this protectionLevel to identify some permissions that users may be concerned about. Android will warn the user about the need for these permissions when installing the program, and the exact behavior may vary depending on the Android version or the installed mobile device.

signature: These permissions are only granted to programs signed with the same key as this program.

signatureOrSystem: Similar to signature, except that the programs in the system also need to be eligible to access. This allows custom Android system applications to also gain permissions, and this level of protection helps integrate the system compilation process.

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

Activity type and usage determine its risks and defense methods, so activities are classified as follows: Private, Public, Partner, In-house

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
11-When Providing an Asset Secondhand, the Asset should be Protected with the Same Level of Protection <br/>
12-Do not send sensitive information as much as possible, and consider the risk that the information of the intent in the public Activity may be stolen by malicious applications

## Vuln Example

[first you need to download InjuredAndroid apk from here](https://m.apkpure.com/ar/injuredandroid/b3nac.injuredandroid/download?from=details)

then install apk with adb tool with 
![](https://i.imgur.com/PVYtWzK.png)

### test with Drozer

first we need to know package name with drozer 
```bash
dz> run app.package.list -f  injured
b3nac.injuredandroid (InjuredAndroid)
```

Identifying the attack surface
![](https://i.imgur.com/jCHQD1q.png)

retrive info about each activity in app and listed exported and hidden activity
![](https://i.imgur.com/hCAT552.png)

Start activity with Drozer 
![](https://i.imgur.com/6rkgTxB.png)

```bash
run app.activity.start --component <package name> <component name>

run app.activity.start --component b3nac.injuredandroid   b3nac.injuredandroid.b25lActivity

```
### with ADB tool form
in this way you need to open apk with jadx-gui and search for  `android:exported="true"`

![](https://i.imgur.com/lBVj0Mp.png)

`adb shell am start -n b3nac.injuredandroid/.b25lActivity`

### with POC java code
![](https://i.imgur.com/wpi3fbu.png)
creat intent that specify first package name and then component name
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
* Write your own app to call startActiviy() or startActivityForResult()


## References
[Android activity developer ](https://developer.android.com/reference/android/app/Activity)

report example <br/>
https://hackerone.com/reports/537670

Book Android Secure Code <br/>
![](https://i.imgur.com/UsvXUER.png)



**At the end** <br/>
![](https://i.imgur.com/uuSwt1q.png)

