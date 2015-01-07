---
layout: post
title:  Et pourquoi pas SQLite ?
tags: linux
---

Intéresons nous aujourd'hui à [SQLite](http://www.sqlite.org/). En effet, il semblerait que j'ai un train de retard ce coup ci : l'habitude de déployer toujours du MySQL, et le fait que j'ai déjà un service [SQL](http://en.wikipedia.org/wiki/SQL) qui tourne m'a détourné du droit chemin de la curiosité. Corrigeons ça tout de suite !

Il existe [pleins](http://en.wikipedia.org/wiki/Comparison_of_relational_database_management_systems) de protagonistes, mais sur Debian on retrouve : [MySQL](http://packages.debian.org/fr/wheezy/database/mysql-server), [PostgreSQL](http://packages.debian.org/fr/wheezy/database/postgresql), [Drizzle](http://packages.debian.org/fr/wheezy/database/drizzle) qui est plutôt orienté cloud, [MongoDB](http://packages.debian.org/fr/wheezy/database/mongodb) qui est orienté orientée objets, [SQLite](http://packages.debian.org/fr/wheezy/database/sqlite3) qui foncitonne sans daemon, [Tarantool](http://packages.debian.org/fr/wheezy/database/tarantool) qui est en [NoSQL](http://en.wikipedia.org/wiki/NoSQL) en ram, et [Virtuoso](http://packages.debian.org/fr/wheezy/database/virtuoso-minimal).

# SQLite, pour et contre

Avantages
- facile à installer, zéro configuration ou presque
- peut être inclus directement dans une application
- développement et mise en oeuvre très rapide
- utilisable pour des sites web qui reçoivent environ 100,000 hits/jour
- permet de remplacer tout stockage du style fopen/fread/fwrite/fclose

Inconvénients
- impossible de faire des restrictions d'accès fines type grant/revoke
- impossible de faire une architecture client-serveur
- pas de métriques permettant l'analyse fine des performances
- pas super pour les bases volumineuses
- pas super s'il y a beaucoup d'écritures concurrentes
- les tables ne sont pas évolutives à par ajout/renommage de colonnes
- pas de procédures stockés

Bref, pour tout ce qui consisterait à stocker et accéder à des données sans qu'elles soient modifiées trop souvent par trop de personnes en même temps, c'est parfait. En résumé : s'il n'y a pas eu besoin de faire du tuning de performances pour votre application qui utilise un autre système de gestion de bases de données, alors vous pouvez utiliser SQLite à la place.

C'est tellement parfait pour certaines choses, que même Google, Adobe, Mozilla, Opera l'ont choisi pour stocker les paramètres, et les données locales de leurs applications. Au final, c'est le système SQL le plus déployé au monde !

# Installation

Comme d'habitude, sous Debian, c'est simple :
- installation `aptitude install sqlite3`
- pour le support PHP `aptitude install php5-sqlite`
- redémarrer le serveur web

Tester que ça marche en créant une page PHP

	<?php
	if(!class_exists('SQLite3'))
	  die("SQLite 3 NOT supported.");
	else
	  echo "SQLite 3 supported.";
	?>

Accéder à cette page depuis votre navigateur devrait valider le bon foncitonnement. Accessoirement, le listing phpinfo montrerait que le PDO sqlite est chargé.

# Utilisation

Un tour dans la [documentation](http://www.sqlite.org/docs.html) et la page `man sqlite3`, et vous saurez sur l'outil en ligne de commande qui permet la gesiton des bases, leur création, modifications, et sauvegarde.

Exemples en ligne de commande :
- crée/ouvrir une base appelée "toto" : `sqlite3 toto.db` (l'extention est au choix)
- sauvegarder : `sqlite3 toto.db '.backup bkp_toto.db'` (les guillemets sont importants)

Idem, en PHP, un tour dans la [documentation](http://www.php.net/manual/fr/class.sqlite3.php) pour le format et la liste des fonctions.

Un tutorial PHP (avec des objets/classes) est disponible [ici](http://www.tutorialspoint.com/sqlite/sqlite_php.htm).

Et pour finir, le subset SQL supporté par la librairie est lisible [ici](http://www.sqlite.org/lang.html).
