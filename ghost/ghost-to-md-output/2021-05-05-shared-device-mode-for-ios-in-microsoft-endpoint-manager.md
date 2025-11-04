---
title: Shared device mode for iOS in Microsoft Endpoint Manager
slug: shared-device-mode-for-ios-in-microsoft-endpoint-manager
date_published: 2021-05-05T13:30:42.000Z
date_updated: 2021-05-05T13:30:42.000Z
tags: iOS, MEM, Intune, Shared Device
excerpt: I often come across scenarios where organisations have users which do not have their own personal device because it's not required for them to do their job. Most often these job roles include nurses, flight crew and blue-collar workers.
---

I often come across scenarios where organisations have users which do not have their own personal device because it's not required for them to do their job. Most often these job roles include nurses, flight crew and blue-collar workers. Since iPadOS 13.4 we have been able to provision shared iPads using a shared device mode built in to the OS, this provides a very good login/logout flow and data separation for users. But more often than not, these users want to carry around an iPhone instead of an iPad. Keep in mind that this feature also works for iPads which don't have the right specifications to support Apples native Shared iPad.

Using iPhones as shared devices in Microsoft Endpoint Manager has been a challenge since we've not had any way of protecting the data on the device without sharing a passcode between multiple users. This poses a security risk as this passcode can easily leak resulting in unauthorised access to business data. Besides shared passcodes, if a user were to sign in to applications without signing out of all applications, other users can access that users personal data.

During Ignite in March, Microsoft released a new feature in preview for iOS called Shared Device Mode. This aims to fix the security concerns we have when sharing iOS devices between multiple users. It provides a login/logout flow on the device where a users signs in with their Azure AD credentials to apps rather than use a passcode, and when they logout from one app, all applications they are logged in to are automatically logged out making the device ready for the next user.

To managed users on the devices, Microsoft Authenticator is used together with the Single Sign On App Extension  to enable Shared Device Mode. To take full advantage of the login/logout flow in applications, your app needs to support handling shared device sign-out. It also needs to be a single account app, multiple account apps are not supported in shared device mode.

Benefits of Share Device Mode:

- User data resides in apps protected by clearing app data on sign-out
- Sign in using AAD credentials
- User must be a valid AAD user

One challenge I see will be to get apps used on shared devices to support this global sign-in/sign-out mode. Of course if you have an app which is developed in-house this might be easier as you can work together with your developers, but if you on the other hand are using an app from the app store, you have to contact the developers of the application and work with them to implement the changes.

Let's have a look at some pre-requisites:

- iOS 13 or higher device
- A user with Cloud Device Administrator role (required during preview only)
- Apps that support shared device sign-out

You can find guidance [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-ios-shared-devices#modify-your-ios-application-to-support-shared-device-mode) on how the application needs to be setup to support the shared device mode. 

Let's get started configuring!

## Create an enrollment profile for testing

In my case I will enroll the device using automated enrollment without having any user affinity. This is just to put the device under management and get the Microsoft Authenticator app installed and the SSO App Extension profile pushed.

1. In the MEM console, go to **Devices -> iOS/iPadOS -> iOS/iPadOS enrollment -> Enrollment program tokens**
2. Click on your token and then go to **Profiles**
3. Click on **Create profile -> iOS/iPadOS**
4. Give the profile a **Name** and then click **Next**
5. For **User Affinity**, choose **Enroll without User Affinity**
6. For **Management Options**, choose the following:

- Supervised: Yes
- Locked Enrollment: Yes
- Shared iPad: No
- Sync with computers: Allow all

7. Under Setup Assistant settings, configure your **Dapartment** and **Departnemt Phone**. Then on Setup Assistant Screen, click **Toggle All** to hide all screens, finally click Next
8. Click **Create**

To assign a test device to your newly created profile, go to devices on your token, find the device you'd like to test with in the list, select it and click Assign profile.

## Create a dynamic group for shared devices

To make assignment of Microsoft Authenticator and the configurations easy, I'm going to setup a dynamic group that automatically adds devices assigned to the enrollment profile we created in the last step.

1. In the MEM console, go to **Groups** and click on **New group**
2. Give the group a name and for **Membership type**, choose **Dynamic Device**
3. Click **Add dynamic query** and provide the following syntax:

- (device.enrollmentProfileName -eq "YOURPROFILENAME")

4. Click on **Save** and then **Create**

![](__GHOST_URL__/content/images/2021/05/image.png)
## Configure a SSO App Extension profile

The next thing we have to do is to create an SSO App Extension profile and put it in shared device mode.

1. In the MEM console, go to **Devices -> iOS/iPadOS -> Configuration Profiles** and click on **Create Profile**
2. For Profile type, choose **Device features** and click **Create**
3. Give you profile a **name** and then click **Next**
4. In the list of settings, choose **Single sign-on app extension**
5. For SSO app extension type, choose **Microsoft Azure AD**
6. Choose to enable **Shared device mode**
7. Click Next
8. Assign the configuration profile to the dynamic group and click Next
9. Click on Create

![](__GHOST_URL__/content/images/2021/05/image-1.png)
## Configure the Microsoft Authenticator App

 In my test, I'm deploying the Microsoft Authenticator app using VPP to be able to silently push it in the background without any Apple ID on the device.

1. Once you have the app in Intune, go to **Apps -> App configuration policies**
2. Click on **Add -> Managed devices**
3. Give your configuration a name, choose iOS/iPadOS as the platform and target the Microsoft Authenticator app
4. Choose **Use configuration designer** and provide the following key:

- Key: sharedDeviceMode
- Type: Boolean
- Value: true

5. Click **Next**, assign the configuration to the dynamic group and then click **Next**
6. Finally click **Create**

![](__GHOST_URL__/content/images/2021/05/image-2.png)
## Prepare device

When the above steps has been completed and the device is enrolled, you now need to open Microsoft Authenticator and sign-in with your Cloud Device Administrator account, doing this, the device will register the shared device to your tenant. Microsoft provides a demo app [here](https://github.com/Azure-Samples/ms-identity-mobile-apple-swift-objc) if you want to test out the login/logout flow. If you want to build and run this app on a real device you will need an Apple Developer account.

If all goes well, this is what the Authenticator app will look like:
![](__GHOST_URL__/content/images/2021/05/image-4.png)
The device is now ready for users to begin signing in to and use their apps. To tidy things up I pushed additional profiles to my device that hides all built in apps. I also configured a new profile using the new Home Layout flow to specify where apps should appear on the home screen.

This is a feature which as stated earlier, is in **preview**. If you are already using applications that take advantage of MSAL and you use shared devices I think this is something you should look into.
