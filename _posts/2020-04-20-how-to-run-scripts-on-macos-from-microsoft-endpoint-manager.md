---
title: How to run scripts on macOS from Microsoft Endpoint Manager
slug: how-to-run-scripts-on-macos-from-microsoft-endpoint-manager
date_published: 2020-04-20 10:00:00.000000000 Z
date_updated: 2020-04-20 10:55:02.000000000 Z
tags:
- macos
- mem
- microsoft intune mdm agent
- script
---
Queue The Pointer Sisters &#8211; I&#8217;m So Excited, this is a good one.

Microsoft Endpoint Manager finally have support for running scripts on macOS devices. This is a feature that other platforms such as VMware Workspace ONE and Jamf has had for years, and one off the reasons why I&#8217;ve been hesitant to managing my customers macOS fleet in MEM. But now it&#8217;s here and I couldn&#8217;t be happier.

## Why have I missed this so much?

Because this opens up a whole world of possibilities for you and gives you the power to extend your management beyond what is possible using MDM API&#8217;s or the application deployment model we have in MEM. One use case for me is to curl and install auditing agents which cannot be repackaged.

At some point in the future there&#8217;s two features I hope Microsoft adds to this. 

1. The option to send custom attributes back to MEM based on script feedback. This is something you can do with Workspace ONE/Jamf and is really useful.  I have used this in Workspace ONE numerous times as a condition when deploying applications and checking if a service is running or not for compliance.
2. CLI to Intune MDM Agent to be able to send custom notifications to the user when running scripts.

It&#8217;s still a preview and new functions are being added almost every week. So if you plan on going all in and deploying scripts to your devices, think twice and test thoroughly. Below are some of the current known issues. I will continue to update this post with new features and known issues.

## Architectural overview
![]({{ site.baseurl }}/assets/images/wordpress/2020/04/Screenshot-2020-04-18-at-23.07.47-1024x450.png)

Each script you assign to devices is run as a separate process and the run status is reported back to MEM for admins to monitor success of execution or error codes. When you configure the scripts to be run on an interval, the scripts are stored locally to save download time.

## Known issues

- **User group assignment:** Shell scripts assigned to user groups doesn&#8217;t apply to devices. User group assignment is currently not supported in preview. Use device group assignment to assign scripts.
- **Collect logs:** &#8220;Collect logs&#8221; action is visible. But when log collection is attempted, it shows &#8220;an error occurred&#8221; and doesn&#8217;t capture logs. Log collection is currently not supported in preview.
- **No script run status:** In the unlikely event that a script is received on the device and the device goes offline before the run status is reported, the device will not report run status for the script in the admin console.
- **User status report:** An empty report issue exists. In the [Microsoft Endpoint Manager Admin Center](https://go.microsoft.com/fwlink/?linkid=2109431), select **Monitor**. The user status shows an empty report.

But never mind these issues and the fact it&#8217;s a preview, let&#8217;s get started.

## Deploying shell scripts with Microsoft Endpoint Manager

We&#8217;re going to deploy two very simple scripts for the point of demonstration.

- Runs as root and creates a .txt file on the current users desktop and writes Hello World in it.

    
    #!/bin/sh
    #touchtestv3.sh
    loggedInUser=$(ls -l /dev/console | awk '{ print $3 }')
    touch /Users/$loggedInUser/Desktop/touchtest.txt
    echo "Hello World" > /Users/$loggedInUser/Desktop/touchtest.txt
    
    

- Runs as the current user and starts the TextEdit application.

    
    #!/bin/sh
    #start.sh
    Open -a textEdit
    

### Prerequisites

- Devices are running macOS 10.12 or later.
- Devices are managed by Intune.
- Shell scripts begin with `#!` and must be in a valid location such as `#!/bin/sh` or `#!/usr/bin/env zsh`.
- Command-line interpreters for the applicable shells are installed.

### Upload and assign

- Go to the [MEM console](https://endpoint.microsoft.com) and sign in. Then, Go to **Devices -> Scripts -> Add** and choose **macOS**.

![]({{ site.baseurl }}/assets/images/wordpress/2020/04/Screenshot-2020-04-18-at-13.20.45-1024x963.png)
- Give the script a **name** and a **description** and click **Next**. On the **Script settings** page you have a number of settings, let&#8217;s break them down.
- **Upload script**
- Any script that begins with #! and point to a valid location (such as #!/bin/sh or #!/usr/bin/env zsh)

- **Run script as signed-in user**
- If you choose **Yes**, the script will run as the currently signed-in user. Default is **No** which means that the script runs as root to be able to make changes that a standard user can&#8217;t.

- **Hide script notifications on devices**
- Not configured
- A notification will be shown on the device when the script runs

- Yes
- The user will not be notified when a script runs

- **Script frequency**
- Specifies if the script will be run on intervals. The options are
- Not configured, default value (run once)
- Every 15/30 min, every 1/2/3/6/12 hour
- Every 1 day or 1 week

- **Max number of times to retry if the script fails**
- Specify how many times to re-run the script if it fails

Choose the settings that apply to your use case then click **Next** and assign your script to a Azure AD Device group. You can use a dynamic group that points to a specific ABM profile with the property enrollmentProfileName, or, if you want to assign to all macs you can use the property deviceOSType with the value OS X.

Below is the settings I used for my scripts:

- **Touchtestv3**

![]({{ site.baseurl }}/assets/images/wordpress/2020/04/ttest.png)
- **Start.sh**

![]({{ site.baseurl }}/assets/images/wordpress/2020/04/stest.png)
### The Intune MDM Agent

#### Install

To run the script on devices an agent called Intune MDM Agent is installed silently on devices targeted by a script in MEM. The agent is installed to /Library/Intune/Microsoft Intune Agent.app and is not visible in /Applications

#### Use

- The agent silently authenticates with Intune services before checking in to receive assigned shell scripts for the macOS device.
- The agent receives assigned shell scripts and runs the scripts based on the configured schedule, retry attempts, notification settings, and other settings set by the admin.
- The agent checks for new or updated scripts with Intune services usually every 8 hours. This check-in process is independent of the MDM check-in.

**Can I force a check in?**
Yes. There are two ways you can force a check in on the device.
1. Open **terminal **and run: `sudo killall IntuneMdmAgent` this immediatly restarts the agent and forces a check in.
2. Open **Activity Monitor**, at the top **choose view -> All Proccesess**, search for Intune. You will now se two Processes, one run as the user and one as root. Quit the process running as **root**.

#### Uninstall

The agent is uninstalled when one of these conditions is true:

- Shell scripts are no longer assigned to the device.
- The macOS device is no longer managed.
- The agent is in an irrecoverable state for more than 24 hours (device-awake time).

### End user experience

So we added two scripts, one that runs as root and adds a textfile to the users desktop called touchtest.txt, this also displays a notification to the user that something is happening. This is the notification the user will see:
![]({{ site.baseurl }}/assets/images/wordpress/2020/04/Screenshot-2020-04-18-at-14.14.30.png)
After the scripts execution we now have a touchiest.txt on the desktop and TextEdit app is open:
![]({{ site.baseurl }}/assets/images/wordpress/2020/04/Screenshot-2020-04-18-at-14.30.53-1024x640.png)
Both scripts runs with an interval of 15 minutes. A tad unesseary for these scripts but I wanted them to run as often as possible for demonstration.

That&#8217;s it folks. I hope this guide was useful for you. I absolutley love this addition to MEM and will continue to test with more complex scenarios. Right now I think it works really well and are looking forward to future enhancements.
