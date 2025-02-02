---
layout: post
title: Apple Cert Requirements for Homelab PKI
excerpt_separator: <!--more-->
updated_date: 2025-02-02
---

This is a high-level overview of Apple's requirements for certificates, and how to trick Apple devices into trusting a custom CA.

<!--more-->

# Summary of Apple's rules for certificates

1.  **Maximum expiry period is 398 days**. 
    - [Source](https://support.apple.com/en-us/102028#:~:text=398%20days%20is%20measured%20with,maximum%20validity%20of%20397%20days.)
2.  **The entire certificate chain must be served by the server**.  
    - Source:  "Trust me bro" (I can't find a good source, but this is what I've seen in practice)
3.  Requirements for various algorithms ([source](https://www.apple.com/certificateauthority/ca_program.html)):
    - **RSA** requires **2048** bits or greater using **SHA-2** hashing
    - **ECDSA** requires **256** bits and must use **NIST P-256** curve
    - Other algorithms are supported with their own set of requirements
4.  The usual stuff (also required for Windows and/or Linux trust):
    - **Must have a valid SAN** (Subject Alternative Name) that matches the DNS for the server (i.e. If I'm serving the cert from mrghorm.com, then "mrghorm.com" must be in the SAN field)
    - The cert must be marked for **"Server Authentication"** in the extended properties


# How to Install Homelab Root Cert on Apple Devices

## Install profile on iPhone or iPad

1. **Obtain certs from your root CA** (and intermediate CA, if applicable).
   - PEM format works fine, [but apple supports other formats too](https://support.apple.com/guide/deployment/intro-to-certificate-management-depb5eff8914/web)
   - I chose to install my root and intermediate certs **separately**, but I've read elsewhere that you can install a single PEM file with the full chain.

2. **Download the profile:** On iPhone and iPad, use the **Files** app and simply tap on the certificate file to open it.  It will download the cert to your settings as a "profile".

3. **Install the profile in settings:** Navigate to your **Settings** app and tap the **"Profile Downloaded"** button near the top.  Review the certs settings, then tap **"Install"** in the top left.  This installs the certificate as a profile on your device.  These can be found under General > VPN & Device Management in Settings.

4. **Enable ultimate trust for your root CA:**  In Settings, navigate to **General > About > Certificate Trust Settings**.  Enable "Full Trust For Root Certificates" for your root certificate.

Repeat steps 1 & 2 for your intermediate CA if applicable.  An intermediate CA won't show up in the ultimate trust settings.


### Install the profile on Apple TV

See [this tutorial](https://lucaslegname.github.io/mitmproxy/2020/04/10/certificate-tvos.html) from Lucas Legname on how to install a profile on an Apple TV using DropBox.  The process is very similar to iPhone or iPad, but requires a download link that you can type into the Apple TV.  In summary:


1. Navigate to **Settings > General > Privacy**.  Scroll to **Share Apple TV Analytics**, but click the **Play** button rather than clicking on it normally.
2. Click on **Add Profile**
3. Enter the URL where your certificate is hosted, then click **Done**
4. Navigate to **Settings > General > About**, then click **Certificate Trust Settings**
5. Click on your certificate to trust it.  It should say **Trusted** next to it when finished


# Troubleshooting

One great tool I've found for troubleshooting is the [TLS Inspector](https://apps.apple.com/us/app/tls-inspector/id1100539810) app on iPhone.  It will show any potential issues with the certificate being served, as well as show all the certificate's details.

---

# Changelog

2025-02-02:  
- Simplified article; gets to the point much faster.
- Updated some sources for requirements
- Added a changelog

2025-02-02:
- Added table of contents