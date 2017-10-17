# Prérequis pour l'intégration de `clients-windows`

Ceci correspond aux configurations testées et validées pour l'intégration de postes `Windows` à un domaine `SE3`. Vu l'infinité de situations possibles, seules ces configurations sont supportées et feront l'objet d'une assistance.

* [Système](#système)
* [Installation](#installation)
* [Ordre de boot](#ordre-de-boot)
* [Drivers](#drivers)
* [Licence et activation](#licence-et-activation)
* [Préparation et mise à jour](#préparation-et-mise-à-jour)
* [Logiciels](#logiciels)
* [Outils](#outils)


## Système

Il faut impérativement partir d'un poste fraîchement installé. Le temps perdu à refaire une installation propre sera toujours du temps qu'on ne perdra pas ensuite à résoudre des incompatibilités.

* **`Windows10` Pro 64 bits**  
L'installeur peut être téléchargé directement chez `Microsoft` sans restrictions particulières, et installé sur une clé `USB`.

* **`Windows7` Pro 32 et 64 bits**  
Il faut utiliser une version officielle sans personnalisations `OEM`, il est admis d'y ajouter les mises à jour ainsi que les drivers à l'aide d'outils tiers : voir
[les outils](#outils) plus bas. En revanche aucune personnalisation de type `GPO` ne doit avoir été faite.
En pratique, pour avoir un système à jour il faut déployer des milliers de mises à jour. C'est très lent ! Il vaut mieux installer W10 !

* **`Windows XP` n'est plus supporté**, ni par `SE3`, ni par `Microsoft`, ni par les applications récentes.  
Si vous vous posez la question d'installer des postes `windowsXP`, [installez des `clients-linux`](../pxe-clients-linux/README.md#installation-de-clients-linux-debian-et-ubuntu-via-se3--intégration-automatique) !


## Installation

Installation en mode `Legacy Bios`. **Surtout pas d'`UEFI` !**.

Une seule partition `windows` + la petite partition de boot qui est créée automatiquement par l'installeur. Pour cette partition, 100 Go sont largement suffisants.

Une astuce permet d'éviter la création de la partition de boot de 100Mo créée automatiquement à l'installation de Windows 7. Cela simplifie le clonage et la création d'images. Voici comment procéder.

**Avant de lancer l'installation de Windows** (par exemple avec gparted inclu dans SystemRescueCD via le boot PXE) :
* supprimer toute partition du disque,
* (re)créer une table de partition ms-dos (pas de gpt),
* créer la partition destinée à recevoir l'OS (100G suffisent),
* formater cette partition en NTFS.

Ainsi, lors de l'installation, en choisissant cette partition, la partition de boot de 100Mo ne sera pas créée.

Il est possible d'avoir un double-boot `Gnu/Linux`, dans ce cas [vous laisserez un espace vide](../pxe-clients-linux/utilisation.md#installation-en-double-boot) en partageant le disque dur en deux parties sensiblement égales.


## Ordre de boot

Boot `PXE` activé (soit systématiquement, soit manuellement avec `F12`).


## Drivers

Tous les pilotes utiles doivent être installés et à jour : voir [les outils](#outils) plus bas.


## Licence et activation

Pour `Windows7` il faut activer à l'aide d'un outil tiers…

Pour `Windows10`, théoriquement aucune activation n'est requise si le poste est éligible à une installation `OEM`. Ceci implique généralement que les clés OEM SLIC V2.4 soient intégrées au Bios. Ce n'est malheureusement pas toujours le cas. Il est possible de récupérer la licence du bios, l'intégration se3 le fera automatiquement en cas de besoin.
En cas d'absence d'activation, un message s'affiche en fond d'écran, mais à part cela tout fonctionne. 

**Attention** Ne pas télécharger et utiliser Windows Loader pour Windows 10 : c'est un faux, qui en revanche installe de vrais virus !

## Préparation et mise à jour

**Sur un serveur à jour, aucune opération manuelle n'est nécessaire pour intégrer windows 10**

**A VÉRIFIER**

Il faut avoir créé un fichier témoin `temoin_w10.txt` vide dans le répertoire `netlogon\domscripts\`. Ensuite il suffit de se connecter sur `\\se3\admhomes` avec le compte `admin` et de lancer `\\se3\admhomes\netlogon\domscripts\rejointse3.exe`.

## ancienne méthode

* pour les postes en w10

Désactiver la mise en veille pour surveiller le poste pendant son intégration l'intégration.
Fusionner le fichier .reg placé dans  \\se3\progs\install\domscripts (voir article windows10) .

Il est possible d'automatiser un certain nombre de réglages, dans l'optique de faire une **image de la machine** avant l'intégration. À vous de choisir parmi les commandes suivantes, celles qui vous conviennent et les mettre dans un fichier `.bat`. 

*Si l'image de la machine possède déjà l’utilisateur adminse3, et que le mdp d'adminse3 est bien donné qu compte administrateur, alors l'intégration à distance de l'image déployée pourra se faire directement par l'interface.*

* Désactiver les 3 pare-feux de Windows7 :

`netsh advfirewall set allprofiles state off`

* Désactiver un compte local "essai" issu de l'installation :

`net user essai /active:no`

* Rendre actif le compte Administrateur :

`net user Administrateur /active:yes`

* Ajouter le compte "adminse3" :

`net user /add adminse3 mdp_admise3`

* Ajouter ce compte adminse3 aux administrateurs locaux :

`net localgroup Administrateurs adminse3 /add`

* Donner aux mots de passe une durée de validité illimitée :

`net accounts /maxpwage:unlimited`

* Désactiver le controle UAC :

`reg.exe ADD HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f`

* Mettre à jour depuis wsusoffline du Se3 :

```
net use W: \\IP_du_Se3\install\wsusoffline mdp_admise3 /user:sambaedu3\adminse3
w:\client\cmd\DoUpdate.cmd /updatecerts /instielatest /updatecpp /instmssl /instdotnet35 /instdotnet4 /instpsh /instofv
net use W: /delete
```
Si par hasard les màj windows update sont bloquées, une piste de résolution [ici](http://www.easy-pc.org/2016/06/fix-windows-7-quand-les-verifications-des-mises-a-jour-prend-trop-de-temps.html).

## Logiciels

Aucun logiciel installable à l'aide de `Wpkg` ne doit être installé préalablement. C'est une source de problème sans fin.

Partez d'une installation de `Windows` de base et laissez le serveur `Wpkg` gérer les applications à installer.


## Outils

…à compléter…
* [Obtenir des ISO légalement](http://www.downflex.com/)
* [Pack de drivers](https://sdi-tool.org/)
* [Créer une clé USB bootable](http://www.winsetupfromusb.com/)
* [Créer une ISO personnalisée](http://rt7lite.com/)
* [Sauvegarder et restaurer l'activation](http://joshcellsoftwares.com/products/advancedtokensmanager/) ([Documentation](http://www.pcastuces.com/pratique/windows/sauvegarder_activation/page1.htm))
