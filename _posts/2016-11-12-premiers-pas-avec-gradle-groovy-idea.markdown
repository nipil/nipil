---
layout: post
title: Premiers pas avec Gradle, Groovy et IDEA
tags: programmation groovy gradle idea
---

Depuis que j'ai changé de job il y a un an et demi, je fais beaucoup de "catch-up" (rattrapage) technologique, cf [article précédent](/2016/10/17/velib-stats-consultation-des-statistiques-velib.html).

Depuis septembre, j'ai aussi repris un autre projet (qui est encore à l'état de prototype) dont je ne connais ni les technos, ni la génèse ; il n'est pas documenté, et les docs de conception sont de type "fil rouge" (ie très succintes).

Bref tout est à faire, mais la base techno est déjà posée et ne changera pas (il y a de bonnes raisons pour ça, qui sont hors sujet ici). Pas grave, ça me dérange pas, je maîtris le "saut dans la piscine en mode éponge" !

Aujourd'hui, je vous parlerai de mes premiers pas avec [Groovy](http://groovy-lang.org/). Et donc avec [Gradle](https://gradle.org/). En utilisant [IDEA](https://www.jetbrains.com/idea/). Prenez une boisson chaude, et des petits gateaux, et allons y relax...

# Groovy, kezako ?

De quoi on parle :

- Groovy est un langage de programmation basé sur Java
- Java est un langage de programmation orienté objet
- Java (et ses dérivés, dont Groovy) est compilés en bytecode
- ce bytecode s'exécute sur une machine virtuelle multiplateforme

Ok ? Mais alors, pourquoi Groovy ?

- Groovy vise à *simplifier* et à "*dé-rigidifier" Java
- Groovy évite le "boilerplate" qui n'apporte rien et qui ne sert qu'à faire "que ça marche"
- Groovy permet de faire des "scripts" java (ie) et de faire du "typage optionnel"

Première illustration :

	// HelloWorld.java
	public class HelloWorld {
		public static void main(String[] args) {
			System.out.println("Hello, World");
		}
	}

	// HelloWorld.groovy
	println "Hello, World"

Je sais pas vous, mais moi il y en a clairement un que je préfère...

Trois choses sont à noter dans cet exemple :

- les `;` en fin de ligne sont optionnels
- on a un script "principal" et non pas une méthode class/méthode "main", ce qui élimine du code "boilerplate" qui n'apporte rien. En interne, l'outil va *générer* une classe et va prendre le code du script et tout coller dans une fonction `main` générée à la volée. Bref, c'est équivalent, mais plus simple.
- on a pas besoin de mettre le chemin de package des trucs les plus souvent utilisés, un certain nombre d'import sont faits automatiquement
- la fonction `println` opère par défaut sur stdout, sans qu'on ait besoin de préciser qu'on println sur la sortie standard `System.out`
- pour les commandes de "**premier** niveau", les parenthèses sont facultatives, ie on aurait pu écrire `println("toto")` mais on peut écrire `println "toto"`. De la même manière, on écrirait `f a, b` au lieu d'écrire `f(a, b)`.

Bref, l'objectif c'est de faire pareil, en plus simple, pour se concentrer sur le fond plutôt que la forme.

Tout groovy est fait sur ce principe, deux autres exemples :

- les types sont optionnels : `def a = new ArrayList<String>()` plutôt que `ArrayList<String> a = new ArrayList<String>()` donc on évite la répétition
- les nulls sont gérés de manière plus sécuritaire : `class A { String s; }; A a=null; b=a?.s` on aura `b==null` plutôt qu'une NullPointerException, sans avoir à tester avant d'accéder !

Pour finir, un script groovy ça s'exécute de la manière suivante :

	# soit avec une compilation à la volée :
	groovy monscript.groovy options

	# soit on le compile en bytecode java
	groovyc a.groovy
	# puis on exécute le bytecode java en incluant les librairies
	java -jar /usr/share/groovy2/embeddable/groovy-all-2.4.5.jar -cp . a

Il a des tonnes d'autres trucs que je suis loin de maîtriser, mais la [documentation](http://groovy-lang.org/documentation.html) du langage est très bien faite et agréable à lire, il faut juste prendre le temps de l'ingurgiter et de la digérer.
