## Étape 0: Installation Caméra

Suivre le [guide d'installation module caméra](http://www.raspberrypi.org/help/camera-module-setup/).

## Étape 1: Tester la caméra

Avec la caméra branchée et activée, entrez la commande suivante dans le Terminal pour capturer une image :

```bash
raspistill -o cam.jpg
```

Vous devriez avoir un aperçu à l'écran lors de la prise de photo.

Maintenant tapez `ls` et vous devriez voir un fichier nommé `cam.jpg`. Ouvrez votre répertoire personnel (*home*) dans l'explorateur de fichier et regarder l'image (clique droit et selectionner `Visionneur d'images`). S'il y a une photo de ce qui était devant votre caméra - elle marche correctement !

Si votre photo est à l'envers, ceci est parce que votre caméra était orientée à l'envers - mais pas de souci - parfois c'est plus facile de l'installer dans ce sens , et vous pouvez simplement inversé l'image.

Si vous avez l'intention de positionner votre caméra à l'envers, faites passer les valeurs de configuration `-hf` et `-vf` pour inverser horizontalement et verticalement l'image (sinon avancez à l'Étape 2) :

```bash
raspistill -hf -vf -o cam2.jpg
```

Vérifiez de nouveau, il devrait avoir désormais un fichier nommé `cam2.jpg` dans le dossier. Ouvrez l'image pour vérifier que c'est à l'endroit.

## Étape 2: Écrire un script

Ensuite nous allons écrire un script "Bash" qui va prendre une photo et l'enregistrer avec la date et l'heure. Ça peut être aussi simple que ce qui suit :

```bash
#!/bin/bash

DATE=$(date +"%Y-%m-%d_%H%M")

raspistill -o /home/pi/camera/$DATE.jpg
```

Pensez à utiliser les "drapeaux" `-vf` et `-hf` si votre caméra est à l'envers.

Créez un nouveau fichier nommé `camera.sh` en ouvrant un editeur de texte, par exemple `nano camera.sh`, collez ou autrement entrez les lignes comme ci-dessus et enregistrez le fichier.

Maintenant vous pouvez rendre le fichier "exécutable" avec la commande suivante :

```bash
chmod +x camera.sh
```

Si on lance ce script, on va enregistrer une photo avec le tampon "date/heure" comme nom de fichier dans un répertoire qui s'appelle `camera` qui se trouve sous notre répertoire peronnel (*home*). Tout d'abord nous allons créer ce dossier :

```bash
mkdir camera
```

Veillez à être situé dans votre répertoire personnel quand vous lancer la commande. Si vous n'y êtes pas, ou vous n'êtes pas certain, tapez simplement `cd` et appuyer sur `Entrée` pour retourner sous votre répertoire personnel.

Vous pouvez utiliser la commande `pwd` (*present working directory* - répertoire actif actuel) afin de vérifier votre position dans l'arborescence et `ls` pour montrer le contenu. Après avoir lancé la commande `mkdir` vous devriez voir un nouveau dossier y apparaître.

Avant de continuer, testez le bon fonctionnement du script en le lançant à partir de la ligne de commande (d'abord il faudrait retourner au répertoire personnel avec la commande `cd`) :

```bash
./camera.sh
```

Vous devriez voir de nouveau un aperçu pendant la prise de photo. Puis utilisez `ls camera` pour regarder à l'intérieur du dossier `camera` et voir la photo que vous venez de prendre stockée sur le disque.

Ouvrez l'explorateur de fichier et prévisualisez pour voir la nouvelle photo. Si vous êtes satisfait du résultat, et que la date et heure sont bien présentes dans le nom du fichier, continuez à l'automatisation.

## Étape 3: Planifier la prise de photos

Puisque vous avez, d'ores et déjà, un script "Bash" qui prend des photos et les nomme avec un tampon date-heure, vous pouvez programmer le script pour s'exécuter périodiquement, par exemple, toutes les minutes.

Pour le réaliser nous allons utiliser `cron`. Tout d'abord, il faudrait ouvrir la table "cron" pour la modifier :

```bash
sudo crontab -e
```

Si vous n'avez jamais exécuté `crontab` auparavant vous serez invité de choisir un editeur de texte - si vous n'avez pas de préférence, choisissez `nano` en appuyant sur `Entrée`.

Maintenant vous voyez le fichier `cron`, faites défiler jusqu'en bas où vous trouverez une ligne avec les en-têtes suivants :

```bash
# m h  dom mon dow   command
```

The layout for a cron entry is made up of six components:

Minute, Hour, Day of Month, Month of Year, Day of Week and the command to be executed.

```
# * * * * *  command to execute
# ┬ ┬ ┬ ┬ ┬
# │ │ │ │ │
# │ │ │ │ │
# │ │ │ │ └───── day of week (0 - 7) (0 to 6 are Sunday to Saturday, or use names; 7 is Sunday, the same as 0)
# │ │ │ └────────── month (1 - 12)
# │ │ └─────────────── day of month (1 - 31)
# │ └──────────────────── hour (0 - 23)
# └───────────────────────── min (0 - 59)
```

Pour programmer toutes les minutes l'exécution du script `camera.sh`, ajoutez la ligne suivante :

```bash
* * * * * /home/pi/camera.sh 2>&1
```

Puis enregistrer et quitter le fichier. Si vous utilisez `nano` comme editeur, c'est `Ctrl + O` pour enregistrer et `Ctrl + X` pour quitter.

Vous devez voir le message suivant :

```bash
crontab: installing new crontab
```

Maintenant vous pouvez retourner au dossier de la caméra pour voir des images qui commencent à apparaître :

```bash
cd ~/camera/
```

et utilisez `ls` pour voir le contenu du dossier. Entrez `date` pour voir l'heure précise et si vous ête près du changement de minutes (`00` secondes) car c'est à ce moment-là qu'une nouvelle image sera prise.

Utilisez `watch ls` pour surveiller les changements du contenu du répertoire. `watch` exécute la commande toutes les 2 secondes (par défaut).

## Étape 4: Patience

Si vous constatez qu'une nouvelle image arrive dans le dossier toutes les minutes, et vous êtes content par rapport à l'orientation des images, positionnez définitivement votre caméra dans la direction désirée.

Vous pouvez utiliser un pied ou tout simplement scotcher le Pi au mur ou à un objet et positionner la caméra à l'aide de ruban adhésif. Vérifiez que la position de la caméra est statique et en restera dans la durée.

Vous pouvez éteindre le Pi, enlever l'écran et le câble ethernet et puis le remettre en marche uniquement avec l'alimentation (quand vous la branchez, le Pi demarrera normalement et `cron` s'exécutera comme prévu) à l'emplacement requis.

Vous pouvez même vous servir d'un bloc de batterie si vous en avez avec suffissament d'autonomie pour répondre à vos besoins. Ceci est particulièrement utile si vous devez placer l'ensemble loin d'une prise eléctrique, comme par exemple sur un toit ou dans un arbre !

### Copier des images à distance

Si vous avez une connexion réseau avec le Pi (avec ou sans fil) ou si votre écran est toujours branché, vous pouvez vérifier l'avancement des photos.

Si votre écran est branché, vous pouvez utiliser `ls`, `watch ls` et même l'explorateur de fichiers pour observer les photos au fer et à mesure qu'elles arrivent, sinon vous pouvez accéder à votre Pi à distance d'un autre ordinateur afin de copier les fichiers sur un autre machine. Voici quelques options, que vous pouvez trouver et lire dans notre documentation :

#### SSH

Vous pouvez accéder à distance à votre Pi en ligne de commande via [SSH](http://www.raspberrypi.org/documentation/remote-access/ssh/README.md) - utilisez `ls` et `watch ls` pour vérifier l'enregristrement des images capturées.

#### SCP

Utilisez [SCP](http://www.raspberrypi.org/documentation/remote-access/ssh/scp.md) (Copie sécurisée) pour copier des fichiers via SSH.

#### rsync

Utilisez [rsync](http://www.raspberrypi.org/documentation/remote-access/ssh/rsync.md) pour synchroniser un dossier sur le Pi avec un dossier sur votre ordinateur.

#### FTP

Mettez en place un serveur [FTP](http://www.raspberrypi.org/documentation/remote-access/ftp.md) sur le Pi et utilisez un client FTP sur un autre ordinateur afin d'accéder à distance au système des fichiers du Pi, et transférer des fichiers.

#### Carte SD

Si vous vous en servez du système d'exploitation GNU/Linux sur un autre ordinateur vous pouvez transférer des fichiers directement de la carte SD, car l'ordinateur pourrait monter la partition du système des fichiers.

## Étape 5: Transformer des images instantanées en vidéo

Maintenant vous aurez besoin de assembler les photos en vidéo afin d'obtenir un effet de time-lapse.

Vous pouvez faire cela sur le Pi via `mencoder`. Cependant le traitement sera lent. Peut-être ce serait préférable de transférer les fichiers d'image sur votre ordinateur de bureau ou un portable et procéder au traitement vidéo sur cette machine-là.

Naviguez vers le dossier qui contient l'ensemble de vos images et créez une liste des noms des images dans un fichier texte. Par exemple :

```bash
ls *.jpg > stills.txt
```

### Sur le Raspberry Pi ou un autre système Linux

Installez le *package* "mencoder" :

```bash
sudo apt-get install mencoder
```

Maintenant lancez la commande suivante :

```bash
mencoder -nosound -ovc lavc -lavcopts vcodec=mpeg4:aspect=16/9:vbitrate=8000000 -vf scale=1920:1080 -o timelapse.avi -mf type=jpeg:fps=24 mf://@stills.txt
```

Une vidéo sera enregistrée sous le nom `timelapse.avi`

## Licence

Sauf si expressément spécifié, tout le contenu de ce répertoire est couvert sous la licence suivante :

![Licence Creative Commons](http://i.creativecommons.org/l/by-sa/4.0/88x31.png)

***Time-lapse Setup*** de la [Fondation Raspberry Pi](http://raspberrypi.org) est licencié sous  [Licence Creative Commons Attribution 4.0 International](http://creativecommons.org/licenses/by-sa/4.0/).

D'après le travail réalisé sous https://github.com/raspberrypilearning/timelapse-setup
