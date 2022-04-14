# Nextdom, Z-Wave, et MQTT sur un RPI.
## Introduction
Toutes les commandes ci-dessous doivent être réalisées avec l’utilisateur : `pi`.
## Installation du raspberry.

Téléchargement de [la dernière version Lite](https://www.raspberrypi.org/downloads/raspberry-pi-os/) et déziper l’archive.

Installer l’image sur une carte SSD :
```sh
sudo dd bs=1M if=2020-05-27-raspios-buster-lite-armhf.img of=/dev/sdX status=progress conv=fsync
```
À la racine de la partition boot, créer un fichier ssh vide à la racine. Cela permet d’installer et d’activer sshd à l'installation.

Lancer le raspberry avec la carte. Il doit être joignable en ssh : `pi@<host>` (password: `raspberry`).
```sh
ssh pi@raspberrypi.local
```

Avec `raspi-config` :
```sh
sudo raspi-config
```
Réaliser les modifications :
- changer le `password` de l'utilisateur pi.
- changer le nom de la machine.
- changer la `Locale` - pour la France ajouter et choisir `fr_FR.UTF-8`.
- changer la `Time/Zone` - pour la France choisir `Europe/Paris`.

Mettre à jour la machine : 
```sh
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

Si vous avez une carte GPIO pour le Z-Wave il faut désactiver le bluetooth :
```sh
echo "dtoverlay=pi3-disable-bt" | sudo tee -a /boot/config.txt
sudo systemctl disable hciuart
```
Optimisation de la ram vidéo :
```sh
echo "gpu_mem=16" | sudo tee -a /boot/config.txt
echo "disable_l2cache=0" | sudo tee -a /boot/config.txt
echo "gpu_freq=250" | sudo tee -a /boot/config.txt
```
redémarrer la machine
```sh
sudo shutdown -r now
```
## Installation du Z-Wave.
### Open-Zwave.
```sh
wget -q -O - razberry.z-wave.me/install | sudo bash
```
Avec l’utilisateur `pi` :
```sh
sudo apt-get install -y libudev-dev git
cd ~
git clone https://github.com/OpenZWave/open-zwave.git
cd open-zwave && make && sudo make install
sudo ldconfig
export LD_LIBRARY_PATH=/usr/local/lib64
sudo sed -i '$a LD_LIBRARY_PATH=/usr/local/lib64' /etc/environment
```
### Mosquitto.
```sh
sudo apt install -y mosquitto mosquitto-clients libmosquitto-dev
sudo systemctl enable mosquitto
```

### Zwave2Mqtt.
#### Installation
```sh
cd ~
sudo apt-get install -y npm git
git clone https://github.com/OpenZWave/Zwave2Mqtt
cd Zwave2Mqtt
npm install
npm run build
cd ~
```
#### Mise en place du service
Créer le fichier suivant
```sh
sudo nano /etc/systemd/system/zwave2mqtt.service
```
Avec
```ini
[Unit]
Description=zwave2mqtt
After=network.target

[Service]
Environment="LD_LIBRARY_PATH=/usr/local/lib64"
ExecStart=/usr/local/bin/npm start
WorkingDirectory=/home/pi/Zwave2Mqtt
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```
```sh
sudo systemctl start zwave2mqtt
sudo systemctl enable zwave2mqtt.service
```

> TODO: Réalisation de La configuration de `zwave2mqtt` : http://host:8091/

## Installation de (Jee|Next)dom.
### Prérequis
Mise en place de libttspico-utils
```sh
wget http://ftp.us.debian.org/debian/pool/non-free/s/svox/libttspico0_1.0+git20130326-9_armhf.deb
wget http://ftp.us.debian.org/debian/pool/non-free/s/svox/libttspico-utils_1.0+git20130326-9_armhf.deb
sudo apt install -yf ./libttspico0_1.0+git20130326-9_armhf.deb ./libttspico-utils_1.0+git20130326-9_armhf.deb
```
### Le choix s’offre à vous.
#### pour Jeedom
```sh
cd ~
wget -O- https://raw.githubusercontent.com/jeedom/core/master/install/install.sh | sudo bash
```
#### pour Nextdom

```sh
sudo apt install -y software-properties-common gnupg wget ca-certificates 
sudo add-apt-repository non-free
sudo wget -qO -  http://debian.nextdom.org/debian/nextdom.gpg.key  | apt-key add -
sudo echo "deb http://debian.nextdom.org/debian  nextdom main" >/etc/apt/sources.list.d/nextdom.list
sudo apt update
sudo apt -y install nextdom-mysql nextdom-common nextdom
```
### Installation du plugin JMQTT
Sur le market installer le plugin `JMQTT` en version `stable`.

Désactiver la configuration  `Installer Mosquitto localement`.

Installer les dépendances du plugin.

Dans le plugin : Ajouter un `Broker` : `zwave`.

> TODO: Réalisation de La configuration du `Broker` 

