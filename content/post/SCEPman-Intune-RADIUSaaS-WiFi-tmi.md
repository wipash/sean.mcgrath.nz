---
title: "802.1x TLS Authentication using Intune, SCEPman, and RadiusAAS - Too much info"
date: 2022-06-07T20:23:10+12:00
draft: true
---

This blog post assumes a few things:
- You have an active Intune subscription
- You have deployed [SCEPman](https://www.scepman.com/)
- You have an active [RADIUSaaS](https://www.radius-as-a-service.com/) subscription
  - RADIUSaaS could be substituted with another RADIUS service, or your own FreeRADIUS server, providing you can work out how to configure it.
- You're using Aerohive/Extreme Networks for your Wi-Fi solution
  - The config within the Wi-Fi solution is very basic, so can easily be translated to other platforms

By following this, you'll end up with:
- A new trusted root certificate
- User and device authentication certificates issued to users and devices
- A single Wi-Fi profile deployed to all your users and devices
- The ability to connect users and devices to different VLANs

---

# SCEPman Setup
Once you've followed the initial SCEPman setup, you should have a functional web app running somewhere like https://sean-scepman.azurewebsites.net/
I have configured some additional SCEPman settings as follows:

**Setting**|**Value**|**Docs**|**Reasoning**
:-----:|:-----:|:-----:|:-----:
`AppConfig:UseRequestedKeyUsages`|`true`|[Docs](https://docs.scepman.com/advanced-configuration/application-settings/certificates#appconfig-userequestedkeyusages)|Allows us to define a custom EKU extension in our Intune SCEP profile
`AppConfig:ValidityPeriodDays`|`365`|[Docs](https://docs.scepman.com/advanced-configuration/application-settings/certificates#appconfig-validityperioddays)|Lets us issue certificates valid for up to one year

Grab the newly issued root CA from your SCEPman interface and hold onto it, we'll need it later on.
![Root CA](/img/scepman-dashboard-rootcert.png)

---

# RADIUSaaS Setup
The only critical steps within RADIUSaaS at this stage are to configure the SCEPman root to be trusted for client authentication, and to create and download the root certificate that will be used to sign the RADIUS responses.

#### SCEPman Trusted Root Certificate
1. Sign in to your RADIUSaaS portal (https://yourname.radius-as-a-service.com)
2. Under Settings -> Authentication Certificates, hit the Add button and enter your SCEPman URL. This will automatically grab your SCEPman root certificate.

#### RADIUSaaS Root Certificate
1. Under Settings -> Server Settings, hit the Add button under the Server Certificates header
2. Choose 'Create your own CA', and press Create
3. Hit the Download button next to your newly created Customer-CA certificate
   ![Download RADIUSaaS certificate](/img/radiusaas-download-ca.png)
4.
---

# Intune Setup
Intune requires a few config profiles to be set up and deployed to your users and devices
- Deploy the

# Topics to cover
- Certificates issued from SCEPman via Intune
- Wi-Fi profile delivered via Intune - key is that the auth mode is "Client & device"
- Cert uses subject to indicate VLAN
- RadiusAAS uses cert subject to dynamically assign VLAN in Radius response
