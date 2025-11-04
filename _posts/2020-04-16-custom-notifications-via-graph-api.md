---
title: Custom notifications via Powershell and Graph API
slug: custom-notifications-via-graph-api
date_published: 2020-04-16 10:08:00.000000000 Z
date_updated: 2020-04-16 10:08:19.000000000 Z
tags:
- compliance
- graph api
- intune
- mem
- powershell
- security
---
I&#8217;ve Been playing around a bit with sending custom notifications via Graph API. In this post I would like to share what I have set up so far, and how you can use this in your own environment.

Sending notfications from the console is a great way of notifying end users if any changes is coming up or if they need to take action on something. The feature lit up in MEM late july 2019 and I have used it sporadically. However, there are functions I miss with this feature. One example is the ability to send a notification to a device if it violates a compliance policy. I think this is a great use case and it&#8217;s what we&#8217;re going to spend some time on. While I know the function is in development to send notifications with compliance policy actions I cannot wait.

I wanted a way to notify users on their devices automatically by not using email. Personally, I think that a notification on the device is more visible. And that, there&#8217;s a greater chance the user will take notice of this. Reason being we can continually push until they are compliant.

## The manual method

First off, let&#8217;s see how to send a custom notification using the console.

- Click on **Tenant administration** and then on **Custom notifications**

![Custom notifications 1]({{ site.baseurl }}/assets/images/wordpress/2020/04/customnot.png)
- Enter a **Title** and **Body**, then click **Next**

![Custom notifications 2]({{ site.baseurl }}/assets/images/wordpress/2020/04/custnot2.png)
- Click **Select groups to include **and find the group you wish to notify. Keep in mind that if the group contains both users and devices only users will be targeted

![Custom notifications 3]({{ site.baseurl }}/assets/images/wordpress/2020/04/custnot3.png)
- Verify the information and click on **Create**

![Custom notifications 4]({{ site.baseurl }}/assets/images/wordpress/2020/04/custnot4.png)
The notification is now being sent to all users targeted in the group you specified. While this is great for information you want to send to users, it becomes more useful when we can automate the process. 

Above I showed how to send to a group. In addition, you can also target a specific device from the device details page (Android Enterprise DO devices not supported).

## The automated method

Let&#8217;s talk about the fun stuff. What we&#8217;re going to do is check device statuses on a compliance policy and if the device is non compliant send a notification specifying which policy it violates.

This is available via beta Graph API today and we&#8217;re going to use Powershell to do it.

Prerequisites:

- You will need access to the Azure AD app **Microsoft Intune PowerShell**
- Be a MEM administator
- Intune Graph Powershell module installed (`Install-Module Microsoft.Graph`)

If you&#8217;re not familliar with administrating MEM via Powershell, the first command you&#8217;ll need to run is:

    
    Connect-MSGraph
    

In addition, since we need to use the Beta API&#8217;s we also have to update our environment.

    
    Update-MSGraphEnvironment -SchemaVersion 'beta'
    #Run connect-msgraph again to use the new environment
    Connect-MSGraph
    

To get the id of the compliance policy you want compliance status from, run the following command and copy the id from the policy you want.

    
    Get-DeviceManagement_DeviceCompliancePolicies -Select displayName, id
    

Now that we are connected to Graph and have the policy id, we&#8217;re ready to get device statuses from a compliance policy and start sending notifications to non compliant devices. Below is the script I used to do this.

    
    #Get non compliant devices from a specific compliance policy
    $Devices = Get-DeviceManagement_DeviceCompliancePolicies_DeviceStatuses -deviceCompliancePolicyId [ID you copied earlier]
    $nonCompliant = $Devices | where {$_.status -eq "noncompliant"}
    
    #Send notification to each non compliant device
    foreach ($device in $nonCompliant){
        
        #Get MEM device ID
        $id = ($device.id).ToString().split("_")
        $id = $id.GetValue(2)
    
        #Set body/message and build Json
        $notificationBody = @{
        notificationTitle = "Device compliance violation"
        notificationBody = "Your device violates the password compliance. Please open settings and set a passcode"
        }
    
        $body = $notificationBody | ConvertTo-Json
    
        $body = @"
    
        $body
    
    "@
    
        #Send notification to the device via Graph API and force a check in
        Invoke-MSGraphRequest -HttpMethod POST -Url "/deviceManagement/managedDevices/$id/sendCustomNotificationToCompanyPortal" -Content $body
    
        Invoke-MSGraphRequest -HttpMethod POST -Url "deviceManagement/managedDevices/$id/syncDevice"
    
    }
    

This is what the user will see on their device if it violates the compliance policy:
![Notification on iOS device]({{ site.baseurl }}/assets/images/wordpress/2020/04/devicenot-473x1024.png)Company portal notification on iOS
And that&#8217;s it. What I like about this is that it could be run from an Azure Runbook or anywhere else where you can execute Powershell scripts. In conclusion, this can help you automate notifications to non compliant devices or any other use case you might have.
