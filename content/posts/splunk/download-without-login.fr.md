---
date: '2024-12-01T10:24:59+01:00'
draft: true
title: 'Téléchargez Splunk sans vous connecter'
slug: 'download-splunk-without-login'
categories:
  - Tuto
tags:
  - Splunk
description: 'Apprenez comment télécharger Splunk sans s’identifier.'
---

[Splunk Enterprise](https://www.splunk.com/en_us/products/splunk-enterprise.html) est un logiciel de traitement de gros volumes de données.
Son téléchargement est gratuit, et son usage est soumis à licence.
Splunk offre différentes licences, notamment par défaut une licence gratuite qui limite la quantité de données que l’on peut y indexer, mais la plupart des fonctionnalités restent accessibles.

Nous allons voir dans cet article comment télécharger Splunk pour différentes plateformes de différentes façons.

<!--more-->

## La méthode naïve

Le téléchargement de Splunk se fait [depuis le site internet de l’éditeur](https://www.splunk.com/fr_fr/download/splunk-enterprise.html).
Vous pouvez sur cette page créer un compte et ensuite vous aurez accès aux liens de téléchargement.
Si vous ne souhaitez pas partager vos données personnelles avec l’éditeur, vous pourrez trouver tout un ensemble de méthodes, notamment l’utilisation d’adresse e-mail jetable, dans un article à venir.

À titre d’exemple, la dernière version (au moment de la rédaction de cet article) de Splunk Enterprise sur Linux en RPM est disponible derrière le lien

```text
https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498.x86_64.rpm
```

La plupart de l’URL est déterministe, nous pouvons la deviner à l’avance.
Cependant les caractères hexadécimaux `d8bb32809498` correspondent probablement à un numéro de build, et nous empêchent de générer toute l’URL de téléchargement automatiquement.


## Depuis le navigateur

En observant la source (`CTRL+U`) de la page de téléchargement de Splunk: [https://www.splunk.com/fr_fr/download/splunk-enterprise.html](https://www.splunk.com/fr_fr/download/splunk-enterprise.html), nous remarquons la présence des liens de téléchargement.
On les retrouve dans des attributs `data-link`:

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

Il suffit d’aller chercher ici les liens qui vous intéressent et les ouvrir dans un autre onglet ou les télécharger avec [cURL](https://curl.se/) par exemple.

## Depuis un shell

Vous pouvez utiliser le script suivant pour effectuer ces actions simplement depuis un shell.

Le script va vous lister les téléchargements possible et vous laisser choisir celui qui vous intéresse.

Il va alors télécharger le fichier en question dans le répertoire courant.

En cas de coupure, pas de panique, relancez le script et le téléchargement de splunk reprendra où il s’est arrêté.

{{< gist K-Yo 0d0aaa9c4c6b4d0ad88867a86b4b3963 >}}
