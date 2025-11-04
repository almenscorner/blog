---
title: The journey to Munki and Intune
slug: the-journey-to-munki-and-intune
date_published: 2023-06-12 09:08:25.000000000 Z
date_updated: 2023-06-12 09:08:25.000000000 Z
tags:
- azure automation
- azure devops
- azure storage
- intune
- macos
- munki
excerpt: Over time, my focus has revolved around effectively managing application
  deployments for Macs using a combination of Intune, Munki, Azure DevOps, and Azure
  Storage. Through dedicated efforts, I've established a highly automated system that
  handles the majority of tasks seamlessly.
---
For quite some time now, I have been extensively involved in managing application deployments for Macs using Intune, relying on a combination of Munki, Azure DevOps, and Azure Storage. This journey has led me to establish a highly automated system, where the majority of tasks are seamlessly handled. Our team only needs to intervene in package management scenarios when a package isn't available in AutoPkg or when dealing with internal applications that have yet to transition to recipe-based deployment, which we are actively exploring.

Today's post aims to provide insights into my current daily setup and share some valuable lessons I've learned throughout the process of deploying applications within a complex organizational structure. While this post won't serve as a technical guide for deployment procedures (as I have already covered that extensively in previous posts), it will shed light on the intricacies of my existing setup.

## Why the move?

When we initially delved into the world of Munki, we encountered significant limitations in Intune's support for package deployment. The packages needed to be wrapped in a specific format compatible with Intune, and unfortunately, some packages proved impossible to deploy. Moreover, the overall process was excessively time-consuming, hampering efficiency.

## Some headaches

### Re-package

The process of adding a post-install script to an app in Intune can be highly time-consuming, often unnecessarily long due to the requirement of re-signing the app and going through additional steps. This can pose significant challenges, particularly when dealing with a large number of applications or frequent updates.

### Updating 

For example, updating an app in Intune would require signing the app every time a new release becomes available if you want to include pre/post-install scripts. This necessitates re-packaging the app, which can be impractical for large enterprises that frequently update their applications.

In such a dynamic environment, the need for a more streamlined approach becomes apparent. To address this challenge, we have implemented a solution that allows for seamless app updates without the need for frequent re-packaging and signing. By leveraging tools like Munki and other deployment mechanisms, we can efficiently manage app updates in a large enterprise setting.

With this setup, updates can be quickly and easily deployed without the hassle of repetitive signing and re-packaging. It allows us to leverage pre/post-install scripts and other configurations while maintaining a smooth and agile update process.

By adopting this approach, we ensure that our enterprise can keep pace with frequent app updates without compromising efficiency or introducing unnecessary complexities. This enables us to provide the latest versions of applications to our users promptly while minimizing administrative overhead.

### Reporting

Obtaining accurate and timely install details on applications in Intune can be a frustrating experience. The current process often results in significant delays, with reports on installation status taking days to surface. This lack of reliability undermines our ability to effectively monitor and manage application deployments.

## Improvements

However, it's worth noting that Microsoft has since made remarkable progress in this regard. They have transitioned from the wrapping approach to utilizing the MDM agent for handling DMG deployments, and there are plans to extend this method to PKG deployments as well. These advancements have significantly improved the deployment experience within Intune.

Despite these positive developments, we have decided to continue utilizing Munki for the time being. This decision is based not only on the investment of time and effort we have dedicated to Munki but also on the fact that certain features we require are still lacking in Intune's current capabilities. 

Some of lacking capabilities include:

- Post/Pre-install script configurations
- Custom detections
- Payload free packages

## The setup

The bits and pieces used to get everything running in an automated manner look like this,
![]({{ site.baseurl }}/assets/images/2023/06/Munki.png)Munki architecture overview
This visual representation provides a clear glimpse into the critical role Azure DevOps plays in our daily operations. It serves as the backbone for running our workflows efficiently and effectively. Our reliance on Azure DevOps spans a wide range of tasks, from the automated import of updates to our Munki repository using AutoPkg, to the seamless promotion of applications across various catalogs, including testing, beta, pre-prod, and production.

The key to this streamlined process lies in our pipelines, which execute on a pre-defined schedule. These pipelines take care of the entire workflow, ensuring that updates are seamlessly imported, applications are appropriately categorized, and promotions are carried out seamlessly.

By leveraging Azure DevOps in this manner, we have achieved a high level of automation and consistency in our operations, ultimately saving valuable time and effort. It serves as a reliable and powerful tool in managing our daily operations.
![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-10.47.46.png)Azure DevOps pipelines
All this is possible thanks to open-source solutions, special thanks to Gusto engineering for [providing the scripts to run AutoPkg and AutoPromote](https://engineering.gusto.com/running-autopkg-in-github-actions/#:~:text=Create%20an%20empty%20Github%20repo%20with%20Actions%20enabled.,in%20overrides%2F%29.%20Add%20the%20override%20filename%20to%20recipe_list.json.).

Furthermore, I have made updates to our scripts to accommodate our specific setup, which involves using Teams instead of Slack as the platform for receiving notifications from our pipelines. This adjustment ensures that our communication channels align with our preferred collaboration tools.

By modifying the scripts, we have seamlessly integrated the pipeline notifications with Teams, allowing us to receive important updates and alerts directly within the Teams platform. This shift in communication not only enhances convenience but also ensures that our team stays informed and connected throughout the pipeline execution process.

The updated scripts reflect our commitment to adapt our workflows to best suit our operational environment, enabling effective collaboration and streamlined communication within our team.
![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-10.52.49.png)AutoPkg imported package notification![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-10.53.16.png)AutoPromote notification![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-10.53.44.png)RepoClean notification
### Manifests

One of the primary concerns we encountered when delving into Munki was how to handle application assignments effectively. While some applications were intended for everyone and could be categorized under "site_default," there were instances where specific licensing or access restrictions dictated that certain applications should only be visible to particular sites or licensed users. We recognized the need for a solution that could leverage Azure AD group membership, as it aligned with our existing assignment processes.

Given the absence of an existing solution, I took the initiative to [develop an open-source tool](https://github.com/almenscorner/munki-manifest-generator) that addresses this challenge comprehensively. This tool takes care of the creation, updating, and maintenance of device-specific manifests, allowing us to manage application assignments based on Azure AD group membership.

Through this tool, we can effortlessly assign "included manifests" to devices based on the Azure AD groups to which users and/or devices belong. This approach has enabled us to establish a seamless "Evergreen" flow, where application updates are deployed in waves to devices. As a result, our deployment process remains efficient, ensuring that updates are effectively delivered while accommodating the specific needs and access privileges of different groups within our organization.

Currently, we rely on a custom attribute that runs on a schedule for each device to update its own manifest. However, one limitation of this approach is that we are restricted to running the updates only once every eight hours. To address this constraint and enhance flexibility, I am exploring an alternative solution.

My plan involves creating a LaunchDaemon that runs a script, leveraging a profile deployed from Intune, which reads the webhook to the Azure Automation Runbook. By implementing this approach, we can overcome the limitations of the scheduled custom attribute updates.

Shifting to a LaunchDaemon-based solution offers several advantages. Firstly, it simplifies the process of updating the webhook, ensuring that modifications can be made easily and efficiently. Additionally, it grants us the flexibility to control the runtime of the script, providing greater adaptability to our evolving requirements.
![]({{ site.baseurl }}/assets/images/2023/06/manifest.png)Device manifest updated by tool
### AutoPkg

AutoPkg plays a pivotal role in our automated environment, serving as a central component of our application management workflow. Our primary objective is to leverage recipes whenever possible, enabling automated updates without the need for manual package creation, modification, or import into Munki. Fortunately, many applications have existing recipes readily available, especially for publicly accessible ones.

However, there are instances where direct access to the package or viable APIs to the package's console is not feasible. In these cases, I have taken the initiative to create recipes that retrieve the binary from Azure Blob Storage and import it into Munki. The results of these tests have been successful thus far, and if there is interest, I would be happy to share examples of these recipes.

It is worth noting that when running AutoPkg in Azure DevOps and pipelines, there is no saved cache available. Consequently, each package needs to be downloaded every time the pipeline runs. This can quickly consume the allocated storage on the pipeline agent, particularly when dealing with numerous binaries. To address this challenge, I modified Gusto's script to clear the cache, freeing up storage space. However, it's important to acknowledge that this can lead to longer pipeline run times.

Looking ahead, I plan to explore the possibility of setting up my own agent on a managed Mac. This will enable me to establish and utilize a cache, significantly reducing run times while preserving storage capacity.
![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-11.26.20-3.png)Pipeline run tasks
### AutoPromote

During the process of importing a new package into Munki through AutoPkg or manually, it is placed within the "testing" catalog. This catalog serves as the initial stage for our internal team to test and evaluate the new package or update. To facilitate the seamless progression of packages through our deployment pipeline, an AutoPromote pipeline is triggered. This pipeline is responsible for promoting the package from the testing catalog to the beta, pre-prod, and ultimately, the production catalog.

To manage the promotion process, we rely on a configuration file within the repository specifically designed for AutoPromote. This file guides the pipeline and ensures that the package follows the appropriate promotion trajectory. Within a week's time, a package successfully moves from the testing catalog all the way to the production catalog, ready for broader deployment.

By adopting this systematic approach, we maintain a streamlined and controlled release process. In the event of any reported issues or problems, we have the flexibility to halt the promotion to further catalogs, effectively preventing any potential negative impact on our end-users. This ability to pause and address concerns ensures that we can prioritize stability and quality throughout our deployment cycle.
![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-11.44.52.png)Package promoted by AutoPromote pipeline
### Munki Repository

Our repository in Azure DevOps serves as the crown jewel of our application management system. It is the ultimate source of truth, housing all our packages, package information, manifests, and other crucial elements. If a package or any related component is not found within this repository, it will not be deployed.

While our clients do not directly connect to this repository to discover available packages and manifests, it is the central hub where administrators like us connect to add new applications, updates, and other relevant changes. This approach ensures that we maintain complete control over the repository and enables us to keep a comprehensive history of all modifications through the commit history feature. Additionally, we can implement an approval flow using Pull Requests, adding an extra layer of protection against undesired changes. This setup empowers us to easily roll back any unintended modifications that might make their way into the production environment.

By leveraging our Azure DevOps repository as the core of our application management system, we guarantee the integrity and reliability of our deployment process. The repository acts as a safeguard, enabling us to monitor changes, enforce approval workflows, and ensure a smooth roll-out of updates while minimizing the risk of unwanted or detrimental alterations.

The repository itself is a Munki repository stored in Git, and while it may appear simple, its significance lies in its functionality and purpose. This Git-based Munki repository serves as a foundational element of our application management system, providing the structure and organization needed to efficiently deploy and manage packages.

By utilizing Git as the underlying technology for our Munki repository, we benefit from its robust features such as branching, merging, and pull requests. This enables efficient collaboration among team members, streamlines the review and approval process, and facilitates the management of changes within the repository.

While the repository may be straightforward in its structure, its role as a reliable and version-controlled Munki repository is pivotal to the success of our application management.
![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-15.52.46.png)Azure DevOps Commit history
### Azure Storage Account

This serves as the front end for all clients, acting as the central hub where they connect to discover available packages and manifests. To ensure seamless synchronization of data, an automated pipeline is set up to trigger whenever changes are made to the main branch in Azure DevOps. This pipeline efficiently synchronizes all relevant information from Azure DevOps to this front-end repository.

To safeguard the integrity and security of the blob storage, we have implemented a read-only Shared Access Signature (SAS) token. This token provides controlled access to the blob storage and is regularly rotated at specified intervals. By employing this security measure, we can mitigate the risk of unauthorized modifications or access to the repository.

The combination of the automated synchronization pipeline and the read-only SAS token mechanism ensures that our front-end repository remains up to date, reflecting the latest changes and additions from Azure DevOps, while maintaining strict access controls. This approach guarantees that clients have access to the most current and accurate information regarding available packages and manifests, while preserving the security and integrity of the repository.
![]({{ site.baseurl }}/assets/images/2023/06/Screenshot-2023-06-09-at-15.48.29.png)Munki repository as Azure Blobs
### Intune

So are we using Intune at all for application deployment? short answer, no!

the only aspect related to application management that we deploy through Intune includes:

- Munki configuration profile to configure the repository URL and SAS token
- Install script that installs Munkitools on our clients
- Custom Attribute script that updates the devices manifest on an interval

That's it.

To install Munkitools, we are using the base shell script for installing packages created by the great Neil Johnson which I have made some modifications to.

First we are checking if the Munki profile has been installed, if not, we continue to wait:

    # function to delay script if the Munki profile is not yet installed
    waitForProfile () {
    	
        #################################################################################################################
        #################################################################################################################
        ##
        ##  Function to pause while the Munki profile is not installed
        ##
        ##  Functions used
        ##
        ##      None
        ##
        ##  Variables used
        ##
        ##      $1 = name of profile to check for
        ##      $2 = length of delay (if missing, function to generate random delay between 10 and 60s)
        ##      $profileState = If value is null, run while loop until profile is installed
        ##
        ###############################################################
        ###############################################################
    
        profileName=$1
        fixedDelay=$2
        profileState=$( sudo /usr/bin/profiles -P | grep -w "www.windowsintune.com.custompayload.$profileName" )
    
        while [[ ! $profileState ]];
        do
            profileState=$( sudo /usr/bin/profiles -P | grep -w "www.windowsintune.com.custompayload.$profileName" )
    
            # If we've been passed a delay we should use it, otherwise we'll create a random delay each run
            if [[ ! $fixedDelay ]]; then
                delay=$(( $RANDOM % 50 + 10 ))
            else
                delay=$fixedDelay
            fi
    
            echo "$(date) | Munki profile not installed, waiting [$delay] seconds"
            sleep $delay
    
        done
        
        echo "$(date) | Munki profile found, safe to proceed"
    
    }
    
    

When the profile is installed, we are then running a Munki check-in:

    # Function to check if Munki has any apps to install
    checkMunkiInstalls () {
    
        echo "$(date) | Checking for Munki installs or upgrades"
        output=$(sudo /usr/local/munki/managedsoftwareupdate --munkipkgsonly | grep "installed or upgraded")
    
        if [[ ! $output ]]; then
            echo "$(date) | Munki up to date, nothing to install"
        else
            echo "$(date) | Munki installs found, installing apps"
            sudo /usr/local/munki/managedsoftwareupdate --auto
        fi
    
        failures=$(sudo cat /Library/Managed\ Installs/Logs/ManagedSoftwareUpdate.log | grep "The install failed")
    
        if [[ ! $failures ]]; then
            echo "$(date) | Munki successfully installed the apps"
        else
            echo "$(date) | App(s) failed to install, check ManagedSoftwareUpdate logs"
        fi
    }
    

In addition, we are installing Dockutil and add Managed Software Center to the dock:

    # function to add MSC to the dock if dockutil is installed
    addToDock () {
        dockutil="/usr/local/bin/dockutil"
    
        currentUser=$( echo "show State:/Users/ConsoleUser" | scutil | awk '/Name :/ { print $3 }' )
    
        # Get uid logged in user
        uid=$(id -u "${currentUser}")
    
        # Current User home folder - do it this way in case the folder isn't in /Users
        userHome=$(dscl . -read /users/${currentUser} NFSHomeDirectory | cut -d " " -f 2)
    
        # Path to plist
        plist="${userHome}/Library/Preferences/com.apple.dock.plist"
    
        # Convenience function to run a command as the current user
        # usage: runAsUser command arguments...
        runAsUser() {  
            if [[ "${currentUser}" != "loginwindow" ]]; then
                launchctl asuser "$uid" sudo -u "${currentUser}" "$@"
            else
                echo "no user logged in"
                exit 1
            fi
        }
    
        if [ -f "$dockutil" ]; then
            dock=$( runAsUser "${dockutil}" --list ${plist} | grep "Managed Software Centre" )
    
            if [[ -z ${dock} ]]; then
                runAsUser "${dockutil}" --add '/Applications/Managed Software Center.app' --position 2 ${plist} > /dev/null 2>&1
            fi
        fi
    }
    

These functions are then used in the "InstallPKG" function:

        # Checking if the app was installed successfully
        if [ "$?" = "0" ]; then
    
            echo "$(date) | $appname Installed"
            echo "$(date) | Cleaning Up"
            rm -rf "$tempdir"
    
            echo "$(date) | Application [$appname] succesfully installed"
            fetchLastModifiedDate update
            updateOctory installed
    
            # Getting additional middleware script for Munki
            echo "$(date) | Installing Middleware"
            curl "$weburl_middleware" -o /usr/local/munki/middleware_azure.py
    
            # Check if Munki profile is installed, if not installed wait 10 seconds
            waitForProfile "$munkiProfileName" 10
    
            # Run managedsoftwareupdate and check for installs
            checkMunkiInstalls
    
            # Add MSC to dock if not present
            addToDock
    
            exit 0
    
        else
    

These changes guarantees that all enrolling clients are properly configured and prepared for use in a controlled and stable manner. By carefully checking for the installation of the Munki profile and implementing a waiting mechanism, we ensure that the necessary components are in place before proceeding. This precautionary step helps create a consistent and reliable environment for our clients, promoting stability and minimizing potential issues.

### Munki Report

No comprehensive setup would be complete without the ability to monitor and proactively address any issues that may arise. To fulfill this requirement, we have implemented Munki Report, which runs within a Docker container as an Azure Web App. This robust solution empowers us to monitor application deployments in near real-time, enabling us to swiftly respond and rectify any potential problems that may occur.

By leveraging Munki Report, we gain valuable insights into the status of our application deployments. We can monitor the progress, identify any errors or anomalies, and promptly take action to ensure a seamless and successful deployment process. This proactive approach allows us to address any issues before they escalate and impact our users.

The Azure Web App hosting further enhances the accessibility and scalability of our monitoring solution. It provides a reliable platform for running Munki Report, ensuring continuous availability and efficient performance.

By utilizing Munki Report within a Docker container as an Azure Web App, we establish a robust monitoring framework that keeps us informed and empowers us to proactively manage our application deployments. This proactive stance ensures a smooth and efficient operation while minimizing any potential disruptions.
![]({{ site.baseurl }}/assets/images/2023/06/mr.png)Munki Report overview

## Final words

Despite the significant time and effort invested, the solution I have developed ultimately streamlines our processes and lightens the workload for us in the future. While there are areas that can still be improved upon, we are currently in an excellent state and have no plans to migrate from this well-established setup.

## Improvement areas

As mentioned earlier, there are areas within the setup that can still be optimized to further enhance the workflow. Two specific areas that require attention are migrating to our own pipeline agents to facilitate caching for AutoPkg and streamlining the onboarding process for new administrators who may have limited Git experience.

By setting up our own pipeline agents, we can take advantage of caching capabilities for AutoPkg. This will significantly reduce the time it takes to download and manage multiple binaries, ultimately improving the speed and efficiency of our pipeline runs.

In addition, refining our internal onboarding process for new administrators who may have limited Git experience is crucial. By developing comprehensive training resources, documentation, and mentorship programs, we can accelerate their learning curve and equip them with the necessary skills to effectively contribute to our setup. This will streamline the onboarding process, allowing new administrators to quickly become proficient in working with Git and other essential tools.

## Who is this setup not for?

Regardless of the size of your environment, I firmly believe that the setup I have described can bring significant benefits. It is a scalable solution that can be adapted to suit various organizational needs. However, it's important to note that if you require support with a Service Level Agreement (SLA) for your solution, our current setup may not fulfill that requirement.

It's worth highlighting that every component we utilize in our setup is open-source. While this means that there is no formal support agreement in place, it also means that we are part of a vibrant and supportive community. Munki, in particular, is widely recognized and loved within the Mac Admin community. Many large organizations rely on Munki, fostering a strong community where knowledge sharing and assistance are readily available.

In fact, the Mac Admin community often provides faster and more comprehensive support than what is typically experienced with paid solutions. The collective expertise and willingness to help among community members can prove invaluable when encountering challenges or seeking guidance.

While our solution may not come with a formal support structure, we leverage the strength of the open-source community to ensure ongoing success and overcome obstacles. The combination of the reliable Munki framework and the support of the Mac Admin community ultimately provides us with a robust and well-supported solution.

## Closing

Throughout this post, I aimed to provide valuable insights into how we effectively leverage Munki and Intune in our operations. By sharing our experiences, I hope to offer practical knowledge and inspiration to others who may be exploring similar approaches.

As we continue to refine and enhance our system, I remain confident that our current setup will continue to serve us well. The combination of Munki and Intune has proven to be a powerful solution, providing us with the tools and capabilities needed to efficiently manage our applications.
