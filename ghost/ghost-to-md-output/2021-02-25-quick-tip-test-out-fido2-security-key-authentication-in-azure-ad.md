---
title: "Quick tip: Test out FIDO2 security key authentication in Azure AD"
slug: quick-tip-test-out-fido2-security-key-authentication-in-azure-ad
date_published: 2021-02-25T15:10:35.000Z
date_updated: 2021-02-25T17:00:32.000Z
tags: AzureAD, MFA, Security
---

Are you struggling to get MFA rolled out? are your users refusing to register their personal devices to receive work related notifications? then a FIDO2 security key might be the answer for you.

Everybody should be using MFA these days, but some are having a harder time pushing it through as not everybody is supplying their staff with a company owned device that can receive the MFA request. I know that some users are extremely hesitant to register a personal device for this, and some that don&#8217;t think it&#8217;s a big deal.

For the users that do not want to register their personal device, it&#8217;s possible to instead hand out a FIDO2 security key. This quick tip walks through configuring Azure AD for FIDO2 authentication as well as the user experience signing in with this type of key.

**Note:** This is a **preview** in Azure AD, it is not recommended to roll this out in production

I will be using this little guy which is a key from Yubico:
![](__GHOST_URL__/content/images/wordpress/2021/02/yubikey-1.jpg)
## Configure FIDO2 in Azure AD

### Requirements

- [Azure AD Multi-Factor Authentication](https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted)
- Enable [Combined security information registration preview](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-registration-mfa-sspr-combined) (enabled in new tenants)
- Compatible [FIDO2 security keys](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless#fido2-security-keys)
- WebAuthN requires Windows 10 version 1903 or higher**

To use security keys for logging in to web apps and services, you must have a browser that supports the WebAuthN protocol. These include Microsoft Edge, Chrome, Firefox, and Safari.

### Configuring Azure AD

1. Sign into [Azure AD portal](https://portal.azure.com)

2. Go to **Azure AD -> Security -> Authentication Methods**

3. Under policies you see **FIDO2 Security Key**, click on it

4. Choose to **enable**, as I will be portraying the user today, I will target my user by clicking **Select users**, then **Save** the policy
![](__GHOST_URL__/content/images/wordpress/2021/02/fi1-1024x295.png)
## User Registration of the FIDO2 key

Now that we have FIDO2 Security Keys enabled, I must register it to my account. If you already have an Azure AD MFA method registered you can immediately register the FIDO2 key, if not you must add one.

1. Sign in to https://myprofile.microsoft.com/

2. Click on **Update Info** on the Security Info card
![](__GHOST_URL__/content/images/wordpress/2021/02/fi2-1024x300.png)
3. Click on **Add Method**, then choose **Security Key** and then **Add**
![](__GHOST_URL__/content/images/wordpress/2021/02/fi3-1024x467.png)
4. Choose the type of key you have, **USB** or **NFC** and then click **Nex**

5. When asked, plug in your FIDO2 Security Key

6. Enter a pin-code and give the key a name

7. You Security Key is now added
![](__GHOST_URL__/content/images/wordpress/2021/02/fi5.png)
## Authenticate to an MFA cloud app using the security key

For test purposes, I created a Conditional Access policy that applies to Exchange Online and All users. When authenticating to Outlook web, I&#8217;m asked to enter my pin-code if the security key is plugged in, if not I&#8217;m asked to plug it in.

1. Browse to [Outlook web app](https://outlook.office365.com) and enter your username and password

2. When asked to verify your identity, click on **Use a security key**
![](__GHOST_URL__/content/images/wordpress/2021/02/fi6.png)
3. Enter your pin-code and then touch the key

- ![](__GHOST_URL__/content/images/wordpress/2021/02/fi7-1.png)
- ![](__GHOST_URL__/content/images/wordpress/2021/02/fi8.png)

4. Voilá, we&#8217;re signed in

![](__GHOST_URL__/content/images/wordpress/2021/02/fi9.png)
## Known issues

### Security key provisioning

Administrator provisioning and de-provisioning of security keys is not available in the public preview.

### UPN changes

Microsoft is working on supporting a feature that allows UPN change on hybrid Azure AD joined and Azure AD joined devices. If a user&#8217;s UPN changes, you can no longer modify FIDO2 security keys to account for the change. The resolution is to reset the device and the user has to re-register.
