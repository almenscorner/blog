---
title: Shared iPads in MEM
slug: shared-ipad-in-mem
date_published: 2021-12-01 16:30:00.000000000 Z
date_updated: 2021-12-01 16:30:00.000000000 Z
tags:
- abm
- ios
- ipados
- shared device
excerpt: Today, we are looking at the built in functionality from Apple to support
  a shared scenario on iPad devices in Microsoft Endpoint Manager.
---
It has been a lot of focus on macOS here lately so It's time for another post on iOS ;)

A few months ago, we had a look at how you can configure shared device mode in Intune for iOS devices, which is a method of using the Authenticator app and MSAL to support a sign in/sign out flow. Today, we are looking at the built-in functionality from Apple to support a shared scenario on iPad devices in Microsoft Endpoint Manager.

Instead of building it on an app level, users are separated in the OS and a sign in/sign out flow is provided by iPadOS directly. This option has been available in iPadOS for quite some time but in the September release of Intune, support for User Channel settings finally became available. This means that even though it's a shared device, we can target specific settings, applications to specific users. Previously we could only target the Device Channel, i.e., all users were targeted by all settings and applications. So, even though the device is shared, we can build a personal experience for the user.

When we looked at setting up shared device mode without using the built-in functionality of iPadOS, there's one thing which might make this a better option for you. The requirement of having MSAL integrated into the apps you want to use with shared device mode. There might be several reasons why this may be a concern for you, for example, it might be a public App Store app and you don't have a relationship with the developer.

If you're not familiar with this option for iPad devices, it has been available since iPadOS 13.4 for business use and even longer for Education with Apple School Manager. By the end of this post, you will have a good understanding of the pre-requisites, how to configure shared iPads and how to deploy them.

## Notes

A few things you need to think about

- 
This feature is in preview

- 
Federated Apple Business Manager

- Since we are signin in with Managed Apple IDs, your ABM should be federated with Azure AD. I have previously written about this, you can [find this post here]({{ site.baseurl }}managing-ios-ipados-part-4-identity/#federated).

- 
Supported devices (must have at least 32GB storage)

- iPad Pro
- iPad (5th generation or later)
- iPad Air 2 (or later)
- iPad mini (4th generation or later)

- 
Content caching

- If content caching is set to iCloud, the data is saved to iCloud using the caching service. If the user signs into a new shared iPad, the data is downloaded from iCloud. Apple recommends planning for users to sign into the same shared iPad the user has used before.

- 
Storage

- Consider the amount of data that will be stored on the device and how much storage the iPad has. The storge left when all data is loaded on the device is how much you can allocate to your users.
- Devices with a storage capacity of 32GB

- 10GB for the system, 8GB for apps and media. The remaining storage is divided among the number of defined users, with 1GB minimum per user.

- Devices with a storage capacity of 64GB or greater

- 10GB for the system, 16GB for apps and media. The remaining storage is divided among the number of defined users, with 2GB minimum per user.

- 
User caching

- Per default, the shared iPad will be configured to cache 10 users. You can configure this value in Intune, on an 32GB or 64GB iPad, you can cache a maximum of 24 users.

- 
Temporary sessions

- Enabled by default, this means that any user can tap the guest icon and sign into the device as a guest without having to enter a Managed Apple ID. This can be blocked by configuring a device restriction configuration profile.
![Screenshot-2021-11-30-at-22.03.26]({{ site.baseurl }}/assets/images/2021/11/Screenshot-2021-11-30-at-22.03.26.png)

## Device configurations

As you can see in the table below, device configurations can be applied to both user and device channel. Generally, device applicable settings take effect on any user on the device and user applicable settings apply on any shared device the user signs in to.
Profile typeSetting nameApplicability on device group assignmentApplicability on user group assignmentDevice featuresHome screen layoutDeviceUserDevice featuresApp notificationsDeviceUserDevice featuresSingle sign on app extensionDeviceUserDevice featuresAirPrint settingsDeviceNot applicableDevice featuresLock screen messageDeviceNot applicableDevice featuresWeb content filterDeviceNot applicableDevice restrictionsBlock Shared iPad temporary sessionsDeviceNot applicableDevice restrictionsDefer software updatesDeviceNot applicableDevice restrictionsForce automatic date and timeDeviceNot applicableDevice restrictionsRequire joining Wi-Fi networks only using configuration profilesDeviceNot applicableDevice restrictionsBlock auto lockDeviceNot applicableDevice restrictionsAllow users to boot devices into recovery mode with unpaired devicesDeviceNot applicableDevice restrictionsBlock Siri for dictationDeviceNot applicableDevice restrictionsAll other settings in device restrictionsDeviceUserEmailAll settingsDeviceUserVPN, Wi-Fi, CertificateAll settingsDeviceNot applicable
When configuring settings for shared iPads, keep the following recommendations in mind,

- If all users have the same role and should have the same settings and applications, assign all settings and applications to a device group containing the shared iPad
- If users have different roles and needs different settings and applications, assign all common settings and applications that should apply to everyone to a device group containing the shared iPad and assign role/user specific settings and applications to a user group
- It is not recommended to push the same setting twice to a device as this can cause conflicts

## Configure enrollment profile

First thing we need to do is to configure an enrollment profile in Intune that the iPads will use.

- Sign in to the [MEM console](https://endpoint.microsoft.com)
- Navigate to **Devices -> iOS/iPadOS Devices -> iOS/iPadOS Enrollment -> Enrollment Program Tokens -> Choose your token -> Profiles**
- Click **Create Profile** and choose **iOS/iPadOS**
- Give the profile a **name** and click **Next**
- Select the following options:

- **Enroll without user affinity**
- **Supervised:** Yes
- **Locked enrollment:** Yes
- **Shared iPad:** Yes
- **Maximum cached users:** Select how many users you want to cache or leave it at the default of 10
- **Maximum seconds after screen lock before password is required:** 0-14,400 seconds
- **Maximum seconds of inactivity until user session logs out:** Available for iPadOS 14.5 and up, let's you configure the amount of inactivity in seconds before the user is signed out
- **Require Shared iPad temporary session only:** When set, users only see the guest welcome screen and won't be able to sign in with their own account, available for iOS 14.5 and up
- **Maximum seconds of inactivity until temporary session logs out:** Available for iPadOS 14.5 and up, let's you configure the amount of inactivity in seconds before the guest user is signed out
- **Sync with computers:** Allow or deny based on your preference
- Click **Next**

- For **Setup Assistant**, fill out **Department**, **Department Phone** and choose which screens, if any, should be shown while enrolling
- Save and create your profile
![sharedprofile]({{ site.baseurl }}/assets/images/2021/11/sharedprofile.PNG)

## Create dynamic group and assign devices

First we will create a filter that we are going to use when targeting our shared iPads with configurations and applications.

- Navigate to **Groups -> New group**, provide a group name and for **membership type** select **dynamic device** and click **Add dynamic query**
- Configure the rule like below,

PropertyOperatorValueenrollmentProfileNameEquals{Your Profile Name}
- Click **Save** and then **Create**

![Screenshot-2021-11-30-at-22.01.28]({{ site.baseurl }}/assets/images/2021/11/Screenshot-2021-11-30-at-22.01.28.png)

Now we have a group that we can target when assigning our shared iPads configurations and applications. Now let's assign the iPad(s) that should be enrolled as shared.

- Navigate to **Devices -> iOS/iPadOS Devices -> iOS/iPadOS Enrollment -> Enrollment Program Tokens -> Choose your token -> Devices**
- Find a iPad in the list you want to be shared, select it and click **Assign profile**
- Choose the profile you created earlier and click **Assign**

![Screenshot-2021-11-30-at-22.02.20]({{ site.baseurl }}/assets/images/2021/11/Screenshot-2021-11-30-at-22.02.20.png)

**Assigning apps**, when assigning apps, it is recommended to assign all apps to a device group and if users should see different apps, assign a home screen layout configuration profile that shows or hides apps for different sets of users. The home screen layout would then be assigned to a user group.

App assignment applicability
App typeApplicability on device group assignmentApplicability on user group assignmentLine-of-business appDeviceNot applicableDevice-licensed volume-purchased or custom app (VPP)DeviceNot applicableUser-licensed volume-purchased or custom app (VPP)Not applicableNot applicableWeb appDeviceUserApp Store appNot applicableNot applicable
## Enrollment and user experience

When enrolling a device assigned to the shared ipad enrollment profile it will automatically enroll without the need to authenticate since we are enrolling without user affinity. Once the enrollment is complete the device will reboot and then you're presented with the sign in screen like below. If you have not turned off temporary sessions, the guest icon is visible in the lower right corner of the screen.
![Screenshot-2021-12-01-at-13.20.40]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-01-at-13.20.40.png)

From here, the user can now sign in with their managed and federated Apple ID. And since it's federated, the user will be redirected to Microsoft to sign in. After authenticating, the user will have to choose language and region.
![Screenshot-2021-12-01-at-13.23.31]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-01-at-13.23.31.png)

![Screenshot-2021-12-01-at-13.23.49]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-01-at-13.23.49.png)

The next step is to set a passcode, this passcode will be used to unlock the iPad from this point on instead of entering the Azure AD password. Once done, the user lands on the home screen.
![Screenshot-2021-12-01-at-13.28.47]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-01-at-13.28.47.png)

In my setup, I installed some applications assigned them to my dynamic device group and used one home screen layout configuration, in addition I created a restriction to hide Outlook and Teams for James May. Below you can see that Jeremy Clarkson can see and use Microsoft Outlook and Teams while these apps are not displayed for James May. To sign out of the iPad, the user only has to swipe down from the top and tap "sign out"
![Screenshot-2021-12-01-at-14.38.43]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-01-at-14.38.43.png)
![Screenshot-2021-12-01-at-14.33.40]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-01-at-14.33.40.png)

## Closing words

I really like this option provided by Apple since there's no requirements on integrated libraries in the applications to provide the flow. And now that User Channel is available and we're able to customize the experience per user/role it's even better.

You can find additional information from [Apple here](https://support.apple.com/en-in/guide/mdm/cad7e2e0cf56/web) and [Microsoft here](https://docs.microsoft.com/en-us/mem/intune/enrollment/device-enrollment-shared-ipad#add-apps-on-shared-ipads)
