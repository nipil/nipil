---
layout: post
title:  GnuPG, plan de secours validé, ouf !
---

Aujourd'hui, l'impensable est arrivé : j'ai perdu ma clé privée [GnuPG](http://www.gnupg.org/). Mais ça n'a été que temporaire, car j'avais tout prévu... Sauf que je n'avais pas testé ce plan de secours. On dit que "fail to plan = plan to fail", mais le billet d'aujourd'hui est là pour montrer que même si on a une solution de secours, si on la teste pas on risque pas d'en voir les écueils et d'être sûr que ça fonctionnera.

<!--more-->

Une clé GnuPG/PGP ça permet :
- à tout le monde de vérifier que ce que je donne a bien été envoyé par moi
- à tout le monde de vérifier que ce que je donne n'a pas été modifié
- à tout le monde de me donner des données cryptées sans aucun risques

Ca permet aussi de construire un réseau de confiance en signant les clés que tu as vérifié (en personne, par téléphone, etc). Ce principe de signature est similaire à ce que font les autorités de certifications lorsqu'ils donnent des certificats, sauf que là c'est d'humain à humain, et qu'il n'y a pas de certificats "racine" : c'est donc une toile de confiance, et non pas une "pyramide de confiance".

Retour au 24 avril 2001, où je découvre le système GnuPG. Aussi sec, je me créé une clé, et sans trop comprendre les implications ni savoir comment ça se gère, je créé une clé qui n'expire pas (d'où les deux colonnes vides ci-dessous), et je publie la partie publique sur un [serveur de clé](http://pgp.mit.edu/) (peu importe lequel, ils se répliquent les uns les autres). 

	pub  1024D/26A2F0AE 2001-04-24            
		 Fingerprint=8A6D 48BB 5DB9 2BDE 6065  3F39 0CC2 128E 26A2 F0AE 
	
	uid Nicolas Pillot <nicolas.pillot@ifrance.com>
	sig  sig   26A2F0AE 2001-04-24 __________ __________ [selfsig]
	
	sub  2048g/FAE19092 2001-04-24            
	sig sbind  26A2F0AE 2001-04-24 __________ __________ []

Remarque : On obtient les mêmes infos sans uploader sa clé via l'option `--list-keys --fingerprint`

# La facture de l'échec, constat et nouveau départ

Le 27 octobre 2007, après de multiples réinstallations et changement de distribution, arrive ce qui devait arriver : j'ai perdu mon répertoire `.gnupg`, et je ne retrouve aucun backup de celui-ci sur les diverses clé usb que j'utilise... l'andouille. En plus comme je n'y connaissais pas grand'chose, je n'ai pas préparé de révocation, ce qui est logique vu que je savais même pas à quoi ça servait.

Dans cette situation, les conséquences sont les suivante (par priorité d'emmerdes décroissantes)
- impossible de décrypter tout ce qui a pu m'être envoyé crypté par le passé
- mon identité numérique "reconnue" est perdue, il faut que je reconstruise ma réputation
- sans révocation, impossible d'annoncer que mon ancienne identité ne doit plus être utilisée

Du coup, j'ai perdu mes données cryptées, j'ai perdu ma réputation, et je risque de recevoir encore des données inutilisables via mon ancienne clé...

Je génère une nouvelle clé via `--gen-key`

Je l'upload vers un serveur de clé via `--send-keys` 

	pub  1024D/2BCABBDA 2007-10-27            
		 Fingerprint=3CDE AC7D 2771 9B5A 7450  C28D 0EDE 91EE 2BCA BBDA 
	
	uid Nicolas Pillot (nipil.org) <nicolas.pillot@gmail.com>
	sig  sig3  2BCABBDA 2007-10-27 __________ __________ [selfsig]
	
	sub  2048g/52E7BBFF 2007-10-27            
	sig sbind  2BCABBDA 2007-10-27 __________ __________ []

Puis je réfléchis à ce que j'aurais dû faire la première fois :
- pas de révocation prête à l'emploi
- pas de backup de ma clé privée

Corrigeons ça pour que ça n'arrive plus.

## Générer le bloc de révocation et exporter sa clé privée

Révoquer une clé ne la fera pas disparaître des serveurs de clé (rien ne disparaît jamais d'un serveur de clé), mais elle indiquera à tous ceux qui l'utiliseraient qu'elle ne doit plus être utilisée pour encrypter de nouvelles données, et qu'aucune signature ne peut être faite avec cette clé. Par contre, ça n'empêche pas de continuer de vérifier les signatures qui ont déjà été faites (c'est une des raisons pour laquelle les clés publiques révoquées restent sur les serveurs de clé). En résumé, révoquer une clé, c'est un peu comme laisser une clé expirer : elle cesse d'être utilisable pour de nouvelles choses, mais continue de fonctionner pour ce qui a été fait auparavant.

Préparer la révocation à l'avance, ça permet de "faire désactiver" une clé, même si on a tout perdu. En effet il faut la clé pour générer la révocation, mais on a besoin de rien pour que la révocation soit prise en compte, il suffit juste de l'uploader et ça désactive la clé. C'est donc important de la créer, mais il faut pas la laisser traîner : n'importe qui qui tombe dessus et déciderait de l'uploader désactivera votre clé sans que ça puisse être annulé. Bref, c'est ultra-utile, mais faut y faire attention.

En résumé :
- soit on finit avec une identité active mais inutilisable (sans révoc)
- soit on finit avec une identité inactive mais inutilisable (avec révoc)

Dans les deux cas l'ancienne est inutilisable, et dans les deux cas il faudra regénérer une identitié. Donc autant que l'ancienne puisse réellement être désactivée.

On génère un certificat de révocation avec l'option `--gen-revoke`, et on peut donner une raison affichée lorsqu'un le publiera, et un commentaire. L'outil demande bien sûr l'accès à la clé privée via le mot de passe, ce qui valide à l'avance la demande. Un tel document est très court, donc facile à sécuriser. Ca ressemble à ça (celui-ci est bien sûr bidon !) :

	-----BEGIN PGP PUBLIC KEY BLOCK-----
	Version: GnuPG v1.4.14 (GNU/Linux)
	Comment: A revocation certificate should follow
	
	iQEfBCABCAAJBQJSPEOGAh0CAAoJEPewhXw/whdWTQgH/24EOPoPa7CsLpgn9JPp
	imy2Y8q9htPgpXaYe/4TFNz7LSm2YxLWR2AoKJbm7Lo0m2VSGkp5UmwfBBtlYSZz
	oxM=
	=abcd
	-----END PGP PUBLIC KEY BLOCK-----

Et on la met de côté, bien à l'abri. Pour plus tard, si un jour (dans 1 mois, dans 1 an, dans 10 ans) on en a finalement besoin.

## Exporter sa clé privée

On exporte sa clé privée via `--armor --export-secret-key`

	-----BEGIN PGP PRIVATE KEY BLOCK-----
	Version: GnuPG v1.4.14 (GNU/Linux)
	
	lQO+BFI8QRkBCADD/5O3H1RY8hOYfi/l5M5CKbBVydHufo05TiDGxMyspZGyAw/s
	WFvEQ96axp9OBc5NxSlin4BjDagBDtpxpjO/dRQFSH8l5X3kr/1nDrjIRpjsqSRq
	...
	...
	...
	/+zckiru+ih//LcXbpTn3rDi5FwsDUN/20bO2m0OklSoIURZZCVzhjvxZrc0H4oZ
	EOB54DLO9iG5Tg==
	=GrR+
	-----END PGP PRIVATE KEY BLOCK-----

A noter que lors de l'export **la clé privée celle-ci reste cryptée** on est en "sécurité" mais *il faudra aussi archiver le mot de passe pour pouvoir tout récupérer* le jour où on en a besoin !

# L'impression papier, l'ultime backup

Pour la sauvegarde, j'ai choisi la solution la plus résiliente. Il aurait été possible de sauvegarder le tout sur CD, disque, disquette, clé, etc etc. Mais étant donné que tous peuvent perdre leurs données à plus ou moins long terme, ou décider de ne plus marcher, j'ai choisi la solution ... du papier.

Et oui, imprimé sur du papier, et classée/rangée avec d'autres papiers importants, de préférence sur un autre site, le plus simple étant de les confier à un membre de la famille à qui on fait confiance (tant sur le fait qu'il ne va pas révoquer votre clé pour le fun, mais aussi sur le fait qu'il conserve ce document)

On prendra soin de sauvegarder
- le fingerprint de la clé
- le bloc de révocation
- le bloc de la clé secrete
- le mot de passe de la clé privée
- la clé publique (uniquement si on l'a pas uploadée)

Le papier c'est simple et fiable, ça brûle/fond pas à moins de 250°C, l'impression laser noire haute qualité résiste très bien au temps. En plus, c'est plus simple car ça prend pas de place, et on en ayant une copie chez soi et chez un proche, on est tranquille contre les problèmes de dégats des eaux et/ou d'incendie ... et si jamais une des copies avait un problème, on l'apprendrait forcément (c'est la famille !) donc on pourrait redonner une copie si l'une ou l'autre avait été détruite.

Reste que comme c'est du papier, et non un format numérique, il va falloir lors de la récupération, retransformer ce qui a été imprimé en deux ou trois fichiers sur un PC. Pour ce faire, on passera par un coup de scan (ou photo numérique), puis un coup de "reconnaissance de caractères" (OCR en anglais) disponible [hors ligne](http://packages.debian.org/search?keywords=tesseract-ocr) ou [en ligne](http://www.onlineocr.net/) sur ces images, et une inévitable étape de correction manuelle du texte.

La phase la plus essentielle une fois qu'on a décidé d'imprimer le tout, c'est de faire plusieurs copies, **avec des polices de caractère bien différentes**, qui permettent de fiabiliser la reconnaissance par OCR, *et surtout* la relecture manuelle (correction des 'numéro un / petit L / grand L', des 'petit o, grand O, et zéro', des 'trois / grand B / huit', etc. Le top est de choisir une première police où tous ces caractères sont bien distincts, puis une/deux/trois autres polices d'aspect différent où ces caractères sont toujours bien différenciables. Et au moins une copie en 'Courier-new', histoire d'avoir une police "simple" de départ pour la reconstruction OCR.

Et après avoir passé 5 heures à vérifier, revérifier à la main les différentes copies pour corriger les typos de l'OCR jusqu'à ce que le checksum (de l'ensemble!) soit bon et que la clé puisse être réimportée, je peux vous garantir que ces précautions sont nécessaires... Si je n'avais pas eu 4 polices de caractères, il y a certains morceaux que je n'aurais pas pu retrouver à l'identique.

Après avoir galéré, je suis tombé sur un [conseil tout bête](http://lists.gnupg.org/pipermail/gnupg-users/2006-January/027750.html) qui m'aurait permis de gagner quelques heures : ajouter avant l'impression, un checksum par ligne. Comme ça, on peut se focaliser sur une ligne jusqu'à ce que le checksum soit bon, et voir beaucoup plus vite si une ligne sur papier est identique à la version OCR. Ceci est possible crâge à un script tout simple :

`cat bloc-a-imprimer.txt | while read n; do echo -en "${n}\t"; echo "${n}" | cksum; done`

On voit l'effet dans le listing suivant, où la première ligne est l'originale, la seconde est celle où on le script a ajouté le checksum :

	/+zckiru+ih//LcXbpTn3rDi5FwsDUN/20bO2m0OklSoIURZZCVzhjvxZrc0H4oZ
	/+zckiru+ih//LcXbpTn3rDi5FwsDUN/20bO2m0OklSoIURZZCVzhjvxZrc0H4oZ        4180382878 65

	EOB54DLO9iG5Tg==
	EOB54DLO9iG5Tg==        2054101035 17

	=GrR+
	=GrR+   3438912180 6

	-----END PGP PRIVATE KEY BLOCK-----
	-----END PGP PRIVATE KEY BLOCK-----     101326596 36

Bien que les chiffres ajoutés (qui représentent le checksum de la ligne et le nombre de caractères par ligne) ne "servent à rien" côté cryptographie, ça nous facilitera la correction des erreurs de recopie/scan car on voit tout de suite qu'une ligne est mauvaise, par exemple dans l'exemple suivant (où on a remplacé un 'zéro' par un 'grand O') où on voit immédiatement que le checksum n'est pas bon pour cette ligne :

	Ce qui a été imprimé initialement sur le papier :
	/+zckiru+ih//LcXbpTn3rDi5FwsDUN/20bO2m0OklSoIURZZCVzhjvxZrc0H4oZ        4180382878 65

	Ce le résultat du mini-script, sur la clé en cours de reconstruction :
	/+zckiru+ih//LcXbpTn3rDi5FwsDUN/2ObO2m0OklSoIURZZCVzhjvxZrc0H4oZ        648820701 65

On évite donc de devoir revérifier *à chaque fois* l'intégralité des milliers de caractères à chaque fois qu'on arrive pas à importer et que gpg nous dit "CRC Error".

Et encore mieux, on peut imprimer en complément un dump hexadécimal du fichier à imprimer :

`cat fichier-a-imprimer.txt | hd`

Ca permettra au prix d'un peu plus de papier de faciliter encore la récupération. En effet on a à la fois le caractère "réel" mais aussi le code hexa qui va avec : si on a un doute on lit le code hexa sur le papier et on sait directement quel est le bon charactère à mettre (pour résoudre les (o, O, 0, etc).

	00000d80  7a 63 6b 69 72 75 2b 31  68 2f 2f 4c 63 58 62 70  |zckiru+1h//LcXbp|
	00000d90  54 6e 33 72 44 69 35 46  77 73 44 55 4e 2f 32 4f  |Tn3rDi5FwsDUN/2O|
	00000da0  62 4f 32 6d 30 4f 6b 38  53 6f 49 55 52 5a 32 43  |bO2m0Ok8SoIURZ2C|
	00000db0  56 7a 42 6a 76 6c 5a 72  63 30 48 34 6f 5a 0a 45  |VzBjvlZrc0H4oZ.E|
	00000dc0  4f 42 35 34 44 4c 4f 39  69 47 35 54 67 3d 3d 0a  |OB54DLO9iG5Tg==.|

Grâce à ça, on voit tout de suite si un caractère est un 'petit i' (69), un 'un' (31), un 'L minuscule' (6C), ou un 'L majuscule' (4C). Pareil avec les autres caractères (B,3,8,o,0,O,w,W,etc) pour tous ceux-là il suffit de lire le code hexa et d'aller voir la [table ASCII](http://fr.wikipedia.org/wiki/American_Standard_Code_for_Information_Interchange).

Maintenant, plus d'excuses pour perdre sa clé privée !

# Précaution additionnelle pour le mot de passe

Comme le mot de passe peut être composé de pleins de caractères spéciaux (accents, etc) prenez bien garde à l'imprimmer en version "normale" mais aussi sous version  hexa (via `hd`). Ca serait dommage de récupérer tout sauf le mot de passe !

**Update 2013-09-21** : Je viens de faire un script qui fait tout ça. Ensuite, ne vous reste plus qu'à l'imprimer (que ça soit directement, ou par un traitement de texte et plusieurs polices)

	#! /bin/bash
	echo "Usage info : $0 NUMERICAL_KEY_ID > hardcopy.txt" >&2
	read -r -p "Please enter your passphrase so it can be appened to the hardcopy : " PASS
	TMP=`tempfile`
	gpg --gen-revoke $1 > $TMP
	echo -e "\nKey information:\n"
	gpg --list-key --fingerprint $KEY
	echo -e "\nPassphrase dump:\n"
	echo $PASS | hd
	echo -e "\nRevocation certificate (ascii with control checksums):\n"
	cat $TMP | while read n; do echo -e -n "${n}\t"; echo "${n}"|cksum; done
	echo -e "\nRevocation certificate (hexdump of raw ascii):\n"
	cat $TMP | hd
	echo -e "\nEncrypted secret key (ascii with control checksums):\n"
	gpg --export-secret-key -a $KEY | while read n; do echo -e -n "${n}\t"; echo "${n}" | cksum; done
	echo -e "\nEncrypted secret key (hexdump of raw ascii):\n"
	gpg --export-secret-key -a $KEY | hd
	rm -f $TMP

