---
layout: post
title:  Certificats gratuits
tags: sécurité serveur certificat
---

Je pensais qu'il fallait payer pour avoir des certificats reconnus par les navigateurs ? Et bien en fait ça n'est pas le cas, grâce à [StartSSL](https://www.startssl.com), qui délivre *gratuitement* des certificats.

<!--more-->

Un certificat est comme une carte d'identité, qui a été validée par quelqu'un "de confiance". Ca permet de décider de faire confiance à celui qui présente sa carte, dans la limite de ce qui a été vérifié.

Deux types de certificats gratuits sont disponibles :
- "email" (appelé aussi S/MIME, ou certificat client)
- "nom de domaine" (appelé certificat serveur)

Dans un certificat "email", seule l'adresse email est vérifiée. Cette vérification est faite via l'habituel mail de confirmation et le lien à cliquer. Aucune autre information n'est vérifiée (ni le nom, ni le prénom, ni rien)

Un certificat personnel "email" permet de signer ses emails envoyés afin de garantir leur authenticité et de les chiffrer si besoin.

Dans un certificat "nom de domaine", seul le nom de domaine est vérifié. Cette vérification est faite en envoyant un email à l'administrateur du domaine, avec un lien à cliquer. Aucune autre information n'est vérifiée (ni le nom de l'entreprise, ni rien)

Un certificat "nom de domaine" permet d'autentifier un serveur, et de chiffre les échanges avec celui-ci. C'est par exemple indispensable pour le HTTPS, mais ça peut être utilisé aussi pour la consultation des emails via IMAP+SSL/TLS, ou l'authentification de l'envoi de mail via SMTP+SSL/TLS **vérifier** et aussi la signature des informations DNS **vérifier**

A noter que la délivrance et le renouvellement de certificats est gratuite, mais que la révocation avant expiration peut avoir un coût, mais si on fait pas les idiots on en aura  jamais besoin.

# Inscription auprès de StartSSL

L'inscription se fait sur le site [StartSSL](https://www.startssl.com), et au lieu de vous demander un identifiant et un mot de passe, ils ont décidé de vous créer un *certificat client*.

Ils vous demandent donc vos infos (nom, prénom, adresse, téléphone et email), et vérifieront que votre email est valide. A noter qu'il vaut mieux mettre des informations réelles, car à la moindre vérification invalide, tous les certificats générés après coup pourraient être révoqués.

On reçoit un email avec un code de confirmation à entrer sur leur site, on attend une dizaine de minute, et on reçoit un second email avec un deuxième lien à cliquer pour rentrer un deuxième code de confirmation.

Ensuite, le site :
- génère une clé privée sur votre ordinateur que vous seul connaissez
- créé un certificat, basé sur cette clé, et le signe de leur autorité
- installe ce certificat dans votre navigateur

Ce premier certificat ne sert qu'à une chose : vous identifier auprès du service StartSSL, en lieu et place des habituels identifiants/mot de passe. Ils ont raison, c'est mieux. La contrepartie, c'est qu'il faut extraire et sauvegarder ce certificat pour l'archiver, et pour pouvoir se connecter au site StartSSL depuis un autre navigateur (même si c'est sur le même PC !)

Pour exporter et sauvegarder le certificat qui permet d'accéder au site, sous FireFox 21, aller dans les "options/préférences", puis dans la section "avancé", puis dans l'onglet "encryption", puis "voir les certificats", puis dans l'onglet "vos certificats".

Vous devriez avoir dans la liste le certificat identifié par votre adresse email, dans la section "StartCom Ltd". Sélectionner le certificat, et cliquer sur "sauvegarder". Il vous demande un mot de passe (que vous stockerez par exemple dans un gestionaire de mots de passe, mais dans tous les cas, que vous n'oublierez pas !) et il créé un fichier avec extension `.p12`, vu que le certificat est au format [PKCS12](http://fr.wikipedia.org/wiki/PKCS12).

En regardant ce fichier, on voit que la seule information qu'il contient est l'adresse email qui a été vérifiée, aucune autre information demandée lors de l'inscription n'est présente (c'est logique vu qu'elles n'ont pas été vérifiées).

Si vous voulez créer un autre certificat pour une autre adresse email, il faut d'abord valider cette autre adresse mail, et ensuite générer un certificat basé sur cette identité.

Pour valider une adresse mail :
- se connecter au site [StartSSL](https://www.startssl.com)
- aller dans l'onglet "validation wizard"
- sélectionner "email address validation"
- entrer l'adresse mail à vérifier
- attendez de recevoir le mail
- entrez le code de confirmation
- terminer la vérification

Pour générer un certificat "email" basé sur une identité :
- aller dans l'onglet "certificates wizard"
- selectionner "S/mime and authentication certificate"
- selectionner "high grade"
- selectionner votre adresse email et sélectionner "SHA1 (default)"
- cliquer terminer, le certificat est installé dans votre navigateur
- exportez le et sauvegardez le comme auparavant

A noter que n'importe quel certificat d'identité créé sur le site permet de s'authentifier auprès de StartSSL, vu qu'ils ont tous été générés par le même "compte".

# Utilisation de ce certificat "email" pour la messagerie

Pour utiliser un certificat avec votre messagerie il est indispensable d'avoir un logiciel de messagerie : il n'est pas possible de signer ses emails quand on utilise un webmail. Il faut donc utiliser un truc style Thunderbird (ou autre).

Aller dans le menu "préférence", "paramètre de comptes", et aller dans la partie "sécurité de vos comptes, cliquer sur "voir les certificats", cliquer sur "importer", sélectionner le fichier P12 créé plus haut, entrer le mot de passe et validez le message d'information comme quoi le certificat a été importé dans Thunderbird.

Il faut maintenant définir où (sur quel compte si vous en avez plusieurs) et comment (signature et/ou encryption) vous voulez utiliser ce certificat.

Pour chacun des comptes que vous souhaitez protéger, allez dans la section sécurité, et dans la partie "signature numérique" cliquer sur sélectionner. Choisissez le certificat à utiliser, et validez.

On vous demande alors si vous voulez utiliser le même certificat pour chiffrer les messages envoyés ou déchiffrer les messages qui vous ont été envoyés chiffrés, répondez "oui". Vérifier pour chacun des comptes que par défaut, on n'encrypte jamais "par défaut" les emails envoyés.

Ensuite gérez votre messagerie comme d'habitude :
- et quand vous voulez signer un message (pour garantir son authenticité et le fait qu'il n'ait pas été modifié) utilisez l'option "signer numeriquement ce message" de la barre d'action ou bien du menu option
- et quand vous voulez chiffrer un message (pour garantir sa confidentialité) utilisez l'option "chiffrer ce message" de la barre d'action ou bien du menu option

Lors de la réception et de la relecture de messages envoyés, on voit que c'est signé grâce à l'enveloppe cachetée affichée dans la barre d'outil, et on voit que c'est chiffré grâce au cadenas dans la barre d'outil. Ou les deux si c'est signé et chiffré.

A noter que
- la ligne "sujet" du message n'est **jamais** chiffrée !
- c'est *votre* certificat qui est utilisé pour signer
- c'est le certificat du *destinataire* qui est utilisé pour chiffrer

Maintenant on a une messagerie sûre (authentifiée, non-modifiable et confidentielle).

# Création d'un certificat serveur

Une remarque importante : StartSSL ne permet de valider des zones directes (exemple.com, azerty.org, etc) et pas des sous domaines

Pour créer un certificat serveur, il faut d'abord valider que vous ayez accès au nom de domaine, puis générer un ou plusieurs certificats serveurs correspondant aux besoins de ce domaine.

Pour valider l'accès au domaine :
- se connecter au site [StartSSL](https://www.startssl.com)
- aller dans l'onglet "validation wizard"
- sélectionner "domain name validation"
- entrer le nom de votre domaine et son extension
- sélectionner une des adresses mail à laquelle vous avez accès
- entrer le code de confirmation reçu

Pour générer un certificat "serveur" basé sur un domaine
- aller dans l'onglet "certificates wizard"
- selectionner "web server ssl/tls certificate"
- selectionner un *bon* mot de passe et confirmez le
- sélectionner 4096 pour la key size
- selectionner SHA1 pour le "secure hash algorithm"
- cliquez "continue" et patientez
- copier/coller le texte fourni dans un fichier "xyz.key"
- sélectionner le nom de domaine qui contient le serveur (example.com)
- sélectionner le nom d'hôte de la machine cible prévue (www.example.com)
- copier/coller le texte fourni dans un fichier "xyz.crt"

Sauvegarder les certificats racines et intermédiaire de l'autorité (enregistrer-sous)
des deux liens "intermediate" et "root", pour qu'il puissent être référencés dans le daemon qui utilisera votre certificat, au besoin.

Dans tous les cas, un certificat serveur est consituté d'une partie "publique" (qui peut être diffusées sans restriction), c'est le bloc de texte délimité par `BEGIN/END CERTIFICATE` alors que la clé privée (qui ne devrait jamais quitter votre serveur) est le bloc de texte délimité par `BEGIN/END PRIVATE KEY`.

# Gestion de la clé privée

Un serveur (web par exemple) aura besoin d'avoir accès à la clé privée à chaque démarrage. Il demandera donc par défaut un mot de passe de manière interactive, et pendant ce temps il ne démarre pas. Pour éviter ça, on préfère généralement décrypter la clé privée (ôter l'effet du password utilisé à la création) via la commande `openssl rsa -in ssl-chiffree.key -out ssl-dechiffree.key`.

A noter qu'il est bon de mettre les bonnes authorisations d'accès à cette clé privée déchifrée, par exemple via un **vérifier** `chown root:www-data ssl-dechiffree.key` puis un `chmod 640 ssl-dechiffree.key`.

Pour info lors de la génération d'un certificat serveur, c'est le site qui a généré la clé privée secrête (c'est pour ça qu'il nous a demandé un mot de passe). Donc dans l'absolu il peut la stocker, et la réutiliser à de mauvaises fins, même si on peut logiquement lui faire confiance.

Pour être parfaitement confidentiel, il faudrait plutôt générer la clé privée sur notre serveur, créer à partir de celle-ci une "requête de signature de certificat" (CSR), et utiliser ce CSR pour demander la génération du certificat (ce que le site permet). De cette manière, la clé privée ne sort jamais de chez nous, et donc personne ne peut la prendre à moins de pénétrer notre serveur ou de faire une erreur de gestion.


