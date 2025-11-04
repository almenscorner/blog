---
title: Working custom actions for non-compliance (sort of)
slug: custom-compliance-actions
date_published: 2020-10-23 16:46:38.000000000 Z
date_updated: 2021-02-22 15:49:33.000000000 Z
tags:
- compliance
- graph api
- mem
- powershell
---
During Ignite a new type of compliance was unveiled for Windows, the ability to run a Powershell script and set the output of that script to a compliance policy. Some OEMs have already created scripts as examples to help you get an idea of what you can do. This is a powerful tool as it lets you set compliance on just about anything you want.

I truly hope this will be added for macOS as well. Soon we&#8217;ll be able to collect custom attributes for macOS and it would be a fantastic addition to also be able to use this for compliance.

But today&#8217;s topic is about the actions we can take today for non-compliant devices. There are a few actions we can take built into the compliance module:

- Send email to user
- Send a push notification
- Lock the device
- Wipe corporate data

And that’s about it. While these are important actions and I know we also tie this together with Conditional Access, App Protection Policies etc. I wish there were a few more actions we can take on device side of things.

In this post I will focus on iOS and Android but it could also apply to Windows and macOS. In other UEMs I&#8217;ve worked with we can take a whole range of actions on non-compliant devices, for example, removing specific profiles or uninstalling applications should the device break compliance. These actions can give your use the extra push to resolve the issue because who wants to be without that important application or the certificate that gives you access?

So how can we do this in MEM?

## **Solution (sort of)**

You guessed it, Powershell and Graph API. I will use a powershell script that checks for non-compliant devices on a compliance policy in MEM. If there&#8217;s a non-compliant device the user will be added to a group that’s excluded from the profile or app.

Oh yeah.. Let&#8217;s talk about the limitations with this solution. I&#8217;m going to use groups to unassign a user from a profile or force uninstalls an application. Why is this a limitation?

1. You are not able to mix user groups and device groups for assignment/exclusion
1. If a user group is assigned a profile or application and you add a device group with the users device to exclusions the user group assignment wins i.e. the profile or app will not be removed.
2. With above limitation if the user has multiple iOS or Android devices enrolled the app or profile will be removed on all devices.

2. Timing. Since I have a Powershell script running to take care of adding/removing the user from a group there&#8217;s a possible delay on 1 hour or more depending on how often the script is executed. So, if the user remediates the issue and reports as compliant, I must wait for the script to execute

With the above limitations in mind, I think it&#8217;s safe to say this might not be a solution ready to be pushed in production, but I wanted to share my experience building this either way.

## **Pre-reqs**

If you&#8217;d like to test this for yourself this is what you need

- Azure AD module
- Intune Graph API module
- Security groups in Azure AD for testing
- Device enrolled with your test account

## **Create test group**

- In the MEM console, go to **Groups -> New Group**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/group1-1024x766.png)
- Give the group a name and click **Create**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/group2.png)
- Search for the group and copy the ID and save it for later

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/group3.png)
## **Add group to exclusion**

Next step is to exclude this group from profiles/applications you want to be uninstalled from the device should it become non-compliant. In the following steps I&#8217;m forcing an uninstall of Outlook on users that are a member of this group.

- In the MEM console, go to **Apps -> Microsoft Outlook -> Properties -> Edit**
- Add your group as **excluded** for **Required** and add your group to** Uninstall** then click **Review + save -> Save**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/assign3-1024x564.png)
## **Create compliance policy**

To test, I will create a compliance policy on which my device will not meet the conditions. I&#8217;m using an iOS 14 device so I will set the rules to require maximum iOS 12 or lower.

- In the MEM console select **Devices -> Compliance Policies -> Create policy > Platform iOS/iPadOS -> Create**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/comp1-1024x536.png)
- Give the policy a name and click on **Next**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/comp2.png)
- Expand **Device Properties** and provide maximum OS, in my case, 12.0. Then click **Next**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/comp3_2-666x1024.png)
- Make sure Mark device noncompliant -> immediately is configured then click **Next**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/comp4-706x1024.png)
- Assign the policy to a group that your test user is a member of, in my case EMS-Users then click **next**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/comp5-633x1024.png)
- Review the configured settings and then click **Create**

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/comp6-671x1024.png)
- Copy the policy ID from your browser and save it for later

![]({{ site.baseurl }}/assets/images/wordpress/2020/10/comp7-1024x104.png)

## Script

Below is the script I used to get compliance and add/remove users from the group. First, since we&#8217;re connecting to Intune and Azure AD, run these commands:

    
    Connect-MSGraph
    

[

## Module

If you don&#8217;t have the Graph module installed, run this command: **Install-Module Microsoft.Graph.Intune**

](

## Module

If you don&#8217;t have the Graph module installed, run this command: **Install-Module Microsoft.Graph.Intune**

)

    
    Connect-AzureAD
    

[

## Module

If you don&#8217;t have the AzureAD module installed, run this command: **Install-Module AzureAD**

](

## Module

If you don&#8217;t have the AzureAD module installed, run this command: **Install-Module AzureAD**

)

Update highlighted lines with the IDs you copied:

    
    #ObjectId of AAD group to target
    $groupObjectId = "YOUR OBJECT ID"
    #Get compliance status for all devices from specified compliance policy
    $complianceStatus = Get-DeviceManagement_DeviceCompliancePolicies_DeviceStatuses -deviceCompliancePolicyId "YOUR COMPLIANCE POLICY ID"
    #Sort out non-compliant devices
    $nonCompliant = $($complianceStatus | where { $_.status -eq "noncompliant" }).UserPrincipalName
    #Sort out compliant devices
    $compliant = $($complianceStatus | where { $_.status -eq "compliant" }).UserPrincipalName
    
    #If compliant devices is found, get group membership for each user, if the user is a member of the exclusion group, remove the user
    if ($compliant) {
    
        foreach ($user in $compliant) {
        
            $memberOf = $(Get-AzureADUserMembership -ObjectId $user).objectid
            $AADUser = Get-AzureADUser -ObjectId $user
            $objectId = $AADUser.ObjectID
    
            if ($memberOf -contains $groupObjectId) {
                Write-Host -ForegroundColor Yellow "Removing $user"
                Remove-AzureADGroupMember -ObjectId $groupObjectId -MemberId $ObjectId
            }
        }
    
    }
    
    #If non-compliant devices is found, get group membership for each user, if the user is a not member of the exclusion group, add the user
    if ($nonCompliant) {
    
        foreach ($user in $nonCompliant) {
    
            $memberOf = $(Get-AzureADUserMembership -ObjectId $user).objectid
            $AADUser = Get-AzureADUser -ObjectId $user
            $objectId = $AADUser.ObjectID
    
            if ($memberOf -notcontains $groupObjectId) {
                Write-Host -ForegroundColor Yellow "Adding $user"
                Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $objectId
            }
    
        }
    
    }
    

## **Verify**

Normally this would be set up to run on a schedule but here I will execute it manually.

Once the script runs successfully you will see that your user is added to the exclusion group.
![]({{ site.baseurl }}/assets/images/wordpress/2020/10/add.png)
When the device syncs with MEM the profile and/or app you excluded will be removed from the deivce.

This was one way I could think of to add some more to compliance. What do you think? Would you like to see this added as actions within MEM? Or do you think this was just a waste of time? 😉
