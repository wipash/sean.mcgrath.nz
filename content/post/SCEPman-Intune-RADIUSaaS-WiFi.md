---
title: "802.1x TLS Authentication using Intune, SCEPman, and RadiusAAS"
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
Once you've followed the [initial SCEPman setup](https://docs.scepman.com/scepman-deployment/deployment-guides/enterprise-guide-1), you should have a functional web app running somewhere like https://sean-scepman.azurewebsites.net/

I have configured some additional SCEPman settings as follows:

**Setting**|**Value**|**Docs**|**Reasoning**
:-----:|:-----:|:-----:|:-----:
`AppConfig:UseRequestedKeyUsages`|`true`|[Docs](https://docs.scepman.com/advanced-configuration/application-settings/certificates#appconfig-userequestedkeyusages)|Allows us to define a custom EKU extension in our Intune SCEP profile
`AppConfig:ValidityPeriodDays`|`365`|[Docs](https://docs.scepman.com/advanced-configuration/application-settings/certificates#appconfig-validityperioddays)|Lets us issue certificates valid for up to one year

Grab the newly issued root CA from your SCEPman interface and hold onto it, we'll need it later on.
![Root CA](/img/scepman-dashboard-rootcert.png)

---

# RADIUSaaS Setup
The only critical steps within RADIUSaaS at this stage are to [configure the SCEPman root to be trusted for client authentication](https://docs.radiusaas.com/portal/settings/settings-trusted-roots/trusted-roots), and to [create](https://docs.radiusaas.com/portal/settings/settings-server/certificates#custom-cas) and [download](https://docs.radiusaas.com/portal/settings/settings-server/certificates#download=) the root certificate that will be used to sign the RADIUS responses.

{{< notice tip >}}
Make sure you edit the downloaded certificate to only include the root certificate, as described in the RADIUSaaS docs [here](https://docs.radiusaas.com/azure/microsoft-intune/trusted-root).
{{< /notice >}}

---

# Intune Setup
Intune requires a few config profiles to be set up and deployed to your users and devices
- Deploy the SCEPman root certificate
- Deploy the RADIUSaaS root certificate
- Deploy a Wi-Fi configuration profile

## SCEPman Root Certificate - Devices
1. Create a new device configuration profile called something like "Devices - SCEPman Trusted Root" using the "Windows 8.1 and later" platform, with profile type "Trusted certificate"
2. Upload your SCEPman root certificate
3. Set the destination store to "Computer certificate store - Root"
4. Deploy the profile to a group of devices

## SCEPman Root Certificate - Users
This is essentially identical to the last profile, but you need the profile to be deployed to both users and devices so that you can use it in the SCEP profiles later on.
1. Create a new device configuration profile called something like "Users - SCEPman Trusted Root" using the "Windows 8.1 and later" platform, with profile type "Trusted certificate"
2. Upload your SCEPman root certificate
3. Set the destination store to "Computer certificate store - Root"
4. Deploy the profile to a group of users



# Topics to cover
- Certificates issued from SCEPman via Intune
- Wi-Fi profile delivered via Intune - key is that the auth mode is "Client & device"
- Cert uses subject to indicate VLAN
- RadiusAAS uses cert subject to dynamically assign VLAN in Radius response
