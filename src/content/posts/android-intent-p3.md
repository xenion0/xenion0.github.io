---
title: Android Intent P3
published: 2022-02-25
description: "Explicit versus implicit intents, the intent-filter model and how the system resolves a match, why intent filters are not a security boundary, and an intent-redirection walkthrough against InsecureShop."
image: ''
tags: [Android]
category: Android
draft: false
lang: ''
---

## What is Intent?
An Intent is a messaging object you use to request an action from another application component — an activity, a service or a broadcast receiver.

It is a data structure that describes an operation to be performed, along with the information needed to perform it.

Let’s look upon the informal way of defining Intents. You can think of Intents as a messaging service that is used to communicate between various components of the Android application.
 For example, if you want to send some message from Egypt to USA using the Post Office facility then you can do so by buying an Envelope and then pass the message in the Envelope and send the message to the desired location.
 
 
 ## Intent Types
 There are two type of Intent:
 
 ### 1-Explicit Intents
 ![](https://i.imgur.com/vd7pS0m.png)
if you want communication between the components of your application only then you can use the Explicit Intent.
Explicit Intents are used to communicate with a particular component of the same application

provide the component name (class name ) This is usually for inter-application components.

  for example , if you want to launch an Activity by clicking some button  on the present Activity then you go to target activity 
code example for Explicit Intent — https://developer.android.com/guide/components/intents-filters#ExampleExplicit

 ### 2-Implicit Intent
used to invoke components of different application 

They don’t  provide the specific component name to be invoked but rely on system to find the best available component to be invoked


don't name a specific component , but instead declare a general action to perform , which allow a component from another app to handle 
For example , if you want to show the user a location on a map , you can use an implicit intent to request that another capable app show a specified location on a map 
code example for Implicit Intent — https://developer.android.com/guide/components/intents-filters#ExampleSend

### How does an implicit intent find its component?
If you call the Implicit Intent then , the Android system will search for all the available components that can be used to start that activity ,
This process is done by comparing the content of the intent with the content present in the intent-filters declared in the AndroidManifest.xml
If there is only one intent-filter that is compatible with the content of the intent the android system will start the desired component . <br/>
But if several intent-filters match the intent, the system shows a chooser listing the applications that can handle it.

## Use Cases for Intents
1. start Activity
2. Start a service
3. Delivering a broadcast 

![](https://i.imgur.com/qLfMfBM.png)

## Intent Components

Intent is a data structure designed to hold information on events or operation that android system used to determine which component to start  (such as the exact component name or component category that should receive the intent)

1-Component Name:  <br/>
This is optional , but it's critical piece of information that makes an Intent explicit , meaning that intent should be delivered only to the app component defined by the component name .
without a component name the intent is implicit and the system decides which component should receive it

2-Action: <br/>
A string that specifies the generic action to perform (such as view or pick)
example of action   ACTION_VIEW
Use this action in an intent with startActivity()  when you have some information that an activity can show to the user And developer can make custom action  to perform  by define app package name
```java
        static final String ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL";
```
3- Extras :  <br/>
you can add extra data to intent in the for of key-value pairs and this extras information can be passed from one activity to the other <br/>
4-Category :  <br/>
A qualifier the intent filter must also match — `DEFAULT` (required for implicit resolution), `BROWSABLE` (reachable from a browser), `LAUNCHER` (app entry point). This is an **implicit**-intent mechanism: an explicit intent names its target directly, so the filter, category included, is never consulted.  <br/>
5-Data : <br/>
The URI of the data to act on, plus its MIME type — this is what the system matches against the `<data>` element of an intent filter.
 
## Intent Filter
Each components can provide Intent-filters   structures that provide information on which Intents can be handled by particular components.

The system then compares filters to the Intent object and select the best available components for it 
If a component does not have Intent-filters, it can only receive explicit Intents.
Note that Intent-filters cannot be relied on for security because one can always send an explicit Intent to it, thus bypassing the filters. 
Component specific permissions should always be defined to restrict who can access a particular component through Intents. In addition, limited data can be passed through Intents. 
However, any sensitive information, such as passwords, should never be sent through Intents, as these can be received by malicious components. 

### Intent Filter component

![](https://i.imgur.com/191sWkn.png)

In each app component that includes an `<intent-filter>` element,
 explicitly set a value for android:exported.<br/>
 This attribute indicates whether the app component is accessible to other apps.
    
In the example scenario below, MainActivity is sending sensitive data to MainActivity2 via intent. But the malicious app did the same definition of `<intent-filter>` to itself. Because of this, the implicit-intent scenario is exposed and the data is transmitted to the third-party application.

![](https://i.imgur.com/2TU79n9.png)

Example: if we have a component that uses an implicit intent (action, data, category) — say an action that calls a phone number — and more than one component matches it, you will see an **Open with: Call Phone · Truecaller · …** chooser. That is the system resolving several matching components for the same intent.

When intent filters are declared for a component in the manifest, that component automatically becomes exported — unless the developer overrides it by setting `exported="false"` on the component.
    
    
    
## Examples of Intent

### Why build a POC app?
    
when want to exploit the bug and create POC will be better if create app make that 

### Example of Explicit Intent
  
![](https://i.imgur.com/B6U1fVV.png)![](https://i.imgur.com/vFoKyw5.png)

    
### Code Example
![](https://i.imgur.com/eIapXrB.png)

    1- Intent object that we will to go from first Activity to Target
    2 setClassName() that to specify the packageName and the class want to go fore 
    3- startActivity() used to start Activity that define in the Intent 
    
    
That small example about how to create Intent the same way can do to start another android component (Broadcast Receivers - Service - Activity)

### Example of Implicit Intent
doesn’t specify the component. In such a case, intent provides information on available components provided by the system that is to be invoked. For example, you may write the following code to view the webpage.
    
![](https://i.imgur.com/OE2JzKw.png)![](https://i.imgur.com/9AYkk1C.png)

    
for example we chose Chrome to accept our action
chrome open and take our url as input to open  
![](https://i.imgur.com/ZZaNjKO.png)
    

### Code Example
![](https://i.imgur.com/WJTuE23.png)

1- Intent object and in constructor  we give it action we need to do <br/>
2- here we give the intent the URI as data to open in a WebView<br/>
3- start the Intent

## Attacks in Intents
    

> The default depends on whether the component is likely to be used externally. For Activities, Services and Broadcast Receivers it follows from how the component is configured with regard to intents: a component that declares an intent filter is assumed to be intentionally reachable by other apps, and is therefore exported by default. A component with no intent filter can only be reached by naming its class explicitly, so it is assumed private — `exported` defaults to false.
>
> — Jeff Six, *Application Security for the Android Platform*

This is the most useful mental model for reading a manifest: **a filter is a declaration that the component is meant to be reachable, and the `exported` default follows from it.**
   
[link for lab Intent Redirection (Access to Protected Components)](https://github.com/optiv/InsecureShop/releases/download/v1.0/InsecureShop.apk)   
reverse the APK with a tool like jadx-gui; the vulnerable code takes your extra and starts a new activity with it.

```xml
          <activity android:name="com.insecureshop.WebView2Activity">
            <intent-filter>
                <action android:name="com.insecureshop.action.WEBVIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
            </intent-filter>
        </activity>
``` 
    
    
this code in `WebView2Activity.java`
    
![](https://i.imgur.com/nHaENjM.png)

The extra intent being passed is not sanitized or filtered in any way, which means we could use this activity to pass an intent which would then be used by the startActivity. That seems a perfect candidate to access the PrivateActivity.

### POC
 ![](https://i.imgur.com/yTXIWNf.png)

make intent as extra that will start new activity this technique look like nested intent intent object will start webView and extra object will start privateActivity
  
![](https://i.imgur.com/r6ooCLK.png)![](https://i.imgur.com/1yzswkL.png)

![](https://i.imgur.com/b5SsgZs.png)
    
    
**When open adb logcat will find** 
![](https://i.imgur.com/F4FrwfB.png)

that mean exploit done 
 
## Explicit vs Implicit Intents
|  | Explicit Intent | Implicit Intent |
| --- | --- | --- |
| Target | Names the exact component (package + class) | Declares an action; the system resolves a component |
| Resolution | The intent filter is **not** consulted | Matched against every registered intent filter via PackageManager |
| Scope | Inside one app, or a known external component | Across applications |
| Data | `putExtra()` / `getStringExtra()` for key-value data | Carries action, data URI and category rather than extras |
| Typical use | Navigating between your own activities | Sharing, opening a URL, dialling a number |

The point worth keeping: in an explicit intent the target is delivered even though the filter is never consulted — which is exactly why intent filters cannot be treated as a security boundary.
    
## References
https://hackerone.com/reports/200427  
https://developer.android.com/guide/components/intents-filters
    
https://medium.com/androiddevelopers/lets-be-explicit-about-our-intent-filters-c5dbe2dbdce0    
  
https://docs.insecureshopapp.com/insecureshop-challenges/access-to-protected-components    
