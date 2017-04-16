---
layout: post
title: Connexion 3G avec raspbian jessie
tags: raspberry 3g modem
---

# L'objectif

En fait, j'aimerais pouvoir faire des trucs à distance, mais sans avoir sur place une connexion permanente (pas d'adsl) pour des questions de coût, de maintenance.

L'idée c'est d'installer sur place un raspberry pi (v1 parce que j'en ai un qui trainte) qui serait alimenté en permanence, vu que ça ne consomme quasiment rien.

Ensuite on déclencherait la connexion 3G en envoyant un SMS au numéro de la carte SIM. Et on finit par une connexion VPN vers une adresse IP prédéfinie (car généralement via une clé 3G on reçoit une adresse "privée" en 10.A.B.C qui n'est pas joignable directement).

À terme en plus on prendra soin des éléments additionnels :

- on fera à terme un filesystem en lecture seule pour ne pas avoir de soucis de carte SD
- on le redémarre une fois par jour au cas où quelque chose a planté

# Modem

J'ai une vieille clé 3G "Option ICON 225" de 2008 et une carte SIM Free-Mobile avec un forfait à 2€.

Elle est reconnue par défaut par la Debian, avec le pilote `hso`, qui créé des périphérique `/dev/ttyHS*`.

Pour savoir quel fichier sert à quoi, on regardera dans `/sys` :

    find /sys/devices/ -name 'hsotype' | xargs grep ''

    ... /ttyHS0/hsotype:Control
    ... /ttyHS1/hsotype:Application
    ... /ttyHS2/hsotype:Diagnostic
    ... /ttyHS3/hsotype:Modem

Le device pour le modem (3G/SMS) est `ttyHS3`.

# Désactivation du code PIN

Pour être sûr qu'on ne soit pas embêtés par le code PIN de la carte SIM, on va le désactiver.

On installe un terminal série :

    sudo apt-get install picocom

Attention : le user qui lance `picocom` doit faire partie du groupe `dialout` ou bien être `root` !

On branche la clé avec la carte SIM, puis on lance un terminal série :

    picocom /dev/ttyHS3

On tape d'abord `AT` et on doit recevoir `OK`, pour voir que la communication série avec le modem se passe bien.

Ensuite on constate qu'on nous demande le code PIN :

    AT+CPIN?
    +CPIN: SIM PIN
    OK

On regarde l'obligation de déverrouillage systématique (`1` veut dire qu'on doit entrer le code PIN à chaque branchement) :

    AT+CLCK="SC",2
    +CLCK: 1
    OK

On désactive l'obligation de déverrouillage (ici code PIN 1234)

    AT+CLCK="SC",0,"1234"
    OK

Ensuite on constate qu'on ne nous demande plus le code PIN :

    AT+CPIN?
    +CPIN: READY
    OK

On quitte picocom en faisant `Ctrl+a` puis `Ctrl+q`.

Maintenant, plus aucun code ne sera nécessaire lors des débranchements/redémarrage de la clé 3G.

# Connexion avec NetworkManager

Quand on dispose d'une interface graphique, rien de plus simple avec NetworkManager :

- faire un clic sur l'icône du NetworkManager, et sélectionner l'option `Activer la connexion mobile"

- on obtient une association avec le point réseau `Free (itinérance UMTS/HSDPA)` mais la connexion 3G n'est pas établie

- cliquer sur `Nouvelle connexion mobile...`

- cliquer sur `Suivant`, sélectionner `France`, sélectionner `Free Mobile`, cliquer sur `Suivant`

- Dans la liste déroulant, sélectionner l'option `Free-Mobile`, contrôler le nom du point d'accès "APN" qui apparait `free`.

- *Attention*: Si ce n'est pas `free` qui apparaît en tant qu'APN, sélectionner l'autre option `Free-Mobile` !

- Cliquer sur `Suivant` puis sur `Appliquer`

Lors de la première connexion, on nous demandera un mot de passe : saisir `free` puis entrée. Ce mot de passe sera mémorisé pour la suite.

Pour se connecter, cliquer sur l'option `Free Mobile Free-mobile` de NetworkManager.

Pour se déconnecter, cliquer sur l'option `Se déconnecter`.

# Connexion avec WVDIAL

Installer le package `wvdial`

    sudo apt-get install wvdial

Configurer `/etc/wvdial.conf` :

    [Dialer Defaults]
    Modem Type = Analog Modem
    ISDN = 0
    Modem = /dev/ttyHS3
    Baud = 115200
    Init1 = AT
    Init3 = ATZ
    Init4 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
    Init5 = AT+CGDCONT=1,"IP","free"
    Stupid mode = 1
    Phone = *99#
    New PPPD = yes
    Check Def Route = 1
    Username = free
    Password = free

On teste la connexion via `sudo wvdial` :

    --> WvDial: Internet dialer version 1.61
    --> Cannot get information for serial port.
    --> Initializing modem.
    --> Sending: AT
    AT
    OK
    --> Sending: ATZ
    ATZ
    OK
    --> Sending: ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
    ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
    OK
    --> Sending: AT+CGDCONT=1,"IP","free"
    AT+CGDCONT=1,"IP","free"
    OK
    --> Modem initialized.
    --> Sending: ATDT*99#
    --> Waiting for carrier.
    ATDT*99#
    CONNECT 7200000
    --> Carrier detected.  Starting PPP immediately.
    --> Starting pppd at Sun Apr 16 11:48:25 2017
    --> Pid of pppd: 14091
    --> Using interface ppp0
    --> pppd: ���v[18]z�
    --> pppd: ���v[18]z�
    --> pppd: ���v[18]z�
    --> pppd: ���v[18]z�
    --> pppd: ���v[18]z�
    --> local  IP address 10.47.118.140
    --> pppd: ���v[18]z�
    --> remote IP address 10.64.64.64
    --> pppd: ���v[18]z�
    --> primary   DNS address 212.27.40.240
    --> pppd: ���v[18]z�
    --> secondary DNS address 212.27.40.241
    --> pppd: ���v[18]z�

On interrompt la connexion via `Ctrl-C`

    ^CCaught signal 2:  Attempting to exit gracefully...
    --> Terminating on signal 15
    --> pppd: ���v[18]z�
    --> Connect time 0.6 minutes.
    --> pppd: ���v[18]z�
    --> pppd: ���v[18]z�
    --> pppd: ���v[18]z�
    --> Disconnecting at Sun Apr 16 11:49:04 2017

Configurer `/etc/network/interfaces` :

    # auto ppp0
    iface ppp0 inet wvdial

Pour activer la connexion : `sudo ifup ppp0`

Après quelques secondes, on constate que ça marche via `ppp0` :

    $ ip route

    default dev ppp0  scope link
    10.64.64.64 dev ppp0  proto kernel  scope link  src 10.191.207.59

    $ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=56 time=352 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=56 time=352 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=56 time=329 ms
    ^C
    --- 8.8.8.8 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 329.094/344.814/352.735/11.136 ms

On désactive la connexion : `sudo ifdown ppp0`

# Connexion avec PPPD

Installer le package `pppd`

    sudo apt-get install pppd

On contrôle/définit les options par défaut `/etc/ppp/options` :

    asyncmap 0
    auth
    crtscts
    lock
    hide-password
    modem
    lcp-echo-interval 30
    lcp-echo-failure 4
    noipx

On définit le script de discussion avec le provider `/etc/ppp/chat/freemobile.chat` :

    #ECHO ON
    ABORT 'BUSY'
    ABORT 'ERROR'
    ABORT 'NO ANSWER'
    ABORT 'NO CARRIER'
    '' AT
    OK ATZ
    OK 'ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0'
    OK AT+CGDCONT=1,"IP","free"
    OK ATDT*99#

On définit les options pour le fournisseur `/etc/ppp/peers/freemobile` :

    115200
    connect 'chat -v -t 60 -f /etc/ppp/chat/freemobile.chat'
    noauth

Pour débugger ce qui se passe : `sudo tail -f /var/log/message`

On active la connexion : `pon freemobile`

    Apr 16 12:11:00 localhost pppd[15397]: pppd 2.4.6 started by root, uid 0
    Apr 16 12:11:01 localhost chat[15399]: abort on (BUSY)
    Apr 16 12:11:01 localhost chat[15399]: abort on (ERROR)
    Apr 16 12:11:01 localhost chat[15399]: abort on (NO ANSWER)
    Apr 16 12:11:01 localhost chat[15399]: abort on (NO CARRIER)
    Apr 16 12:11:01 localhost chat[15399]: send (AT^M)
    Apr 16 12:11:01 localhost chat[15399]: expect (OK)
    Apr 16 12:11:01 localhost chat[15399]: AT^M^M
    Apr 16 12:11:01 localhost chat[15399]: OK
    Apr 16 12:11:01 localhost chat[15399]:  -- got it
    Apr 16 12:11:01 localhost chat[15399]: send (ATZ^M)
    Apr 16 12:11:01 localhost chat[15399]: expect (OK)
    Apr 16 12:11:01 localhost chat[15399]: ^M
    Apr 16 12:11:01 localhost chat[15399]: ATZ^M^M
    Apr 16 12:11:01 localhost chat[15399]: OK
    Apr 16 12:11:01 localhost chat[15399]:  -- got it
    Apr 16 12:11:01 localhost chat[15399]: send (ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0^M)
    Apr 16 12:11:02 localhost chat[15399]: expect (OK)
    Apr 16 12:11:02 localhost chat[15399]: ^M
    Apr 16 12:11:02 localhost chat[15399]: ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0^M^M
    Apr 16 12:11:02 localhost chat[15399]: OK
    Apr 16 12:11:02 localhost chat[15399]:  -- got it
    Apr 16 12:11:02 localhost chat[15399]: send (AT+CGDCONT=1,"IP","free"^M)
    Apr 16 12:11:02 localhost chat[15399]: expect (OK)
    Apr 16 12:11:02 localhost chat[15399]: ^M
    Apr 16 12:11:02 localhost chat[15399]: AT+CGDCONT=1,"IP","free"^M^M
    Apr 16 12:11:02 localhost chat[15399]: OK
    Apr 16 12:11:02 localhost chat[15399]:  -- got it
    Apr 16 12:11:02 localhost chat[15399]: send (ATDT*99#^M)
    Apr 16 12:11:02 localhost pppd[15397]: Serial connection established.
    Apr 16 12:11:02 localhost pppd[15397]: Using interface ppp0
    Apr 16 12:11:02 localhost pppd[15397]: Connect: ppp0 <--> /dev/ttyHS3
    Apr 16 12:11:03 localhost pppd[15397]: PAP authentication succeeded
    Apr 16 12:11:07 localhost pppd[15397]: Could not determine remote IP address: defaulting to 10.64.64.64
    Apr 16 12:11:07 localhost pppd[15397]: local  IP address 10.103.160.93
    Apr 16 12:11:07 localhost pppd[15397]: remote IP address 10.64.64.64
    Apr 16 12:11:07 localhost pppd[15397]: primary   DNS address 212.27.40.240
    Apr 16 12:11:07 localhost pppd[15397]: secondary DNS address 212.27.40.241

On constate que ça marche via `ppp0` :

    $ ip route

    default dev ppp0  scope link
    10.64.64.64 dev ppp0  proto kernel  scope link  src 10.191.207.59

    $ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=56 time=107 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=56 time=126 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=56 time=125 ms
    ^C
    --- 8.8.8.8 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 126.383/140.820/168.964/19.905 ms

Pour désactiver la connexion : `poff`

    Apr 16 12:45:20 localhost pppd[2493]: Terminating on signal 15
    Apr 16 12:45:20 localhost pppd[2493]: Connect time 0.5 minutes.
    Apr 16 12:45:20 localhost pppd[2493]: Sent 480 bytes, received 252 bytes.
    Apr 16 12:45:20 localhost pppd[2493]: Connection terminated.
    Apr 16 12:45:21 localhost pppd[2493]: Exit.

Configurer `/etc/network/interfaces` :

    # auto ppp0
    iface ppp0 inet ppp
        provider freemobile

Pour activer la connexion : `sudo ifup ppp0`

On désactive la connexion : `sudo ifdown ppp0`

## Routage restreint

Si on ne souhaite pas que la route par défaut soit utilisée via la connexion 3G, mais plutôt définir des routes spécifiques :

- d'abord ajouter `nodefaultroute` à `/etc/ppp/peers/freemobile`

- ensuite, ajouter des lignes `post-up` et `pre-down` à la définition de l'interface `ppp0` dans `/etc/network/interfaces` :

Par exemple :

    iface ppp0 inet ppp
        provider freemobile
        post-up sleep 10 && /sbin/ip route add 8.8.8.8 dev ppp0
        pre-down /sbin/ip route del 8.8.8.8 dev ppp0

Remarque: ici le `sleep 10` sert à attendre que l'interface `ppp0` arrive avant d'essayer d'ajouter des routes dessus.

On affiche le routage une fois la connexion établie : `ip route`

    default via 192.168.1.1 dev eth0
    8.8.8.8 dev ppp0  scope link
    10.64.64.64 dev ppp0  proto kernel  scope link  src 10.92.216.131
    192.168.1.0/24 dev eth0  proto kernel  scope link  src 192.168.1.3  metric 202

Ici, seul le trafic vers 8.8.8.8 partira via la connexion 3G (latence élevée) et le reste du trafic partira via la route par défaut :

    $ ping -c 3 8.8.8.8

    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=59 time=628 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=59 time=506 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=59 time=316 ms

    --- 8.8.8.8 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 316.072/483.529/628.004/128.381 ms

    $ ping -c 3 8.8.4.4

    PING 8.8.4.4 (8.8.4.4) 56(84) bytes of data.
    64 bytes from 8.8.4.4: icmp_seq=1 ttl=58 time=2.34 ms
    64 bytes from 8.8.4.4: icmp_seq=2 ttl=58 time=1.77 ms
    64 bytes from 8.8.4.4: icmp_seq=3 ttl=58 time=2.48 ms

    --- 8.8.4.4 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 1.773/2.203/2.488/0.309 ms

De cette manière, on peut restreindre le trafic qui consomme le forfait 3G de l'abonnement !
