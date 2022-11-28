# TP_actionneur
Le but de ce TP est de contrôler un hacheur depuis la carte électronique STM32. Pour cela, nous avons utilisé le PowerModule 70097 qui permet de faire l'adaptation entre l'étage numérique et l'étage de puissance du moteur.

## Auteurs
Ramos Medina Lizianne: [lil05RM](https://github.com/lil05RM)
Yule Giraldo Juan Sebastian: [JuanYule](https://github.com/JuanYule)

**Sommaire**
1. [Configuration des pins](https://github.com/JuanYule/TP_actionneur/README.md#configuration-des-broches)
2. [Github et Doxygen](https://github.com/JuanYule/TP_actionneur/README.md#github-et-doxygen)
3. [Console UART](https://github.com/JuanYule/TP_actionneur/README.md#console-UART)
4. [Commande-mcc-basique](https://github.com/JuanYule/TP_actionneur/README.md#commande-mcc-basique)

## Configuration des broches
L'image suivante illustre la configuration des broches. Celle-ci a été effectuée à partir du fichier .ioc.

![configuration des broches](/images/pins.png "Configuration des broches")

## Github et Doxygen
Tout d'abord, nous avons créé un dépôt github afin de téléverser toutes les informations concernant la réalisation du TP.

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
Dans cette partie, nous avons avons implémenté le code disponible dans moodle pour créer notre shell. [Voir lien ici](https://moodle.ensea.fr/mod/resource/view.php?id=46898). D'ailleurs nous avons implémenté des variables et des fonctions pour leur fonctionnement.
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
Nous avons généré quatre signaux PWM à partir du TIMER 1. Le cahier des charges pour ce TIMER est le suivant :
* Fréquence de la PWM : 16 kHz
* Temps mort minimum : 2 µs
* Résolution minimum : 10 bits.

### Configuration du TIMER
Afin d'avoir une fréquence de 16 kHz dans le TIMER 1, Nous avons réglé le PSC à *5-1*, l'ARR à *1022* et le Countermode à *TIM_COUNTERMODE_CENTERALIGNED1*. Puis, nous avons activé les deux canaux du TIMER.

### Temps mort
Pour configurer, le temps mort. il faut coché l'option *Break and Dead Time management* et insérer 200 dans la case vide *Dead Time*.

L'image suivante illustre les quatre sigaux PWM avec un temps mort de 2 us.
![Temps mort](/images/Temps_mort.jpg "Doxygen")

### Prise en main du hacheur et câblage
Afin de connecter notre STM32 au hacheur, nous avons utilisé une carte d'adaptation. Voici la liste des connexions des différentes broches utilisées :

*Tableau avec broches*

### Commande start
Selon la fiche technique du hacheur, un pulse minimum de 2 µs est nécessaire pour démarrer le système. Pour commander cette séquence nous avons choisi d'utiliser l'appui du bouton bleu de la carte avec une gestion d'interruption en mode EXTI. Ainsi, avec les fonctions *HAL* et une broche GPIO de sortie, nous avons généré une impulsion de 6.68 µs comme l'illustre la figure suivante.

![Commande start](/images/start.jpg "commande start")

### Commande de vitesse
Pour controler la vitesse du moteur, nous avons envoyer un séquence via liaison UART de la forme:
```
speed=XXXX
```
Pour obtenir les 4 dernières données entrées sur la ligne de commande, nous avons vérifie si l'entrée de commande fait 10 caractères. Ensuite, nous avons les quatre dernieres chifres et si la valeur est plus grande que la vitesse maximale autorisé *100*, on l'abaisse à cette valeur.
La figure suivante illustre les quatre signaux PWM avec un rapport cyclique de 60%.

![PWM 60%](/images/PWM_60.jpg "PWM 60%")

La figure suivante illustre les quatre signaux PWM avec un rapport cyclique de 100%.
![PWM 100 %](/images/PWM_100.jpg "PWM 100 %")

### Premiers tests
Après avoir correctement branché le moteur, nous avons fait un test avec un rapport cyclique de 50% puis de 70%. Pour le premier test, le moteur ne tourne pas car il est à son point d'équilibre. Nous avons augmenté le rapport cyclique de deux en deux pour atteindre 56% et voir le moteur commencer à tourner. Ensuite, nous avons appliqué un rapport cyclique de 70% et le moteur ne tournais plus car l'augmentation du rapport cyclique était trop rapide.

Pour pallier à ce problème, nous avons programmé une montée progressive du rapport cyclique jusqu'à la vitesse souhaitée via la commande. Pour cela, nous avons mis en place une boucle *for* avec un délai, tout en gardant l'ancienne valeur de commande de vitesse. Ainsi, nous avons incrémenté la valeur du rapport cyclique de un en un à partir de l'ancienne valeur et jusqu'à la nouvelle valeur de commande de vitesse.
La ligne [267](https://github.com/JuanYule/TP_actionneur/blob/main/TP_actionneur/Core/Src/main.c#L267) montre la manière comment nous avons mis en œuvre cette solution.

## Mesure de Vitesse et de courant