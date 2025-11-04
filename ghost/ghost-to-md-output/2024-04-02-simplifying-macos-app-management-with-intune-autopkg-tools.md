---
title: Simplifying macOS App Management with Intune AutoPkg Tools
slug: simplifying-macos-app-management-with-intune-autopkg-tools
date_published: 2024-04-02T06:44:14.000Z
date_updated: 2024-04-02T07:12:37.000Z
tags: macOS, AutoPkg, Python, automation, Azure DevOps, GitHub
excerpt: In today's fast-paced digital landscape, efficient management of applications is crucial for ensuring the smooth operation of organizations. However, alongside this efficiency, security remains paramount
---

In today's fast-paced digital landscape, efficient management of applications is crucial for ensuring the smooth operation of organizations. However, alongside this efficiency, security remains paramount. As technology evolves, so do the methods employed by malicious actors to exploit vulnerabilities in applications and systems.

Implementing robust security measures is essential to safeguard sensitive data, protect against cyber threats, and maintain the trust of customers and stakeholders. This includes regularly updating software.

Microsoft Intune offers a platform for managing macOS apps, but the process of creating, updating, promoting, and deleting these apps can be time-consuming and complex. However, with the Intune AutoPkg Tools repository, managing macOS apps in Intune becomes a breeze.

**Automating Intune App Management**

The [Intune AutoPkg Tools repository](https://github.com/almenscorner/intune-autopkg-tools) houses a script that acts as a wrapper around AutoPkg, a popular tool for automating the packaging and deployment of macOS applications. This script streamlines the entire process of managing macOS apps in Intune, from creation to deletion, using the Intune Uploader set of AutoPkg custom processors.

**Getting Started**

For those new to Intune Uploader, the [repository's wiki](https://github.com/almenscorner/intune-uploader/wiki) provides comprehensive guidance on getting started with the processor.

**How It Works**

The heart of the Intune AutoPkg Tools repository lies in its script, `autopkg_tools.py`, which orchestrates the entire process. This script utilizes a list of overrides for applications to manage, this repository contains two examples, one for Firefox and one for UTM. Adding additional overrides for other apps you want to manage is as easy as creating a new override using AutoPkg.

The repository includes three key JSON files:

1. `autopkg_tools/recipe_list.json` - List of recipes to run
2. `autopkg_tools/promote_list.json` - List of recipes to promote to groups in stages, see [here](https://github.com/almenscorner/intune-uploader/wiki/IntuneAppPromoter) for more information on how to configure this.
3. `autopkg_tools/cleanup_list.json` - List of recipes to delete old versions in Intune, when adding a new version, configure the `keep` value to the number of versions to keep. If not configured, the default is 3.

**Integration with CI/CD Pipelines**

Whether you prefer GitHub Actions or Azure Pipelines, integrating Intune AutoPkg Tools into your CI/CD workflow is simple and straightforward.

In this post the focus is on GitHub Actions and Azure Pipelines, however, this script can be run on any macOS agent.

**Teams notifications**

If you configure a webhook to a Teams channel, status messages from the CI/CD run will be sent to Teams so you can get information about when a application was updated, promoted or removed.

![](__GHOST_URL__/content/images/2024/03/import.png)

![](__GHOST_URL__/content/images/2024/03/promote.png)

![](__GHOST_URL__/content/images/2024/03/remove.png)

![](__GHOST_URL__/content/images/2024/03/error.png)

Teams notifications

### **Preparation: **

**Clone the repo**

1. Open your terminal or command prompt.
2. Navigate to the directory where you want to clone the repository.
3. Use the `git clone` command followed by the URL of the Intune AutoPkg Tools URL:

    git clone https://github.com/almenscorner/intune-autopkg-tools
    

**Create Teams webhook (optional)**

1. Navigate to the channel where you want to add the webhook.
2. Click on the ellipsis (...) next to the channel name and Select "Manage Channel."
3. Click "Edit" on Connectors.
4. In the connectors list, find "Incoming Webhook" and click "Configure."
5. Give your webhook a name and (optionally) upload an image to represent it.
6. Click "Create" or "Add."
7. After the webhook is created, you'll be provided with a unique URL. This URL is what you'll use to send messages to the channel. Save this URL for later use.
8. You can configure additional options like specifying a custom name for the bot posting messages, choosing an icon, etc. This depends on your requirements.
9. Once you've copied the webhook URL and configured any additional options, save your settings.

### **For GitHub Actions:**

**Step 1: Create a New Private Repository**

1. Log in to your GitHub account.
2. In the upper-right corner of any page, click the "+" icon, then select "New repository" from the dropdown menu.
3. On the "Create a new repository" page, enter a name for your repository. Optionally, add a description for your repository.
4. Select "Private" as the visibility level for your repository.
5. Choose whether to initialize the repository with a README file, add a .gitignore, and choose a license. This is optional.
6. Click the "Create repository" button.

**Step 2: Clone Your Private Repository**

1. Once your private repository is created, click the "Code" button to get the repository's URL.
2. In your terminal or command prompt, navigate to the directory where you want to clone your repository.
3. Use the following command to clone your repository, replace `<repository_URL>` with the URL of your private repository:

    git clone <repository_URL>
    

1. Copy the contents of the cloned `intune-autopkg-tools` repository to your repository.

**Step 3: Add Secret Action Variables**

1. Go to the "Settings" tab of your repository.
2. In the left sidebar, click on "Secrets and variables" and then "Actions".
3. Click on the "New repository secret" button.
4. Add the following secrets to your repository:
1. `CLIENT_SECRET`
2. `TEAMS_WEBHOOK` (optional)

![](__GHOST_URL__/content/images/2024/03/actionsecrets.png)GitHub Action secrets
**Step 4: Update the action variables**

1. Open up the `autopkg.yml` file in a text editor and update variables `TENANT_ID` and `CLIENT_ID` in the `.github/workflows/autopkg.yml` file to match your Entra ID settings.

![](__GHOST_URL__/content/images/2024/03/Screenshot-2024-03-30-at-12.08.37.png)GitHub Actions variables
**Step 5: Commit and Push Changes**

1. Use the following commands to commit and push your changes to your private repository:

    git add .
    git commit -m "initial commit"
    git push
    

That's it! You've successfully created a private repository on GitHub and added secret action variables for use in your GitHub Actions workflows.

**Step 6: Update or add overrides**

1. As mentioned there are two overrides provided as an example, Firefox and UTM. Firefox contains information to use the promotion feature, either add group IDs and modify as needed or remove the promotion information from the override as well as the `promote_list.json`. If you have any other applications you want to include, create and add overrides for these. Firefox is also added to `cleanup_list.json` to remove old versions as updates are uploaded.

### **For Azure Pipelines:**

**Step 1: Create a new project**

1. In the "Projects" section, locate and click on the "New project" button. It's usually positioned in the top-right corner of the screen.
2. **Enter Project Details:**
- **Project Name:** Enter a name for your project. Choose a descriptive name that reflects the purpose or nature of your project.
- **Visibility:** Choose Private
- **Description (Optional):** Optionally, you can add a description to provide more context about your project.

3. Click the "Create" button to create the project.

**Step 2: Clone Your Private Repository**

1. Once your private repository is created, click on it to open it.
2. Click on the "Clone" button to copy the repository URL.
3. In your terminal or command prompt, navigate to the directory where you want to clone your repository.
4. Use the following command to clone your repository, replace `<repository_URL>` with the URL of your private repository:

    git clone <repository_URL>
    

1. Copy the contents of the cloned `intune-autopkg-tools` repository to your repository.

**Step 3: Update the pipeline variables**

1. Open up the `autopkg-azure.yml` file in either a text editor and update variables `TENANT_ID` and `CLIENT_ID`  in the `autopkg-azure.yml` file to match your Entra ID settings.

![](__GHOST_URL__/content/images/2024/03/Screenshot-2024-03-30-at-12.07.03.png)Pipeline variables
**Step 5: Commit and Push Changes**

1. Use the following commands to commit and push your changes to your private repository:

    git add .
    git commit -m "initial commit"
    git push
    

**Step 6: Create the Pipeline**

1. Click on the "Pipelines" menu item in the left sidebar.
2. In the Pipelines page, click on the "New pipeline" button. This button is usually located at the top right corner of the page.
3. Azure DevOps will prompt you to choose the location of your source code. Select the Azure Repos Git.
4. Azure DevOps will analyze your repository and provide you with options for pipeline configuration. Since you want to create a pipeline from an existing file, select the option that says "Existing Azure Pipelines YAML file."
1. Click on the "Continue" button.

5. In the next screen, Azure DevOps will ask you to specify the path to your existing pipeline YAML file. Click on the folder icon to browse for the file and select `autopkg-azure.yml`.
1. Once you've selected the file, click on the "Continue" button.

6. Click on the Variables button in the top right and add the following Secret Variables:
1. `CLIENT_SECRET`
2. `TEAMS_WEBHOOK` (optional)

7. Review and save your pipeline.

![](__GHOST_URL__/content/images/2024/03/Screenshot-2024-03-30-at-11.59.58.png)
That's it! You've successfully created a private repository on Azure DevOps and added secret variables for use in your Pipeline.

**Step 7: Update or add overrides**

1. As mentioned there are two overrides provided as an example, Firefox and UTM. Firefox contains information to use the promotion feature, either add group IDs and modify as needed or remove the promotion information from the override as well as the `promote_list.json`. If you have any other applications you want to include, create and add overrides for these. Firefox is also added to `cleanup_list.json` to remove old versions as updates are uploaded.

**Logging**

Each run from a GitHub action or Azure Pipeline will create an artifact with the logs as a zip. To review the logs you can just download the artifacts, unzip and review. If you require more verbose output for troubleshooting, add `--debug` to the parameters when executing the script in either `autopkg.yml` or `autopkg-azure.yml`.
![](__GHOST_URL__/content/images/2024/03/artifact.png)GitHub artifacts![](__GHOST_URL__/content/images/2024/03/Capture-2024-03-31-103804.png)Azure DevOps artifacts

**Conclusion**

Follow the steps above, you should now have a complete setup to run AutoPkg in a CI/CD workflow and automatically update and maintain your macOS apps in Intune.

The Intune AutoPkg Tools repository simplifies macOS app management in Intune, providing efficient automation for packaging, updating, promoting, and deleting apps. This automation streamlines workflows, allowing you to prioritize delivering value to your users.

0:00

                            /0:23
1×

Intune AutoPkg tools run
