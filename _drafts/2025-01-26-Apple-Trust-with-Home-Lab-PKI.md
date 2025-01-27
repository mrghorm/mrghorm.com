---
layout: post
title: Apple Cert Requirements for Homelab PKI 
---

After recently setting up a Jellyfin server with HTTPS, my usual devices (Windows & Linux) were happy connecting, but my Apple devices would reject the HTTPS, and often wouldn't give a specific error.  I came to find that Apple devices have requirements for PKI trust that are a little more strict than standard.

This is a high-level overview of Apple's requirements, and what it takes to coerce Apple devices into trusting your homelab's CA server and issued certs.


# Summary of Apple's rules for certificates

1.  Apple devices won't trust certificates with an expiry period of greater than 398 days. 
    - [Source](https://support.apple.com/en-us/102028#:~:text=398%20days%20is%20measured%20with,maximum%20validity%20of%20397%20days.)
2.  Apple devices won't trust a certificate chain unless the **entire** certificate chain is served by the server.  
    - Source:  "Trust me bro" (I can't find a good source, but this is what I've seen in practice)
3.  Apple has some requirements for the algorithm used.  Here are some examples:
    - RSA certs require 2048 bits or greater using SHA-2 hashing
    - ECDSA certs require 256 bits and using NIST P-256 curve
4.  The usual stuff (also required for Windows and/or Linux trust):
    - The cert must contain a SAN (Subject Alternative Name) that matches the serving FQDN (i.e. if your iPhone is reaching out to "server.lan", then the certificate must have "server.lan" in the SAN field)
    - The cert must be marked for "Server Authentication" in the extended properties


# How to set up homelab PKI trust for Apple Devices

First step is to obtain your certificates for your root CA (and intermediate CA if applicable).  In my case, the root certificate is self-signed.  I've found that PEM format works fine, but [other sources](https://support.apple.com/guide/deployment/intro-to-certificate-management-depb5eff8914/web) show that Apple can accept a range of certificate formats.  

I've chosen to install my root certificate and intermediate certificate separately, but I've read elsewhere that the whole chain can be installed at the same time; it's typical for orgs to install entire cert chains at once using MDM.


### Install the profile on iPhone or iPad

1. **Download the profile:** On iPhone and iPad, use the **Files** app and simply tap on the certificate file to open it.  It will download the cert to your settings as a "profile".

2. **Install the profile in settings:** Navigate to your **Settings** app and tap the **"Profile Downloaded"** button near the top.  Review the certs settings, then tap **"Install"** in the top left.  This installs the certificate as a profile on your device.  These can be found under General > VPN & Device Management in Settings.

3. **Enable ultimate trust for your root CA:**  In Settings, navigate to **General > About > Certificate Trust Settings**.  Enable "Full Trust For Root Certificates" for your root certificate.

Repeat steps 1 & 2 for your intermediate CA if applicable.  An intermediate CA won't show up in the ultimate trust settings.


### Install the profile on Apple TV

See [this tutorial](https://lucaslegname.github.io/mitmproxy/2020/04/10/certificate-tvos.html) from Lucas Legname on how to install a profile on an Apple TV using DropBox:

The process is very similar to iPhone or iPad, but requires a download link that you can type into the Apple TV.  To download the certificate into the Apple TV, navigate to **Settings > General > Privacy**.  Scroll to **Share Apple TV Analytics**, but click the **Play** button rather than clicking on it normally.


# Generating a Certificate for your Server

Make sure you generate your certificate based on the rules above.

Due to the 398-day expiry requirement, it's best to have the server automatically renew certificates assigned to it.  In my own environment, and at the recommendation of step-ca, I tend to run my certs with much shorter expiry times (as short as 1 week in some instances).  [See the step-ca docs](https://smallstep.com/docs/step-ca/certificate-authority-server-production/#use-short-lived-certificates)

The server needs to serve the *entire* chain, or your iPhone won't trust the cert.  If the server can use PEM format, then it's easy to combine the server's cert, the intermediate cert, and the root cert into one file.

For example, Jellyfin with its native HTTPS will *only* serve the HTTPS cert assigned to it, and not the whole chain.  This breaks Apple's trust.  In order to get around this, I had to set up an nginx proxy in front of Jellyfin that serves the entire certificate chain (including the root, intermediate, and server certs).  Linux & Windows on the other hand are happy to only receive the server certificate, as long as the root and intermediate certs are installed.


# Troubleshooting

One great tool I've found for troubleshooting is the [TLS Inspector](https://apps.apple.com/us/app/tls-inspector/id1100539810) app on iPhone.  It will show any potential issues with the certificate being served.
