---
title: Secure messages to Teams with Azure Logic App
slug: secure-messages-to-teams
date_published: 2022-01-10 19:42:42.000000000 Z
date_updated: 2022-01-11 07:24:41.000000000 Z
tags:
- azure
- azure devops
- logic app
- python
excerpt: Today we're setting up an Azure Logic App and configure authorization policies
  to require an OAuth token to send a message to a Teams channel
---
In the last couple of posts where we have used Azure DevOps Pipelines to [add new packages for macOS](__GHOST_URL__/) and [move them between catalogs](__GHOST_URL__/), you were able to setup an incoming Teams webhook to get notified when something happens.

Not all organizations allow use of these webhooks though since anybody with the link can send a message to the channel without the need of authenticating. In this post we'll address this by setting up an Azure Logic App and configure authorization policies to require an OAuth token to send the message.

As mentioned, we're going to use OAuth in this post. Other options you can use to secure your logic app include,

- IP based restriction
- Restrict External Guest User Access
- Expose Logic App as API
- Restrict permitted Enterprise Application Users and Groups and Conditional Access policies.
- Scopes and Roles Authorization in Logic Apps.
- Logic Apps and APIM (Azure API Management).

Configuring this will require some changes to the scripts I provided in previous posts as well as some new variables in the pipelines. We will walk through what needs to be changed and how to do it.

### Create the Logic App

Let's start with creating the Logic App!

- Head over to the [Azure Portal](https://portal.azure.com) and navigate to **Logic Apps**
- Click **Add** in the top left corner
- Choose your **Subscription**, **Resource group (or create new)**, for type, choose **Consumption**, provide a **Name**, choose your **region**, and then click **Review + Create**

![]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-22-at-13.44.26.png)
Once the deployment for the logic app is complete, open the logic app and navigate to **Authorization** in the left menu. Here, we are going to configure the claims we'll use. Add claims and configure them like below,

- **Issuer:**[https://sts.windows.net/](https://sts.windows.net/){YOUR AZURE AD TENANT ID}/
- **Audience:**[https://management.core.windows.net/](https://management.core.windows.net/)

We are also adding one custom claim, appid, this will ensure that only tokens from a specific app in Azure AD are accepted. To configure, click **Add custom claim** and configure it like below,

- **appid:** {YOUR CLIENT ID}

If you don't have an app registration, create a new app in Azure AD and copy the Client ID, add a new secret, and save it for later.
![]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-22-at-13.54.01-1.png)
Now we are ready to configure our flow, navigate to **overview** on the app and under **Start with a common trigger **select **When a HTTP request is received**. Leave this step with default configurations and click **+ New step **and choose **Parse JSON**.

- For Content, choose **Body** from the HTTP request step
- For Schema, configure the following,

    {
        "properties": {
            "message": {
                "type": "string"
            },
            "title": {
                "type": "string"
            },
            "type": {
                "type": "string"
            }
        },
        "type": "object"
    }
    

![]({{ site.baseurl }}/assets/images/2021/12/Screenshot-2021-12-22-at-14.06.24.png)
Click **+ New step **again and choose **Post Adaptive card in a chat or channel**. You will have to sign in to create the connection to Teams, if you don't want your name on every message, use a service account. Once signed in, configure the step like below,

- **Post as:** Flow bot
- **Post in:** Channel
- **Team:** {YOUR TEAM}
- **Channel:** {YOUR CHANNEL}

For **Adaptive Card**, copy and paste in the JSON below,

    {
    
        "type": "AdaptiveCard",
      
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      
        "version": "1.2",
      
        "body": [
      
          {
      
            "type": "TextBlock",
      
            "text": "@{body('Parse_JSON')?['type']}",
      
            "wrap": true,
      
            "size": "large"
      
          },
      
      {
      
         "type": "ColumnSet",
      
         "columns": [
      
          {
      
           "type": "Column",
      
           "width" : "250px",
      
           "items": [
      
            {
      
              "type": "TextBlock",
      
              "text": "@{body('Parse_JSON')?['title']}",
      
              "wrap": true,
      
              "isSubtle": true
      
            },
      
            {
      
              "type": "TextBlock",
              
              "text": "@{body('Parse_JSON')?['message']}",
              
              "wrap": true
      
            }
      
           ]
      
          }
      
         ]
      
        }
      
       ]
      
      }
    

Once you have saved the flow, copy the **HTTP POST URL** from the first step in the flow and remove everything after **?api-version=2016-10-01**. Save this URL since we'll need it later

To further protect our Logic App, we're going to make sure the app cannot be triggered without a authorization header. To do so, open the **When a HTTP request is received step**, click the three dots at the top right and click **settings**. Scroll down an enter the following under **Trigger Conditions**, `@startsWith(triggerOutputs()?['headers']?['Authorization'], 'Bearer' )`. Click **</ > Code View** at the top menu bar and enter `"operationOptions": "IncludeAuthorizationHeadersInOutputs",` under **triggers**
![codeview]({{ site.baseurl }}/assets/images/2022/01/codeview.png)

### Prepare Azure DevOps Pipelines

The bulk of the work is now done, all we must do now is to edit the autopkg_tools.py and autopromote.py scripts, add a module and get some secrets into DevOps and variables into the pipeline.

First up, copy the below script and save it as **logic_app_message.py** in the root of the AutoPkg and/or the autopromote repo,

    import os
    import requests
    import json
    from adal import AuthenticationContext
    
    LOGIC_APP_URL = os.getenv("LOGIC_APP_URL")
    CLIENT_ID = os.getenv("CLIENT_ID")
    CLIENT_SECRET = os.getenv("CLIENT_SECRET")
    TENANT_ID = os.getenv("TENANT_ID")
    
    def obtain_accesstoken(tenantid,clientid,clientsecret):
    
        authority_url = 'https://login.microsoftonline.com/'+tenantid
        context = AuthenticationContext(authority_url)
        token = context.acquire_token_with_client_credentials(
            resource='https://management.core.windows.net/',
            client_id=clientid,
            client_secret=clientsecret,
        )
        return token
    
    def trigger_logic_app(source,type,title,message):
    
        data = json.dumps(
            {
                "source": source,
                "type": type,
                "title": title,
                "message": message
            }
        )
    
        token = obtain_accesstoken(TENANT_ID,CLIENT_ID,CLIENT_SECRET)
    
        headers = {'Content-Type':'application/json', \
            'Authorization':'Bearer {0}'.format(token['accessToken'])}
    
        response = requests.post(url=LOGIC_APP_URL,data=data,headers=headers)
    
        if response.status_code != 202:
            raise ValueError(
                "Request to Teams returned an error %s, the response is:\n%s"
                % (response.status_code, response.text)
            )
    

![Screenshot-2022-01-10-at-15.52.48]({{ site.baseurl }}/assets/images/2022/01/Screenshot-2022-01-10-at-15.52.48.png)

You can find updated scripts prepared for using a Logic app for AutoPkg [here](https://github.com/almenscorner/AutoPkgAzureDevOps/blob/main/autopkg_tools_logicapp.py) and autopromote [here](https://github.com/almenscorner/autopromote-AzureDevOps/blob/main/autopromote_logicapp.py), just replace the script content in your repository.

Last thing before we move on to the pipelines is to add **"adal"** in the **requirements.txt** file. To do this, just open requirements.txt and add adal in the list.

Once you have edited the scripts it's now time to update the pipeline(s) with required variables. Add the following variables in the YAML file,

      CLIENT_ID: "{YOUR CLIENT ID}"
      TENANT_ID: "{YOUR TENANT ID}"
      LOGIC_APP_URL: "{YOUR LOGIC APP URL}"
    

In addition to these, add the client secret as a secret variable by,

- Browsing to **Pipelines** and click the three dots at the end of your pipeline and choose **Edit**
- Click **Variables** at the top right, then click **+**
- Provide the name **CLIENT_SECRET**, paste your secret in the **Value** field, tick the box **Keep this value secret** and then click **Ok**

Now edit the pipeline(s) and change the variable "TEAMS_WEBHOOK" to "CLIENT_SECRET", it's to be edited in the script run step in the pipeline. The example below shows how it should look for the AutoPkg pipeline,

    - script: python3 $(REPO_DIR)/autopkg_tools.py -l $(REPO_DIR)/recipe_list.json
      displayName: Run AutoPkg
      condition: eq(variables.OVERRIDESEXISTS, true)
      env:
        CLIENT_SECRET: $(CLIENT_SECRET)
    

Once the steps above have been completed, the scripts now use the logic app to notify Teams,
![Screenshot-2022-01-10-at-17.42.15]({{ site.baseurl }}/assets/images/2022/01/Screenshot-2022-01-10-at-17.42.15.png)

![Screenshot-2022-01-10-at-17.42.28]({{ site.baseurl }}/assets/images/2022/01/Screenshot-2022-01-10-at-17.42.28.png)
