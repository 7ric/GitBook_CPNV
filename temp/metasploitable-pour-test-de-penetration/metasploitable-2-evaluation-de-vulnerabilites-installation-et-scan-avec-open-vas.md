# Metasploitable 2 – Évaluation de vulnérabilités, installation et scan avec Open-VAS

## 

[LinuxFrench](https://linuxfrench.wordpress.com/author/linuxfrench/)[20 septembre 2016](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/)[Hacking\_Training](https://linuxfrench.wordpress.com/category/hacking_training/), [Linux](https://linuxfrench.wordpress.com/category/linux/), [Metasploitable 2](https://linuxfrench.wordpress.com/category/metasploitable-2/), [netstat](https://linuxfrench.wordpress.com/category/netstat/), [nmap](https://linuxfrench.wordpress.com/category/nmap/), [Open-VAS](https://linuxfrench.wordpress.com/category/open-vas/), [réseau](https://linuxfrench.wordpress.com/category/reseau/), [Sécurité informatique](https://linuxfrench.wordpress.com/category/securite-informatique/), [Sécurité système](https://linuxfrench.wordpress.com/category/securite-systeme/)

### Navigation des articles

[Précédent](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-vsftpd/)[Suivant](https://linuxfrench.wordpress.com/2016/09/24/hacking-unreal-ircd-3-2-8-1-avec-metasploitable-2/)![](https://linuxfrench.files.wordpress.com/2016/09/metasploitable-2-vulnerability-assessment-2156x1032-702x336.jpg)

Bonjour à tous.

Une évaluation de la vulnérabilité est un élément crucial dans tous les tests de pénétration. Dans cette partie du tutoriel, nous allons évaluer les vulnérabilités disponibles sur le côté réseau de la machine virtuelle Metasploitable 2. Dans le tutoriel précédent « Metasploitable – Énumération réseau avec Nmap » , nous avons appris que la machine Metasploitable 2 contient beaucoup de vulnérabilités. Nous avons collecté des informations précieuses sur notre hôte-cible que nous allons utiliser pour trouver des vulnérabilités connues à la fois en ligne et hors ligne. Dans ce tutoriel, nous allons examiner quelques façons différentes pour effectuer une analyse de vulnérabilité. Nous allons être amener à chercher manuellement des exploits, utiliser des outils d’analyse tels que Nmap avec des scripts et nous allons examiner l’utilisation de scanners de vulnérabilités automatisés comme Open-Vas. Chaque technique de balayage et leur méthode ont leurs propres avantages et inconvénients. C’est ce que nous verrons dans ce tutoriel.  
  
Comme mentionné précédemment, il existe de nombreuses façons d’effectuer une analyse de vulnérabilité, de recherche manuelle par le biais d’exploiter une base de données avec des test entièrement automatique avec des outils comme ‘Open-Vas’ et le scanner de vulnérabilité ‘Nessus’  . L’analyse des vulnérabilités avec des outils automatisés est une façon très agressive d’un balayage de vulnérabilité car il faut beaucoup de demandes et de la circulation pour terminer ce genre d’analyses. Dans certains cas, la grande partie du trafic se fait bloquer \(DOS\) par les hôtes et des services ciblés de sorte qu’il est conseillé d’être prudent lors de l’utilisation de ces types d’outils. Méfiez-vous d’utiliser uniquement ces scans de vulnérabilité sur les hôtes que vous avez la permission de numériser. Lorsque vous utilisez des outils automatisés pour la numérisation de la vulnérabilité, il est toujours sage d’utiliser plusieurs outils pour exclure les faux positifs. De plus, il est important de maîtriser également les moyens manuels d’analyse de  vulnérabilité afin que vous ne soyez pas trop dépendants des scanners automatisés.

[![2e76eb7](https://linuxfrench.files.wordpress.com/2016/09/2e76eb7.png?w=1136&h=429)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/2e76eb7/)[![edb-2015-theme-logo641](https://linuxfrench.files.wordpress.com/2016/09/edb-2015-theme-logo641.png?w=602&h=181)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/edb-2015-theme-logo641/)[![logo\_nmap](https://linuxfrench.files.wordpress.com/2016/09/logo_nmap.jpg?w=234&h=181)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/logo_nmap/)[![msf](https://linuxfrench.files.wordpress.com/2016/09/msf.png?w=292&h=181)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/msf/)  
**Metasploitable 2 – Énumération d’informations :**

Commençons cette évaluation de vulnérabilité en regardant ce que nous savons déjà sur la machine à partir de la phase d’énumération précédente Metasploitable 2– Il est en cours d’exécution sur un système d’exploitation  Linux 2.6.9 – 2.6.33  
– Le nom du serveur est METASPLOITABLE.  
– Il y a 35 comptes d’utilisateurs disponibles.  
– L’utilisateur ‘msfadmin’ est le compte administrateur.  
– Il n’y a pas de date d’expiration pour le mot de passe du compte ‘msfadmin’.  
– Nous savons quels services sont en cours d’exécution, les versions de ces services et sur quel port ils écoutent.  
– Il y a plusieurs serveurs dont un serveur Web et SQL en cours d’exécution sur la machine.

Puis n’oublions pas toutes les informations que nous a fourni le scan d’Nmap :

| **Service** | **Port** | **Status** |
| :--- | :--- | :--- |
| Vsftpd 2.3.4 | 21 | Open |
| OpenSSH 4.7p1 Debian 8ubuntu 1 \(protocol 2.0\) | 22 | Open |
| Linux telnetd service | 23 | Open |
| Postfix smtpd | 25 | Open |
| ISC BIND 9.4.2 | 53 | Open |
| Apache httpd 2.2.8 Ubuntu DAV/2 | 80 | Open |
| A RPCbind service | 111 | Open |
| Samba smbd 3.X | 139 & 445 | Open |
| 3 r services | 512, 513 & 514 | Open |
| GNU Classpath grmiregistry | 1099 | Open |
| Metasploitable root shell | 1524 | Open |
| A NFS service | 2048 | Open |
| ProFTPD 1.3.1 | 2121 | Open |
| MySQL 5.0.51a-3ubuntu5 | 3306 | Open |
| PostgreSQL DB 8.3.0 – 8.3.7 | 5432 | Open |
| VNC protocol v1.3 | 5900 | Open |
| X11 service | 6000 | Open |
| Unreal ircd | 6667 | Open |
| Apache Jserv protocol 1.3 | 8009 | Open |
| Apache Tomcat/Coyote JSP engine 1.1 | 8180 | Open |

Beaucoup de ces services contiennent des vulnérabilités connues qui peuvent être exploitées. L’étape suivante est de savoir quels sont les services vulnérables et de recueillir des informations sur la façon dont ils peuvent être exploités. Il existe plusieurs sources qui peuvent être utilisées pour déterminer si un service est vulnérable ou non. Les sources les plus populaires et sont ‘[exploit-db de Offensif Security](https://www.exploit-db.com/)‘ et la base de données Open Source des vulnérabilités \([OSVDB](https://blog.osvdb.org/)\). Nous chercherons aussi sur ‘[searchsploit](https://github.com/offensive-security/exploit-database/blob/master/searchsploit)‘, une base de données hors ligne exploit inclus avec Kali Linux. ‘Searchsploit’ est une grande source en ligne lors d’évaluation de vulnérabilité, car il contient beaucoup d’informations sur les vulnérabilités connues et contient aussi des codes d’exploitation.

**La vulnérabilité de VSFTPD :**

Bien qu’un tutoriel sur l’exploitation de la vulnérabilité de VSFTPD a déjà été traité dans l’article précédent, je me dois pour ce tutoriel d’y revenir un minimum.  
  
Pour déterminer les vulnérabilités du service VSFTPD v2.3.4, nous allons consulter plusieurs ressources. Lorsque nous cherchons sur Google des vulnérabilités connues pour ce service, il nous ramène sur une vulnérabilité connue de type backdoor  qui a été introduit dans un téléchargement du logiciel dans la version 2.3.4 :

=&gt; [la vulnérabilité connue](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor)

Cela signifie que seule une partie des installations de VSFTPD v2.3.4 sera vulnérable à ce backdoor qui a été ajouté volontairement, puis a été supprimé quelques jours plus tard. Néanmoins, il serait bien d’essayer de voir si l’installation sur la machine Metasploitable 2 est vulnérable. Il y a aussi un module Metasploit disponible pour exploiter cette vulnérabilité mais ça nous l’avons dans le tutoriel précédent.  
  
**CVE:** CVE-2011-02523

**OSVDB:** 73573

VSFTPD – Le script Nmap :

Nous pourrions lancer Metasploit et voir si le service en cours d’exécution sur la machine Metasploitable 2 est vulnérable, mais il y a une autre façon et nous allons l’utiliser. Pour déterminer si le service FTP contient un backdoor avec la possibilité d’obtenir un ‘shell’, nous pouvons utiliser un script Nmap. Le scripts Nmap est « ftp-vsftpd-backdoor » pour VSFTPD v2.3.4. Lançons note scan Nmap avec le script afin e numériser notre hôte-cible en utilisant la commande suivante :

| **\#** **nmap –-script ftp-vsftpd-backdoor –p 21 \[target host\]** **\#** **nmap –script ftp-vsftpd-backdoor -p 21 192.168.1.14** |
| :--- |


Jetons un oeil à ce que nous renvois le script Nmap pour déterminer si le service VSFTPD est en cours d’exécution et si il est vulnérables :

![nmap-1.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-12.jpg?w=1140)

Vous pourrez trouver plus d’informations sur ce script =&gt; [ici](https://nmap.org/nsedoc/scripts/ftp-vsftpd-backdoor.html)

**Vulnérabilité – Unreal IRCD :**

Jetons un coup d’œil au service ‘Unreal IRCD’.Le service IRC est connu pour supporter de nombreuses plates-formes. La seule chose que nous savons à propos de ce service jusqu’à maintenant grâce à notre analyse avec Nmap, c’est qu’il est en cours d’exécution sur le port 6667 . Nous ne savons pas d’autres détails tels que la version qui nous aiderait beaucoup dans la détermination et la recherche de vulnérabilité. Une méthode commune pour déterminer la version d’un service, on utilise la technique ‘de la bannière’ . Netcat est un outil qui peut être utilisé à cet effet \(parmi bien d’autres bien sur\). Voyons voir si nous pouvons saisir une bannière à l’aide Netcat :

| **\#** **nc \[IP CIBLE\] 6667** **\#** **nc 192.168.1.14 6667** |
| :--- |


Malheureusement, Netcat ne nous a pas retourné de bannière sur notre tentative de connexion sur le serveur IRC :

![nc-1](https://linuxfrench.files.wordpress.com/2016/09/nc-1.jpg?w=1140)

_\(sachez que normalement, certains d’entre vous obtiendrons plus de résultat depuis cette commande. Pour ma part, je manque de ressource au niveau mémoire et du coup, Netcat ne sait terminer sa commande.\)_

Revenons à Nmap et utilisons la commande suivante pour déclencher une analyse complète sur le port 6667 :

| **\# nmap –A –p 6667 \[IP CIBLE\]** **\# nmap -A -6667 192.168.1.4** |
| :--- |


Nmap nous a retourné le numéro de version du service ‘Unreal ircd’. Lorsque nous cherchons sur Google si il existe une faille pour cette version, nous trouvons rapidement que cette version contient un backdoor :

=&gt; [source](https://www.rapid7.com/db/modules/exploit/unix/irc/unreal_ircd_3281_backdoor)

Il y a aussi un script NMap disponibles pour numériser les hôtes-cibles le backdoor de cette version Unreal IRCD. Utilisez la commande suivante avec Nmap sur notre hôte-cible :

| **\# nmap –sV –-script irc-unrealircd-backdoor –p 6667 \[IP CIBLE\]** **\#** **nmap -sV –script irc-unrealircd-backdoor -p 6667 192.168.1.14** |
| :--- |


La sortie du scripts nous  indique que notre hôte-cible est probablement vulnérable ….ou non. Le script émet une commande sur notre hôte-cible, mais comme il n’y a aucun moyen de retourner la sortie de la commande sur notre session de terminal, nous ne pouvons pas être sûr à 100% que l’hôte est vulnérable en utilisant le script de cette manière. Pour cela …. voir le tutoriel précèdent qui montre comment exploiter ette faille avec nmap, Metasploit et Netcat.

![nmap-unrealircd.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-unrealircd.jpg?w=1140)

Jusqu’à présent, nous ne sommes pas sûr à 100% si le service ‘Unreal IRCD 3.2.8.1’ est vulnérable. Nous ne pouvons que soupçonner qu’il est.

**Évaluation de vulnérabilités en utilisant ‘exploit-db’ :**

Une autre grande source pour trouver des vulnérabilités connues est la base de données maintenue par ‘Offensive Security Exploit’. Exploit-db offre une énorme quantité et de détails sur des exploits, documents, shellcodes et peuvent être recherchés en utilisant le CVE ou les identificateurs de OSVDB. Lorsque nous cherchons la base de données pour exploiter la version vulnérable de ‘Unreal IRCD’, on constate qu’il y a plusieurs exploits proposés :

![exploit-db.jpg](https://linuxfrench.files.wordpress.com/2016/09/exploit-db.jpg?w=1140)

En effet, d’après le site ‘exploit-db’, cette version de ‘Unreal IRCD’ pour Linux semble contenir de multiples vulnérabilités :

CVE: 2010-2075 : [Backdoor Command Execution \(Metasploit\)](https://www.exploit-db.com/exploits/16922/)

CVE: 2010-2075 : [Remote Downloader/Execute Trojan](https://www.exploit-db.com/exploits/13853/)

Le premier exploit est une vulnérabilité qui ne vise que le système d’exploitation Windows, celui-ci ne sont donc pas utilisables pour notre machine Metasploitable 2. Lorsque l’on clique sur les vulnérabilités trouvées, nous pouvons télécharger le code d’exploitation pour exploiter la vulnérabilité. Souvent, Exploit-db contient également une version vulnérable du logiciel qui peut être téléchargé et être utilisé à des fins de test dans un environnement contrôlé tel que nous le faisons ici.

Un code d’exploitation est souvent écrit dans les langages de programmation tels que le Ruby \(le Framework de Metasploit par exemle\), C, Perl ou en Python. S’il vous plaît noter que la gratuité et l’offre de ses code d’exploitation a souvent besoin de petites modifications \(votre système d’exploitation , votre IDE,ect\) pour être utilisé avec succès sur une cible. Cela vous oblige à avoir un minimum une certaine connaissance et une expérience en programmation pour être en mesure de modifier le code. De nombreux chercheurs en sécurité veulent éviter que tout le monde \(Surtout les script kiddies\) peuvent utiliser les codes d’exploitations de la base de données d’exploit-db sans aucune connaissance préalable du sujet et ben souvent la preuve du concepts \(POC\). Bien sûr, cela ne vaut pas quand il y a un module Metasploit disponible qui peut être utilisé hors de la base sans aucune modification.

**Soyez prudent avec le téléchargement des exploits !**  
  
Soyez prudent lorsque vous téléchargez des exploits d’autres ressources que sur Exploit-DB . Exploits peuvent contenir des shellcode-malveillants qui peuvent nuire à votre système, votre vie privée ou à les deux. Bien évidemment, quoi de plus simple d’introduire un code malveillants dans un exploits dit ‘safe’ ? Rien bien sur. Pour éviter ce genre de comportement inattendu, il est conseillé de vérifier le code et voir si il fonctionne comme annoncé. Par exemple, lorsque vous rencontrez un exploit pour un contrôle à distance qui ne pas utiliser les sockets-réseau, il est probablement pas fait pour une exploitation à distance et donc, cet indication devrez vous rendre prudent lorsque vous le  compilerez et l’exécuterez. Il y a quelque temps, un exploit concernant le serveur Web et la dernière version d’Apache comme un exploit 0Day non corrigé. En analysant le code, il c’est avéré que celui contenait un code malveillant qui permettez à l’attaquant d’obtenir un ‘shellcode’ avec les droit super-utilisateurs sur votre machine.  


![1143757](https://linuxfrench.files.wordpress.com/2016/09/1143757.jpg?w=1140)

 **Databases CVE :**

Une autre grande source pour rechercher des vulnérabilités et des informations est la base de données CVE. Voici le lien =&gt; [ici](https://cve.mitre.org/cve/cve.html)[![cve](https://linuxfrench.files.wordpress.com/2016/09/cve1.jpg?w=424&h=193)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/cve-2/)[![cve-2](https://linuxfrench.files.wordpress.com/2016/09/cve-2.jpg?w=363&h=193)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/cve-2-2/)[![cve-3](https://linuxfrench.files.wordpress.com/2016/09/cve-3.jpg?w=341&h=193)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/cve-3/)[![cve-4](https://linuxfrench.files.wordpress.com/2016/09/cve-4.jpg?w=588&h=262)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/cve-4/)[![cve-5](https://linuxfrench.files.wordpress.com/2016/09/cve-5.jpg?w=544&h=262)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/cve-5/)

Quand nous  faisons un recherche sur une vulnérabilité pour le CVE 2010-2075 , nous pouvons trouver une liste des sources avec des rapports de divulgations complètes et quelques autres liens vers des informations qui pourraient nous aider à obtenir une meilleure compréhension de la vulnérabilité et donc, de la façon de l’exploiter.

Une autre grande source pour les évaluations de vulnérabilité est le site de détails CVE. Nous pouvons chercher cette base de données pour les logiciels et services spécifiques pour déterminer si elles contiennent des vulnérabilités connues. Quand nous faisons une recherche pour le service ‘Proftpd 1.3.1’, nous trouvons une liste de bogues et de vulnérabilités connues applicables à cette version spécifique. Y compris certaines vulnérabilités avec une cote de difficulté :

![CVE-proftpd.jpg](https://linuxfrench.files.wordpress.com/2016/09/cve-proftpd.jpg?w=1140)

**SearchSploit :**

Un autre grand outil Open Source pour trouver des vulnérabilités et des exploits est searchSploit. SearchSploit est inclus avec Kali Linux par défaut. Utilisez la commande suivante pour rechercher des vulnérabilités pour le service ‘Unreal Ircd’ en utilisant SearchSploit :

| **\#** **searchsploit unreal ircd** |
| :--- |


![searchsploit.jpg](https://linuxfrench.files.wordpress.com/2016/09/searchsploit.jpg?w=1140)

Nous ne pouvons afficher le contenu des fichiers avec le terminal en utilisant la commande ‘cat’ comme ceci :

| **\#** **cat /usr/share/exploitdb/platforms/linux/remote/16922.rb** _et le concaténer dans un fichier texte par exemple en y rajoutant :_ **\#** **cat /usr/share/exploitdb/platforms/linux/remote/16922.rb &gt; result.txt** |
| :--- |


![cat](https://linuxfrench.files.wordpress.com/2016/09/cat.jpg?w=1140)![cat-1](https://linuxfrench.files.wordpress.com/2016/09/cat-1.jpg?w=1140)

**Open-VAS – Un scanner de vulnérabilité :**

Jusqu’à présent, nous avons seulement utilisé Nmap et quelques techniques manuelles pour découvrir des vulnérabilités connues pour notre évaluation de vulnérabilité. Il existe d’autres façons de tester rigoureusement une foule de vulnérabilités en utilisant des scanners de vulnérabilité hautement automatisés tels que Open-Vas et Nessus. S’il vous plaît, soyez conscient que l’utilisation de ces scanners va générer beaucoup de trafic à un point que vous pourriez faire un DOS sur votre hôte-cible. De plus, utiliser ce type de scanner sur des machines dont vous n’avez pas l’autorisation d’audit sur celles ci, est totalement illégal. C’est pour cela que des Lab comme Metasploitable \(entre autres\) ont été crée. Enfin, vous voilà prévenu.

Maintenant, avant de continuer, nous allons devoir faire un petit détour afin d’installer et de mettre en route notre scanner Open-VAS. Allons y ….

Open-VAS – Installation :  
  
Commençons donc par installer OpenVAS et exécuter les commandes suivantes dans une session de terminal pour télécharger et installer OpenVAS :

| **\#** **apt-get install open-vas** **\#** **openvas-setup** |
| :--- |


![openvas-1.jpg](https://linuxfrench.files.wordpress.com/2016/09/openvas-1.jpg?w=1140)

Les dernières commandes met en place OpenVAS et est la synchronisation de l’alimentation NVT avec sa collection sur votre machine. En fonction de votre vitesse de connexion, cela peut prendre un certain temps pour se terminer.

Lorsque le processus d’installation est terminée, sur la dernière ligne, il vous sera présenté un long mot de passe . Ce mot de passe est utilisé pour accéder à l’interface web d’OpenVAS. Enregistrer-le quelque part et changer le après la première connexion.

![openvas-5.jpg](https://linuxfrench.files.wordpress.com/2016/09/openvas-5.jpg?w=1140)

Lorsque le processus d’installation est terminée, le gestionnaire Web d’OpenVAS, le scanner et ses services sont à l’écoute sur les ports 9390, 9391, 9392 et sur le port 80 \(https\). Vous pouvez utiliser la commande netstat suivante pour vérifier si ces services sont bien à l’écoute sur ses ports :

| **\#** **netstat -antp** |
| :--- |


**Netstat –antp : commande explication**  
-a : all \(tout\)  
-n  : Montre l’IP au lieu des noms d’hôte  
-t : Montre les connexions TCP actives  
-p : Montre les noms/id des processus

**Open-VAS – Démarrage :**

Si les service Open-VAS ne sont pas en cours OpenVAS, utiliser la commande suivante pour démarrer ces services :

| **\# openvas-start** **\#** **service apache2 start** _\(puis tapé votre IP ‘localhost’ sur le port 9392 dans votre navigateur\)_ [**https://127.0.0.1:9392**](https://127.0.0.1:9392/) |
| :--- |


![openvas-6-1](https://linuxfrench.files.wordpress.com/2016/09/openvas-6-11.jpg?w=1140)![openvas-6](https://linuxfrench.files.wordpress.com/2016/09/openvas-6.jpg?w=1140) ![openvas-7](https://linuxfrench.files.wordpress.com/2016/09/openvas-7.jpg?w=1140)Acceptez le certificat SSL auto-généré et connectez-vous avec l’utilisateur ‘admin’ et le mot de passe généré au cours du processus d’installation. L’interface Web après la connexion devrait ressembler à ceci :

![open-vas-8.jpg](https://linuxfrench.files.wordpress.com/2016/09/open-vas-8.jpg?w=1140)

![openvas-7.1.jpg](https://linuxfrench.files.wordpress.com/2016/09/openvas-7-1.jpg?w=1140)

**Open-VAS – Scanner notre hôte-cible \(Metasploitable\) :**  
  
Démarrer un scan avec OpenVAS est très facile et simple. Il suffit d’entrer le nom de l’hôte ou l’adresse IP de la cible dans le champ de démarrage rapide et appuyez sur le bouton «Start Scan». Assurez-vous que vous ayez le droit d’analyser vos des cibles avec OpenVAS et l’autorisation de numériser le tout. Open-VAS va scanner les vulnérabilités et va générer beaucoup de trafic réseau qui peut conduire à des accidents \(ou DOS. Globalement,faire un scan avec Open-VAS peut faire planter un serveur et des services sur l’hôte-cible.

Lorsque l’analyse est terminée, cliquez sur la page des rapports dans le menu « Scan Management » et jeter un œil à un aperçu des résultats de l’analyse :

Comme vous pouvez voir la machine Metasploitable est très vulnérable et contient \(dans l’ordre de la plus dangereuse à la moins\) 2 très dangereuse, 19 dangereuse, 32 moyenne et 6 vulnérabilités de gravité nominale basse. Lorsque vous cliquez sur un rapport, vous pouvez voir un aperçu plus détaillé des vulnérabilités trouvées. La liste des vulnérabilités connues est ordonnée par gravité :

[![openvas-9](https://linuxfrench.files.wordpress.com/2016/09/openvas-9.jpg?w=769&h=115)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-9/)[![openvas-10](https://linuxfrench.files.wordpress.com/2016/09/openvas-10.jpg?w=199&h=115)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-10/)[![openvas-11-2](https://linuxfrench.files.wordpress.com/2016/09/openvas-11-2.jpg?w=160&h=115)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-11-2/)[![openvas-11-3](https://linuxfrench.files.wordpress.com/2016/09/openvas-11-3.jpg?w=759&h=519)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-11-3/)[![openvas-11-4](https://linuxfrench.files.wordpress.com/2016/09/openvas-11-4.jpg?w=373&h=304)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-11-4/)[![openvas-11](https://linuxfrench.files.wordpress.com/2016/09/openvas-11.jpg?w=373&h=211)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-11/)[![openvas-12](https://linuxfrench.files.wordpress.com/2016/09/openvas-12.jpg?w=571&h=342)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-12/)[![openvas-13-1](https://linuxfrench.files.wordpress.com/2016/09/openvas-13-1.jpg?w=571&h=407)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-13-1/)[![openvas-13](https://linuxfrench.files.wordpress.com/2016/09/openvas-13.jpg?w=561&h=753)](https://linuxfrench.wordpress.com/2016/09/20/metasploitable-2-evaluation-de-vulnerabilites-et-installation-de-open-vas/openvas-13/)  
Openvas offre beaucoup plus de fonctionnalités, y compris la base de données de vulnérabilité alimentés et classés de CVE , NVT et CPE. L’outil de gestion ‘Secinfo’ offre également un joli tableau de bord montrant quelques statistiques de haut niveau sur les vulnérabilités :![openvas-14](https://linuxfrench.files.wordpress.com/2016/09/openvas-14.jpg?w=1140)![openvas-14-1](https://linuxfrench.files.wordpress.com/2016/09/openvas-14-1.jpg?w=1140) Avec le tableau de bord des vulnérabilités, nous allons conclure ce tutoriel. Nous vous conseillons de vous familiariser avec OpenVAS, les rapports et la base de données des vulnérabilités en exécutant plusieurs balayages et de comparer les résultats avec d’autres scanner de vulnérabilité. Les résultats ont été triés sur la gravité et comme vous pouvez le voir, Open-Vas a détecté un grand nombre de vulnérabilités graves. Il est sage d’utiliser plusieurs scanners de vulnérabilité afin d’exclure les faux-positifs qui peuvent se produire fréquemment pendant le balayage automatique des vulnérabilités.  
  
Jusqu’à présent, notre évaluation de la vulnérabilité a découvert un grand nombre de vulnérabilités sur la machine Metasploitable 2 pour seulement 2 services en utilisant différentes techniques. Tant que les services ‘Unreal Ircd ‘et ‘VSFTPD’ contiennent des backdoors qui peuvent être facilement exploitées à la fois manuellement et automatiquement avec Metasploit \(par exemple\). Nous avons également examiné le scanner de vulnérabilité automatique Open-VAS et nus avons remarqué un grand nombre de vulnérabilités graves._Voilà la fin de ce tutoriel. Merci de m’avoir lu et j’espère à très vite._

