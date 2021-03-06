Page du projet: https://github.com/lpefferkorn/pppoesk

Qu'est-ce que c'est ? 
=====================

PPPoE session killer

pppoesk est un petit programme en C permettant de terminer une session PPPoE fantôme de manière automatique.

Installation
============

pppoesk nécessite les bibliothèques libnet et libpcap, ainsi que les autotools.

     sh autogen.sh
     ./configure
     make && make install
     
PPPoE
===== 

PPPoE est un protocole qui permet d'établir une connexion entre le concentrateur d'accès de votre FAI et votre équipement réseau. Il encapsule successivement les protocoles PPP puis IP.

Une session PPPoE s'établie de la manière suivante: rfc2516

   * envoi par votre équipement d'un paquet PADI (PPPoE Active Discovery Initiation)
   * réponse du(des) concentrateur(s) par un paquet PADO (PPPoE Active Discovery Offer)
   * choix du concentrateur par un paquet PADR (PPPoE Active Discovery Request)
   * confirmation par le concentrateur choisi avec un paquet PADS (PPPoE Active Discovery Session-confirmation) puis choix d'un identiant unique à la session. 

A présent la session PPPoE est établie et est unique par les 3 paramètres suivants :

   * adresse MAC du concentrateur
   * adresse MAC de votre équipement réseau
   * identifiant de session 

Clore la session peut être fait au niveau PPP ou au niveau PPPoE avec un paquet PADT (Terminate) qui doit être envoyé au concentrateur d'accès.

Session fantôme 
===============

Durant la session PPPoE, il peut arriver que celle-ci soit brutalement coupée (mauvaise qualité de la ligne,...), sans que le concentrateur côté FAI en soit informé (il ne reçoit pas le paquet de fin de session PADT)

Du point de vue de ce dernier, la session est toujours établie. Un certain laps de temps est alors nécessaire au concentrateur pour clore la session fantôme, et ainsi permettre à une nouvelle session d'être établie.

Avec les paquets cela donne :

   * client : envoi d'un paquet PADI
   * concentrateur : réponse avec un paquet PADO
   * client : choix du concentrateur avec un paquet PADR
   * concentrateur : hum, il existe déjà une session, je ne réponds pas
   * client : ... 

Ce délai dépends apparement du paramétrage du concentrateur d'accès, et est au maximum de 24h (la fameuse déconnexion des 24h est faite pour cela)

Avec mon FAI il me faut attendre en général 1h ou 2.

Fonctionnement de pppoesk 
=========================

Forge un paquet PPPoE de type PADT (Terminate) avec les paramètres de la session PPPoE :

   * adresse MAC côté client
   * adresse MAC côté FAI
   * identifiant de session 

puis envoi le paquet afin de clore la session du côté du concentrateur d'accès.

Il est tout à fait possible de le faire manuellement avec la commande pppoe et les paramètres -p -e comme décrit dans son manuel :

>-k     Causes  pppoe to terminate an existing session by sending a PADT
>       frame, and then exit.  You must use the -e option in conjunction
>       with  this  option  to specify the session to kill.  This may be
>       useful for killing sessions when a buggy peer does  not  realize
>       the session has ended

Seulement il faut récupérer les adresses MAC et l'identifiant de session (soit dans les logs, soit avec tcpdump), puis taper la commande pppoe avec les données adéquates.

pppoesk fait cela automatiquement :

   * capture le traffic sur l'interface ethernet reliée au modem
   * récupére les adresses MAC et l'identifiant de session
   * forge le paquet PPPoE de type PADT adéquat
   * envoi le paquet 

Résultat 
========

La session PPPoE est close côté concentrateur d'accès, et il est possible d'en rétablir une immédiatement.

Notes
===== 

Bien qu'il soit possible de faire la même chose de manière relativement aisée avec un script shell, j'ai écrit ce programme pour simplifier drastiquement cette procédure.

J'en ai aussi profité pour améliorer mes connaissances en C et découvrir les bibliothèques libpcap et libnet, permettant respectivement de sniffer des paquets et d'en forger. 

N'ayant plus d'accès PPPoE, je ne peux plus tester le bon fonctionnement de pppoesk.
