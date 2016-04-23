## Étape 0: Installation Caméra

Suivre le [guide d'installation module caméra](http://www.raspberrypi.org/help/camera-module-setup/).

## Étape 1: Tester la camera

Avec la caméra branchée et activée, entrez la commande suivante dans le Terminal pour capturer une image :

```bash
raspistill -o cam.jpg
```

Vous devriez avoir un aperçu à l'écran lors de la prise de photo.

Maintenant tapez `ls` et vous devriez voir un fichier nommé `cam.jpg`. Ouvrez votre répertoire personnel (*home*) dans l'explorateur de fichier et regarder l'image (clique droit et selectionner `Visionneur d'images`). S'il y a une photo de ce qui était devant votre caméra - elle marche correctement !

Si votre photo était à l'envers, ceci est parce que votre caméra était orientée à l'envers - mais pas de souci - parfois c'est plus facile de l'installer dans ce sens , et vous pouvez simplement inversé l'image.

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

Véillez à être dans votre répertoire personnel quand vous lancer la commande. Si vous n'y êtes pas, ou vous n'êtes pas certain, tapez simplement `cd` et appuyer sur `Entrée` pour retourner sous votre répertoire personnel.

Vvous pouvez utiliser la commande `pwd` (*present working directory* - répertoire actif actuel) afin de vérifier votre position dans l'arborescence et `ls` to show the contents. After running `mkdir` you should see a new folder there.

Before continuing, test the script works as intended by running it from the command line (first return to the home directory with `cd`):

```bash
./camera.sh
```

You should see the preview again as the picture is taken. Now use `ls camera` to look inside the `camera` folder to see the picture you just captured on disk.

Open the file browser and preview the image to see the picture itself. If you're happy this worked as intended, and the date and time were given in the filename, continue to automation.

## Étape 3: Schedule taking pictures

Now you have a Bash script which takes pictures and timestamps them, you can schedule the script to be run at an interval, say every minute.

To do this we'll use `cron`. First open the cron table for editing:

```bash
sudo crontab -e
```

If you've not run `crontab` before you'll be prompted to select an editor - if you don't know the difference, choose `nano` by hitting `Enter`.

Now you'll see the `cron` file, scroll to the bottom where you'll see a line with the following column headers:

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

To schedule for the `camera.sh` script to be executed every minute, add the following line:

```bash
* * * * * /home/pi/camera.sh 2>&1
```

Now save and exit. If you're using `nano` as your editor, that's `Ctrl + O` to save and `Ctrl + X` to exit.

You should see the following message:

```bash
crontab: installing new crontab
```

Now return to the camera directory to see the photos start to appear:

```bash
cd ~/camera/
```

and use `ls` to see the contents of the folder. Enter `date` to see how close you are to the minute (`00` seconds) as a new picture should be captured at this precise time.

Use `watch ls` to see changes to the contents of the folder. `watch` runs the command runs every 2 seconds (by default).

## Étape 4: Patience

If you see pictures landing in the `camera` folder every minute, and you're happy with the orientation of the pictures, now position the camera wherever you want it to point at.

Perhaps use a camera mount or simply tape the Pi to a wall or object and position the camera with tape. Make sure the camera position is static and will remain in place over time.

You can shut down the Pi, remove it from the monitor and ethernet and simply have it running on power (when you plug it in, it will boot as normal and `cron` will run as expected) in the position you require.

You can even use a battery pack if you have one that lasts long enough for your requirements. This is especially handy if you need to position the power out of reach of a power socket, such as on a roof or in a tree!

### Copying pictures remotely

If you have network connection with the Pi (wired or wireless) or your monitor is still attached, you can check the progress of the photos.

If your monitor is attached, you can use `ls`, `watch ls` and even the file browser to see photos as they are captured, otherwise you can remotely access your Pi from another computer to copy the files to your computer. Here are some options, which you can read about in our documentation:

#### SSH

You can gain remote access to the command line using [SSH](http://www.raspberrypi.org/documentation/remote-access/ssh/README.md) use `ls` and `watch ls` to verify the pictures are being captured.

#### SCP

Use [SCP](http://www.raspberrypi.org/documentation/remote-access/ssh/scp.md) (Secure copy) to copy files over SSH.

#### rsync

Use [rsync](http://www.raspberrypi.org/documentation/remote-access/ssh/rsync.md) to syncronise a folder on the Pi with a folder on your computer.

#### FTP

Set up an [FTP](http://www.raspberrypi.org/documentation/remote-access/ftp.md) server on the Pi and use an FTP client on another computer to access the Pi's filesystem remotely, and copy files over.

#### SD Card

If you're using Linux on another computer you can transfer the files directly from the SD card, as it can mount the filesystem partition.

## Étape 5: Turn stills in to a video

Now you'll need to stitch the photos together in to a video to achieve the time-lapse effect.

You can do this on the Pi using `mencoder` but the processing will be slow. You may prefer to transfer the image files to your desktop computer or laptop and processing the video there.

Navigate to the folder containing all your images and list the file names in to a text file. For example:

```bash
ls *.jpg > stills.txt
```

### On Raspberry Pi or other Linux computer

Install the package mencoder:

```bash
sudo apt-get install mencoder
```

Now run the following command:

```bash
mencoder -nosound -ovc lavc -lavcopts vcodec=mpeg4:aspect=16/9:vbitrate=8000000 -vf scale=1920:1080 -o timelapse.avi -mf type=jpeg:fps=24 mf://@stills.txt
```

This will save a video called `timelapse.avi`

## Licence

Unless otherwise specified, everything in this repository is covered by the following licence:

![Creative Commons License](http://i.creativecommons.org/l/by-sa/4.0/88x31.png)

***Time-lapse Setup*** by the [Raspberry Pi Foundation](http://raspberrypi.org) is licenced under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

Based on a work at https://github.com/raspberrypilearning/timelapse-setup
