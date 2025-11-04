---
title: Personal data in MS Authenticator but it's managed, now what?
slug: personal-data-in-ms-authenticator-but-its-managed-now-what
date_published: 1970-01-01T00:00:00.000Z
date_updated: 2021-12-22T15:59:59.000Z
draft: true
---

I had a [discussion on Twitter](https://twitter.com/considerITman/status/1470646189309763584?s=20) a couple of weeks ago regarding users wanting to back up their Microsoft Authenticator data but it's a managed app and the restriction "Block managed apps from storing data in iCloud” which blocks backing up managed app data to iCloud is configured. In this thread, I said that if you unassign a VPP app from a user it will be un-installed since the license is revoked. No Tobias, [that is not how it works](https://docs.microsoft.com/en-us/mem/intune/apps/vpp-apps-ios#revoking-app-licenses)...
![](__GHOST_URL__/content/images/2021/12/Screenshot-2021-12-22-at-14.21.44.png)
What really needs to happen for an application to be removed and license to be revoked from the device is you have to first unassign the app from the user/device, then you have to revoke the license and if you want the application to be removed, you have to change the assignment to "uninstall". But hey, we all make mistakes, right? ;)

Even though the app is not removed, the problem still stands. Since the application is installed from MDM, it is still managed and the restrictions apply.

To try and find a solution to allow for users to back up their personal accounts used with Microsoft Authenticator I tried to make the application unmanaged by removing the assignment within Intune and re-install the application using a personal Apple ID

Steps I tried:

- Unassign the app from Intune
- Revoke the license
- Re-install the app from App Store with personal Apple ID

Even though assignment is removed, license is revoked, and I re-installed the app with my own personal Apple ID, the app is still managed, and data is kept. I also let the device sit for 8 hours with no difference. As soon as the app is re-installed, it shows up as a managed app under the device management profile.

The only way I was able to remove all data from Authenticator and move it to unmanaged is by doing a full factory reset of the device.

This led to create a bug report with Microsoft.
