# TP_actionneur
TP d'Actionneur et automatique appliquée. Option ESE
Liziaine RAMOS MEDINA
Juan Sebastian YULE GIRALDO

**Table of Contents**
1. [Github et Doxygen](https://github.com/JuanYule/TP_actionneur/README.md#github-et-doxygen)
2. [Configuration des pins]()

## Configuration des broches
L'image suivante illustre la configuration des broches. La configuration des broches a été effectuée à partir du fichier .ioc.

![configuration des broches](/images/pins.png "Configuration des broches")

## Github et Doxygen
Tout d'abord, nous avons créé un dépôt github afin de téléverse toutes les informations concernant la réalisation du TP.

### Configuration de Doxygen
Nous avons suivi la configuration pour l'installation comme indiqué sur le site officiel.
Nous avons utilisé les commandes suivantes pour l'installation sur un ordinateur linux. [Voir ici documentation](https://www.doxygen.nl/download.html)
```
git clone https://github.com/doxygen/doxygen.git
cd doxygen
mkdir build
cd build
cmake -G "Unix Makefiles" ..
make
```
Noud avons eu des problèmes pour générer la documentation, donc nous avons téléchargé et utilisé l'interface graphique ```doxywizard```.
L'image suivante illustre la page html avec l'information du projet.

![Doxygen](/images/doxygen.png "Doxygen")

## Console UART