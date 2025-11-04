---
title: Get devices from Samsung Knox Mobile Enrollment using Powershell
slug: get-devices-from-samsung-knox-mobile-enrollment-using-powershell
date_published: 2020-10-17T21:28:52.000Z
date_updated: 2021-02-22T15:52:03.000Z
---

I was recently tasked with exporting data from a customer&#8217;s UEM and 3rd party portals using REST-API, this data was then to be populated in Power BI. For the most part there were no issues until I started work on exporting device information from the Samsung Knox Mobile Enrollment portal. I thought I&#8217;d share my solution using Powershell.

First off, if you want to use REST-API with the KME portal you need to do some preparation. Let&#8217;s walk through them step by step.

## 1. Get access to the API

To get access to the Knox Cloud API you need to apply for this in the portal

- Log in to your account and apply for a [Knox Cloud Service](https://docs.samsungknox.com/dev/common/web-services.htm) on your [Knox Dashboard](https://www2.samsungknox.com/en/user).
- Apply for Knox Cloud API access by contacting your local Samsung representative.
- If you can&#8217;t get access through a Samsung Knox representative, please create a support ticket from the [Knox Dashboard](https://www2.samsungknox.com/en/user) to request access.

## 2. Generate your key pair from the Knox API Portal

- Launch the [Knox API Portal](https://us-kcs-api.samsungknox.com/kapi/) from the Samsung [Knox Dashboard](https://www2.samsungknox.com/en/user).
- From the portal, generate a public/private key pair called &#8220;keys.json&#8221; by clicking **Download**. This is a JSON file that contains the following information. It only needs to be generated once:
- **Public key**: Key that is sent in the body of the `accesstoken` REST API and stored by Samsung Knox to validate signed requests.
- **Private key**: Key that should be stored and never revealed. This key is used for signing the Client Identifier and the Access Token returned by the REST API call. This key is not stored by Samsung Knox and we will never request it from you.

## 3. Generate your unique Client Identifier from the Knox API Portal

Obtain your Client Identifier from the [Knox API Portal](https://us-kcs-api.samsungknox.com/kapi/). This unique identifier only needs to be generated once, unless either:

- Privileges need to be changed.
- The Client Identifier is lost or stolen.

In both cases, re-generating the Client Identifier invalidates the previous one. This image below shows the page you see when generating the Client Identifier from the portal.
![Knox Cloud Portal](https://docs.samsungknox.com/dev/common/images/client-identifier.PNG)
---

Once the above steps have been completed, we&#8217;re ready to get devices from the KME portal. Below is the script I wrote for this. This works great for me as I only need to run it once a month. If it&#8217;s to be run more often some handling needs to be added to check for an expired token. Here I&#8217;m requesting a new token each time the script runs. 

I&#8217;m sure there&#8217;s improvements that can be done with this script 🙂

First thing before you run it is to copy your keys.json file to ScriptRoot\SupportFiles and insert you Client ID on line 108. You can download a .zip [here](https://github.com/almenscorner/Powershell) with all needed files and directories.

    
    <#
    .SYNOPSIS
        This script gets devices from KME.
    .DESCRIPTION
        This script uses assemblies created by Samsung to generate and sign JWT and access tokens with the keys.json file. To run this script you need a keys.json file downloaded from your
        Samsung Knox portal and a generated Client Identifier.
        All needed assemblies are stored in the "NotLoaded" folder and are copied to "SupportFiles" once the script runs.
    
        keys.json needs to be stored in "SupportFiles".
    .NOTES
        Written by: Tobias Almen
        Version: 1.0
    #>
    
    $notLoaded = Get-ChildItem -Path "$PSScriptRoot\SupportFiles\NotLoaded" -File
    
    #If assemblies has not benn copied to SupportFiles, copy each assembly
    if ( -not (Test-Path -path "$PSScriptRoot\SupportFiles\*.dll")) {
        Write-Host -ForegroundColor Green "Copying assemblies to supportfiles"
        foreach ($file in $notLoaded) {
            Copy-Item -Path $file.FullName -Destination "$PSScriptRoot\SupportFiles" -Force
        }
    }
    
    sleep 2
    function Get-SignedJwt {
    
        param(
            [Parameter(Mandatory = $true)]
            [string]$jsonPath,
            [string]$clientID
        )
    
        #Load assemblies
        Add-Type -Path "$PSScriptRoot\SupportFiles\Microsoft.CSharp.dll"
        Add-Type -Path "$PSScriptRoot\SupportFiles\Microsoft.IdentityModel.Logging.dll"
        Add-Type -Path "$PSScriptRoot\SupportFiles\System.Security.Cryptography.Cng.dll"
        Add-Type -Path "$PSScriptRoot\SupportFiles\Microsoft.IdentityModel.Tokens.dll"
        Add-Type -Path "$PSScriptRoot\SupportFiles\Microsoft.IdentityModel.JsonWebTokens.dll"
        Add-Type -Path "$PSScriptRoot\SupportFiles\Newtonsoft.Json.dll"
        Add-Type -Path "$PSScriptRoot\SupportFiles\BouncyCastle.Crypto.dll"
        Add-Type -Path "$PSScriptRoot\SupportFiles\knoxAPIUtility.dll"
    
        #Generate signed JWT
        try {
            $script:signedClientId = [knoxAPIUtility.KnoxTokenUtility]::generateSignedClientIdentifierJWT("$jsonPath", "$clientID")
        }
        catch {
            Write-Error $_.Exception | format-list -force
            Break
        }
    }
    
    function Get-AccessToken {
    
        param(
            [Parameter(Mandatory = $true)]
            [string]$jsonPath
        )
    
        #Get json content from keys.json
        $keyJson = Get-Content $jsonPath | ConvertFrom-Json
        $pubKey = $keyJson.public
    
        #Build body for access token request
        $body = [ordered]@{
            clientIdentifierJwt          = "$signedClientId"
            base64EncodedStringPublicKey = "$pubKey"
            #validityForAccessTokenInMinutes = 30
        }
    
        #Convert body to json format
        $requestObject = $body | ConvertTo-Json
    
        $requestbody = @"
    $requestObject
    "@
    
        #Set access token request headers
        $Headers = @{’Content-type’ = ‘application/json’ }
    
        #If JWT has been generated, request access token
        if ($body.clientIdentifierJwt) {
            $tokenquery = Invoke-RestMethod -Method Post -Uri "https://eu-kcs-api.samsungknox.com/ams/v1/users/accesstoken" -Body $requestBody -Headers $Headers
        }
        else {
            Write-Error "Not able to get JWT"
        }
    
        #If access token request failed, break
        if (!$tokenquery.accesstoken) {
            Write-Error "Not able to get access token"
            Break
        }
        else {
            try {
                $script:signedAccessToken = [knoxAPIUtility.KnoxTokenUtility]::generateSignedAccessTokenJWT("$jsonPath", $tokenquery.accessToken)
            }
            catch {
                Write-Error $_.Exception | format-list -force
                Break
            }
        }
    }
    
    #Generate JWT
    Write-Host -ForegroundColor Cyan "Generating signed JWT"
    Get-SignedJwt -jsonPath "$PSScriptRoot\SupportFiles\keys.json" -clientID "YOUR_CLIENT_ID"
    
    sleep 2
    
    #Get access token and sign with KnoxAPI utility
    Write-Host -ForegroundColor Cyan "Getting access token and signing the token"
    Get-AccessToken -jsonPath "$PSScriptRoot\SupportFiles\keys.json"
    
    #Check if JTW and Token has been generated, if not, break
    if (!$signedClientId -or !$signedAccessToken) {
        Write-Error "not able to generate signed JWT or access token"
        break
    }
    
    sleep 2
    
    #If access token is generated get devices
    if ($signedAccessToken) {
        $Headers = @{’Content-type’ = ‘application/json’; 'x-knox-apitoken' = "$signedAccessToken"; 'cache-control' = 'no-cache' }
        $devicequery = Invoke-RestMethod -Method Get -Uri "https://eu-kcs-api.samsungknox.com/kcs/v1/kme/devices/list" -Headers $Headers
        Write-Host -ForegroundColor Yellow "Found $($devicequery.totalCount) devices"
    }
    

When the script runs successfully this is the output you&#8217;ll see
![](__GHOST_URL__/content/images/wordpress/2020/10/kmeapi-1024x269.png)
Here I only output the total number of devices uploaded to the portal. You can change this to fit your use case. I for example extract Model, Serial Number, Date Added and populate Power BI.

Hopefully, this post was useful and will help you get going using Samsung Knox REST-APIs with Powershell.
