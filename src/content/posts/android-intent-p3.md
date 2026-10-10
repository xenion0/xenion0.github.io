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
Intent is a messaging object you can use to request an action from another application components (activities , services and Broadcast Receivers) 

Intents themselves are object containing information on operations to be performed .

Intent is data structure designed to hold info on events or operations to be performed.

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
         code example for Explict Intent https://developer.android.com/guide/components/intents-filters#ExampleExplicit

 ### 2-Implicit Intent
used to invoke components of different application 

They don’t  provide the specific component name to be invoked but rely on system to find the best available component to be invoked


don't name a specific component , but instead declare a general action to perform , which allow a component from another app to handle 
For example , if you want to show the user a location on a map , you can use an implicit intent to request that another capable app show a specified location on a map 
code example for Implicit Intent   https://developer.android.com/guide/components/intents-filters#ExampleSend

### How Implicit Intent find component for Intent??
If you call the Implicit Intent then , the Android system will search for all the available components that can be used to start that activity ,
This process is done by comparing the content of the intent with the content present in the intent-filters declared in the AndroidManifest.xml
If there is only one intent-filter that is compatible with the content of the intent the android system will start the desired component . <br/>
But if there are am number of intent-filters that are compatible with the content of the intent then the android system will show you a list of application that can be used to perform 

## Use cases Example of Intents
1. start Activity
2. Start a service
3. Delivering a broadcast 

![](https://i.imgur.com/qLfMfBM.png)

## Intent Component

Intent is a data structure designed to hold information on events or operation that android system used to determine which component to start  (such as the exact component name or component category that should receive the intent)

1-Component Name:  <br/>
This is optional , but it's critical piece of information that makes an Intent explicit , meaning that intent should be delivered only to the app component defined by the component name .
without a component name the intent is implicit and Android system will decides which component should recive the intent

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
Category is used in case of Explicit Intents where you need to specify the type of application that will be used to perform a particular action.  <br/>
5-Data : <br/>
important to specify the type of data (its MIME type) on which the action is to be performed by Android system which the help of intents
 
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
    
In the example scenario below, MainActivity is sending sensitive data to MainActivity2 via intent. But the malicious app did the same definition of `<intent-filter>` to itself. Because of this situation, the implict intent scenario exposed and the relevant data was transmitted to the 3rd party application.    

![](https://i.imgur.com/2TU79n9.png)

Example: if we have a component file which will use implicit intent (action, data, category) like action call a tel and there are more than one component with the same contents of call a tel, and uses the same intent. You may see the option Open with: Call Phone - Truecaller - etc.. and that's because of the existing of more than one component matches and handles same intent.

When intent filters are mentioned for a component in the Manifest file that component automatically becomes exported unless the developer overrides that by mentioning exported=FALSE in the component attribute.
    
Example: if we have a component file which will use implicit intent (action, data, category) like action call a tel and there are more than one component with the same contents of call a tel, and uses the same intent. You may see the option Open with: Call Phone - Truecaller - etc.. and that's because of the existing of more than one component matches and handles same intent.

**When intent filters are mentioned for a component in the Manifest file that component automatically becomes exported unless the developer overrides that by mentioning exported=FALSE in the component attribute.**
    
    
    
## Examples of Intent

### Why we need to know about how to create Intent?
    
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

    
for example we chose chrome to accept ower action to do 
chrome open and take our url as input to open  
![](https://i.imgur.com/ZZaNjKO.png)
    

### Code Example
![](https://i.imgur.com/WJTuE23.png)

1- Intent object and in constructor  we give it action we need to do <br/>
2- here give intent the  uri  as data to oben in webview <br/>
3- start the Intent

## Attacks in Intents
    

**The default behavior depends on whether the component is likely to be used externally. 
In the case of Activities, Services, and Broadcast Receivers, the default depends on how the component is configured with regard to Intents. 
As we have seen, a component can specify an Intent filter that allows it to receive Intents from other apps to carry out tasks.
As such, if a component specifies an Intent filter, it is assumed that you want the component to be accessed by other apps and it is, by default, exported and thus public.
If, however, no Intent filter is specified for a component, the only way to send an Intent to it is to fully specify the component’s class name. 
Therefore, it is assumed that you do not want to make this component publicly accessible, so the default exported is false and the component is private.**

```    
this paragraph important to understand i take it from 
book Application Security for the Android Platform by Jeff Six
```    
   
[link for lab Intent Redirection (Access to Protected Components)](https://github.com/optiv/InsecureShop/releases/download/v1.0/InsecureShop.apk)   
first you need to reverse apk with tool like jadx-gui and look in code will find this code vul take your extra and start with it a new activity 

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
 
## Diff between Explicit & Implicit Intent
<table>

<thead>

<tr>

<th>

Explicit Intent

</th>

<th>

Implicit Intent

</th>

</tr>

</thead>

<tbody>

<tr>

<td>

As already mentioned above explicit intents are those in which the user has a clear vision and knows exactly which activity can handle the requests.

**Example**: When you have a Listview screen on tap of each item you will go to detail activity  

**Intent = Intent(applicationContext,DetailActivity::class.java)**

**startActivity(intent)**

</td>

<td>

Implicit intents do not name a specific component like explicit intent, instead declare general action to perform, which allows a component from another app to handle.

**Example:** When you tap the share button in any app you can see the Gmail, Bluetooth, and other sharing app options. Here user sends a request is the implicit intent request which can be handled by these Gmail, Bluetooth-like app.

</td>

</tr>

<tr>

<td>Explicit intent can do the specific application action which is set by the code like changing activity, downloading the file in the background, etc.</td>

<td>It specifies the only action to be performed and does not directly specify Android Components.</td>

</tr>

<tr>

<td>

In explicit intent, you can pass data to other activities by using the putExtra method and retrieve by using getIntent().

**Example:**

val intent = Intent(this, SecondActivity:: class.java).apply{

putExtra(“key”,”New Value”)

}

startActivity(intent)

**Second Screen:**

val secondIntent = intent.getStringExtra(“key”)

</td>

<td>Here we just mention the action in the intent and OS decides which applications are suitable to handle the task, action across two different applications.</td>

</tr>

<tr>

<td>Explicit intents are used for communication inside the application. Like changing activities inside the application.</td>

<td>They are used for communication across two different applications.</td>

</tr>

<tr>

<td>In explicit intent, the action target is delivered even the filter is not consulted.</td>

<td>When you make an implicit call with the intent. OS look at the action and then it matches with all the filters intent-filters of all the registered activities of all application using PackageManager and then populates the result as a list, this process is called as intent Resolution. </td>

</tr>

</tbody>

</table>
    
## References
https://hackerone.com/reports/200427  
https://developer.android.com/guide/components/intents-filters
    
https://medium.com/androiddevelopers/lets-be-explicit-about-our-intent-filters-c5dbe2dbdce0    
  
https://docs.insecureshopapp.com/insecureshop-challenges/access-to-protected-components    
