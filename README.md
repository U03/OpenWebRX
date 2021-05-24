# Description du repo

Ce repo contient un fichier `docker-compose.yaml` et une configuration de base pour utiliser OpenWebRX avec un dongle RTL_SDR dans un contexte hors fréquences radioamateurs. Vous devez avoir un dongle compatible RTL_SDR connecté à la machine que vous utiliserez comme serveur.

# Présentation de OpenWebRX

OpenWebRX est un logiciel serveur qui permet de partager un ou plusieurs récepteurs SDR entre plusieurs utilisateurs via un navigateur supportant HTLML 5. Les débits nécessaires (200 kb/s par utilisateur) permettent un partage sur internet.

## Multi-utilisateurs et réseau 

Des *profils* sont définis dans OpenWebRX, un *profil* correspond à un récepteur SDR, un fréquence centrale et une fréquence d'échantillonnage. Un récepteur SDR peut être partagé entre plusieurs utilisateurs par OpenWebRX à condition que ces utilisateurs utilisent le même profil, mais chaque utilisateur peut écouter une fréquence différente à l'intérieur de la bande couverte par le *profil*.

Une solution type `rtl_tcp` permet l'utilisation à distance d'un dongle SDR mais nécessite jusqu'à 70 Mbit/s, ce qui ne la rend utilisable en pratique que sur un LAN.

Le débit nécessaire pour OpenWebRX est d'environ 200 kb/s par utilisateur (canal audio et FFT pour la *waterfall* compris). Une suite de process est lancée pour chaque utilisateur, cette consommation CPU dépend du type de signaux écoutés et s'ajoute à la consommation réseau dans le dimensionnement d'une installation partagée.

# Installation de OpenWebRX avec Docker

## Paramètres système

```
sudo tee /etc/modprobe.d/blacklist-dvb.conf << EOF
blacklist dvb_usb_rtl28xxu
EOF
```

> Il est nécessaire de rebooter pour prendre en compte ce paramètre sinon l'erreur "*Kernel driver is active, or device is claimed by second instance of librtlsdr*" va se produire lors du lancement de OpenWebRX.

## Install de Docker

> Les commandes suivantes sont testées sur une machine Ubuntu 20.04 LTS

```
sudo apt-get update \
 && sudo apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common \
 && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - \
 && sudo add-apt-repository "deb https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
 && sudo apt update \
 && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose git \
 && sudo usermod -a -G docker $USER
```

> Se déconnecter et se reconnecter pour prendre en compte le rajout du groupe unix sur l'utilisateur

## Images Docker

Une image Docker est disponible à cette adresse <https://hub.docker.com/r/jketterl/openwebrx/tags>, elle est utilisée par le fichier `docker-compose.yaml` présent dans ce repo.

## Utilisation de OpenWebRX à partir de ce repo

Cloner le repo:

```
git clone https://github.com/U03/OpenWebRX/
cd OpenWebRX
```

Notez que le fichier `docker-compose.yaml` contient le couple user/password (par défaut `MasterOfPuppets/MaitreDuMonde`) utilisé pour créer un utilisateur qui sera administrateur dans l'interface web. Vous devez le personnaliser si vous désirez mettre votre site en ligne.

```
docker-compose up -d
docker-compose logs -f
```

Ceci va lancer OpenWebRX dans un container Docker. Si tout s'est bien passé la commande `docker-compose logs -f` devrait afficher les messages suivants (notez la création de l'utilisateur `MasterOfPuppets` à l'avant-dernière ligne):

```
openwebrx    | [s6-init] making user provided files available at /var/run/s6/etc...exited 0.
openwebrx    | [s6-init] ensuring user provided files have correct perms...exited 0.
openwebrx    | [fix-attrs.d] applying ownership & permissions fixes...
openwebrx    | [fix-attrs.d] done.
openwebrx    | [cont-init.d] executing container initialization scripts...
openwebrx    | [cont-init.d] done.
openwebrx    | [services.d] starting services
openwebrx    | [services.d] done.
openwebrx    | Creating user MasterOfPuppets...
openwebrx    | 2021-05-24 15:23:17,346 - owrx.__main__ - INFO - OpenWebRX version v1.0.0 starting up..
```

Vous pouvez désormais vous connecter à http://xxx.xxx.xxx.xxx:8073/ (où xxx.xxx.xxx.xxx est l'adresse de votre machine) pour explorer un peu les ondes avec les quelques *profils* et *signets* définis.

## Principe de la configuration du logiciel

> Les fichiers ne doivent pas être modifiés directement, désormais l'option *Settings* en haut à droite de l'interface de OpenWebRX permet de modifier ces paramètres.

Dans le menu *settings* les principaux paramètres sont organisés de la façon suivante:

* General Settings: les informations sur la station de réception (localisation, identité, images), les paramètres généraux SDR (waterfall, compression pour le canal audio) et l'intégration Google Map
* SDR device settings: Les différents récepteurs SDR et les profils associés (fréquence centrale, fréquence d’échantillonnage)
* Bookmark editor: les différentes fréquences d’intérêt et leur modulation (leur codage dans le cas des diffusions en numérique)

En cas de besoin les principaux fichiers de configuration sont les suivants:

* `/var/lib/openwebrx/bookmarks.json`; Contient les *bookmarks* (*signets*)
* `/var/lib/openwebrx/settings.json`: Contient tout le paramétrage et en particulier la liste des récepteurs SDR, avec pour chaque récepteur, une liste de profils (fréquence centrale, fréquence d'échantillonnage, et fréquence initiale).

# Exemple d'exploration autour de 446 MHz (PMR)

Autour de 446 MHz on trouve les PMR (*Private Mobile Radio* ou talkie-walkie). En mode analogique il y a 16 canaux, la plupart des talkie-walkies ne disposent que des 8 premiers canaux. Le profil "*RTLSDR PMR 446 MHz*" vous permettra d'explorer cette partie du sceptre radio et vous trouverez des *signets* pour chacun des 16 canaux analogiques.

![Signets PMR 446](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/PMR_446.png)

Si des gens utilisent des talkie-walkies dans votre zone vous les verrez et pourrez les écouter. Si vous avez un talkie-walkie évitez de l'utiliser trop près de l'antenne de votre SDR, vous verriez le *waterfall* saturer et seriez capable d'entendre votre voix sur plusieurs canaux (avec une qualité de plus en plus mauvaise en vous éloignant du canal sur lequel est réglé votre talkie-walkie.)

D'une manière générale l'antenne de votre SDR doit être loin des perturbations radio (ou alors vous devez utiliser un filtre passe-bande pour atténuer les perturbations.)

Un exemple édifiant est visible ci-dessous avec une raie qui apparaît à 445.500 MHz et des raies parasites autour, c'est typique d'une antenne trop proche d'un émetteur... Il s'agit ici de la fréquence rayonnée par des câbles HDMI (ceux de mon ordinateur et de ma console de jeu), les différentes intensités correspondent à quand zéro, un ou deux câbles HDMI sont connectés.

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/HDMI_445500.png)


# Sites web utiles

<https://www.openwebrx.de/>

<https://github.com/jketterl/openwebrx/wiki/Getting-Started-using-Docker>

# Copies d'écran

Citizen-Band (27 MHz)

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/CB_27.png)

Bande FM autour de 107 MHz (radios 'FM')

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/FM_107.png)

Parasites causés par des câbles HDMI

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/HDMI_445500.png)

LDP433 (Low Power Devices autour de 433/434 MHz)

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/LPD_434.png)

Pagers 87 MHz

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/Pager_87.png)

Pagers 143 MHz (pompiers)

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/Pager_173.png)

Pagers 466 MHz (eMessages anciennement Alphapage)

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/Pager_466.png)

Personnal Mobile Radio 446 MHz (talkie-walkies)

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/PMR_446.png)

SRD860 (Short Range Devices autour de 860 MHz)

![Parasites HDMI](https://raw.githubusercontent.com/U03/OpenWebRX/master/images/SRD_860.png)

