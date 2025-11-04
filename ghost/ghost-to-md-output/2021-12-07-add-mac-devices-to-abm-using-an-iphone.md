---
title: Add mac devices to ABM using an iPhone
slug: add-mac-devices-to-abm-using-an-iphone
date_published: 2021-12-07T15:33:46.000Z
date_updated: 2021-12-07T15:33:46.000Z
tags: macOS, ABM
excerpt: Since iOS 11 we have been able to add iOS devices to Apple Business Manager using Apple Configurator 2 on a macOS device by connecting the device to the mac. Ever since this became available, most of us have wanted this feature for mac devices as well.
---

It's here, it's live and I love it.

Since iOS 11 we have been able to add iOS devices to Apple Business Manager using Apple Configurator 2 on a macOS device by connecting the device to the mac. Ever since this became available, most of us have wanted this feature for mac devices as well. Until now, they still needed to be bought from an authorized reseller to get them in.

During WWDC in June, Apple showed us that it will be possible to add macs to ABM using Apple Configurator on an iPhone, which finally is live in the App Store. The flow is very similar to adding an iOS/iPadOS device in that a server called "Apple Configurator 2" is created in the ABM console from which we must reassign the device to the MDM server you want to enroll the device to.

While you still need to be close to the device you at least do not have to physically connect to it to complete the assignment. I talk about Apple Business manager in this post, but it could just as well be done for Apple School Manager.

There are some pre-requisites to adding a mac to ABM this way,

- The iPhone must have **iOS 15**
- The macOS device must have **macOS Monterey** installed
- The macOS device must have a **T2 security chip** or **Apple Silicon**
- You need to be an administrator in ABM with the **Device Enrollment Manager role**

If you can check all the boxes above, you're ready to start assigning devices. Let's get into the fun part, actually doing it.

## Install Apple Configurator

- Start by going to the App Store on an iPhone with iOS 15 and search for "Apple Configurator", you will see an app with the icon below. Click on it and install the app
![appstore-1](__GHOST_URL__/content/images/2021/12/appstore-1.jpeg)
- Once installed, open the app, now you are presented with a login screen. Sign in using the credentials you normally use for the ABM console
![acsignin](__GHOST_URL__/content/images/2021/12/acsignin.jpeg)
- Grant access to the camera
![accamera](__GHOST_URL__/content/images/2021/12/accamera.jpeg)

That's it, the app is now prepared. There are some additional settings you can configure by tapping the gear icon. Here you can choose if you want to share the network you are connected to with the mac, or, if you want to use a configuration profile. If you do not want to share the network, create a configuration profile in Apple Configurator on a macOS device.
![acsettings](__GHOST_URL__/content/images/2021/12/acsettings.jpeg)

## Assign mac to ABM

- Bring the iPhone close to the mac you wish to assign. Make sure Bluetooth is enabled and that you are connected to a network if you want to share it. The mac will automatically present the assign screen
- Scan the screen with the Apple Configurator app
![acscan](__GHOST_URL__/content/images/2021/12/acscan.jpeg)
- The device will now be assigned, you can see devices assigned in the app by navigating to the menu in the lower right corner
![acdevicestatus](__GHOST_URL__/content/images/2021/12/acdevicestatus.jpeg)

## Assign the device to MDM

Now that the mac is in ABM, all you have to do is re-assign it to your MDM server.

- Go to [ABM](https://business.apple.com)
- Navigate to **Settings -> MDM servers** and click on **Apple Configurator 2**
![abmservers](__GHOST_URL__/content/images/2021/12/abmservers.png)
- Click on **Show devices**
![abmshowdevices](__GHOST_URL__/content/images/2021/12/abmshowdevices.png)
- Click your device and then click **Edit Device Management**
![abmedit](__GHOST_URL__/content/images/2021/12/abmedit.png)
- Choose your **MDM server** and click **Continue**
![abmassign](__GHOST_URL__/content/images/2021/12/abmassign.png)

Done, if you synchronize devices in Intune now, the mac device will show up just like any other ABM device and you'll be able to enroll it just like one.

To synchronize devices in Intune,

- Browse to the [MEM console](https://endpoint.microsoft.com)
- Navigate to **Devices -> iOS/iPadOS -> iOS/iPadOS enrollment -> Enrollment program tokens -> Click on the token where you have your profile -> Devices -> Sync**
