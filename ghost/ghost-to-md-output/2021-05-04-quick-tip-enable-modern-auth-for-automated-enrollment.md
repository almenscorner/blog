---
title: "Quick tip: Enable modern auth in Setup Assistant"
slug: quick-tip-enable-modern-auth-for-automated-enrollment
date_published: 2021-05-04T13:34:19.000Z
date_updated: 2021-05-04T13:34:19.000Z
tags: iOS, Intune, ABM
excerpt: With the update for week of April 12th Microsoft released New modern authentication method with Apple Setup Assistant in public preview. This is something I've been waiting for ever since custom DEP was a thing and now it's finally here.
---

With the update for week of April 12th Microsoft released New modern authentication method with Apple Setup Assistant in public preview. This is something I've been waiting for ever since custom DEP was a thing and now it's finally here.

This means we don't have to put the Company Portal in single app mode and require the user to sign in before they can use the device. This has been a frustrating flow for both users and admins, many times the devices locked up in guided access mode without any straightforward way to get out of it.

For organisations that require MFA to enroll devices the Company Portal method has long been the only way to accomplish user affinity with MFA, but not anymore.

## Modern auth in setup assistant

> This method provides all the security from Setup Assistant but avoids the issue of leaving end users stuck on a device they can't use while the Company Portal installs on the device

To gain access to resources protected by Conditional Access, the user still has to sign into Company Portal post enrollment, but, the device is immediately registered to the user so apps and configurations assigned to the user flows down to the device before this sign-in occurs. This is a huge win as I've seen many users confused about having to open an app and sign in before they get their apps. Keep in mind though that the Azure AD registration will not be complete until the Company Portal sign-in is complete.

For iOS/iPadOS, the correct Company Portal version is automatically pushed to devices as a required app. For macOS you have to add the app to MEM. You can find guidance on how to do that [here](https://docs.microsoft.com/en-us/mem/intune/apps/apps-company-portal-macos). 

- 
For iOS/iPadOS: The Install Company Portal setting will not be there when choosing this flow for iOS/iPadOS. The CP will be a required app on the device with the correct app configuration policy on it once the end user lands on the home screen. User must sign in with Azure AD credentials into the CP after enrollment to gain access to resources protected by Conditional Access and be fully AAD registered.

- 
For macOS: Users must sign into the Company Portal to complete Azure AD registration and gain access to resources protected by Conditional Access. The end user will not be locked to the CP after landing on the home page, but an additional login into the CP will be required to access corporate resources and be compliant.

## Configure modern auth in Setup Assistant

Let's get started configuring!

1. Sign into the [MEM console](https://endpoint.microsoft.com)
2. Go to **Devices -> iOS/iPadOS -> iOS/iPadOS enrollment -> Enrollment program tokens**
3. Click on the token where you have the device you'd like to test with and then click on **Profiles**
4. Click on Create **profile -> iOS/iPadOS**
5. Give the profile a **Name** and click **Next**
6. For **User affinity** select **Enroll with user affinity**
7. Under **Authentication method** you can now see the new **Setup Assistant with modern authentication (preview)**
8. Configure the rest of the enrollment profile as you normally would

- As a reference, this is my profile![modernauthprofile](__GHOST_URL__/content/images/2021/05/modernauthprofile.png)

9. Save and create your profile
10. Assign your new profile to a test device

This is the end user experience with modern auth in Setup Assistant
![](__GHOST_URL__/content/images/2021/05/modernauth.gif)
