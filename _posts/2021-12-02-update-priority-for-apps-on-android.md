---
title: Update priority for apps on Android
slug: update-priority-for-apps-on-android
date_published: 2021-12-02 17:33:49.000000000 Z
date_updated: 2021-12-02 17:33:48.000000000 Z
tags:
- android
- android enterprise
- intune
- mem
- security
excerpt: In the 2111 release of Microsoft Endpoint Manager, a small but important
  feature was released in regards of application update management in Android Enterprise.
---
In the 2111 release of Microsoft Endpoint Manager, a small but important feature was released in regards of application update management in Android Enterprise.

Keeping apps updated is an important aspect of managing our devices as it improves stability with bug fixes, makes new features available to our users and improves security. Applications are updated automatically by default on Android Enterprise devices, but there are many conditions that the device has to meet before an app actually updates,

- The device is connected to a Wi-Fi network.
- The device is charging.
- The device is idle (not actively used).
- The app to be updated is not running in the foreground.

With all of these conditions, it might take a long time before an important update is installed on all devices.

With the 2111 release however, you can now set the behaviour of app updates using a new configuration on apps imported through Managed Google Play. When configured, it overrides the update settings of the default configuration and lets you as an admin make sure that critical applications always receive updates. No matter if the user has configured to not update applications or if they rarely connect to a WIFI network.

The options you can choose from are the following,

- Default: The app's updates are subject to default conditions (described above).
- High Priority: The app will update as soon as possible from when a new update is released, disregarding all of the default conditions. This may be disruptive for some users since the update can occur while the device is being used. If the device is offline, the app will update as soon as the device connects to a network

You can configure this setting for dedicated, fully managed, and corporate-owned work profile devices. So personal devices with Work Profile won't be affected by this configuration.

It's important to keep in mind that the update will happen no matter of the condition, if the application is in use, the app will be closed and the update will happen, if the device is on cellular network, the update will happen.

So, while you are planning to implement this, do not target all applications with high priority since this might eat up your user's cellular data quota or cause a bad user experience. Instead, select a subset of critical applications that should always get the update.

## Configuring update priority

Let's have a look at how we can set an app to have a high priority

- Sign into the [MEM console](https://endpoint.microsoft.com)
- Navigate to **Apps -> Android** and click on a Managed Google Play app which you want to change priority for
- Navigate to **Properties -> Assignments** and click **edit**
- From here you can edit the update priority by clicking on **default** under **update priority**
- Note that you can set different priorities for different groups
![Screenshot-2021-12-02-at-10.50.11]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-02-at-10.50.11.png)

- Choose **High Priority** and click **Ok**
![Screenshot-2021-12-02-at-10.52.12]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-02-at-10.52.12.png)

That's it, the default settings will now be overridden, and the app will always get the latest update.

If some devices do not get the app immediately on release, there are other factors that can have an effect on install times,

- App release settings

- Android app developers can roll out app updates gradually. As a result, an app update may initially only be available to some devices in your fleet

- Pending installs

- App updates are queued and installed one at a time. If a device has several apps with pending updates, it may take longer than expected to install all the updates
