---
title: The binding of macs
slug: the-ad-binding-of-macs
date_published: 1970-01-01T00:00:00.000Z
date_updated: 2021-08-20T19:35:17.000Z
draft: true
---

Lately I've had many discussions regarding binding of macs to AD. There seems to be a confusion on what options we have available and when to use a specific product or option to achieve the goal and whether you should bind to AD or not. It's a hot topic and it's not always easy to say what's the better choice.

Let's make one thing clear, there is no one size fits all. Every organization have their specific needs and you might very well have a good reason for binding your macs to AD. For example, you might have requirements for Wi-Fi authentication to only allow domain joined devices or file shares that require bound devices.

If you on the other hand are starting to move away from an on-premise architecture and move data to the cloud, you're better of using a solution where passwords for the local accounts on macs are synced with your IdP.

You can even use these sync type of solutions if you are binding to AD but use local accounts in favor of mobile. 

In almost all cases these are architectural discussions which you can solve by using an MDM solution to manage your macs. All popular MDM vendors supports integrating to your ADCS infrastructure to issue certificates for Wi-Fi, VPN or what have you. 

## Options for local accounts

So what options do we have for local accounts?

### Kerberos SSO Extension

Since macOS Catalina, we've had the option to use the Kerberos SSO Extension built in to macOS to be able to cater for macs that are not bound to AD or use local accounts. Before this, Apple had something called Enterprise Connect, a paid tool that let you get Kerberos tickets and sync passwords from AD to local accounts. This is now built in and free to use. With this extension you are able to issue Kerberos tickets and sync AD password to the local account.

**Requirements:**

- 
An Active Directory domain running Windows Server 2008 or later. The Kerberos SSO extension isn’t intended for use with Azure Active Directory. It requires a traditional on-premise Active Directory domain.

- 
Access to the network where the Active Directory domain is hosted. This network access can be through Wi-Fi, Ethernet or VPN.

- 
Devices managed with a mobile device management (MDM) solution with support for the Extensible Single Sign-on (SSO) configuration profile payload. Contact your MDM vendor to ask about their support for this configuration profile payload.

As you can see from the requirements above, this option works when you have access to the corporate network when setting up the macs.

### Jamf Connect

Formerly known as NoMAD, Jamf Connect is a tool that accomplishes much of the same tasks as the Kerberos SSO Extension but with a major difference, you can use Jamf Connect to sync passwords to local accounts with IdPs other than an on-premise AD. If you for example want to sync to Okta or AAD instead, this is possible using Jamf Connect.

I prefer implementing Jamf Connect as you don't have the restrictions of needing access to the corporate network in order to get the mac setup. Because you sync with a cloud IdP, the mac can enroll and be ready for use anywhere. This is especially important in these times of remote work.

While this is a paid product compared to Kerberos SSO Extension, I still see this as the way forward as most organizations I talk to increase their investment in cloud technologies and remote work.

NoMAD is still an option if you want to use an open source tool for your local accounts needs and Jamf will continue to support this.

## Summary

When the tools we discussed above are paired with managing mac using MDM, you are in a situation where binding to AD may not even be needed. Again, you can still have a reason to bind, but you definitely should think about what moving away from it would mean for the user experience and support.

Are you using a cloud IdP such as Okta or AAD? use Jamf Connect

Are you using on-premise AD without any federation? use Kerberos SSO Extension
