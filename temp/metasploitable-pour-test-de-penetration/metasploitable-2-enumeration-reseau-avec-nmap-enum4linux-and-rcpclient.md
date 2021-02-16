# Metasploitable 2 – Enumération réseau avec Nmap, Enum4Linux & Rcpclient

[LinuxFrench](https://linuxfrench.wordpress.com/author/linuxfrench/)[15 septembre 2016](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/)[enum4linux](https://linuxfrench.wordpress.com/category/enum4linux/), [Hacking\_Training](https://linuxfrench.wordpress.com/category/hacking_training/), [Linux](https://linuxfrench.wordpress.com/category/linux/), [Metasploit](https://linuxfrench.wordpress.com/category/metasploit/), [Metasploitable 2](https://linuxfrench.wordpress.com/category/metasploitable-2/), [netdiscover](https://linuxfrench.wordpress.com/category/netdiscover/), [nmap](https://linuxfrench.wordpress.com/category/nmap/), [réseau](https://linuxfrench.wordpress.com/category/reseau/), [rcpclient / rpcclient](https://linuxfrench.wordpress.com/category/rcpclient-rpcclient/), [Sécurité informatique](https://linuxfrench.wordpress.com/category/securite-informatique/), [Sécurité système](https://linuxfrench.wordpress.com/category/securite-systeme/)

### Navigation des articles

[Précédent](https://linuxfrench.wordpress.com/2016/06/22/hollywood-simuler-une-fenetre-de-hacking-comme-au-cinema-sur-ubuntu/)[Suivant](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-vsftpd/)![](https://linuxfrench.files.wordpress.com/2016/09/metasploitable-2-enumeration-702x3361.jpg)

Bonjour à tous.Dans ce nouveau tutoriel, nous allons « énumérer » la machine virtuelle Metasploitable 2 afin de recueillir des informations utiles pour une évaluation de la vulnérabilité. Énumération en mathématiques ou dans l’informatique veut dire énumérer un certain nombre d’éléments dans un ensemble.L’énumération dans le contexte de piratage est le processus de récupération des noms d’utilisateur, des actions, des services, des annuaires du web, des groupes, des ordinateurs sur un réseau. Il est également appelé « [énumération réseau](https://www.aldeid.com/wiki/Attaques/Enumeration-Scanning/Decouverte-cartographie-reseau)« . Au cours de ce processus, nous allons également recueillir d’autres informations relatives au réseau utile pour effectuer un test de pénétration. Une partie importante du processus d’énumération Metasploitable 2 est le processus de balayage de port et d’empreintes digitales. Le balayage de Port est utilisé pour sonder un serveur ou un hôte pour découvrir si le trafic TPC est ouvert ainsi que les ports UDP. [Dactyloscopie](https://fr.wiktionary.org/wiki/dactyloscopie) est le processus d’identification des services liés à ces ports. Un outil très populaire utilisé pour le dénombrement de réseau, la numérisation de port et d’empreintes digitales est [Nmap](https://fr.wikipedia.org/wiki/Nmap) \(Network Mapper – cet outil est un véritable bijou \) que nous allons utiliser tout au long de ce tutoriel. Nous allons également utiliser un outil de recensement appelé « [enum4linux](https://labs.portcullis.co.uk/tools/enum4linux/)« . Enum4linux est un outil utilisé pour l’énumération des informations à partir d’hôtes Windows et Samba.  


[![2dlogo](https://linuxfrench.files.wordpress.com/2016/09/2dlogo.jpg?w=1136&h=498)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/2dlogo/)[![enum](https://linuxfrench.files.wordpress.com/2016/09/enum.png?w=344&h=215)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum/)[![fond-de-dactyloscopie-7057806](https://linuxfrench.files.wordpress.com/2016/09/fond-de-dactyloscopie-7057806.jpg?w=262&h=215)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/fond-de-dactyloscopie-7057806/)[![dnmap\_architecture](https://linuxfrench.files.wordpress.com/2016/09/dnmap_architecture.png?w=522&h=215)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/dnmap_architecture/)Après que nous avons terminé avec le succès de l’énumération de Metasploitable 2 \(nous ferons une évaluation de la vulnérabilité sur le côté réseau dans un prochain tutoriel\).  
Avec des informations récupérées à partir du processus d’énumération, par exemple la version du système d’exploitation et des services avec la version en cours d’exécution, nous allons examiner les vulnérabilités connues dans ces services. Nous allons utiliser la base de données Open Source de la vulnérabilité \([OSVDB](https://en.wikipedia.org/wiki/Open_Source_Vulnerability_Database)\) et les Common Vulnerabilities and Exposures \(CVE\) à cet effet. La dernière étape consiste à analyser l’hôte cible de ces vulnérabilités avec un scanner de vulnérabilité appelé « [OpenVAS](https://fr.wikipedia.org/wiki/OpenVAS) » sur Kali Linux.[![openvas-8-structure](https://linuxfrench.files.wordpress.com/2016/09/openvas-8-structure.png?w=633&h=374)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/openvas-8-structure/)[![osvdb-lens](https://linuxfrench.files.wordpress.com/2016/09/osvdb-lens.png?w=499&h=374)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/osvdb-lens/)

**Metasploitable – énumération d’hôte :**

Dans cette partie, nous allons énumérer les services, les comptes en cours d’exécution et réaliser une analyse sur les ports ouvert. Nous allons utiliser Nmap pour scanner la machine virtuelle pour les ports ouverts et comme ça nous connaitrons les empreintes digitales des services connectés. Dans ce tutoriel, nous nous concentrerons sur l’énumération du côté réseau de la machine Metasploitable 2.  
\(nous couvrirons le côté web dans un autre tutoriel où nous allons énumérer les applications Web et des répertoires, effectuer des attaques SQL et exploiter les services Web vulnérables.\)

Je suppose que vous avez déjà installé la machine virtuelle Metasploitable du tutoriel ? Il est temps de l’allumer maintenant. Lorsque vous vous connectez à l’hôte vulnérable avec « msfadmin » comme nom d’utilisateur et mot de passe, vous pouvez utiliser la commande « ifconfig » pour déterminer son adresse IP. Vous pouvez également utiliser « netdiscover » sur la machine Kali Linux pour balayer une plage d’adresses IP pour l’hôte cible. Utilisez la commande suivante dans le terminal :

| **$ sudo ifconfig eth0** **\# netdiscover -r 192.168.1.0/24** |
| :--- |


Cette commande renvoie tous les hôtes en direct sur la plage IP donnée. Soit de 192.168.11.0 à 192.168.1.0 à 255 dans notre exemple. Bien sûr, vous devez analyser la gamme IP de votre installation de votre VM pour obtenir un résultat correct.  


![ifconfig](https://linuxfrench.files.wordpress.com/2016/09/ifconfig.jpg?w=1140)![netdiscover](https://linuxfrench.files.wordpress.com/2016/09/netdiscover.jpg?w=1140)

Nous pouvons voir que notre hôte-cible Metasploitable se trouve sur l’IP 192.168.1.14.

![ifconfig - h&#xF4;te.jpg](https://linuxfrench.files.wordpress.com/2016/09/ifconfig-hc3b4te.jpg?w=1140)

**Nmap – Scan de ports et de services :**

Nous allons commencer par le port ouvert découvert après le scan avec la numérisation de l’hôte cible avec Nmap. Nous allons utiliser un scan TCP SYN à cette fin et nous allons balayer la cible pour les ports UDP ouverts. Le [scan SYN](https://nmap.org/man/fr/man-port-scanning-techniques.html) est connu comme un scan de port furtif, car il ne termine la synchronisation TCP complète. Une connexion TCP complète commence par une synchronisation à trois voies où un paquet SYN est envoyé par Nmap. Quand un port sur une machine cible est ouvert, il répondra avec un paquet SYN-ACK. Quand il n’y a pas de réponse de la cible sur le premier paquet de données SYN, que le port ou le service est fermé ou filtré par un pare-feu. La 3ème étape de ce processus, c’est la machine hôte-cible qui doit répondre avec un SYN-ACK et un paquet ACK pour terminer la négociation de la communication TCP complète.

Lorsque vous démarrez un « scan SYN » \(ou tout autre scan de port\) à partir de Nmap sans spécifier la plage de ports, Nmap va scanner seulement les 1.000 premiers ports qui sont considérés comme des ports les plus importants au lieu de tous \(soit 65,535 ports\). Pour analyser tous les ports, vous devez utiliser l’option « -p-« .  
La commande scan Nmap SYN utilise l’option « -sS » tel qu’il est utilisé dans la commande suivante pour le « SYN scan » sur le port 1 au port 65,535 :

| **\# nmap -sS -p- \[IP CIBLE\] \# nmap -sS -p- 192.168.1.14** |
| :--- |


ATTENTION ! =&gt; _Le scan Nmap SYN est souvent appelé « scan furtif » car il passe inaperçue. Cela est vrai pour les anciens pare-feu, qui ne se connectent que par connexions TCP complètes, mais pas pour les pare-feu modernes qui enregistrent également les connexions TCP inachevées._ ![nmap-1.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-1.jpg?w=1140) ![tcp-3-way-handshake](https://linuxfrench.files.wordpress.com/2016/09/tcp-3-way-handshake.png?w=1140)

**Connaître les ports vulnérables :**  
  
Ce n’est pas parce qu’un port est ouvert que cela signifie que le logiciel sous-jacent est vulnérable. Nous avons besoin de connaître la version du système d’exploitation et des services en cours d’exécution. Avec ces informations, nous pouvons déterminer s’il y a des vulnérabilités connues disponibles qui peuvent être exploitées. Le résultat de l’analyse des services et OS va nous donner les bonnes informations pour étudier l’évaluation de la vulnérabilité. Pour obtenir ces informations, nous allons lancer le scan de port avec l’option « -sV » pour la détection de la version de l’application qui tourne sur notre hôte-cible et l’option « -O » pour la détection de l’OS \(Windows et sa version, Linux ou autres\). Le scan Nmap de l’OS et des versions des applications nous permet d’avoir un résultat de scan assez complet qui nous permettra de cibler une faille spécifique sur un service  en cours d’exécution.  
Voici la commande que nous allons taper pour nos recherche d’OS et versions :

| **\#** **nmap -sS -sV -O \[IP CIBLE\]** **\# nmap -sS -sV -O 192.168.1.14** |
| :--- |


_\(Après l’exécution de cette commande, Nmap retournera une liste des ports ouverts et les services connectés\)_

![nmap-2.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-2.jpg?w=1140)

Les ports et services du résultat du scan avec Nmap renvoie un grand nombre de ports ouverts, des services d’écoute et la version du système d’exploitation. L’hôte cible est en cours d’exécution et il est basé sur un système d’exploitation Linux 2.6.9 – 2.6.33. Nous pouvons voir que l’hôte a en cours d’exécution un service SSH utilisant OpenSSH, un service telnet, un serveur web Apache 2.2.8, 2 serveurs SQL et quelques autres services. Résumons tous les services avec la version et le port dans une liste.  


* Vsftpd 2.3.4 on open port 21
* OpenSSH 4.7p1 Debian 8ubuntu 1 \(protocol 2.0\) on open port 22
* Linux telnetd service on open port 23
* Postfix smtpd on port 25
* ISC BIND 9.4.2 on open port 53
* Apache httpd 2.2.8 Ubuntu DAV/2 on port 80
* A RPCbind service on port 111
* Samba smbd 3.X on port 139 and 445
* 3 r services on port 512, 513 and 514
* GNU Classpath grmiregistry on port 1099
* Metasploitable root shell on port 1524
* A NFS service on port 2049
* ProFTPD 1.3.1 on port 2121
* MySQL 5.0.51a-3ubuntu5 on port 3306
* PostgreSQL DB 8.3.0 – 8.3.7 on port 5432
* VNC protocol v1.3 on port 5900
* X11 service on port 6000
* Unreal ircd on port 6667
* Apache Jserv protocol 1.3 on port 8009

![nmap-3.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-3.jpg?w=1140)

_\(ce qui nous fait pas mal de travaille …\)_

Bien sûr, nous savons que la machine virtuelle Metasploitable 2 est volontairement vulnérable. et donc on peut soupçonner que la plupart, sinon la totalité des services contiennent des vulnérabilités, backdoors, etc.  
Dans ce tutoriel, nous allons utiliser la tactique du dénombrement et de balayage de ports avec une évaluation de la vulnérabilité du côté du réseau. Continuons avec l’énumération d’utilisateur.

**Nmap – UDP scan :**

Jusqu’à présent, nous avons seulement scanné les ports TCP ouverts, ce qui est la valeur par défaut pour Nmap contrairement aux  ports UDP ouverts. Nous allons utiliser la commande suivante pour lancer une analyse UDP :

Nous pouvons également utiliser l’option « -p » pour définir les ports à numériser. Le scan UDP prendra un peu plus de temps à se faire qu’un balayage de TCP. Nmap retourne les informations suivantes sur les ports UDP ouverts qu’il a trouvé :  


| **\#** **nmap -sU \[IP CIBLE\]** **\#** **nmap -sU 192.168.1.14** |
| :--- |


![nmap-4.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-4.jpg?w=1140)

_\(s’il vous plaît noter que les scans UDP peuvent causer beaucoup de faux-positifs. Les faux positifs peuvent se produire parce que l’UDP ne dispose pas l’équivalent d’un paquet TCP SYN. Quand un port UDP numérisé est fermé, le système répondra avec un message inaccessible du port ICMP. Une telle absence indique que le port UDP est ouvert pour de nombreux outils d’analyse. Quand un pare-feu bloque est présent sur l’hôte-cible, le message ICMP est inaccessible sur tous les ports UDP qui pourraient être ouvert. Lorsque les pare-feu bloque un seul port du scan, les autres ports seront également signaler à tort que le port est ouvert.\)_

**Metasploitable – Énumération des utilisateurs :**

L’énumération d’utilisateur est une étape importante dans tous les tests de pénétration et doit être fait très soigneusement. En énumérant les utilisateurs, le pentesteur arrive à voir les utilisateurs qui ont accès au serveur et les utilisateurs qui existent sur le réseau. Un autre but de l’énumération d’utilisateur est d’essayer avoir accès à la machine en utilisant des techniques de [force brute](https://fr.wikipedia.org/wiki/Attaque_par_force_brute). Étant donné que le nom d’utilisateur est déjà connu par le pentesteur, la seule chose qui lui reste à faire et de découvrir le mot de passe. Il y a plusieurs façons d’énumérer les utilisateurs sur un système Linux. Nous allons examiner 2 méthodes différentes :

– Énumération utilisateurs en utilisant un script Nmap nommé « [smb-enum-users](https://nmap.org/nsedoc/scripts/smb-enum-users.html)« .  
– Énumération utilisateurs par le biais d’une des sessions nulles utilisant « [rpcclient](https://www.samba.org/samba/docs/man/manpages/rpcclient.1.html)« .

Commençons par énumèrent les utilisateurs en utilisant le script Nmap.

**Énumération utilisateurs avec Nmap :**

Afin d’énumérer les comptes utilisateurs disponibles sur la machine cible, nous allons utiliser le script Nmap suivant: « smb-enum-users ».  
Nous pouvons exécuter le script Nmap en utilisant la commande suivante :

| \# **nmap –script smb-enum-users.nse –p 445 \[IP CIBLE\]** **\#** **nmap -script smb-enum-users.nse -p 445 192.168.1.14** |
| :--- |


La sortie du script nous donne une longue liste d’utilisateurs disponibles sur l’hôte :

![nmap-enum-1](https://linuxfrench.files.wordpress.com/2016/09/nmap-enum-1.jpg?w=1140)![nmap-enum-2](https://linuxfrench.files.wordpress.com/2016/09/nmap-enum-2.jpg?w=1140)

Comme vous pouvez le voir il y a beaucoup de noms d’utilisateur sur la machine Metasploitable 2. Parmi eux, un grand nombre de comptes,de service et le compte administrateur qui est nommé « msfadmin ». Jetons un coup d’oeil à la seconde méthode pour récupérer une liste de comptes d’utilisateurs à partir du serveur Metasploitable 2 en utilisant une session nulle sur le serveur Samba.

**Énumération de comptes avec RPCCLIENT :**

Rpcclient est un outil Linux qui est utilisé pour exécuter des fonctions « MS-RPC » côté client. Une session nulle est une connexion avec un serveur samba ou SMB qui ne nécessite pas d’authentification par mot de passe. Aucun nom d’utilisateur ou mot de passe est nécessaire pour mettre en place la connexion. C’est pour cela qu’on l’appel « session null ». L’allocation des sessions nul a été activé par défaut sur les systèmes existants, mais a été désactivé à partir de Windows XP SP2 et Windows Server 2003. Mais nous avons vu suite au résultat obtenu par nos scan que le port 445 était ouvert sur notre hôte-cible.

Ouvrons une nouvelle fenêtre de terminal et mettons en en place une « session nulle » avec le serveur samba sur la machine Metasploitable 2 en utilisant la commande suivante :

| **\#** **rpcclient -U «  » \[IP CIBLE\]** **\# rpcclient -U «  » 192.168.1.14** |
| :--- |


_\(L’option -U définit un nom d’utilisateur « nulle » suivi de l’adresse IP de notre hôte-cible \(VM Metasploitable 2\). On vous demandera un mot de passe, appuyez sur Entrée pour continuer.\)_

![nmap-rpcclient.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-rpcclient.jpg?w=1140)

  
La ligne de commande va changer une fois connecté depuis le client RPC. Le prompt du « rpcclient » sera indiqué par un « rpcclient$&gt; ». Maintenant, exécutez la commande suivante dans le contexte de rpcclient :

| **rpcclient $&gt; querydominfo** |
| :--- |


![nmap-rpcclien-2](https://linuxfrench.files.wordpress.com/2016/09/nmap-rpcclien-2.jpg?w=1140)

La commande « querydominfo » renvoie le nom de domaine, serveur, la totalité des utilisateurs du système et d’autres informations utiles. Le résultat nous indique qu’il y a un total de 35 comptes utilisateurs disponibles sur le système cible. Maintenant, exécutez la commande suivante pour récupérer une liste de ces 35 utilisateurs :

| **rpcclient $&gt;** **enumdomusers** |
| :--- |


a  
![nmap-rpcclient-3.jpg](https://linuxfrench.files.wordpress.com/2016/09/nmap-rpcclient-3.jpg?w=1140)

Le résultat est une liste de tous les comptes d’utilisateurs disponibles sur le système. Maintenant que nous savons quelles comptes d’utilisateurs sont disponibles sur le serveur, nous pouvons utiliser le « rpcclient ». Pour obtenir plus d’informations sur un utilisateur en particulier _\(dans notre cas, msfadmin\)_, on va utiliser la commande suivante :

| **rcpclient $&gt;** **queryuser \[username\]** **rcpclient $&gt;** **queryuser msfadmin** |
| :--- |


Ceci renvoie des informations sur le chemin sur le profil du serveur, le lecteur , les paramètres de mot de passe liés et beaucoup plus. Ce sont d’excellentes informations qui peuvent être interrogé sans accès administrateur ! Si vous voulez en savoir plus sur la façon d’utiliser le « rpcclient », il suffit de taper la commande d’aide pour un aperçu des options disponibles.

![nmap-rpcclient-4](https://linuxfrench.files.wordpress.com/2016/09/nmap-rpcclient-4.jpg?w=1140)

\(Utilisez la commande « netshareenum » sur l’hôte-cible Metasploitable 2 pour énumérer ses partages réseau.

| **rcpclient $&gt;** **netshareenum** |
| :--- |


![nmap-rpcclient-5](https://linuxfrench.files.wordpress.com/2016/09/nmap-rpcclient-5.jpg?w=1140)

**Énumération avec Enum4Linux :**

Enum4linux est un outil écrit en Perl utilisé pour énumérer les hôtes Windows et Samba. L’outil est essentiellement un « wrapper » pour smbclient, rpcclient, net et nmblookup. Jetons un regard sur la façon d’utiliser enum4linux et exécutons le sur notre hôte-cible Metasploitable 2. Voici les options les plus couramment utilisés avec « enum4linux ». Pour obtenir un aperçu des différentes options utilisent la commande  « -help ».

> Usage: ./enum4linux.pl \[options\]ip
>
> -U        get userlist  
> -M        get machine list\*  
> -S        get sharelist  
> -P        get password policy information  
> -G        get group and member list  
> -d        be detailed, applies to -U and -S  
> -u user   specify username to use \(default “”\)  
> -p pass   specify password to use \(default “”  
> -a        Do all simple enumeration \(-U -S -G -P -r -o -n -i\).  
> -o        Get OS information  
> -i        Get printer information

Utilisons « enum4linux » avec toutes ses options sur notre hôte-cible avec la commande suivante :

| **$ enum4liux \[IP CIBLE\]** **$ enum4linux 192.168.1.14** |
| :--- |


  
Après la commande « enum4linux » terminé, on peut remarquer qu’il nous renvoie beaucoup d’informations utiles. Dont un aperçu des actions disponibles sur notre hôte cible.

 [![enum4linux-1](https://linuxfrench.files.wordpress.com/2016/09/enum4linux-1.jpg?w=566&h=566&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4linux-1/)[![enum4linux-2](https://linuxfrench.files.wordpress.com/2016/09/enum4linux-2.jpg?w=566&h=566&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4linux-2/)[![enum4linux-3](https://linuxfrench.files.wordpress.com/2016/09/enum4linux-3.jpg?w=376&h=376&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4linux-3/)[![enum4linux-5](https://linuxfrench.files.wordpress.com/2016/09/enum4linux-5.jpg?w=376&h=376&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4linux-5/)[![enum4linux-6](https://linuxfrench.files.wordpress.com/2016/09/enum4linux-6.jpg?w=376&h=376&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4linux-6/)[![enum4llinux-4](https://linuxfrench.files.wordpress.com/2016/09/enum4llinux-4.jpg?w=376&h=376&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4llinux-4/)[![enum4linux-7](https://linuxfrench.files.wordpress.com/2016/09/enum4linux-7.jpg?w=376&h=376&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4linux-7/)[![enum4linux-7-1](https://linuxfrench.files.wordpress.com/2016/09/enum4linux-7-1.jpg?w=376&h=376&crop=1)](https://linuxfrench.wordpress.com/2016/09/15/metasploitable-2-enumeration-reseau-avec-nmap/enum4linux-7-1/)

Jusqu’à présent, nous avons recueilli des informations sur le système d’exploitation, les comptes utilisateurs, les ports ouverts et les services en cours d’exécution avec les numéros de version dans ce tutoriel d’énumération sur Metasploitable 2. Nous avons également recueilli des informations sur la politique des mots de passe \(il n’y a personne\) qui soulève des questions sur la force des mots de passe utilisé que nous allons étudier au cours de la phase d’exploitation. Nous avons utilisé des outils tels que Nmap, rpcclient et enum4linux afin de rassembler toutes ces informations. Pour l’instant nous pouvons utiliser cette information pour une évaluation de la vulnérabilité que nous allons faire dans le prochain tutoriel concernant Metasploitable 2.

_Bonne journée à vous, merci de me lire et à bientôt._

