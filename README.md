# TP_actionneur
TP d'Actionneur et automatique appliquée. Option ESE
Liziaine RAMOS MEDINA
Juan Sebastian YULE GIRALDO

**Table of Contents**
1. [Configuration des pins](https://github.com/JuanYule/TP_actionneur/README.md#configuration-des-broches)
2. [Github et Doxygen](https://github.com/JuanYule/TP_actionneur/README.md#github-et-doxygen)
3. [Console UART](https://github.com/JuanYule/TP_actionneur/README.md#console-UART)
4. [Console UART](https://github.com/JuanYule/TP_actionneur/README.md#commande-mcc-basique)

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
Dans cette partie, nous avons avons implmenté le code discponible dans moodle pour crées notre shell. [Voir lien ici](https://moodle.ensea.fr/mod/resource/view.php?id=46898). D'ailleur nous avond imlementé des variables et des fonctions pour leur fonctionnement.
```
char cmd[CMD_BUFFER_SIZE] : contenant la commande en cours
int idxCmd : contenant l'index du prochain caractère à remplir
const uint8_t prompt[] : contenant le prompt comme sur un shell linux
const uint8_t started[] : contenant un message de bienvenue au démarrage du microprocesseur
const uint8_t newLine[] : contenant la chaine de caractère pour faire un retour à la ligne
const uint8_t help[] : contenant le message d'aide, la liste des fonctions
const uint8_t pinout[] : contenant la liste des pin utilisées
const uint8_t powerOn[] : contenant le message d'allumage du moteur
const uint8_t powerOff[] : contenant le message d'extinction du moteur
const uint8_t cmdNotFound[] : contenant le message du commande non reconnue
uint32_t uartRxReceived : flag de récéption d'un caractère sur la liaison uart
uint8_t uartRxBuffer[UART_RX_BUFFER_SIZE] : buffer de réception de donnée de l'uart
uint8_t uartTxBuffer[UART_TX_BUFFER_SIZE] : buffer d'émission des données de l'uart
```
## Commande MCC basique
Nous avons généré quatre signaux PWM à partir du TIMER 1. Le cahier des charges pour ce TIMER est le suivante:
1. Fréquence de la PWM : 16 kHz
2. Temps mort minimum : 2 us
3. Résolution minimum : 10 bits.

### Configuration du TIMER
Afin d'avoir une fréquence de 16 kHz dans le TIMER 1, Nous avons réglé le PSC à *5-1*, l'ARR à *1022* et le Countermode à *TIM_COUNTERMODE_CENTERALIGNED1*. Puis, nous avons active les deux canaux du TIMER.

### Temps mort
Pour configurer, le temps mort. il faut taper sur l'option *Break and Dead Time management* et insérer 200 dans la case vide *Dead Time*.

L'image suivante illustre les quatre sigaux PWM avec un temps mort de 2 us.
![Temps mort](/images/Temps_mort.jpg "Doxygen")

### Commande start
Selon la fiche technique du hacheur, un pulse minimun de 2 us est nécessaire pour démarrer le système. Donc avec les fonction *HAL*, nous avons généré une impulsion de 6.68 us comme l'illustre la figure suivante.

![Commande start](/images/start.jpg "commande start")

### Commande de vitesse

![PWM 60%](/images/PWM_60.jpg "PWM 60%")

![PWM 100 %](/images/PWM_100.jpg "PWM 100 %")