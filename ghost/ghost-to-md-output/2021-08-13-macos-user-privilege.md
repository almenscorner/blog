---
title: macOS user privilege, admin or not?
slug: macos-user-privilege
date_published: 2021-08-13T13:21:00.000Z
date_updated: 2021-08-31T12:19:13.000Z
tags: macOS, Security, Privileges
excerpt: Many tasks on macOS requires the user to be an admin to be able to do their job, especially when talking about developers. But do they need to be an admin non-stop 24/7? 
---

For years we have been locking our users on Windows devices out from being administrators, and for a good reason. For macOS, this is a different story. Many tasks on macOS requires the user to be an admin to be able to do their job, especially when talking about developers. But do they need to be an admin non stop 24/7? 

Today we're going to have a look at enrolling our mac users as standard users, but giving them the ability to elevate their privileges when needed.

To do this, we'll walk through how to deploy an app called Privileges in Microsoft Endpoint Manager. 

## What is Privileges?

Developed by the great guys at SAP, and which were nice enough to release it as open source, Privileges is an app that allows your users to work as standard users performing their day-to-day tasks and provide an easy way to request admin privileges when needed. All you have to do is click an icon in the dock.

> We believe all users, including all developers, can benefit from using **Privileges.app**. Working as a standard user instead of an administrator adds another layer of security to your Mac and is considered a security best practice. **Privileges.app** helps enable users to act as administrators of the system only when required.

### Requirements

**Privileges** supports the following macOS versions:

- macOS 10.12.x
- macOS 10.13.x
- macOS 10.14.x
- macOS 10.15.x
- macOS 11.x

## Deploy Privileges with MEM

As Privileges comes as a .app file, we have to install it using a scripted method instead of wrapping it. This script is provided by Microsoft and includes good logging which you can collect from the MEM console, we're going to have a look at that when the script has run. Below you can find the script I modified to download Privileges directly from SAP's GitHub repo.

**Note: **at the time of writing, the version is 1.5.2

To use this script, first do the following

- Copy the script
- Save it as installprivileges.sh
- In the terminal on a mac, run this command to make it executable

- `chmod +x /path/installprivileges.sh`

    #!/bin/bash
    #set -x
    
    ############################################################################################
    ##
    ## Script to install the latest GNU Imagine Manipulation client
    ##
    ############################################################################################
    
    ## Copyright (c) 2020 Microsoft Corp. All rights reserved.
    ## Scripts are not supported under any Microsoft standard support program or service. The scripts are provided AS IS without warranty of any kind.
    ## Microsoft disclaims all implied warranties including, without limitation, any implied warranties of merchantability or of fitness for a
    ## particular purpose. The entire risk arising out of the use or performance of the scripts and documentation remains with you. In no event shall
    ## Microsoft, its authors, or anyone else involved in the creation, production, or delivery of the scripts be liable for any damages whatsoever
    ## (including, without limitation, damages for loss of business profits, business interruption, loss of business information, or other pecuniary
    ## loss) arising out of the use of or inability to use the sample scripts or documentation, even if Microsoft has been advised of the possibility
    ## of such damages.
    ## Feedback: neiljohn@microsoft.com
    
    # User Defined variables
    weburl="https://github.com/SAP/macOS-enterprise-privileges/releases/download/1.5.2/Privileges.zip"      # What is the Azure Blob Storage URL?
    appname="Privileges"                                                                              # The name of our App deployment script (also used for Octory monitor)
    app="Privileges.app"                                                                              # The actual name of our App once installed
    logandmetadir="/Library/Logs/Microsoft/IntuneScripts/installPrivileges"                           # The location of our logs and last updated data
    processpath="/Applications/Privileges.app/Contents/MacOS/Privileges"                                    # The process name of the App we are installing
    terminateprocess="false"                                                                    # Do we want to terminate the running process? If false we'll wait until its not running
    autoUpdate="false"                                                                          # Application updates itself, if already installed we should exit
    
    # Generated variables
    tempdir=$(mktemp -d)
    log="$logandmetadir/$appname.log"                                                           # The location of the script log file
    metafile="$logandmetadir/$appname.meta"                                                     # The location of our meta file (for updates)
    
    # function to delay script if the specified process is running
    waitForProcess () {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  Function to pause while a specified process is running
        ##
        ##  Functions used
        ##
        ##      None
        ##
        ##  Variables used
        ##
        ##      $1 = name of process to check for
        ##      $2 = length of delay (if missing, function to generate random delay between 10 and 60s)
        ##      $3 = true/false if = "true" terminate process, if "false" wait for it to close
        ##
        ###############################################################
        ###############################################################
    
        processName=$1
        fixedDelay=$2
        terminate=$3
    
        echo "$(date) | Waiting for other [$processName] processes to end"
        while ps aux | grep "$processName" | grep -v grep &>/dev/null; do
    
            if [[ $terminate == "true" ]]; then
                echo "$(date) | + [$appname] running, terminating [$processpath]..."
                pkill -f "$processName"
                return
            fi
    
            # If we've been passed a delay we should use it, otherwise we'll create a random delay each run
            if [[ ! $fixedDelay ]]; then
                delay=$(( $RANDOM % 50 + 10 ))
            else
                delay=$fixedDelay
            fi
    
            echo "$(date) |  + Another instance of $processName is running, waiting [$delay] seconds"
            sleep $delay
        done
        
        echo "$(date) | No instances of [$processName] found, safe to proceed"
    
    }
    
    # function to check if we need Rosetta 2
    checkForRosetta2 () {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  Simple function to install Rosetta 2 if needed.
        ##
        ##  Functions
        ##
        ##      waitForProcess (used to pause script if another instance of softwareupdate is running)
        ##
        ##  Variables
        ##
        ##      None
        ##
        ###############################################################
        ###############################################################
    
        echo "$(date) | Checking if we need Rosetta 2 or not"
    
        # if Software update is already running, we need to wait...
        waitForProcess "/usr/sbin/softwareupdate"
    
        processor=$(/usr/sbin/sysctl -n machdep.cpu.brand_string)
        if [[ "$processor" == *"Intel"* ]]; then
    
            echo "$(date) | [$processor] found, Rosetta not needed"
            
        else
    
            echo "$(date) | [$processor] founbd, is Rosetta already installed?"
    
            # Check Rosetta LaunchDaemon. If no LaunchDaemon is found,
            # perform a non-interactive install of Rosetta.
            
            if [[ ! -f "/Library/Apple/System/Library/LaunchDaemons/com.apple.oahd.plist" ]]; then
                /usr/sbin/softwareupdate --install-rosetta --agree-to-license
            
                if [[ $? -eq 0 ]]; then
                    echo "$(date) | Rosetta has been successfully installed."
                    return
                else
                    echo "$(date) | Rosetta installation failed!"
                fi
        
            else
                echo "$(date) | Rosetta is already installed. Nothing to do."
            fi
        fi
    
    }
    
    # Function to update the last modified date for this app
    fetchLastModifiedDate() {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  This function takes the following global variables and downloads the URL provided to a temporary location
        ##
        ##  Functions
        ##
        ##      none
        ##
        ##  Variables
        ##
        ##      $logandmetadir = Directory to read nand write meta data to
        ##      $metafile = Location of meta file (used to store last update time)
        ##      $weburl = URL of download location
        ##      $tempfile = location of temporary DMG file downloaded
        ##      $lastmodified = Generated by the function as the last-modified http header from the curl request
        ##
        ##  Notes
        ##
        ##      If called with "fetchLastModifiedDate update" the function will overwrite the current lastmodified date into metafile
        ##
        ###############################################################
        ###############################################################
    
        ## Check if the log directory has been created
        if [[ ! -d "$logandmetadir" ]]; then
            ## Creating Metadirectory
            echo "$(date) | Creating [$logandmetadir] to store metadata"
            mkdir -p "$logandmetadir"
        fi
    
        # generate the last modified date of the file we need to download
        lastmodified=$(curl -sIL "$weburl" | grep -i "last-modified" | awk '{$1=""; print $0}' | awk '{ sub(/^[ \t]+/, ""); print }' | tr -d '\r')
    
        if [[ $1 == "update" ]]; then
            echo "$(date) | Writing last modifieddate [$lastmodified] to [$metafile]"
            echo "$lastmodified" > "$metafile"
        fi
    
    }
    
    function downloadApp () {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  This function takes the following global variables and downloads the URL provided to a temporary location
        ##
        ##  Functions
        ##
        ##      waitForCurl (Pauses download until all other instances of Curl have finished)
        ##      downloadSize (Generates human readable size of the download for the logs)
        ##
        ##  Variables
        ##
        ##      $appname = Description of the App we are installing
        ##      $weburl = URL of download location
        ##      $tempfile = location of temporary DMG file downloaded
        ##
        ###############################################################
        ###############################################################
    
        echo "$(date) | Starting downlading of [$appname]"
    
        # wait for other downloads to complete
        waitForProcess "curl -f"
    
        #download the file
        updateOctory installing
        echo "$(date) | Downloading $appname"
    
        cd "$tempdir"
        curl -f -s --connect-timeout 30 --retry 5 --retry-delay 60 -L -J -O "$weburl"
        if [ $? == 0 ]; then
    
                # We have downloaded a file, we need to know what the file is called and what type of file it is
                tempSearchPath="$tempdir/*"
                for f in $tempSearchPath; do
                    tempfile=$f
                done
    
                case $tempfile in
    
                *.pkg|*.PKG)
                    packageType="PKG"
                    ;;
    
                *.zip|*.ZIP)
                    packageType="ZIP"
                    ;;
    
                *.dmg|*.DMG)
                    packageType="DMG"
                    ;;
    
                *)
                    # We can't tell what this is by the file name, lets look at the metadata
                    echo "$(date) | Unknown file type [$f], analysing metadata"
                    metadata=$(file "$tempfile")
                    if [[ "$metadata" == *"Zip archive data"* ]]; then
                        packageType="ZIP"
                        mv "$tempfile" "$tempdir/install.zip"
                        tempfile="$tempdir/install.zip"
                    fi
    
                    if [[ "$metadata" == *"xar archive"* ]]; then
                        packageType="PKG"
                        mv "$tempfile" "$tempdir/install.pkg"
                        tempfile="$tempdir/install.pkg"
                    fi
    
                    if [[ "$metadata" == *"bzip2 compressed data"* ]] || [[ "$metadata" == *"zlib compressed data"* ]] ; then
                        packageType="DMG"
                        mv "$tempfile" "$tempdir/install.dmg"
                        tempfile="$tempdir/install.dmg"
                    fi
                    ;;
                esac
    
                if [[ ! $packageType ]]; then
                    echo "Failed to determine temp file type"
                    rm -rf "$tempdir"
                else
                    echo "$(date) | Downloaded [$app] to [$tempfile]"
                    echo "$(date) | Detected install type as [$packageType]"
                fi
             
        else
        
            echo "$(date) | Failure to download [$weburl] to [$tempfile]"
             updateOctory failed
             exit 1
        fi
    
    }
    
    # Function to check if we need to update or not
    function updateCheck() {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  This function takes the following dependencies and variables and exits if no update is required
        ##
        ##  Functions
        ##
        ##      fetchLastModifiedDate
        ##
        ##  Variables
        ##
        ##      $appname = Description of the App we are installing
        ##      $tempfile = location of temporary DMG file downloaded
        ##      $volume = name of volume mount point
        ##      $app = name of Application directory under /Applications
        ##
        ###############################################################
        ###############################################################
    
    
        echo "$(date) | Checking if we need to install or update [$appname]"
    
        ## Is the app already installed?
        if [ -d "/Applications/$app" ]; then
    
        # App is installed, if it's updates are handled by MAU we should quietly exit
        if [[ $autoUpdate == "true" ]]; then
            echo "$(date) | [$appname] is already installed and handles updates itself, exiting"
            exit 0
        fi
    
        # App is already installed, we need to determine if it requires updating or not
            echo "$(date) | [$appname] already installed, let's see if we need to update"
            fetchLastModifiedDate
    
            ## Did we store the last modified date last time we installed/updated?
            if [[ -d "$logandmetadir" ]]; then
    
                if [ -f "$metafile" ]; then
                    previouslastmodifieddate=$(cat "$metafile")
                    if [[ "$previouslastmodifieddate" != "$lastmodified" ]]; then
                        echo "$(date) | Update found, previous [$previouslastmodifieddate] and current [$lastmodified]"
                        update="update"
                    else
                        echo "$(date) | No update between previous [$previouslastmodifieddate] and current [$lastmodified]"
                        echo "$(date) | Exiting, nothing to do"
                        exit 0
                    fi
                else
                    echo "$(date) | Meta file [$metafile] not found"
                    echo "$(date) | Unable to determine if update required, updating [$appname] anyway"
    
                fi
                
            fi
    
        else
            echo "$(date) | [$appname] not installed, need to download and install"
        fi
    
    }
    
    ## Install PKG Function
    function installPKG () {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  This function takes the following global variables and installs the PKG file
        ##
        ##  Functions
        ##
        ##      isAppRunning (Pauses installation if the process defined in global variable $processpath is running )
        ##      fetchLastModifiedDate (Called with update flag which causes the function to write the new lastmodified date to the metadata file)
        ##
        ##  Variables
        ##
        ##      $appname = Description of the App we are installing
        ##      $tempfile = location of temporary DMG file downloaded
        ##      $volume = name of volume mount point
        ##      $app = name of Application directory under /Applications
        ##
        ###############################################################
        ###############################################################
    
    
        # Check if app is running, if it is we need to wait.
        waitForProcess "$processpath" "300" "$terminateprocess"
    
        echo "$(date) | Installing $appname"
    
    
        # Update Octory monitor
        updateOctory installing
    
        # Remove existing files if present
        if [[ -d "/Applications/$app" ]]; then
            rm -rf "/Applications/$app"
        fi
    
        installer -pkg "$tempfile" -target /Applications
    
        # Checking if the app was installed successfully
        if [ "$?" = "0" ]; then
    
            echo "$(date) | $appname Installed"
            echo "$(date) | Cleaning Up"
            rm -rf "$tempdir"
    
            echo "$(date) | Application [$appname] succesfully installed"
            fetchLastModifiedDate update
            updateOctory installed
            exit 0
    
        else
    
            echo "$(date) | Failed to install $appname"
            rm -rf "$tempdir"
            updateOctory failed
            exit 1
        fi
    
    }
    
    ## Install DMG Function
    function installDMG () {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  This function takes the following global variables and installs the DMG file into /Applications
        ##
        ##  Functions
        ##
        ##      isAppRunning (Pauses installation if the process defined in global variable $processpath is running )
        ##      fetchLastModifiedDate (Called with update flag which causes the function to write the new lastmodified date to the metadata file)
        ##
        ##  Variables
        ##
        ##      $appname = Description of the App we are installing
        ##      $tempfile = location of temporary DMG file downloaded
        ##      $volume = name of volume mount point
        ##      $app = name of Application directory under /Applications
        ##
        ###############################################################
        ###############################################################
    
    
        # Check if app is running, if it is we need to wait.
        waitForProcess "$processpath" "300" "$terminateprocess"
    
    
    
        echo "$(date) | Installing [$appname]"
        updateOctory installing
    
        # Mount the dmg file...
        volume="$tempdir/$appname"
        echo "$(date) | Mounting Image"
        hdiutil attach -quiet -nobrowse -mountpoint "$volume" "$tempfile"
    
        # Remove existing files if present
        if [[ -d "/Applications/$app" ]]; then
            echo "$(date) | Removing existing files"
            rm -rf "/Applications/$app"
        fi
    
        # Sync the application and unmount once complete
        echo "$(date) | Copying app files to /Applications/$app"
        rsync -a "$volume"/*.app/ "/Applications/$app"
    
        # Unmount the dmg
        echo "$(date) | Un-mounting [$volume]"
        hdiutil detach -quiet "$volume"
    
        # Checking if the app was installed successfully
    
        if [[ -a "/Applications/$app" ]]; then
    
            echo "$(date) | [$appname] Installed"
            echo "$(date) | Cleaning Up"
            rm -rf "$tempfile"
    
            echo "$(date) | Fixing up permissions"
            sudo chown -R root:wheel "/Applications/$app"
            echo "$(date) | Application [$appname] succesfully installed"
            fetchLastModifiedDate update
            updateOctory installed
            exit 0
        else
            echo "$(date) | Failed to install [$appname]"
            rm -rf "$tempdir"
            updateOctory failed
            exit 1
        fi
    
    }
    
    ## Install ZIP Function
    function installZIP () {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  This function takes the following global variables and installs the DMG file into /Applications
        ##
        ##  Functions
        ##
        ##      isAppRunning (Pauses installation if the process defined in global variable $processpath is running )
        ##      fetchLastModifiedDate (Called with update flag which causes the function to write the new lastmodified date to the metadata file)
        ##
        ##  Variables
        ##
        ##      $appname = Description of the App we are installing
        ##      $tempfile = location of temporary DMG file downloaded
        ##      $volume = name of volume mount point
        ##      $app = name of Application directory under /Applications
        ##
        ###############################################################
        ###############################################################
    
    
        # Check if app is running, if it is we need to wait.
        waitForProcess "$processpath" "300" "$terminateprocess"
    
        echo "$(date) | Installing $appname"
        updateOctory installing
    
        # Change into temp dir
        cd "$tempdir"
        if [ "$?" = "0" ]; then
          echo "$(date) | Changed current directory to $tempdir"
        else
          echo "$(date) | failed to change to $tempfile"
          if [ -d "$tempdir" ]; then rm -rf $tempdir; fi
          updateOctory failed
          exit 1
        fi
    
        # Unzip files in temp dir
        unzip -qq -o "$tempfile"
        if [ "$?" = "0" ]; then
          echo "$(date) | $tempfile unzipped"
        else
          echo "$(date) | failed to unzip $tempfile"
          if [ -d "$tempdir" ]; then rm -rf $tempdir; fi
          updateOctory failed
          exit 1
        fi
    
        # If app is already installed, remove all old files
        if [[ -a "/Applications/$app" ]]; then
        
          echo "$(date) | Removing old installation at /Applications/$app"
          rm -rf "/Applications/$app"
        
        fi
    
        # Copy over new files
        rsync -a "$app/" "/Applications/$app"
        if [ "$?" = "0" ]; then
          echo "$(date) | $appname moved into /Applications"
        else
          echo "$(date) | failed to move $appname to /Applications"
          if [ -d "$tempdir" ]; then rm -rf $tempdir; fi
          updateOctory failed
          exit 1
        fi
    
        # Make sure permissions are correct
        echo "$(date) | Fix up permissions"
        sudo chown -R root:wheel "/Applications/$app"
        if [ "$?" = "0" ]; then
          echo "$(date) | correctly applied permissions to $appname"
        else
          echo "$(date) | failed to apply permissions to $appname"
          if [ -d "$tempdir" ]; then rm -rf $tempdir; fi
          updateOctory failed
          exit 1
        fi
    
        # Checking if the app was installed successfully
        if [ "$?" = "0" ]; then
            if [[ -a "/Applications/$app" ]]; then
    
                echo "$(date) | $appname Installed"
                updateOctory installed
                echo "$(date) | Cleaning Up"
                rm -rf "$tempfile"
    
                # Update metadata
                fetchLastModifiedDate update
    
                echo "$(date) | Fixing up permissions"
                sudo chown -R root:wheel "/Applications/$app"
                echo "$(date) | Application [$appname] succesfully installed"
                exit 0
            else
                echo "$(date) | Failed to install $appname"
                exit 1
            fi
        else
    
            # Something went wrong here, either the download failed or the install Failed
            # intune will pick up the exit status and the IT Pro can use that to determine what went wrong.
            # Intune can also return the log file if requested by the admin
            
            echo "$(date) | Failed to install $appname"
            if [ -d "$tempdir" ]; then rm -rf $tempdir; fi
            exit 1
        fi
    }
    
    function updateOctory () {
    
        #################################################################################################################
        #################################################################################################################
        ##
        ##  This function is designed to update Octory status (if required)
        ##
        ##
        ##  Parameters (updateOctory parameter)
        ##
        ##      notInstalled
        ##      installing
        ##      installed
        ##
        ###############################################################
        ###############################################################
    
        # Is Octory present
        if [[ -a "/Library/Application Support/Octory" ]]; then
    
            # Octory is installed, but is it running?
            if [[ $(ps aux | grep -i "Octory" | grep -v grep) ]]; then
                echo "$(date) | Updating Octory monitor for [$appname] to [$1]"
                /usr/local/bin/octo-notifier monitor "$appname" --state $1 >/dev/null
            fi
        fi
    
    }
    
    function startLog() {
    
        ###################################################
        ###################################################
        ##
        ##  start logging - Output to log file and STDOUT
        ##
        ####################
        ####################
    
        if [[ ! -d "$logandmetadir" ]]; then
            ## Creating Metadirectory
            echo "$(date) | Creating [$logandmetadir] to store logs"
            mkdir -p "$logandmetadir"
        fi
    
        exec &> >(tee -a "$log")
        
    }
    
    # function to delay until the user has finished setup assistant.
    waitForDesktop () {
      until ps aux | grep /System/Library/CoreServices/Dock.app/Contents/MacOS/Dock | grep -v grep &>/dev/null; do
        delay=$(( $RANDOM % 50 + 10 ))
        echo "$(date) |  + Dock not running, waiting [$delay] seconds"
        sleep $delay
      done
      echo "$(date) | Dock is here, lets carry on"
    }
    
    ###################################################################################
    ###################################################################################
    ##
    ## Begin Script Body
    ##
    #####################################
    #####################################
    
    # Initiate logging
    startLog
    
    echo ""
    echo "##############################################################"
    echo "# $(date) | Logging install of [$appname] to [$log]"
    echo "############################################################"
    echo ""
    
    # Install Rosetta if we need it
    checkForRosetta2
    
    # Test if we need to install or update
    updateCheck
    
    # Wait for Desktop
    waitForDesktop
    
    # Download app
    downloadApp
    
    # Install PKG file
    if [[ $packageType == "PKG" ]]; then
        installPKG
    fi
    
    # Install PKG file
    if [[ $packageType == "ZIP" ]]; then
        installZIP
    fi
    
    # Install PKG file
    if [[ $packageType == "DMG" ]]; then
        installDMG
    fi
    

Now that the script is prepared, it can be added to MEM. To upload the script, follow the steps below.

1. Sign in to the [MEM console](https://endpoint.microsoft.com)
2. Go to **Devices -> macOS -> Shell scripts** and click **Add**
3. Give your script a **Name**, then click **Next**
4. Upload the script annd configure script settings, I will be using the defaults from Microsofts example

- Run script as signed-in user: **No**
- Hide script notifiations on devices: **Yes**
- Script frequency: **Every 1 day**
- Max number of times to retry if the script fails: **3 times**

5. Click **Next**, assign the script to a test device then click **Next** and **Add**

### Troubleshooting

If the script fails, you can see additional details on the script result and also collect logs by going to **Device status **in the script details.

- To collect logs, click **Collect Logs** and provide the following path, **/Library/Logs/Microsoft/IntuneScripts/installPrivileges/Privileges.log**
- If you click on Show details, the last output from the script is displayed

![](__GHOST_URL__/content/images/2021/08/Screenshot-2021-08-13-at-11.22.31.png)
![Screenshot-2021-08-13-at-11.35.25-1](__GHOST_URL__/content/images/2021/08/Screenshot-2021-08-13-at-11.35.25-1.png)

- Once the logs has been collected, you can now download the logs by clicking **Download logs**. Inside the zip file will be the requested file, and the two Intune script agent (user) and daemon (root) logs, which are always returned.

![Screenshot-2021-08-13-at-13.00.57](__GHOST_URL__/content/images/2021/08/Screenshot-2021-08-13-at-13.00.57.png)

## Configuring Privileges

To configure Privileges we're going to use a custom configuration profile. If you are fine with the user only having to click the app to be an administrator for 20 minutes you don't have to change anything. I'm going to set 3 additional settings,

1. `DockToggleMaxTimeout` when this key is configured, it allows the user to choose timeout values to something shorter than 20 minutes but 20 will always be the maximum amout of time they can be an admin before it reverts back to a standard user.
2. `RequireAuthentication` Requires the user to enter their password before admin rights are granted
3. `ReasonRequired` Requires the user to enter a reason why they need admin rights. The given reason is logged. `ReasonMinLength` must also be set which specifies the minimum amount of characters the user has to enter as the reason. Maximum number of characters are 100.

Besides the additional keys I'm configuring, you can also set these configurations,

- `DockToggleTimeout` sets a fixed timeout before the user is reverted to a standard user
- `EnforcePrivileges` Allows you to enforce certain privileges

- admin: administrator rights always set by Privileges.
- user: standard user rights are always set by Privileges.
- none: Privileges.app and the PrivilegesCLI command line tool are disabled and it is not possible to change user privileges using these tools.

- `LimitToGroup` limit the use of Privileges to a specified user group
- `LimitToUser` limit the use of Privileges to a specified user
- `RemoteLogging` used to send the logging for Privileges.app to a remote syslog server

If you want to test the application with the same settings I'm using, copy the profile below and save it as a `.mobileconfig` file.

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
    	<key>PayloadContent</key>
    	<array>
    		<dict>
    			<key>PayloadContent</key>
    			<dict>
    				<key>corp.sap.privileges</key>
    				<dict>
    					<key>Forced</key>
    					<array>
    						<dict>
    							<key>mcx_preference_settings</key>
    							<dict>
    								<key>DockToggleMaxTimeout</key>
    								<integer>20</integer>
                                    <key>RequireAuthentication</key>
                                    <true/>
                                    <key>ReasonRequired</key>
    								<true/>
                                    <key>ReasonMinLength</key>
    								<integer>5</integer>
    							</dict>
    						</dict>
    					</array>
    				</dict>
    			</dict>
    			<key>PayloadDescription</key>
    			<string/>
    			<key>PayloadDisplayName</key>
    			<string>Privileges configuration</string>
    			<key>PayloadEnabled</key>
    			<true/>
    			<key>PayloadIdentifier</key>
    			<string>com.apple.ManagedClient.preferences.AEC544EE-591D-4834-900E-4C94858E7871</string>
    			<key>PayloadOrganization</key>
    			<string>DemoOrg</string>
    			<key>PayloadType</key>
    			<string>com.apple.ManagedClient.preferences</string>
    			<key>PayloadUUID</key>
    			<string>AEC544EE-591D-4834-900E-4C94858E7871</string>
    			<key>PayloadVersion</key>
    			<integer>1</integer>
    		</dict>
    	</array>
    	<key>PayloadDescription</key>
    	<string>Configures the Privileges app.</string>
    	<key>PayloadDisplayName</key>
    	<string>Privileges configuration</string>
    	<key>PayloadEnabled</key>
    	<true/>
    	<key>PayloadIdentifier</key>
    	<string>82A2ED21-27DA-428E-8516-446A52169C71</string>
    	<key>PayloadOrganization</key>
    	<string>SAP SE</string>
    	<key>PayloadRemovalDisallowed</key>
    	<true/>
    	<key>PayloadScope</key>
    	<string>System</string>
    	<key>PayloadType</key>
    	<string>Configuration</string>
    	<key>PayloadUUID</key>
    	<string>C2F39834-001F-4930-AC7D-E5BA0DE82529</string>
    	<key>PayloadVersion</key>
    	<integer>1</integer>
    </dict>
    </plist>
    

Once you have the file saved follow the process below.

1. Open the [MEM console](https://endpoint.microsoft.com)
2. Go to **Devices -> Configuration profile** and click **Create**, choose **macOS** for **Platform** and **Profile type** as **Template** then, choose **Custom** and click **Create**
3. Give your profile a **Name** and click **Next**
4. Upload the .mobileconfig file, set the **Configuration profile name** and click **Next**
5. Assign the profile to your test device and click **Next**
6. Click **Create**

## End user experience

This is how the process looks for the end user. All they have to do is click the Privileges app, provide a reason, enter their password and they're admin. A visual indication shows if they are admin or not, if the Privileges icon is green, the user is a standard user. If the icon is orange, the user is an admin.
![](__GHOST_URL__/content/images/2021/08/Privileges.gif)
