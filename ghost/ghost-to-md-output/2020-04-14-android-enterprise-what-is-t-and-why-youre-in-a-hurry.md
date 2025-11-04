---
title: Android Enterprise, what is it and why you're in a hurry
slug: android-enterprise-what-is-t-and-why-youre-in-a-hurry
date_published: 2020-04-14T08:21:20.000Z
date_updated: 2021-03-11T21:46:56.000Z
tags: Android, Intune, MEM
---

If you&#8217;re in charge of the mobile fleet in your organization you probably already know what Android Enterprise and Device Admin is. But I&#8217;ve had surprisingly many conversations with customers who know next to nothing about Android Enterprise and why they have to move away from Device Admin so I thought we should spend some time going over what&#8217;s needed to migrate, why you need to do it and what it is

## So what is it?

### History

let&#8217;s start with talking a bit about what Android Enterprise is.

Lets go back to the beginning. With Android 2.2 DA was introduced to help with managing Android in the enterprise, not many APIs were released with Device Admin initially though and could certainly not compete with the likes of iOS and BlackBerry. But since Android is open source and manufacturers can tweak the OS this was the beginning for Samsung&#8217;s big role in the enterprise market.

Samsung started early with their Knox (SAFE) which has over 1000 API&#8217;s available, it&#8217;s by far, also the most supported manufacturer by UEM platforms. Other manufacturers also added API&#8217;s for enterprise management through UEM but could not compete with Samsung&#8217;s portfolio.

This created a very fragmented management for enterprises as all manufacturers had their own API&#8217;s, on some platforms you could not even configure email if you did not use Samsung with Knox capability. Google took note of this and released Android For Work.

### Present

In Android 5.0 (Lollipop) Google released Android For Work, today known as Android Enterprise. At this time it was still optional for manufacturers to implement this framework. In Android 6.0 (Marshmallow) that changed and was mandatory, and thank god for that. Since then we&#8217;ve had a common way of managing Android devices instead of relying on different API&#8217;s from all manufacturers.

Android Enterprise also came with some much needed features for a enterprise , some of these include:

- Mandatory encryption
- Silent app installation (no user interaction)
- Per-App VPN
- App Config
- Managed Google Play
- Managed Google Accounts
- OOBE enrolment (Google Zero Touch/Samsung Knox Mobile Enrollment)
- Being able to configure a device as a Kiosk
- Separated work and personal data
- A common way of managing device from different manufacturers

There are different ways you can enrol and manage a device:
![](__GHOST_URL__/content/images/wordpress/2020/04/Annotation-2020-04-12-222821-2.png)
As you can see there&#8217;s flexibility in how you can manage an Android Enterprise device, currently MEM supports BYOD (Work Profile), Corporate Owned (Device Owner) and Kiosk. So why is Corporate Owned, Personally Enabled (COPE) not supported? it&#8217;s because Microsoft uses Google&#8217;s API&#8217;s and not their own DPC. But that&#8217;s for another time.

## Why do you need to migrate?

Short answer, Google has deprecated Device Admin as a option to manage Android using UEM platforms. This means that management API&#8217;s will begin to stop functioning. With the release of Android 10 you&#8217;re not able to enforce new password requirements and this is just the beginning. Additional limitations for Device Admin managed devices include:

- For all devices running Android 10 and later, Google has restricted the ability for device administrator management agents like Company Portal to access device identifier information. This restriction impacts the following Intune features after a device is updated to Android 10 or later:Network access control for VPN will no longer work.
- Identifying devices as corporate-owned with an IMEI or serial number won&#8217;t automatically mark devices as corporate-owned.
- The IMEI and serial number will no longer be visible to IT admins in Intune

For now, Samsung devices will not be impacted by the password deprecation since they can be managed via Knox but who knows for how long?

> Microsoft states that
> Intune will only be able to provide full support for device administrator-managed Android devices running Android 10 and later through Q2 CY2020

So yes, if you have not started yet. You are in a hurry.

## What now, what should you do?

First of, you need to decide which of the above management modes you want to use and what they mean for the end user and you as an admin.

If you go for BYOD (Work Profile) work and personal data will be separated on the device. This means that the user will have two play stores for example. Users probably won&#8217;t know how to manage this so they need information and training prior to migrating. From the admin side of things you can only wipe the work area on the device, i.e. you cannot perform a device wipe. Have your App Protection policys in place to ensure corporate data is not copied to the personal area. Enroling as Work Profile does not require the user to factory reset their device.

If you go for Corporate Owned (Device Owner) the user will have to factory reset their device before they enrol the device in Android Enterprise. This is more intrusive for the end user but gives you more control over the device. For example when the device is enroled with Device Owner you can perform a full device wipe. It also gives you the ability to use Google Zero Touch and Samsung Knox Mobile Enrolment for automatic registration of devices with MEM. You can also completley lock down Google Play to only show the apps you approve or give the user the option to also add their own Google account.

Independatly of which mode you choose, end users will need lots of information and training.

In this post I won&#8217;t show how to integrate MEM with Android Enterprise. You can find details on how to do that here: [https://docs.microsoft.com/mem/intune/enrollment/android-enterprise-overview](https://docs.microsoft.com/sv-se/mem/intune/enrollment/android-enterprise-overview)

If you think this post was useful, feel free to drop me a comment 😊
