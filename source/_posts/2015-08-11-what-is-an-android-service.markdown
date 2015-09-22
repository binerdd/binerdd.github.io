---
layout: post
title: "Overview of Attack Surfaces in Android Apps"
date: 2015-08-11 21:35:46 +0900
comments: true
categories: android 
---

When auditing the security of Android apps, it is crucial to enumerate attack surfaces available. There are 3 main attack surfaces available in an Android app: Service, Broadcast Receiver, Content Provider. 

# Android Service

According to [Google](http://developer.android.com/reference/android/app/Service.html), a service is an application component representing either an application's desire to perform a longer-running operation while not interacting with the user or to supply functionality for other applications to use. So it can be thought as background process without an graphical interface.

Service can be either started or bound. When a service is started, it 

When auditing, one can check which services are available by searching for `<service>` element tag in `AndroidManifest.xml` file. There can be many or none, but it is important to distinguish which service is actually reachable from outside the app. 

## Finding out which services are reachable
Example of a service is as follows:

{% highlight xml%}
<service android:exported="false" android:name=".FotaRegisterService">
	<intent-filter>
		<category android:name="com.sec.android.fotaclient"/>
	</intent-filter>
</service>
{% endhighlight %}

As you can see, services have attributes that need to be considered before digging deep in to the code. Certain attributes make services not interactable. For a complete reference, checkout [android API guide](http://developer.android.com/guide/topics/manifest/service-element.html).

#### android:exported
This attribute defines whether this service is exported outside so that components of other apps can interact with it. If this attribute is set to `False`, there is no need to investigate further. Default value is `True`.

#### android:enabled
Although not shown in previous examples, if this value set to false then this service cannot be instantiated, meaning it cannot be run. Default value is `True`.

#### android:permission
This attribute sets name of the permission an entitiy must have in order to lauch or bind with the service. If this options is not set, service inherits permission set by `<application>` element. Therefore if we don't have the specified permission, we cannot interact with the service. 

#### android:name
This is the name of the `Service` subclass that implements the service. When a service is reachable from outside and it is worth looking at closely, value of `android:name` must be searched inside the decompiled code. 


# Broadcast Receivers
Broadcast Receivers receive intents that are broadcasted by the system or other apps. Broadcast Receivers can be statically declared in the `AndroidManifest.xml` or dynamically in the code by `Context.registerReceiver()` method. Because it processes external data(which is intent), it is an another attack surface that should be looked closely. Difference between starting a service is that intent has a specific target(service) wheras broadcast does not have a specific destination and it is up to the broadcast receiver to process it or not.  

## Finding out which broascast receivers are reachable
Unfortunately, it is same for broadcast receivers that attributes must be looked at closely in order to distinguish receivers that is reachable from the outside. For a full reference of the attributes, refer to the [android API guide](http://developer.android.com/guide/topics/manifest/receiver-element.html).

#### android:exported
This attribute defines whether the broadcast receiver can receive broadcast from outside of the application. Set to `True` if it can, `False` if not. 

# Content Providers

# Reference
http://translate.wooyun.io/2015/08/05/Android-Service-Security.html
http://translate.wooyun.io/2015/07/22/Android-Broadcast-Security.html