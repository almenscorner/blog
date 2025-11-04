---
title: 'Introducing: IntuneCD tool'
slug: introducing-intunecd-tool
date_published: 2021-11-17 20:27:09.000000000 Z
date_updated: 2022-09-30 09:05:02.000000000 Z
tags:
- azure devops
- intune
- mem
- pipeline
- python
excerpt: IntuneCD or, Intune Continuous Delivery as it stands for is a Python package
  that is used to backup data from Intune and update configurations in Intune.
---
[pypi link](https://pypi.org/project/IntuneCD)
[GitHub link](https://github.com/almenscorner/IntuneCD)

if you like it, consider leaving a ⭐️ on GitHub!

# IntuneCD tool

IntuneCD or, Intune Continuous Delivery as it stands for is a Python package that is used to back up and update configurations in Intune. It was created with running it from a pipeline in mind. Using this approach we get complete history of which configurations have been changed and what setting has been changed.

The main function is to back up configurations from Intune to a Git repository from a DEV environment and if any configurations has changed, push them to PROD Intune environment.

The package can also be run standalone outside a pipeline, or in one to only backup data. Since 1.0.4, configurations are also created if they cannot be found. This means this tool could be used in a tenant to tenant migration scenario as well.

# Exiting news 📣

The front end for IntuneCD has now been released. Check it out [here](https://github.com/almenscorner/intunecd-monitor)

## What's new in 1.1.2

- Added new exclusions for backup and update, it's now possible to exclude certain configurations from backup and update.

- Example to exclude in backup: `IntuneCD-startbackup -e assignments AppConfigurations Profiles`
- Example to exclude in update: `IntuneCD-startupdate -e AppConfigurations Profiles`

- Added capabilities to update the IntuneCD frontend with data

- Once the frontend is available all that will be needed to update with data is to add `-f <frontend_url>` to startbackup and startupdate command and set the API key in ENV variables.

- Added ability to configure title, intro, tenant and updated lines in the documentation using a JSON string, example:

- `-j "{\"title\": \"demo\", \"intro\": \"demo\", \"tenant\": \"demo\", \"updated\": \"demo\"}"`

- Added unit tests
- Changed deprecated OptionParser to ArgumentParser
- Improved the documentation
- Improved overall code readability

## What's new in 1.1.1

- Added ability to split documentation into categories using `-s Y` in `intunecd-startdocumentation`
- Added ability to set max length of output in documentation using `-m {int_value}` in `intunecd-startdocumentation`
- Added backup and documentation of Group Policy Configurations
- Added retry if 503 is encountered during a graph call

## What's new in 1.1.0

- Bugfix for App Protection policies not being able to be created in a tenant to tenant scenario
- Bugfix for Configuration Profiles not being able to update assignment in a tenant to tenant scenario
- Bugfix for Windows Autopilot profiles not being able to update assignment in a tenant to tenant scenario
- Bugfix for assignment updates where updating assignments when creating new configurations were not possible if the group does not exist

## Install this package

    pip install IntuneCD
    

## Update this package

    pip install IntuneCD --upgrade
    

## What is backed up, updated, created and documented?
PayloadBack upUpdateDocumentCreateNotesApple Push Notification🎉🎉Apple Volume Purchase Program tokens🎉🎉Application Configuration Policies🎉🎉🎉🎉Application Protection Policies🎉🎉🎉🎉Applications🎉🎉Compliance Policies🎉🎉🎉🎉Device Configurations🎉🎉🎉🎉For custom macOS and iOS configurations,
mobileconfigs are backed upGroup Policy Configurations🎉🎉Enrollment profiles🎉 [[1]](#fn1)🎉🎉🎉 [[2]](#fn2)Endpoint Security🎉🎉🎉🎉Security Baselines
Antivirus
Disk Encryption
Firewall
Endpoint Detection and Response
Attack Surface Reduction
Account ProtectionFilters🎉🎉🎉🎉Managed Google Play🎉🎉Notification Templates🎉🎉🎉🎉Proactive Remediations🎉🎉🎉🎉Partner Connections🎉🎉Compliance
Management
Remote AssistanceShell Scripts🎉🎉🎉🎉Powershell Scripts🎉🎉🎉🎉Settings Catalog Policies🎉🎉🎉🎉
## Required Azure AD application Graph API permissions

- DeviceManagementApps.ReadWrite.All
- DeviceManagementConfiguration.ReadWrite.All
- DeviceManagementServiceConfig.ReadWrite.All
- Group.Read.All

If you just want to back up you can get away with only Read permission (except for DeviceManagementConfiguration)!

## How do I use it?

You have two options, using a pipeline or running it locally. Let's have a look at both.

## Parameters

To see which parameters you have to provide just type: IntuneCD-startbackup --help, IntuneCD-startupdate --help or IntuneCD-startdocumentation --help

Example options:

- -h, --help  show this help message and exit
- -o OUTPUT, --output=OUTPUT

- The format backups will be saved as, valid options are

json or yaml. Default is json

- -p PATH, --path=PATH

- The path to which the configurations will be saved.

Default value is $(Build.SourcesDirectory)

- -m MODE, --mode=MODE

- The mode in which the script is run, 0 = devtoprod

(backup from dev -> update to prod) uses os.environ

DEV_TENANT_NAME, DEV_CLIENT_ID, DEV_CLIENT_SECRET, 1 =

standalone (backup from prod) uses os.environ

TENANT_NAME, CLIENT_ID, CLIENT_SECRET

- -a LOCALAUTH, --localauth=LOCALAUTH

- When this paramater is set, provide a path to a local

json file containing the following keys:

params:TENANT_NAME, CLIENT_ID, CLIENT_SECRET when run

in standalone mode and params:DEV_TENANT_NAME,

DEV_CLIENT_ID, DEV_CLIENT_SECRET when run in devtoprod

For IntuneCD-startupdate 1.0.4 the -u parameter has been added which, if set, updates assignments for existing configurations. Again the groups are matched with displayName, so they must be the same in both tenants.

### Run locally

First install the package using pip, then you must create a json which contains authentication parameters in the following format:

    {
        "params":{
            "TENANT_NAME": "",
            "CLIENT_ID": "",
            "CLIENT_SECRET": ""
        }
    }
    

When you have created the json, you can now run these commands

    IntuneCD-startbackup -m 1 -o yaml -p /path/to/save/in -a /path/to/auth.json/
    

If you run without the -m parameter, make sure you have one auth.json pointing to DEV and another pointing to PROD, example:

    {
        "params":{
            "DEV_TENANT_NAME": "",
            "DEV_CLIENT_ID": "",
            "DEV_CLIENT_SECRET": ""
        }
    }
    

    {
        "params":{
            "PROD_TENANT_NAME": "",
            "PROD_CLIENT_ID": "",
            "PROD_CLIENT_SECRET": ""
        }
    }
    

    IntuneCD-startbackup -o yaml -p /path/to/save/in -a /path/to/auth_DEV.json/
    

    IntuneCD-startupdate -p /path/to/save/in -a /path/to/auth_PROD.json/
    

### Run from a pipeline

I have tested this with Azure DevOps which is what I will give an example to. But it could just as well be run using GitHub Actions.

In the example pipeline below I'm running with the parameters -m 1 (standalone mode) and -o yaml (output configurations in yaml format). If you are running this in DEV -> PROD mode, remove -m and add DEV_ in front of all env: variables except for REPO_DIR. CLIENT_SECRET should be added as a secret variable.

DEV env variables:

      env:
        REPO_DIR: $(REPO_DIR)
        DEV_TENANT_NAME: $(TENANT_NAME)
        DEV_CLIENT_ID: $(CLIENT_ID)
        DEV_CLIENT_SECRET: $(CLIENT_SECRET)
    

**Example backup pipeline:**

    pool:
      vmImage: ubuntu-latest
    
    variables:
      REPO_DIR: $(Build.SourcesDirectory)
      TENANT_NAME: example.onmicrosoft.com
      CLIENT_ID: xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx
    
    steps:
    
    - checkout: self
      persistCredentials: true
    
    - script: pip3 install IntuneCD
      displayName: Install IntuneCD
    
    - script: |
          git config --global user.name "devopspipeline"
          git config --global user.email "devopspipeline@azuredevops.local"
      displayName: Configure Git
    
    - script: IntuneCD-startbackup -m 1 -o yaml
      env:
        REPO_DIR: $(REPO_DIR)
        TENANT_NAME: $(TENANT_NAME)
        CLIENT_ID: $(CLIENT_ID)
        CLIENT_SECRET: $(CLIENT_SECRET)
      displayName: Run IntuneCD backup
    
    - script: |
        cd $(REPO_DIR)
        git add --all
        git commit -m "Updated configurations"
        git push origin HEAD:main
      displayName: Commit changes
    

The following shows a pipeline which updates configurations in Intune. Again I'm running with -m 1. If this should update PROD, add PROD_ in front of all env: variables except REPO_DIR. CLIENT_SECRET should be added as a secret variable.

PROD env variables:

      env:
        REPO_DIR: $(REPO_DIR)
        PROD_TENANT_NAME: $(TENANT_NAME)
        PROD_CLIENT_ID: $(CLIENT_ID)
        PROD_CLIENT_SECRET: $(CLIENT_SECRET)
    

**Example update pipeline:**

    pool:
      vmImage: ubuntu-latest
    
    variables:
      REPO_DIR: $(Build.SourcesDirectory)
      TENANT_NAME: example.onmicrosoft.com
      CLIENT_ID: xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx
    
    steps:
    
    - checkout: self
      persistCredentials: true
    
    - script: pip3 install IntuneCD
      displayName: Install IntuneCD
    
    - script: IntuneCD-startupdate -m 1
      env:
        REPO_DIR: $(REPO_DIR)
        TENANT_NAME: $(TENANT_NAME)
        CLIENT_ID: $(CLIENT_ID)
        CLIENT_SECRET: $(CLIENT_SECRET)
      displayName: Run update
    

## Run documentation locally

To create a Markdown document from the backup files, run this command

    IntuneCD-startdocumentation -p /path/to/backup/directory -o /path/to/create/markdown.md -t nameoftenant -i 'This is a demo introduction'
    

## Run documentation in a pipeline

This step should be added to the backup pipeline to make sure the markdown document is updated when configurations changes. By default it writes to the README.md file in the repo, you can change this with the -o option

    - script: IntuneCD-startdocumentation -t $(TENANT_NAME) -i 'This is a demo introduction'
      env:
        REPO_DIR: $(REPO_DIR)
      displayName: Run IntuneCD documentation
    

## Good to know

When this tool tries to update configurations, it matches the display name. Therefore, the display name from DEV must match in PROD.

## Current known limitations

Updating Windows Update Rings configurations is currently not supported, the tool can however create update rings if they don't exist.

---

1. 
Only Apple Business Manager and Windows Autopilot profiles are backed up. [↩︎](#fnref1)

2. 
Only Windows Autopilot profiles are created. [↩︎](#fnref2)
