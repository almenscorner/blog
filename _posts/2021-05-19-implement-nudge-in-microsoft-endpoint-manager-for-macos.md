---
title: Nudge your users to update macOS in Microsoft Endpoint Manager
slug: implement-nudge-in-microsoft-endpoint-manager-for-macos
date_published: 2021-05-19 14:02:50.000000000 Z
date_updated: 2021-05-19 14:02:50.000000000 Z
tags:
- macos
- mem
- nudge
- open source
- security
- update
excerpt: This last few weeks has reminded us how important it is to keep you macOS
  devices up to date. Recently Apple released patches for vulnerability CVE-2021–30657
  where attackers could bypass Gatekeeper, a mechanism to enforce code signing and
  verifying of an app before it's allowed to run
---
This last few weeks has reminded us how important it is to keep you macOS devices up to date. Recently Apple released patches for vulnerability [CVE-2021–30657](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-30657) where attackers could bypass Gatekeeper, a mechanism to enforce code signing and verifying of an app before it's allowed to run. This is a vulnerability which is known to have been exploited by a malware called Shlayer, which could run an app without user intervention and access sensitive data without the user knowing anything. It was reported to Apple on March 25 and patched on March 30. There are new vulnerabilities being reported continuously which is why you need to keep on your toes and make sure users updates their devices. 

Update management is somewhat a complex task to handle on macOS, often different methods works for different versions of the OS. Let's have a look at some methods available to handle updates on macOS:

- MDM Profile

- Enables automatic updates via the native controls in macOS. Downside is that we can only use Apples built in notifications which are not that intrusive and a user can ignore them infinetly

- MDM Command

- Push a command to devices forcing them to update. Basically, you have no control, you push it and hope for the best

- softwareupdate binary

- Allows you to install updates using a scripted method. In recent years, this has shown to be a buggy method of updating macs

In Microsoft Endpoint Manager we do not have any good ways to control or enforce updates on macOS, all we can really do is defer updates or deploy a script. This is where I think Nudge comes into play, not just for MEM, but for all MDMs that can take advantage of the above methods as well.

## What is Nudge?

Written by Erik Gomez in Swift/SwiftUI 5.2, Nudge is essentially a UI tool that prompts your users to install updates. What is great about Nudge is that is uses tried and tested methods of upgrading macOS. Rather than using `softwareupdate` to install updates, it uses system preferences and to install major os upgrades like Big Sur it uses the standard method. In other words, upgrades/updates are installed the way you would install them as a consumer. Major upgrades has not yet been tested as no major version has been released since Big Sur. Although the standard methods are used to install the updates, `softwareupdate` is used to download updates in the background.

Nudge is also highly customizable so that you can get you organisations look and feel.

Any MDM solution that supports the installation of .pkgs and .mobileconfig files is supported.

Nudge consists of three parts:

- Nudge.app installed to `/Applications/Utilities/Nudge.app`
- a LaunchAgent installed to `/Library/LaunchAgents`
- a Preference file, either in JSON or mobileconfig format

Using the standard LaunchAgent that comes with the tool, Nudge will open every 30 minutes, at the 0 and 30 minute mark. If you would like to change this behaviour you have to create your own LaunchAgent.

### Supported OS versions

The following operating system and versions have been tested.

- 11.0, 11.0.1, 11.1, 11.2, 11.2.1

If you need to enforce updates for earlier versions than 11.0 of macOS, you should look at using [nudge-python](https://github.com/macadmins/nudge-python).

## Deploying Nudge in Microsoft Endpoint Manager

The first thing you need to do is download the latest version of Nudge [here](https://github.com/macadmins/nudge/releases). This app is safe to deploy as it will do absolutely nothing without a configuration and LaunchAgent configured to open it. Download the Nudge-1.x.x.x.x.x.pkg.

### Install Nudge

Before we can deploy Nudge with Microsoft Endpoint Manager, it has to be wrapped to an .intunemac file. To do so, follow the steps below (has to be done on a mac).

- 
Download the macOS wrapper tool [here](https://github.com/msintuneappsdk/intune-app-wrapping-tool-mac)

- 
Unzip the archive, open terminal and change directory to the unziped folder

- 
Execute the following command:
`sudo chmod +x IntuneAppUtil`

- 
Copy the Nudge-1.x.x.x.x.x.pkg into the intune app wrapping tool folder and then run this command:
`./IntuneAppUtil -c ./Nudge-{VERSION}.pkg -o ./ `

![Screenshot-2021-05-11-at-14.23.55]({{ site.baseurl }}/assets/images/2021/05/Screenshot-2021-05-11-at-14.23.55.png)

Now that we have our .intunemac bundle, we're ready to upload and deploy in MEM.

- Open the [MEM console](https://endpoint.microsoft.com)
- Go to **Apps -> macOS -> Add**, choose **Line-of-business app** and click **Select**
- Click on **Select app package file** and upload the .intunemac file created in the previous step
- Enter a **Publisher**, set **Minimum operating system** to **macOS Big Sur 11.0** and click **Next**
- Assign the app to a device you'd like to test on and click **Next**
- Click **Create**

The app will now start to deploy to your device
![]({{ site.baseurl }}/assets/images/2021/05/Screenshot-2021-05-11-at-14.31.39.png)
### Configure

Nudge can be configured using two methods, JSON or a .mobileconfig, In this post I'm going to focus on using a .mobileconfig which we then will push via MEM as a custom profile. If you go for the JSON option, it can either be a file local to the mac where Nudge is running or a web server which you point to in the configuration.

First download this [example profile](https://raw.githubusercontent.com/macadmins/nudge/main/Example%20Assets/com.github.macadmins.Nudge.mobileconfig), right click anywhere and choose Save as. This example profile contains all default values for Nudge, do not deploy this example, we will make some changes.

***Only settings which are different from the defaults need to be changed. For instance, if you want to change the text on the quit button, you would change the `primaryQuitButtonText` key. If you are happy with the default text of "Later", you do not need to set the key***

Delete everything from the profile you downloaded except for the following:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
      <dict>
        <key>PayloadContent</key>
        <array>
          <dict>
            <key>PayloadDescription</key>
            <string>Configures all Nudge preferences</string>
            <key>PayloadDisplayName</key>
            <string>Nudge Preferences</string>
            <key>PayloadIdentifier</key>
            <string>com.github.macadmins.Nudge.preferences.example</string>
            <key>PayloadOrganization</key>
            <string></string>
            <key>PayloadType</key>
            <string>com.github.macadmins.Nudge</string>
            <key>PayloadUUID</key>
            <string>CA02957C-7472-446B-9F77-3E0414405556</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
            <key>osVersionRequirements</key>
            <array>
              <dict>
                <key>requiredInstallationDate</key>
                <date>2021-02-28T00:00:00Z</date>
                <key>requiredMinimumOSVersion</key>
                <string>11.2.1</string>
                <key>targetedOSVersions</key>
                <array>
                  <string>11.0</string>
                  <string>11.0.1</string>
                  <string>11.1</string>
                  <string>11.2</string>
                </array>
              </dict>
            </array>
          </dict>
        </array>
        <key>PayloadDescription</key>
        <string>Configures Nudge application</string>
        <key>PayloadDisplayName</key>
        <string>Nudge</string>
        <key>PayloadIdentifier</key>
        <string>com.github.macadmins.Nudge.example</string>
        <key>PayloadOrganization</key>
        <string>Nudge</string>
        <key>PayloadScope</key>
        <string>System</string>
        <key>PayloadType</key>
        <string>Configuration</string>
        <key>PayloadUUID</key>
        <string>2F54F734-132D-4539-B583-F1DCF23DB5EB</string>
        <key>PayloadVersion</key>
        <integer>1</integer>
      </dict>
    </plist>
    

The focus here is the `osVersionRequirements` key:

            <key>osVersionRequirements</key>
            <array>
              <dict>
                <key>requiredInstallationDate</key>
                <date>2021-05-31T00:00:00Z</date>
                <key>requiredMinimumOSVersion</key>
                <string>11.3.1</string>
                <key>targetedOSVersions</key>
                <array>
                  <string>11.0</string>
                  <string>11.0.1</string>
                  <string>11.1</string>
                  <string>11.2</string>
                </array>
              </dict>
            </array>
    
    

Primary keys for Nudge to work:

- `requiredInstallationDate` is the required installation date for Nudge to enforce the required operating system version. If a user has not installed the update after this date, you can nudge (annoy) the user more agressivly.
- `requiredMinimumOSVersion` is the required minimum operating system version. That is, Nudge will ask people to update until they are at this version or later. If the current version of macOS is higher than or equal to the `requiredMinimumOSVersion`, Nudge will not open.
- `targetedOSVersions` is a list of macOS versions which determines which versions of macOS that Nudge will launch on. If the current version of macOS on the machine launching Nudge is not in this list, Nudge will not open.

For testing purposes when you want to get a prompt, you need to make sure that the macOS version you are running Nudge on is present in `targetedOSVersions` list and lower than the value in `requiredMinimumOSVersion`. Change the `osVersionRequirements` so that Nudge will prompt you, make sure the version you are currently running is in the `targetedOSVersions` list.

**Do not, EVER, deploy this in production, this configuration is for testing ONLY**

            <key>osVersionRequirements</key>
            <array>
              <dict>
                <key>requiredInstallationDate</key>
                <date>2021-05-31T00:00:00Z</date>
                <key>requiredMinimumOSVersion</key>
                <string>11.99.1</string>
                <key>targetedOSVersions</key>
                <array>
                  <string>11.0</string>
                  <string>11.0.1</string>
                  <string>11.1</string>
                  <string>11.2</string>
                  <string>11.2.1</string>
                  <string>11.2.2</string>
                  <string>11.2.3</string>
                  <string>11.3</string>
                  <string>11.3.1</string>
                </array>
              </dict>
            </array>
    
    

Here is my test profile:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
      <dict>
        <key>PayloadContent</key>
        <array>
          <dict>
            <key>PayloadDescription</key>
            <string>Configures all Nudge preferences</string>
            <key>PayloadDisplayName</key>
            <string>Nudge Preferences</string>
            <key>PayloadIdentifier</key>
            <string>com.github.macadmins.Nudge.preferences.example</string>
            <key>PayloadOrganization</key>
            <string></string>
            <key>PayloadType</key>
            <string>com.github.macadmins.Nudge</string>
            <key>PayloadUUID</key>
            <string>CA02957C-7472-446B-9F77-3E0414405556</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
            <key>osVersionRequirements</key>
            <array>
              <dict>
                <key>requiredInstallationDate</key>
                <date>2021-05-31T00:00:00Z</date>
                <key>requiredMinimumOSVersion</key>
                <string>11.99.1</string>
                <key>targetedOSVersions</key>
                <array>
                  <string>11.0</string>
                  <string>11.0.1</string>
                  <string>11.1</string>
                  <string>11.2</string>
                  <string>11.2.1</string>
                  <string>11.2.2</string>
                  <string>11.2.3</string>
                  <string>11.3</string>
                  <string>11.3.1</string>
                </array>
              </dict>
            </array>
          </dict>
        </array>
        <key>PayloadDescription</key>
        <string>Configures Nudge application</string>
        <key>PayloadDisplayName</key>
        <string>Nudge</string>
        <key>PayloadIdentifier</key>
        <string>com.github.macadmins.Nudge.example</string>
        <key>PayloadOrganization</key>
        <string>Nudge</string>
        <key>PayloadScope</key>
        <string>System</string>
        <key>PayloadType</key>
        <string>Configuration</string>
        <key>PayloadUUID</key>
        <string>2F54F734-132D-4539-B583-F1DCF23DB5EB</string>
        <key>PayloadVersion</key>
        <integer>1</integer>
      </dict>
    </plist>
    

Once you've made the changes, save the .mobileconfig file. Next, we'll create a custom profile in MEM.

- Open the [MEM console](https://endpoint.microsoft.com)
- Go to **Devices -> Configuration profile** and click **Create**, choose **macOS** for **Platform** and **Profile type** as **Template** then, choose **Custom** and click **Create**
- Give your profile a **Name** and click **Next**
- Upload your edited .mobileconfig, set the **Configuration profile name** and click **Next**
- Assign the profile to your test device and click **Next**
- Click **Create**
![Screenshot-2021-05-11-at-15.14.32-1]({{ site.baseurl }}/assets/images/2021/05/Screenshot-2021-05-11-at-15.14.32-1.png)

Once the app and profile are in place, we're ready to test it on the device. This will be a manual test to make sure everything is working before we push the LaunchAgent.

To test, open the Nudge.app from the /Applications/Utilities folder, this is what you should see:
![]({{ site.baseurl }}/assets/images/2021/05/Screenshot-2021-05-18-at-15.06.46.png)
### LaunchAgent

For the Nudge app to prompt users automatically on devices, a LaunchAgent needs to be pushed. This is an agent that starts Nudge on a given interval. As stated earlier, per default this happens every 30 minutes, at the 0 and 30 minute mark. We will have a look at deploying the LaunchAgent as well as changing this interval.

Since the LaunchAgent is a .plist file that's placed in /Library/LaunchAgents, MEM won't be able to detect the install of the default provided PKG as successful when deployed as an .intunemac file. To work around this, we need to deploy the LaunchAgent using a script. I'm going to use a GitHub repository where I have created some example LaunchAgents. The script will curl the file from the repository and put it under /Library/LaunchAgents. You can use whatever you want to curl the file from, for example, your own web server, Amazon s3 bucket, Azure Storage Account etc.

The script I will be using to curl the file and load the agent:

    #!/bin/zsh
    
    # URL to raw file on GitHub
    baseURL="https://raw.githubusercontent.com/almenscorner/macOS/main/LaunchAgents/Nudge"
    # Name of plist in the repository
    fileName="Default.plist"
    # If you change your agent file name, update the following line
    launch_agent_plist_name='com.github.macadmins.Nudge.plist'
    # Base paths
    launch_agent_base_path='Library/LaunchAgents/'
    
    curl -LJ ${baseURL}/${fileName} -o "$3/${launch_agent_base_path}${launch_agent_plist_name}"
    
    # Copyright 2021-Present Erik Gomez.
    #
    # Licensed under the Apache License, Version 2.0 (the 'License');
    # you may not use this file except in compliance with the License.
    # You may obtain a copy of the License at
    #
    #      https://www.apache.org/licenses/LICENSE-2.0
    #
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an 'AS IS' BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    # See the License for the specific language governing permissions and
    # limitations under the License.
    
    # Fail the install if the admin forgets to change their paths and they don't exist.
    if [ ! -e "$3/${launch_agent_base_path}${launch_agent_plist_name}" ]; then
      echo "LaunchAgent missing, exiting"
      exit 1
    fi
    
      # Current console user information
      console_user=$(/usr/bin/stat -f "%Su" /dev/console)
      console_user_uid=$(/usr/bin/id -u "$console_user")
    
      # Only enable the LaunchAgent if there is a user logged in, otherwise rely on built in LaunchAgent behavior
      if [[ -z "$console_user" ]]; then
        echo "Did not detect user"
      elif [[ "$console_user" == "loginwindow" ]]; then
        echo "Detected Loginwindow Environment"
      elif [[ "$console_user" == "_mbsetupuser" ]]; then
        echo "Detect SetupAssistant Environment"
      elif [[ "$console_user" == "root" ]]; then
        echo "Detect root as currently logged-in user"
      else
        # Unload the agent so it can be triggered on re-install
        /bin/launchctl asuser "${console_user_uid}" /bin/launchctl unload -w "$3/${launch_agent_base_path}${launch_agent_plist_name}"
        # Kill Nudge just in case (say someone manually opens it and not launched via launchagent
        /usr/bin/killall Nudge
        # Load the launch agent
        /bin/launchctl asuser "${console_user_uid}" /bin/launchctl load -w "$3/${launch_agent_base_path}${launch_agent_plist_name}"
      fi
    

Default LaunchAgent that will be downloaded:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
    	<key>Label</key>
    	<string>com.github.macadmins.Nudge</string>
    	<key>LimitLoadToSessionType</key>
    	<array>
    		<string>Aqua</string>
    	</array>
    	<key>ProgramArguments</key>
    	<array>
    		<string>/Applications/Utilities/Nudge.app/Contents/MacOS/Nudge</string>
    		<!-- <string>-json-url</string> -->
    		<!-- <string>https://raw.githubusercontent.com/macadmins/nudge/main/Nudge/example.json</string> -->
    		<!-- <string>-demo-mode</string> -->
    	</array>
    	<key>RunAtLoad</key>
    	<true/>
    	<key>StartCalendarInterval</key>
    	<array>
    		<dict>
    			<key>Minute</key>
    			<integer>0</integer>
    		</dict>
    		<dict>
    			<key>Minute</key>
    			<integer>30</integer>
    		</dict>
    	</array>
    </dict>
    </plist>
    

As you can see from the example above, the `StartCalendarInterval` is where you set the start interval for Nudge. So what are we able to configure here?

`Minute <integer>`

The minute on which this job will be run.

`Hour <integer>`

The hour on which this job will be run.

`Day <integer>`

The day on which this job will be run.

`Weekday <integer>`

The weekday on which this job will be run (0 and 7 are Sunday).

`Month <integer>`

The month on which this job will be run.

An example of the `StartCalendarInterval`

    <key>StartCalendarInterval</key>
    <array>
        <dict>
          <key>Hour</key>
          <integer>8</integer>
          <key>Minute</key>
          <integer>0</integer>
          <key>Weekday</key>
          <integer>3</integer>
        </dict>
    </array>
    

If configured like above, the LaunchAgent would run Nudge at 8:00 every Wednesday. In my [repository](https://github.com/almenscorner/macOS/tree/main/LaunchAgents/Nudge) I have a few examples:

- **Default.plist** - The default LanuchAgent provided for Nudge where it runs every 30 minutes
- **EveryMinute.plist** - The LaunchAgent will run every minute and start Nudge (test purposes 😉)
- **EveryWeekDayMorningNoonEvening.plist** - The LaunchAgent will run every weekday at 8:00, 12:00 and 17:00 and start Nudge
- **SpecificTimeDay.plist** - The LaunchAgent will run every Wednesday at 10:50 and start Nudge

Now that we know what a LaunchAgent is and what we can configure, lets create our script in MEM. Remember to update the **baseURL** and **fileName** if you don't want to use my examples.

- Open the [MEM console](https://endpoint.microsoft.com)
- Go to **Devices -> macOS -> Shell scripts** and click **Add**
- Give your script a **Name** and click **Next**
- Upload the script from above as a .sh file and configure these settings:

- Run script as signed-in user: **No**
- Hide script notification on devices: **Yes**
- Script frequency: **Not configured**
- Max number of times to retry if script fails: **3 times**

- Assign the script to your test device and click **Next**
- Click **Add**
![Screenshot-2021-05-19-at-11.18.54]({{ site.baseurl }}/assets/images/2021/05/Screenshot-2021-05-19-at-11.18.54.png)

Once the script runs on your device, a file named com.github.macadmins.Nudge.plist will be created under /Library/LaunchAgents. Nudge will also be launched as soon as the agent is loaded.
![]({{ site.baseurl }}/assets/images/2021/05/Screenshot-2021-05-19-at-11.21.04.png)
With this deployed, all pieces are in place to deploy to your macs. Before you deploy, remember to update or create new configurations to target the correct OS versions.

### Digging deeper into configurations

Nudge's stages as described by Neil Martin at London Apple Admins,

Inactive

macOS is up to date or not targeted, i.e. Nudge **will not** be displayed
Initial

macOS is not up to date and is targeted
`initialRefreshCycle` number of seconds for Window refresh/deferral rate
`approachingWindowTime` hours greater than time remaining until `requiredInstallationDate`, Nudge **will be** displayed **with primary quit button**
Approaching

macOS is not up to date and is targeted
`approachingRefreshCycle` number of seconds for Window refresh/deferral rate
`approachingWindowTime` hours less than time remaining until `requiredInstallationDate`, Nudge **will be** displayed **with primary and secondary quit button**
Imminent

macOS is not up to date and is targeted
`imminentRefreshCycle` number of seconds for Window refresh/deferral rate
`imminentWindowTime` hours less than time remaining until `requiredInstallationDate`, Nudge **will be** displayed **with primary and secondary quit button**
Elapsed

macOS is not up to date and is targeted
`elapsedRefreshCycle` number of seconds for Window refresh/deferral rate
`requierdInstallationDate` has passed, Nudge **will be** displayed **without any quit buttons**

I encourage you to play around with these settings and deferrals to find the sweet spot that will work in your organisation.

A massive thanks to Erik Gomez for providing this tool to make all mac admins life easier. He has a lot of great stuff, make sure to [check them out.](https://github.com/erikng)
