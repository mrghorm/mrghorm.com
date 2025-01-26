---
layout: post
title: Apple Cert Requirements for Homelab PKI 
---

Apple devices can be a little touchy when trying to get them to trust your homelab PKI, if you've set them up.  This can interfere with some applications' abilities to work correctly with HTTPS, and sometimes prevents applications from working correctly at all.

A couple examples in my own homelab:  Mylio Photos wasn't trusting my Minio S3 storage server when using the app on my iPhone or iPad.  This was causing some syncing issues when other devices, like my desktop, were turned off.  Another example would be that I wasn't able to get HTTPS working correctly using the Infuse app while connecting to my Jellyfin media server.

This tutorial is a high-level overview of what's needed to get homelab trust working on Apple Devices.


# Summary of Apple's rules for certificates

1.  Apple devices won't trust certificates with an expiry period of greater than 398 days. 
    - [Source](https://support.apple.com/en-us/102028#:~:text=398%20days%20is%20measured%20with,maximum%20validity%20of%20397%20days.)
2.  Apple devices won't trust a certificate chain unless the **entire** certificate chain is served by the server.  
    - Source:  "Trust me bro" (I can't find a good source, but this is what I've seen in practice)
3.  Apple has some requirements depending on the algorithm used.  Here are some examples:
    - RSA with 2048 bits or greater using SHA-2 hashing
    - ECDSA with 256 bits and using NIST P-256 curve
4.  The usual stuff:
    - The cert must contain a SAN (Subject Alternative Name) that matches the serving FQDN (i.e. if your phone is reaching out to "server.lan", then the certificate must have "server.lan" in the SAN settings)
    - The cert must be marked for "Server Authentication" in the extended properties


# How to set up homelab PKI trust for Apple Devices

First step is to obtain your certificates (signed public keys) for your CA.  In my case, the root certificate is self-signed.  I've found that PEM format works fine, but [other sources](https://support.apple.com/guide/deployment/intro-to-certificate-management-depb5eff8914/web) show that Apple can accept a range of certificate formats.  I've chosen to install my root certificate and intermediate certificate separately, but I've read elsewhere that the whole chain can be installed at the same time.


### Install the profile on iPhone or iPad

1. On iPhone and iPad, use the files app and simply open the **root** certificate.  It will download the certificate to your settings as a "profile".

2. Navigate to your settings app and tap the "Profile Downloaded" button near the top.  Review the settings if you wish, then tap **"Install"** in the top left.  This installs the certificate as a profile on your device.

However, seeing as this is the root certificate, it still hasn't been given "Ultimate Trust" in the system yet.  To do this in Settings, navigate to General > About > Scroll down to the bottom > Certificate Trust Settings > Enable "Full Trust For Root Certificates" for your root certificate.

Repeat these steps (except enabling ultimate trust) for your intermediate certificate if you're installing it separate.


### Install the profile on Apple TV

See [this tutorial](https://lucaslegname.github.io/mitmproxy/2020/04/10/certificate-tvos.html) from Lucas Legname on how to install a profile on an Apple TV using DropBox:

The process is very similar to iPhone or iPad, but requires a download link that you can type into the Apple TV.


# Generating a Certificate for your Server

Lastly, whatever server you're using needs its certificate to serve to your Apple Device.

Make sure you generate your certificate based on the rules above.  

Due to the 398-day expiry requirement, it's best to have the server automatically renew certificates assigned to it.  In my own environment, and at the recommendation of step-ca, I tend to run my certs with much shorter expiry times (as low as 1 day in some instances).  [See the step-ca docs](https://smallstep.com/docs/step-ca/certificate-authority-server-production/#use-short-lived-certificates)

Don't forget:  the server needs to serve the *entire* chain, or your iPhone won't trust the cert.  If the server can use PEM format, then it's easy to combine the server's cert, the intermediate cert, and the root cert into one file.


# Troubleshooting

One great tool I've found for troubleshooting is the [TLS Inspector](https://apps.apple.com/us/app/tls-inspector/id1100539810) app on iPhone/iPad.  You can use it to connect to whatever service you're trying to serve your certificate on, and it will show you how the certificate is being served and if it's trusted or not, and why it might not be trusted on your device.


---

Questions or comments?  Feel free to connect with me!  matt@mrghorm.com
