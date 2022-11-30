# TP_actionneur
Le but de ce TP est de contrôler un hacheur depuis la carte électronique STM32. Pour cela, nous avons utilisé le PowerModule 70097 qui permet de faire l'adaptation entre l'étage numérique et l'étage de puissance du moteur.

## Auteurs
Ramos Medina Lizianne : [lil05RM](https://github.com/lil05RM)

Yule Giraldo Juan Sebastian : [JuanYule](https://github.com/JuanYule)

**Sommaire**
1. [Configuration des broches](https://github.com/JuanYule/TP_actionneur/README.md#configuration-des-broches)
2. [Github et Doxygen](https://github.com/JuanYule/TP_actionneur/README.md#github-et-doxygen)
3. [Console UART](https://github.com/JuanYule/TP_actionneur/README.md#console-UART)
4. [Commande MCC basique](https://github.com/JuanYule/TP_actionneur/README.md#commande-mcc-basique)
5. [Mesure de vitesse et de courant](https://github.com/JuanYule/TP_actionneur/README.md#mesure-de-vitesse-et-de-courant)

## Configuration des broches
L'image suivante illustre la configuration des broches. Celle-ci a été effectuée à partir du fichier .ioc.

![configuration des broches](/images/pins.png "Configuration des broches")

## Github et Doxygen
Tout d'abord, nous avons créé un dépôt Github afin de téléverser toutes les informations concernant la réalisation du TP.

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
Nous avons eu des problèmes pour générer la documentation, donc nous avons téléchargé et utilisé l'interface graphique ```doxywizard```.
L'image suivante illustre la page HTML avec les informations du projet.

![Doxygen](/images/doxygen.png "Doxygen")

## Console UART
Dans cette partie, nous avons avons implémenté le code disponible dans moodle pour créer notre shell. [Voir lien ici](https://moodle.ensea.fr/mod/resource/view.php?id=46898). D'ailleurs nous avons implémenté des variables et des fonctions pour le fonctionnement du shell.
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
Afin d'avoir une fréquence de 16 kHz dans le TIMER 1, nous avons réglé le PSC à *5-1*, l'ARR à *1023* et le Counter Mode à *TIM1_COUNTERMODE_CENTERALIGNED1*. La valeur de ARR est ainsi car la résolution est de 10 bits donc 2^10 = 1024. La valeur de PSC nous permet d'obtenir une fréquence de 16 kHz en tenant compte du paramètre Counter Mode. Enfin, nous avons activé les deux canaux du TIMER en *PWM Generation CHx CHxN*.

### Temps mort
Pour configurer le temps mort, il faut cocher l'option *Break and Dead Time management* et insérer 200 dans la case vide *Dead Time*.

L'image suivante illustre les quatre sigaux PWM avec un temps mort de 2 µs.
![Temps mort](/images/Temps_mort.jpg "Doxygen")

### Prise en main du hacheur et câblage
Afin de connecter notre STM32 au hacheur, nous avons utilisé une carte d'adaptation. Voici la liste des connexions des différentes broches utilisées entre la carte d'adaptation, le hacheur et la STM32 :

| Nom broche hacheur | Broche hacheur | Broche carte adaptation| Broche STM32 |
| :------------: | :---------------: | :---------------:|:---------------:|
| CMD_B_TOP | 11 | 11 | PA8 |
| CMD_Y_TOP | 12 | 12 | PA9 |
| CMD_B_BOT | 29 | 30 | PA11 |
| CMD_Y_BOT | 30 | 31 | PA12 |
| ISO_RESET | 33 | 34 | PC3 |
| ISO_GND | 36 | 37 | GND |
| ISO_+5V | 37 | 38 | 5V |

La broche permettant l'allumage du hacheur est *ISO_RESET*. Celle-ci est connectée à une sortie GPIO qui, après un appuie sur le bouton utilisateur, envoie la séquence d'amorçage. Cette séquence est décrite dans la sous-partie suivante pour la commande start.

### Commande start
Selon la fiche technique du hacheur, un pulse minimum de 2 µs est nécessaire pour démarrer le système. Pour commander cette séquence nous avons choisi d'utiliser l'appui sur le bouton bleu utilisateur de la carte avec une gestion d'interruption en mode EXTI. Le contenu de la fonction se situe dans le fichier main.c et son nom est [powerUp()](https://github.com/JuanYule/TP_actionneur/blob/main/TP_actionneur/Core/Src/main.c#L693). Ainsi, avec les fonctions *HAL* et une broche GPIO de sortie, nous avons généré une impulsion de 6.68 µs comme l'illustre la figure suivante.

![Commande start](/images/start.jpg "commande start")

### Commande de vitesse
Pour contrôler la vitesse du moteur, nous avons envoyé une commande via la liaison UART de la forme suivante :
```
speed=XXXX
```
Pour obtenir les 4 derniers caratères entrés sur la ligne de commande, nous avons vérifié si l'entrée de commande fait 10 caractères. Ensuite, nous avons traité les quatre derniers chiffres et si la valeur est plus grande que la vitesse maximale autorisé *100*, on l'abaisse à cette valeur.
La figure suivante illustre les quatre signaux PWM avec un rapport cyclique de 60%.

![PWM 60%](/images/PWM_60.jpg "PWM 60%")

La figure suivante illustre les quatre signaux PWM avec un rapport cyclique de 99%.

![PWM 99 %](/images/PWM_100.jpg "PWM 99 %")

### Premiers tests
Après avoir correctement branché le moteur, nous avons fait un test avec un rapport cyclique de 50% puis de 70%. Pour le premier test, le moteur ne tourne pas car il est à son point d'équilibre. Nous avons augmenté le rapport cyclique de deux en deux pour atteindre 56% et voir le moteur commencer à tourner. Ensuite, nous avons appliqué un rapport cyclique de 70% et le moteur ne tournait plus car l'augmentation du rapport cyclique était trop rapide.

Pour pallier à ce problème, nous avons programmé une montée progressive du rapport cyclique jusqu'à la vitesse souhaitée via la commande. Pour cela, nous avons mis en place une boucle *for* avec un délai, tout en gardant l'ancienne valeur de commande de vitesse. Ainsi, nous avons incrémenté la valeur du rapport cyclique de un en un à partir de l'ancienne valeur et jusqu'à la nouvelle valeur de commande de vitesse.
La ligne [267](https://github.com/JuanYule/TP_actionneur/blob/main/TP_actionneur/Core/Src/main.c#L267) montre comment nous avons mis en œuvre cette solution.

## Mesure de vitesse et de courant
Dans cette section, nous avons utilisé les broches retournées par le hacheur pour mesurer le courant qui traverse le moteur. Pour cela, nous avons utilisé l'interruption de l'ADC pour capturer les données de mesure. De cette façon il n'y aura pas de problèmes avec l'affichage de caractères dans le shell.

L'implémentation de cette partie du code fonctionne mais ne montre pas les valeurs correctes du courant, c'est-à-dire qu'il y'a une lecture erronée au moment de la conversion ADC.
