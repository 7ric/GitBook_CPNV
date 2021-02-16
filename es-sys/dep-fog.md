---
description: Déploiement de stations et d'applications avec Free Open Ghost
---

# DEP-FOG

[FOG, pour Free Open-Source Ghost](https://fogproject.org/), est une solution de clonage et de déploiement de systèmes d'exploitation et de logiciels sur des postes PC. Les systèmes d'exploitation supportés sont Windows et Linux.

![](../.gitbook/assets/image%20%282%29.png)

## Objectifs du laboratoire

### **Objectifs principales**

* Permettre le déploiement de stations
* Permettre le déploiement d'applications

### **Compétences**

* Mise en production d'un service
  * Déploiement d'un service
  * Mettre au point une procédure d'installation de la solution
  * Automatiser l'installation de la solution
  * Mettre en exploitation le service
* Gestion des configurations
  * Recueil d'informations sur une configuration et ses éléments
  * Suivi d'une configuration et de ses éléments

### **Objectifs opérationnels**

* Automatiser l'installation d'un service
* Installer, configurer, administrer un service
* Administrer un système
* Contrôler et améliorer les performances d'un système
* Automatiser une tâche d'administration

### **Échéance**

* 28 périodes pour la réalisation du labo \(4 semaines\)
* `Proof Of Concept` lors de la dernière semaine de décembre \(Lundi 14.12.20 au vendredi 18.12.20\)
* Documentation [https://www.gitbook.com](https://www.gitbook.com/) \(vendredi 18.12.20 à 12h00\)

## Mise en situation

L'objectif global est de montrer comment déployer des images de stations et d'applications via le réseau en utilisant [Free Open Ghost \(FOG\)](https://fogproject.org/). Ce labo est mis en œuvre avec des machines virtuelles. Nous travaillerons sur un parc mixte comportant des machines sous Windows et Linux.

Les différents objectifs à atteindre sont les suivants :

* Avoir une vue d'ensemble de l'application FOG sous Ubuntu Server LTS
* Déployer une image type Windows ou Linux sur des machines cibles
* Utiliser les snapins pour déployer un anti-virus sur des machines Windows.

## Réalisation

**FOG**, pour Free Open-Source Ghost, est une solution de clonage et de déploiement de systèmes d'exploitation et de logiciels sur des postes PC. Les systèmes d'exploitation supportés sont Windows et Linux.

### Installation de Free Open Ghost

* Installez FOG sur un serveur Linux dédié au déploiement. \(Sans GUI\)
* Installez et configurer un serveur DHCP pour FOG
* Ajoutez au contexte un client disposant d'un navigateur afin de prendre la main sur le serveur.

### Travaux post installation

* À l'aide de l'interface de gestion, changez le mot de passe attribué par défaut à l'utilisateur FOG.
  * Testez en vous déconnectant et en vous reconnectant.
* Vérifiez que les services associés à LAMP sont présents et à l'écoute sur le serveur.
* Comment s'appelle la base de données créée sur le serveur ?
* Le timeout du menu PXE de FOG est par défaut de 3 secondes. Trouvez comment modifier cette valeur à 10 secondes.
* La configuration de FOG est sauvegardée dans une base de données. Trouvez comment sauvegarder puis restaurer cette configuration.
* FOG trace toutes les tâches effectuées \(inventaires, images, connexion utilisateurs, snapins\). Exporter sous forme de rapport au format PDF ou CSV les traces de ces activités.

### Création des images

* Choisissez deux machines modèle dont il faut réaliser les images. L'une sous Windows et l'autre sous Debian sans interface graphique. 
* Inventoriez ces machines sur votre serveur à l'aide du menu de boot PXE de FOG.
* Créez ensuite deux images types respectivement pour les machines de type Linux et pour les machines de type Windows. Pour chaque image, choisissez le type d'image approprié selon que vos machines disposent de plusieurs disques et partitions.
* Associez les machines inventoriées à leur image et "uploadez" vos machines sur le serveur en lançant les tâches de remontée d'image. Vérifiez la présence de ces deux images dans le répertoire _`/images_FOG`_ qui doit se trouver sur un disque dédié.
* Trouvez comment changer le répertoire cible d'upload des images sur le serveur.

### Déploiement unicast/multicast

* Inventoriez trois nouvelles machines : une qui servira de cible pour un déploiement de Linux et les deux autres pour un déploiement de Windows.
* Associez deux de ces machines à un groupe nommé `GrpeWindows`.
* Réalisez un déploiement unicast de la machine modèle Linux vers une machine cible.
* Réalisez un déploiement multicast de la machine modèle Windows vers deux machines cibles.

### Déploiement d'un anti-virus

* Créez un snapin permettant de déployer un antivirus sur les machines du groupe GrpeWindows.
* Localisez le journal d'activité sur le client et vérifiez le bon déroulement de la procédure.

## Proof Of Concept

\[ \_\_ \] **Serveur FOG** \(2 pts\)

* \[ \_\_ \] \[1 pt.\] Installation de FOG sur un serveur LINUX \(IP fixe\)
* \[ \_\_ \] \[1 pt.\] Connexion depuis une machine cliente sur le serveur FOG \(Navigateur WEB\)
  * `http://<nom-ou-ip-serveur>/fog`
  * Mot de passe par défaut changé \(fog/password\)

\[ \_\_ \] **Configuration et administration du serveur FOG** \(4,5 pts\)

* \[ \_\_ \] \[1 pt.\] Menu PXE de FOG : Time 10 secondes
* \[ \_\_ \] \[0,5 pt.\] Changez le répertoire des images modèles → `/images_FOG`
* \[ \_\_ \] \[1 pt.\] Sauvegardez la configuration de FOG
* \[ \_\_ \] \[1 pt.\] Restaurez la configuration de FOG
* \[ \_\_ \] \[1 pt.\] Exportation du rapport de log FOG au format PDF

\[ \_\_ \] **5 machines virtuelles \(les deux premières servant d'image modèle à uploader les trois autres faisant office de cible du déploiement\)** \(1 pt\)

* `FOG_VM_01` \(Modèle Windows\)
  * 2 G
  * 1 HDD 100 G
* `FOG_VM_02` \(Modèle Linux sans GUI\)
  * 1 G
  * 1 HDD 80 G
* `FOG_VM_03` \(Cible du déploiement Linux\)
  * 1 G
  * 1 HDD 40 G
* `FOG_VM_04` \(Cible du déploiement Windows\)
  * 2 G
  * 1 HDD 100 G
* `FOG_VM_05` \(Cible du déploiement Windows\)
  * 2 G
  * 1 HDD 70 G

\[ \_\_ \] **Enregistrer des 5 machines sur le serveur FOG** \(1 pt\)

* \[ \_\_ \] \[1 pt.\] FOG → Host management

\[ \_\_ \] **Création des images modèles \(Chemin pour le stockage par défaut des images → /images\_FOG\)** \(2 pts\)

* \[ \_\_ \] \[1 pt.\] `FOG_VM_01` → image modèle Windows \(Single Disk : Resizable\)
* \[ \_\_ \] \[1 pt.\] `FOG_VM_02` → modèle Linux \(Single Disk : Resizable\)

\[ \_\_ \] **Association de l’image modèle avec le client \(Host Name et MAC\)** \(1,5 pt\)

* \[ \_\_ \] \[0,5 pt.\] `FOG_VM_03` → Host image LINUX
* \[ \_\_ \] \[0,5 pt.\] `FOG_VM_04` → Host image Windows
* \[ \_\_ \] \[0,5 pt.\] `FOG_VM_05` → Host image Windows

\[ \_\_ \] **Déploiement unicast** \(2 pts\)

* \[ \_\_ \] \[2 pts.\] `FOG_VM_03` → Host image LINUX \(Redimensionnement des partitions\)

\[ \_\_ \] **Déploiement multicast** \(3 pts\)

* \[ \_\_ \] \[1 pt.\] Création d'un groupe de machines `GrpeWindows`
  * `FOG_VM_04`
  * `FOG_VM_05`
* \[ \_\_ \] \[2 pts.\] Déploiement multicast sur les machines du groupe `GrpeWindows`
  * `FOG_VM_04` → Host image Windows \(Redimensionnement des partitions\)
  * `FOG_VM_05` → Host image Windows \(Redimensionnement des partitions\)

\[ \_\_ \]  **Déploiement d’applications avec FOG** \(3 pts\)

* \[ \_\_ \] \[1 pt.\] Installation du client FOG sur les machines `FOG_VM_04` et `FOG_VM_05`
* \[ \_\_ \] \[1 pt.\] Mise en place d'un snapin `AntiVirus` sur le serveur FOG
* \[ \_\_ \] \[1 pt.\] Affectation du snapin aux machines cibles `FOG_VM_04` et `FOG_VM_05`

**Total 20 pts** \[ \_\_ \]

