---
date: '2024-12-01T10:24:59+01:00'
draft: true
title: 'Download Splunk Without Login'
slug: 'download-splunk-without-login'
categories:
  - HowTo
tags:
  - Splunk
description: 'Learn how to download splunk without logging in.'
---

[Splunk Enterprise](https://www.splunk.com/en_us/products/splunk-enterprise.html) is software for processing large volumes of data.
Its download is free, and its use is subject to licensing.
Splunk offers different licenses, including a free license by default that limits the amount of data that can be indexed, but most features remain accessible.

In this article, we will see how to download Splunk for different platforms in various ways.

<!--more-->

## The naive method

Downloading Splunk is done [from the publisher's website](https://www.splunk.com/en_us/download/splunk-enterprise.html).
You can create an account on this page, and then you will have access to the download links.
If you do not wish to share your personal data with the publisher, you will find a whole set of methods, including the use of disposable email addresses, in an upcoming article.

For example, the latest version (at the time of writing this article) of Splunk Enterprise on Linux in RPM is available behind the link

```text
https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498.x86_64.rpm
```

Most of the URL is deterministic, we can guess it in advance.
However, the hexadecimal characters `d8bb32809498` probably correspond to a build number, preventing us from generating the entire download URL automatically.

## From the browser

By observing the source (`CTRL+U`) of the Splunk download page: [https://www.splunk.com/en_us/download/splunk-enterprise.html](https://www.splunk.com/en_us/download/splunk-enterprise.html), we notice the presence of the download links.
They can be found in `data-link` attributes:

```html
<a class="splunk-btn sp-btn-solid sp-btn-pink"
 data-arch="x86_64"
 data-filename="splunk-9.3.2-d8bb32809498-x64-release.msi"
 data-link="https://download.splunk.com/products/splunk/releases/9.3.2/windows/splunk-9.3.2-d8bb32809498-x64-release.msi"
 data-md5="https://download.splunk.com/products/splunk/releases/9.3.2/windows/splunk-9.3.2-d8bb32809498-x64-release.msi.md5"
 data-wget="wget -O splunk-9.3.2-d8bb32809498-x64-release.msi &#34;https://download.splunk.com/products/splunk/releases/9.3.2/windows/splunk-9.3.2-d8bb32809498-x64-release.msi&#34;"
 data-oplatform="windows"
 data-platform="windows"
 data-sha512="https://download.splunk.com/products/splunk/releases/9.3.2/windows/splunk-9.3.2-d8bb32809498-x64-release.msi.sha512"
 data-thankyou="/content/splunkcom/fr_fr/download/splunk-enterprise/thank-you-enterprise.html"
 data-track-analytics="true"
 data-product="splunk"
 data-version="9.3.2"
 href="#"><span>Télécharger maintenant</span>
```

You just need to look here for the links that interest you and open them in another tab or download them with [cURL](https://curl.se/) for example.

## From a shell

You can use the following script to perform these actions simply from a shell.

The script will list the possible downloads and let you choose the one that interests you.

It will then download the file in question to the current directory.

In case of a disconnection, don't panic, restart the script and the download of Splunk will resume where it left off.

{{< gist K-Yo 0d0aaa9c4c6b4d0ad88867a86b4b3963 >}}
