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

As you can see, services have attributes that need to be considered before digging deep in to the code. Certain attributes make services not interactable. For a complete reference, checkout [android API guide](http://developer.android.com/guide/topics/manifest/service-element.html).

#### android:exported
This attribute defines whether this service is exported outside so that components of other apps can interact with it. If this attribute is set to `False`, there is no need to investigate further. Default value is `True`.

#### android:enabled
Although not shown in previous examples, if this value set to false then this service cannot be instantiated, meaning it cannot be run. Default value is `True`.

#### android:permission
This attribute sets name of the permission an entitiy must have in order to lauch or bind with the service. If this options is not set, service inherits permission set by `<application>` element. Therefore if we don't have the specified permission, we cannot interact with the service. 

#### android:name
This is the name of the `Service` subclass that implements the service. When a service is reachable from outside and it is worth looking at closely, value of `android:name` must be searched inside the decompiled code. 
## Example
{% highlight xml%}
<service android:exported="false" android:name=".FotaRegisterService">
	<intent-filter>
		<category android:name="com.sec.android.fotaclient"/>
	</intent-filter>
</service>
{% endhighlight %}

# Broadcast Receivers
Broadcast Receivers receive intents that are broadcasted by the system or other apps. Broadcast Receivers can be statically declared in the `AndroidManifest.xml` or dynamically in the code by `Context.registerReceiver()` method. Because it processes external data(which is intent), it is an another attack surface that should be looked closely. Difference between starting a service is that intent has a specific target(service) wheras broadcast does not have a specific destination and it is up to the broadcast receiver to process it or not.  

## Finding out which broascast receivers are reachable
Unfortunately, it is same for broadcast receivers that attributes must be looked at closely in order to distinguish receivers that is reachable from the outside. For a full reference of the attributes, refer to the [android API guide](http://developer.android.com/guide/topics/manifest/receiver-element.html).

#### android:exported
This attribute defines whether the broadcast receiver can receive broadcast from outside of the application. Set to `True` if it can, `False` if not. 

#### android:enabled
If this value set to `false` then this receiver cannot be instantiated, meaning it cannot be run. Default value is `True`.

#### android:permission
This is the name of a permission that broascasters must have in order to send broadcasts to this specific receiver. Therefore, without this permission it is impossible to trigger the vulnereability in the receiver. If it is not set, recevier inherits permission set by the `<application>` element's `permission` attribute. If neither is set, no permission is needed to send broadcasts.

#### android:name
This is the name of the class that implements the broadcast receiver. Value of `android:name` should be searched inside the code in order to look more deeply.

## Example
{% highlight xml%}
<receiver android:exported="false" android:name=".FotaRegisterService">
	
</receiver>
{% endhighlight %}

# Content Providers
`<provider>` element declares a content provider component. Content provider provides structured access to data managed by the application. Data can be accessed by using URIs defined in the `android:authorities` element in the manifest file. If content provider is not appropriately secured by setting attributes, its contents can be leaked. 

## Finding out which content providers are reachable
Below are attributes that must be considered before diving into the provider for possible leakage of data. These define whether it is accessible from outside the app, etc. 

#### android:authorities
This attribute lists URI authorities that identify data offered by the content provider. Naming convention is using full package name and the provider name.

Example: `com.example.provider.providername`

Authority name is usually the name of the class that implements `ContentProvider`.
 
#### android:enabled
If this value set to `false` then this content provider cannot be instantiated, meaning it cannot be run. Default value is `True`.

`<application>` element has its own `enabled` attribute that applies to all of the application components. Therefore `enabled` attribute of `<application>` and `<provider>` must equal to `true` in order to be enabled. 

#### android:exported
This attribute defines whether the provider is available to other applications. Set to `True` if it can, `False` to only allow applications that have same user ID(UID).

#### android:permission 
Defines name of the permission that one must have in order to read/write to the data. However, permission defined here is overwritten by `android:readPermssion` and `android:writePermission`. 

## Example
{% highlight xml%}
<provider android:exported="false" android:name=".FotaRegisterService">
	
</provider>
{% endhighlight %}

# Reference
http://translate.wooyun.io/2015/08/05/Android-Service-Security.html
http://translate.wooyun.io/2015/07/22/Android-Broadcast-Security.html
http://developer.android.com/