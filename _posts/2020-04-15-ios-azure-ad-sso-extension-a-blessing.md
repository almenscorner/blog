---
title: iOS Azure AD SSO Extension
slug: ios-azure-ad-sso-extension-a-blessing
date_published: 2020-04-15 10:04:00.000000000 Z
date_updated: 2020-08-27 10:17:54.000000000 Z
tags:
- azure ad
- intune
- ios
- mem
- sso
- sso extension
---
I jumped up and down with excitment when I saw this one a couple of weeks ago in the release notes!

With the 2003 service release Microsoft released the first version of iOS Azure AD SSO Extension. I am very happy to see Microsoft taking the lead here and implements this. Right now, MEM is the only platform that has their own SSO Extension for iOS. And let&#8217;s be honest, SSO has not worked all that well for iOS devices managed in MEM. It works great for Office apps that share a key, yes, but for third party apps that rely on SAML/Oauth the experience has not been the same.

Apple released this new function with iOS 13 and macOS Catalina to make the end user experience even more seamless and password less. Too many methods and places exist to manage authentication including, OpenID, SAML, Oauth, WS-Fed, Kerberos etc. With SSO Extension a range of benefits opens up:

- Native apps and Safari authentication
- Uses your existing enterprise account
- It requires management by MDM
- The UI can be web, native or non-existent

SSO Extension provides:

- Native user interface
- Multifactor
- SEP generated keys
- Trust score data
- idP specific features
- Federated auth
- WebAuthN

There are two types of  SSO Extensions, Redirect and Credential. Microsoft chose to developed a redirect extension which makes sense as it&#8217;s designed for modern authentication methods such as OpenID Connect, Oauth and SAML. This of course plays nice with AzureAD as our idP. Luckily, Microsoft went down the road of using Authenticator as their host app, this means that you probably already have the host on users devices and logged in. Only a profile to activate the function is needed.

## How it works

Let&#8217;s have a look at how this works for a webpage in safari.
![]({{ site.baseurl }}/assets/images/wordpress/2020/04/image-1024x210.png)Safari authentication flow
1. Instead of seeing the login page, the OS redirects the request to the SSO Extension. The Extension receives the URL, headers and body.
2. SSO Extension completes authentication with idP
3. URL Response sent to Safari

Finally, we will probably see SSO to Microsoft 365 in Safari on iOS folks 🙂

But it&#8217;s not just Safari sessions that can take advantage of this. Native apps is supported as well. Let&#8217;s have a look.
![]({{ site.baseurl }}/assets/images/wordpress/2020/04/appsso-1024x216.png)Native app authentication flow
1. App sends a command to the SSO Extension such as &#8220;Login&#8221;
2. SSO Extension completes authentication with idP
3. URL Response + tokens returned to the app

The beauty here is that developers can use SSO Extension as a authentication library which means they do not have to implement different library&#8217;s in the application and maintain them. That&#8217;s one of the big reasons we might see a whole lot more SSO support coming to native apps on iOS.

## Now and the future

In first release of the Azure AD SSO Extension we do have some limitations:

- Apps that use the MSAL v1.1.0 library
- AAD registered devices

So right now the app must have MSAL v1.1.0 library integrated.

What I expect to see in the future:

- Support for Safari web SSO
- Support for ASWebAuthenticationSession
- OpenIDConenct, OAuth and SAML support
- macOS extension
- Support for WKWebview (maybe)

There&#8217;s not a lot details on the plans for future versions but this is what I want to see to make the extension more useful than requiring the integration off MSAL to accommodate for  the vast majority of apps used in the enterprise.

## Start testing

It&#8217;s easy to set this up in MEM if you want to test it. A few prerequisites

- Microsoft Authenticator needs to be installed on the device
- The device must be running iOS 13 or later
- Device must be MDM enrolled

![]({{ site.baseurl }}/assets/images/wordpress/2020/04/createprofile-1024x524.png)
To configure, go to **Devices -> Device Configurations**, click **Create Profile**, choose **iOS/iPadOS** and **Device Features** for profile type and then click **create**.
![]({{ site.baseurl }}/assets/images/wordpress/2020/04/createprofile2-1024x747.png)
Give the profile a name, click on **Single sign on app extension**, choose **Redirect **as type, enter the following Extension ID:

**com.microsoft.azureauthenticator.ssoextension.**

Configure the following URLs:

- [https://login.microsoftonline.com](https://login.microsoftonline.com)
- [https://login.windows.net](https://login.windows.net)
- [https://login.microsoft.com](https://login.microsoft.com)
- [https://sts.windows.net](https://sts.windows.net)
- [https://login.partner.microsoftonline.cn](https://login.partner.microsoftonline.cn)
- [https://login.chinacloudapi.cn](https://login.chinacloudapi.cn)
- [https://login.microsoftonline.de](https://login.microsoftonline.de)
- [https://login.microsoftonline.us](https://login.microsoftonline.us)
- [https://login.usgovcloudapi.net](https://login.usgovcloudapi.net)
- [https://login-us.microsoftonline.com](https://login-us.microsoftonline.com)

Save the profile and assign to devices.

Hope this post was useful and informative, feel free to drop me a comment 😊
