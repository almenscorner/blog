---
title: How to integrate Jamf with Microsoft Endpoint Manager
slug: how-to-integrate-jamf-with-microsoft-endpoint-manager
date_published: 2020-04-17T22:47:16.000Z
date_updated: 2020-04-20T10:49:26.000Z
tags: Azure AD, Conditional Access, Jamf, macOS, MEM
---

Since the preview release of [shell script support](https://almenscorner.io/2020/04/19/how-to-run-scripts-on-macos-from-microsoft-endpoint-manager/) for macs in Microsoft Endpoint Manager the platform is becoming increasingly more competent to manage more complex macOS scenarios in an enterprise. Microsoft also states that for simpler management MEM is ready for enterprise. If you on the other hand have more complex management scenarios such as:

- Complex application deployment
- More complex script deployment and actions
- Custom PLIST
- IFTT
- Complex printer deploy

You will need to run MEM and Jamf togheter. Let&#8217;s face it, you cannot achive a better Apple management than you can with Jamf who has been leading iOS and macOS management for years.

This is what Microsoft said about the Mac management landscape at Ignite 2019
![](__GHOST_URL__/content/images/wordpress/2020/04/image-311-1024x592-1.png)
The beauty in all this is Microsoft&#8217;s partnership with Jamf which means that we can get the best management available while also being able to use conditional access policies on Azure based on compliance. This is a really powerful tool.

![](__GHOST_URL__/content/images/wordpress/2020/04/jamf-848x1024.png)

**1.** Mac is managed by Jamf Pro

**2.** Mac is registered with Microsoft Intune

**3.** Jamf sends macOS device inventory to Microsoft Intune

**4.** Microsoft Intune evaluates compliance, generates a report, and enforces conditional access via Azure AD

**5.** Allow access from compliant devices

**6.** Block access from noncompliant devices, and provide a user-friendly remediation experience powered by Microsoft Intune and Jamf

---

## Let&#8217;s get started with integration

First, let&#8217;s have a look at some prerequisites.

- Jamf Pro 10.1.0 or later
- [Company Portal app for macOS](https://aka.ms/macoscompanyportal)
- macOS devices with OS X 10.12 Yosemite or later

We&#8217;re going to make the integration in three steps

1. Register a new app in Azure
2. Enable integration in MEM
3. Configure Jamf for integration

### 1. Register app in Azure

- login to the [azure portal](https://portal.azure.com) and then open **Azure Active Directory -> App Registrations**, select **new registration**.

![](__GHOST_URL__/content/images/wordpress/2020/04/appreg1-1024x885.png)

---

- Give the app a name, for example Jamf Pro CA, choose **Accounts in any organizational directory** and then click on **Register**.

![](__GHOST_URL__/content/images/wordpress/2020/04/appreg2-1024x856.png)

On the **overview** page, copy the **Application (client) ID**. We&#8217;ll need this later.

---

- Go to **Certificates & secrets** and click on **New client secret**, give it a **Description**, select any **Expires** option then click on **Add**. Be sure to copy the **Value** from the client secret before you leave the page, we&#8217;ll need this later.

![](__GHOST_URL__/content/images/wordpress/2020/04/appreg3-1024x678.png)
---

- Next, click on **API permissions -> Add a permission** and select **Intune** and then **Application permissions**.

![](__GHOST_URL__/content/images/wordpress/2020/04/appreg4-1024x597.png)
---

- Select **update_device_attributes** and then click on **Add permissions**.

![](__GHOST_URL__/content/images/wordpress/2020/04/appreg5.png)
---

- Click on **Grand admin consent for [your tenant]** and make sure status changes to **Granted for [your tenant]**. After this, the app registration is complete.

![](__GHOST_URL__/content/images/wordpress/2020/04/appreg6-1024x391.png)
---

### 2. Enable integration in Microsoft Endpoint Manager

- Sign in to the [MEM console](https://endpoint.microsoft.com), then click on **Tenant administration -> Connectors and tokens**.

![](__GHOST_URL__/content/images/wordpress/2020/04/mem1.png)
---

- Choose **Partner device management** and activate the integration by pasting the **Application (client) ID** you copied earlier, then click **Save**.

![](__GHOST_URL__/content/images/wordpress/2020/04/mem2-1024x864.png)
---

### 3. Configure Jamf for integration

- Sign in to your Jamf Pro console, then click on **Settings** and under **Global management**, select **Conditional access**.

![](__GHOST_URL__/content/images/wordpress/2020/04/jamf1-1024x574.png)
---

- Check the **Enable Intune Integration for macOS** box. For **Azure AD Tenant Name**, enter your **Azure AD Directory ID** which you can find in **Azure AD -> Properties**. I know it says Tenant name but the ID is what is required, at least in my testing. Then continue to enter your **Application (client) ID** and** Client Secret** that you copied earlier, then click on **Save**.

![](__GHOST_URL__/content/images/wordpress/2020/04/jamf2-1024x726.png)
---

- Go back to the **Partner device management** page in MEM console, the connection is now active. If you want all users to enroll with Jamf by default leave **Assign to** the default **All users** value. Otherwise you can select specific users by adding your group.

![](__GHOST_URL__/content/images/wordpress/2020/04/jamf3.png)
Next you will need to [deploy Company Portal to devices using Jamf](https://docs.microsoft.com/en-us/mem/intune/protect/conditional-access-assign-jamf#deploy-the-company-portal-app-for-macos-in-jamf-pro) and setup [compliance policies for macs managed by Jamf](https://docs.microsoft.com/en-us/mem/intune/protect/conditional-access-assign-jamf). 

Devices that are managed by Jamf will now start showing up in MEM as compliant devices and you can start using Conditional Access on them. Pretty cool if you ask me 😊.
![](__GHOST_URL__/content/images/wordpress/2020/04/45121078_332697134163303_2872554009661538304_o-1024x371.jpg)
