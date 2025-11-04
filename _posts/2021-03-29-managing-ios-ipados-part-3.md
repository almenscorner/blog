---
title: 'Managing iOS/iPadOS Part 3: Configurations & Apps'
slug: managing-ios-ipados-part-3
date_published: 2021-03-29 15:52:23.000000000 Z
date_updated: 2021-03-29 15:52:23.000000000 Z
tags:
- ios
- mem
excerpt: Today's topic is all about configuring our iOS/iPadOS devices, we'll have
  a look at how to set up configuration profiles in MEM as well as compliance policies,
  how to integrate ABM/ASM to be able to push apps to devices without Apple IDs and
  adding apps from the App Store.
---
Today's topic is all about configuring our iOS/iPadOS devices, we'll have a look at how to set up configuration profiles in MEM as well as compliance policies, how to integrate ABM/ASM to be able to push apps to devices without Apple IDs and adding apps from the App Store.

This is part three of this series covering management of iOS/iPadOS, you can find part one and two here:

[Managing iOS/iPadOS Part 1: The basics](__GHOST_URL__/managing-ios-ipados-part-1-the-basics/)
[Managing iOS/iPadOS Part 2: Deployment methods](__GHOST_URL__/managing-ios-ipados-part-2-deployment-methods/)

**Table of contents for easy navigation**

- [**Configuration Profiles**](#configuration-profiles)
- [Creating a Configuration Profile in MEM](#creating-a-configuration-profile-in-MEM)

- [**Compliance Policies**](#compliance-policies)
- [**Apps**](#apps)
- [Integrating ABM/ASM apps with MEM](#integrating-ABM-ASM-apps-with-MEM)
- [Add apps to Volume Purchase Program (VPP)](#add-apps-to-volume-purchase-program)
- [Add apps via App Store](#add-apps-via-app-store)
- [Line of Business apps](#line-of-business-apps)
- [*Uploading to MEM*](#uploading-to-MEM)
- [*Custom Apps*](#custom-apps)

- [**Wrapping up**](#wrapping-up)

# Configuration Profiles

To do a quick recap on configuration profiles, a **configuration profile** is what we configure in MEM to be able to automatically configure settings like restrictions and credentials. **Payloads** is what we add to the configuration profile, this is the settings we add to the configuration profile and that's applied to the device. Many of the payloads we can configure requires the device to be supervised as they are more intrusive for the user.

Settings requiring supervision include:
Device FeaturesDevice RestrictionsHome Screen LayoutBlock App StoreApp notificationsBlock Installing apps using App StoreLock Screen MessageBlock automatic app downloadsWeb Content FilterBlock playback of explicit music, podcast and iTunes UWallpaperBlock adding Game Center friendsBlock Game CenterBlock multiplayer gamingBlock access to network drive in Files appSingle App ModeBlock cameraBlock FaceTimeRequie Siri profanity filterBlock user-generated content in SiriBlock Apple NewsBlock Apple BooksBlock iMessageBlock PodcastsMusic serviceBlock iTunes RadioBlock iTunes storeBlock Find My iPhoneBlock Find My FriendsBlock user modification to the Find My Friends settingsBlock removal of system apps from deviceBlock SafariBlock Safari AutofillBlock iCloud backupBlock iCloud document and data syncBlock iCloud Keychain syncBlock AirDropBlock pairing with Apple WatchBlock modifying bluetooth settingsBlock pairing with non-Configurator hostsBlock AirPrintBlock storage of AirPrint credentials in KeychainRequire AirPrint to destinations with trusted certificatesBlock iBeacon discovery of AirPrint printersBlock setting up new nearby devicesBlock access to USB drive in Files appSafari password domainsBlock modification of diagnostics settingsBlock remote AirPlay, view screen by Classroom app, and screen sharingAllow Classroom app to perform AirPlay and view screen without promptingBlock modification of account settingsBlock Screen TimeBlock use of erase all content and settingsBlock modification of device nameBlock modification of notifications settingsBlock modification of WallpaperBlock configuration profile changesAllow activation lockBlock removing appsAllow USB accessories while device is lockedForce automatic date and timeRequire teacher permission to leave Classroom app unmanaged classesAllow Classroom to lock to an app and lock the device without promptingAllow students to automatically join Classroom classes without promptingBlock VPN creationBlock modification of eSIM settingsDefer software updatesDelay visibility of software updatesBlock word definition lookupBlock predictive keyboardsBlock auto-correctionBlock spell checkBlock keyboard shortcutsBlock dictationBlock QuickPathKiosk settingsBlock Wallet notifications in lock screenBlock passcode modificationBlock modification of Touch ID fingerprints and Face ID facesBlock password AutoFillBlock password proximity requestsBlock password sharingRequire Touch ID or Face ID authentication for AutoFillRestricted AppsShared iPadShow or hide appsBlock changes to app cellular data usage settingsBlock changes to cellular plan settingsBlock modification of personal hotspotRequire joining Wi-Fi networks only using configuration profilesRequire Wi-Fi always on
As you can see from the table above, many of the restrictions are supported only on supervised devices. This list grows with new iOS updates, it is recommended to use supervision for company owned devices.

## Creating a Configuration Profile in MEM

To create profiles in MEM, follow the steps below

1. Go to the [MEM console](https://endpoint.microsoft.com)
2. Navigate to **Devices -> Configuration Profiles**
3. Click **Create Profile**
4. For **Platform**, choose **iOS/iPadOS**
5. Now you see all the several types of payloads we can configure for the devices. Options include:

- Custom
- Derived Credential
- Device Features
- Device Restrictions
- Email
- PKCS certificate
- PKCS imported certificate
- SCEP certificate
- Trusted certificate
- VPN
- Wi-Fi

6. Choose the type of profile you want to configure and click **Create**. If you for example would like to set a passcode requirement, choose device restrictions
7. Give the profile a **Name** and a **Description**, then click **Next**
8. Configure the settings you want to apply to devices and then click **Next**
9. Assign your profile to a group and click **Next**, then click **Create**

You have now created a profile that will be automatically applied to devices and the settings you picked will be configured on the device.

Look into the different profile types available to see what you can configure and what might apply to your specific use case. Some of the most common profiles include **Passcode, Wi-Fi, VPN** and **Restrictions**.

# Compliance Policies

Compliance policies is an important part of every platform you wish to enroll to MEM. Using these policies, you can require the device to meet certain security measures before the user can access corporate data. For example, if a device does not have a passcode set or if the device has been jailbroken, you can set the device as noncompliant and block access to resources using conditional access policies.

Compliance also ties into Mobile Threat Defense (MTD) like Microsoft Defender For Endpoint which offers protection against phishing and unsafe network connections from apps, emails and websites. If a threat is detected on a device, the Defender app reports back to the Microsoft Defender Security Center where you define the threat level. You can then require the device to be at or under a certain threat level before allowing access. For devices managed by App Protection Policies (MAM) this would be under the Conditional Launch settings in the policy.

Configure a compliance policy,

1. In the MEM console, navigate to **Devices -> Compliance policies**
2. Click **Create Policy** and choose **iOS/iPadOS** for Platform, then click **Create**
3. Give the compliance policy a **Name** and a **Description**, then click **Next**
4. For **Compliance settings** we can configure the following

- **Unable to set up email on the device**
- A managed email account is required. If the user already has an email account on the device, the email account must be removed so Intune can set one up correctly. If no email account exists on the device, the user should contact the IT administrator to configure a managed email account.

- **Device Health**
- Jailbroken devices

- Allow or block devices detected as jailbroken

- Require the device to be at or under the Device Threat Level

- Select the maximum allowed device threat level for devices evaluated by your connected Threat Defense services. Devices which exceed this threat level get marked as noncompliant. (This compliance check is supported for devices with OS versions iOS 8.0 and above)

- **Device Properties**
- Minimum OS version
- Maximum OS version
- Minimum OS build version
- Maximum OS build version

- **Microsoft Defender for Endpoint**
- Select the maximum allowed machine risk score for devices evaluated by Microsoft Defender for Endpoint. Devices which exceed this score get marked as noncompliant.

- **System Security**
- Require a password to unlock mobile devices
- Device Security

- Add bundle IDs for apps that should be restricted. A device that has at least one app installed which is found on the restricted apps list will be marked as non-compliant.

- Select the settings that best fit your use case, at least you should require a passcode and block jailbroken devices, then click **Next**

5. Under **Actions for noncompliance** you can configure what should happen if the device does not meet the compliance policy

ActionSchedule
 (days after noncompliance)Message templateAdditional recipients
 (via email)The action to take if a device
 does not meet complianceSchedule what should happen after
 specified number of days noncompliantIf you select to send an email to the end user,
 you can configure a message template of your ownIf send email is selected,
 choose if you want additional recipients such as an admin
- Available actions

- Mark device as noncompliant
- Send email to end user
- Send push notification to end user
- Remotely lock the noncompliant device
- Retire the noncompliant device

- With these options you can easily build out a flow of what should happen if the device become noncompliant, in the example below I mark the device as noncompliant as soon as it is detected that the device is breaking a compliance rule, on day two I send both a push notification to the user and an email, if the device still fails compliance on day three I retire the device (enterprise wipe) and send an email to the user and an admin

    flowchart TB;
    subgraph Day3
    a1(Send email to user and 
 admin) --- a2(Retire device)
    end
    subgraph Day2
    b1(Send push notification 
 to device) --- b2(Send email to user)
    end
    subgraph Immediately
    c1(Mark device noncompliant) --- c2(Send email to user)
    end
    Immediately --> Day2
    Day2 --> Day3

- When you have configured your actions for noncompliance, click **Next**

1. Assign you compliance policy and click **Next**
2. Review your settings and then click **Create**

# Apps

There are different ways of deploying applications to devices from MEM. You can either add applications directly in the MEM console from the App Store if your organization does not use ABM/ASM, or, you can integrate ABM/ASM and distribute apps from these portals. The benefit of using ABM/ASM is that you can install applications on devices without requiring an Apple ID, you can get licenses for applications and revoke those licenses if a user leave the company.

When ABM/ASM is integrated and you get licenses for a free or paid app, the app inventory is synchronized to the MEM console. From here you're able to assign the apps to groups of users or devices, Apple then handles the installation of these apps. This way, you have a good overview within MEM on the usage of licenses for apps.

We'll walk through the steps necessary to integrate the apps section in ABM/ASM with MEM, as well as adding apps from the App Store and touching on how to handle Line of Business apps.

## Integrating ABM/ASM apps with MEM

I will do these steps in ABM, the same steps apply to ASM.

1. Sign into the [ABM portal](https://business.apple.com)
2. Navigate to **Settings -> Apps and Books**, under **My server tokens**, click on **Download** for your location
3. Go to the [MEM console](https://endpoint.microsoft.com)
4. Navigate to **Tenant administration -> Connectors and tokens -> Apple VPP tokens** and click on **Create**
5. Provide a **Name** for the token, enter the **Apple ID** you used to download the token, **upload the token** you downloaded in step 2, then click **Next**
6. Choose **Country/region**, specify the type of VPP account you have, **Business** or **Education**, select whether you want all apps associated with this token to be **automatically updated** or not, then check **"I grant Microsoft permission to send both user and device information to Apple."** and click **Next**
7. If you are using scope tags add them now, otherwise click **Next** and then **Create**

Once above steps are completed you will see that Status is active.
![]({{ site.baseurl }}/assets/images/2021/03/vpp.png)Successful VPP integration
**Note:** This token must be renewed annually

 mark {
   background: #FFFF00 !important;
    }

## Add apps to Volume Purchase Program (VPP)

Now that we have successfully integrated ABM to MEM, the next step is to add apps. Follow the steps below to add apps and synchronize them to MEM.

1. Sign into the [ABM portal](https://business.apple.com)
2. Navigate to **Apps and books**
3. At the top, search for the application you want and click on it

- Specify the **location** you wish to add the app for
- Enter the **quantity** of licenses needed
- Click **Get**

4. The licenses will now be processed, depending on how many licenses you requested it might take a while
5. When the licenses have been processed, the app will synchronize to MEM.

- If you don't see the app you can speed up the sync by clicking **(...)** to the right on Apple VPP tokens overview and click **Sync**

6. In MEM, navigate to **Apps -> iOS/iPadOS apps**. The app should now be in the list with a type of **iOS volume purchase program app**
7. Click on the newly synchronized app

- Click **Properties**
- Click **Edit** on **Assignments**
- Choose whether you want the app to be **Required** or Available for **enrolled devices**
- Add a **group** or **All devices/users**
- Once a group is added, click on **User** under **License Type** and switch to **Device licensing**, with this option enabled users won't need an Apple ID
- **Uninstall on device removal** allows you to specify if the app should be removed from the device if it unenrolls from MEM. By doing so, you're also reclaiming the license
- **Install as removable** allows you to specify if the user should be able to uninstall the app or not
- Once you're finished with the configuration, clock **Ok -> Review + save -> Save**

If you chose Required, the app will now be pushed to the users you assigned the app to. If the device is supervised, this process will happen in the background and the user won't be prompted to accept the installation.
![]({{ site.baseurl }}/assets/images/2021/03/vppdevice.png)VPP device licensing
## Add apps via App Store

If you're not able to use the above method for whatever reason, you can still push apps from the App Store to your devices. This requires the user to be signed into an Apple ID on the device and accept the app installation. To add App Store apps, follow the steps below.

1. In the MEM console, navigate to **Apps -> iOS/iPadOS** and click **Add**
2. Choose **iOS store app** as the type and click **Select**
3. Click **Search the App Store**, enter the name of the app you wish to add, click on the app, and then click **Select**
4. Review the **App information screen**, then click **Next**
- Choose whether you want the app to be **Required** or Available for **enrolled devices**
- Add a **group** or **All devices/users**
- Click **Next**

5. On the **Review + create screen**, click **Create**

If the app was added as a Required app, it will now start to push to devices and users will have to accept the installation.

## Line of Business Apps

### Uploading to MEM

If you have a line of business app developed in-house or by a third-party developer, you can upload the .ipa file to MEM and push the app to devices. When you use this method to deploy apps, a provisioning profile comes bundled with the app, this profile expires and must be renewed on an ongoing basis. A provisioning profile authorizes developers and devices to create and run Apple iOS applications. If this profile is not renewed the app will stop working.

### Custom Apps

Instead of using the above method to upload in-house or third-party developer apps, I would recommend using Custom Apps in ABM. With this option you can use the same deployment method as for all other apps added using this service. You can also use TestFlight to push beta apps to users before they are pushed widely in the organization. Apple states that most enterprise customers should see this as the way forward.

Benefits of using Custom Apps,

- One program to manage all of your apps (internal and external)
- Apps don’t expire
- Apps are managed individually
- Apps can be distributed to much larger audiences
- Easier to work with third-party software vendors
- Additional App Store features
- TestFlight and App Store Connect tools

# Wrapping up

We're reaching the end of this four part series of managing iOS/iPadOS in MEM. So far we've looked at the basics of iOS/iPadOS in MDM to get an understanding of the core concepts of management, different deployment methods available to handle both BYOD and corporate owned devices and today how we can configure our devices. Next time, and the last part, we'll look into managing identities. Specifically how we can create a good user experience with SSO and federation to handle managed Apple IDs.
