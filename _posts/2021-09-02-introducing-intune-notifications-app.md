---
title: 'Introducing: Intune Notifications app'
slug: introducing-intune-notifications-app
date_published: 2021-09-02 14:43:15.000000000 Z
date_updated: 2023-06-10 07:25:24.000000000 Z
tags:
- power apps
- power automate
- tools
excerpt: Throughout the years of managing devices in MDM it has been interesting to
  keep track of all tokens and certificates we use to connect to Apple Business Manager
  (ABM), Apple Push Notification Server (APNs), Managed Google Play etc.
---
Throughout the years of managing devices in MDM it has been interesting to keep track of all tokens and certificates we use to connect to Apple Business Manager (ABM), Apple Push Notification Server (APNs), Managed Google Play etc. While it's not my specialty, I really like what you can achieve when using Power Automate together with Power Apps and Microsoft Graph. You can very easily build any application to solve a pain point in your company or for you as an administrator with no to very little code. That's why I looked at using these tools to make my life easier as an administrator of mobile devices in Microsoft Endpoint Manager.

Enter, Intune Notifications. The goal with this application is to notify you and your team and give you a visual indication if any tokens/certificates is about to expire or if a sync fails to connected services. For me this is a really handy tool when on the go, there's been multiple times when a sync has failed to ABM/VPP and I only knew once I signed in to the MEM console to check.

**Note: **Use this blog post for first time setup, I will publish new versions and updates on [GitHub](https://github.com/almenscorner/IntuneNotifications)

## App overview

### Home screen

The home screen is where you see all notifications, at this time it monitors APNs certificate expiry, ABM token expiry, VPP token expiry, ABM sync failures and VPP sync failures. If a token/certificate is about to expire in 30 days, the notification will be shown here as a warning, indicated by the orange circle. If a sync fails to any of the services, a notification with a red circle is shown to indicate an error. At the bottom of the screen is an overview of the services sync status. If any token fails a sync the green check mark will be switched to the orange warning sign.

### ABM details screen

Shows details about all connected ABM tokens, here you can also see which one has failed the sync.

### VPP details screen

Shows details about all connected VPP tokens, here you can also see which one has failed the sync.

### Managed Google Play details screen

Shows details about Managed Google Play, here you can also see if it has failed to sync.

![]({{ site.baseurl }}/assets/images/2021/09/HomeScreen-1.JPEG)

![]({{ site.baseurl }}/assets/images/2021/09/ABMScreen.JPEG)

![]({{ site.baseurl }}/assets/images/2021/09/VPPScreen.PNG)

![]({{ site.baseurl }}/assets/images/2021/09/GPlayScreen.PNG)

## Flow overview

All information displayed in the application is collected with Power Automate flows. The flow which displays all notifications is called "Stream" and is setup like below. At a basic level it gets the status of ABM,VPP and APNs with HTTP GET from Microsoft Graph, if there's an issue or if something is about to expire a message is added to an array. At the end, all arrays from the different services is joined and a Response is created which is then shown in the application.
![]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-13.29.59.png)
To get the details and status of all services, a flow per service is run that collects all the information.
![]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-10.26.25.png)
To send a push notification if any issues are detected, a recurring flow is setup
![]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-13.31.34.png)
**How can I use this? **

To make it easy, I have exported and uploaded this application to my GitHub [here](https://github.com/almenscorner/IntuneNotifications), the Power App is under releases and the push notification flow is uploaded to the repository. Download both. I will contiune to work on the application and add new features as needed and this makes it easy for you to get going and update the app once an update is available. You can modify the application or flows as you wish, if you do, create your own updates for the app.

**!!!NOTE!!! **In these flows the secret is stored in a variable which causes it to be visible to anyone who uses the flow and also in the flow outputs. I **strongly** recommend that you instead store the secret in Azure Key Vault, delete the "Secret" variable and replace it with a action to retrieve it from a Key Vault. I just did not have access to one when I set this up 😒

## Setup and use

Let's look at what you need to do to start using the application.

**Pre-requisites**

Of course there are some pre-requisites you need to take into consideration.

- You need to able to create an app registration in Azure AD and grant admin  consent
- The user of the application must have a Flow license since we're using premium features (you can start a 90-day trial)

### Create an Azure AD app registration

First we're going to create an app in Azure AD that will be used by the flows to authenticate to Microsoft Graph.

- Sign into [portal.azure.com](https://portal.azure.com) and open **Azure Active Directory - App Registrations** then click **New registration**
- Give it a **Name**, for example Intune Notifications and click **Register**
- Copy the **Application (Client) ID** and **Directory (Tenant) ID**, we'll need those later
![Screenshot-2021-09-02-at-11.49.48]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-11.49.48.png)
- Open **API Permissions**, click **Add a permission**, choose **Microsoft Graph - Application permissions** and add the following permissions

- DeviceManagementConfiguration.Read.All
- DeviceManagementServiceConfig.Read.All

- Click **Grant admin consent**
![Screenshot-2021-09-02-at-11.45.08]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-11.45.08.png)
- Open **Certificates & secrets** and under **Client secrets**, click **New client secret**, provide a description then click **Add**. Copy the **Value**, we'll need this later
![Screenshot-2021-09-02-at-11.48.05]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-11.48.05.png)

### Import application

When the above steps has been complteted we're ready to import the application to  Power Apps.

- Sign in to [make.powerapps.com](https://make.powerapps.com)
- Go to **Apps** and click **Import canvas app**
![Screenshot-2021-09-02-at-14.12.28]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-14.12.28.png)
- Click **Upload** and choose the ZIP you downloaded from my GitHub
- Verify that the application and all flows have green check marks and that the import setup is specified as "Create as new", then click **Import**
![Screenshot-2021-09-02-at-14.19.27]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-14.19.27.png)
- Sign in to [flow.microsoft.com](https://flow.microsoft.com)
- Open the newly created "Stream" flow and edit the following variables to inlcude your Secret, Tenant ID and Client ID
![Screenshot-2021-09-02-at-14.30.03]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-14.30.03.png)
- Open and edit the HTTP action for GetGPlayStatus, GetDEPStatus and GetVPPStatus to include your Secret, Tenant ID and Client ID
![Screenshot-2021-09-02-at-14.33.42]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-14.33.42.png)
- When the above edits has been made, turn on each flow by clicking the three dots and then click **Turn on**
![Screenshot-2021-09-02-at-14.36.32]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-14.36.32.png)

### Import push notification flow

Since the recurring flow for push notifications is not part of the Power App we have to import that separetly.

- Sign in to [flow.microsoft.com](https://flow.microsoft.com)
- Go to **My flows** and click **Import**
- Click **Upload** and choose the ZIP downloaded from my GitHub
- Create the connection with Power Apps by clicking the configure button
![Screenshot-2021-09-02-at-15.11.21]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-15.11.21.png)
- Click **Create New**
![Screenshot-2021-09-02-at-15.15.07]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-15.15.07.png)
- Click **Create a connection**, choose **Power Apps Push Notification V2** and click **Create**
![Screenshot-2021-09-02-at-15.21.27]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-15.21.27.png)
- Navigate back to the import tab, now you will have a connection, click **Power Apps Push Notification V2** then click **Save**
![Screenshot-2021-09-02-at-15.23.15]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-15.23.15.png)
- Click **Import**
- Open the flow GetStatusRecurring and edit the following variables to inlcude your Secret, Tenant ID and Client ID. In this flow there's an additional variable to specify users who should receive the push notifications. Enter the users in the array you wish to notify
![Screenshot-2021-09-02-at-15.00.59]({{ site.baseurl }}/assets/images/2021/09/Screenshot-2021-09-02-at-15.00.59.png)
- Save the flow

That's all you need to do to use the application. The recurring flow is run once a day which you can change by editing the schedule in the flow. The default time frame to check if a token or certificate expires is 30 days. If you would like to change this to a lower or higher value, just edit the variable **Time To Expiration **in the "Stream" and "GetStatusRecurring" flows.

When opening the application you will be presented with the current status of your environment.
