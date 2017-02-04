Official Raspberry Pi Setup Procedure for Matrioshka
====================================================

This is the official procedure to setup a Raspberry Pi from scratch.  We use Raspbian Jessie Lite as the operating system.


Installing Jessie Lite on SD Card
---------------------------------

First we'll prepare a micro SD card of at least 4GB (8GB+ strongly recommended) with the Raspbian Jessie Lite operating system.
 
Download the latest Raspbian Jessie Lite image from [www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/).

Follow the instructions ([Linux](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md), [Mac](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md) or [Windows](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md)) to transfer the image to the SD card.


Configuration of basic settings on SD Card
------------------------------------------

Mount the SD card on your PC (if it is not already).

### SSH

From the command line, browse to the root of the /boot folder of the SD card and run:

    sudo touch ./ssh

This will enable ssh on boot (it is disabled by default).


### Ethernet static fallback

From the command line, browse to the root of the / folder of the SD card and edit the dhcpcd.conf file.  If you're a fan of the vi text editor, run:

    sudo vi /etc/dhcpcd.conf

Paste the following lines at the bottom of the file and save:

    # Define static profile
    profile static_eth0
    static ip_address=10.0.50.100/24
    static routers=10.0.50.1
    static domain_name_servers=10.0.50.1

    # Fallback to static profile on eth0
    interface eth0
    fallback static_eth0

This will configure the Pi to use IP address __10.0.50.100__ when it is connected to Ethernet without DHCP.  This provides a sure-fire means of connecting to the Pi from a PC to make changes/updates.

The SD card is ready!


First connection
----------------

Complete the following in order:
- Insert the SD card in the Pi.
- Connect the Pi to your PC via a network cable.
- Apply power to the Pi.
- Configure your PC's Ethernet adapter to use an address in the same subnet as the Pi (ex: 10.0.50.101).

From the command line, ssh into the Pi as follows:

    ssh pi@10.0.50.100

The default password will be _raspberry_.

The connection to the Pi is now established.


raspi-config
------------

Configure the Pi by running the following from the command line:

    sudo raspi-config

This should open a text-based menu.

### Change User Password

Select the option to change the user password.  Choose a password that cannot be easily guessed, but is nonetheless straightforward to enter from the command line next time you'll ssh in.

### Change Hostname

Select _Advanced Options_, then _Hostname_.  Change the hostname to __matrioshka-pi__.

Select _Finish_, then select _Yes_ when asked to reboot.


Connect to WiFi
---------------

SSH back into the Pi once the reboot is complete (using the new password).  Open the wpa_supplicant.conf file for editing:

    sudo vi /etc/wpa_supplicant/wpa_supplicant.conf 

For each WiFi network that the Pi will connect to, add the following to the bottom of the file:

    network={
      ssid="ENTER_SSID_HERE"
      psk="ENTER_PASSWORD_HERE"
      key_mgmt=WPA-PSK
    }

Ensure that the list includes an Internet-connected WiFi network in range.  Save the file and exit.  If your PC and the Pi are now configured to connect to the same WiFi network, the Ethernet link between them will no longer be required.  Enter the following from the command line to shut down the Pi:

    sudo shutdown now

If desired, disconnect the network cable from the Pi.  Cycle power.  SSH back into the Pi (which will have a different IP address if you're connecting over WiFi!).


Prepare Pi software
-------------------

Update packages to the latest version by running the following from the command line:

    sudo apt-get update

### Install Node.js

Raspbian will likely include an old version of Node.js. Here we'll provide the instructions for installing the latest LTS (long term support version) which will be required. From the command line on the Pi, execute the following in order (optionally change 6.9.2 below throughout to the latest LTS version as indicated [here](https://nodejs.org/)):

    mkdir ~/Downloads
    cd ~/Downloads
    wget https://nodejs.org/dist/v6.9.2/node-v6.9.2-linux-armv7l.tar.xz
    tar -xf node-v6.9.2-linux-armv7l.tar.xz
    sudo mv node-v6.9.2-linux-armv7l /usr/local/node
    cd /usr/local/bin
    sudo ln -s /usr/local/node/bin/node node
    sudo ln -s /usr/local/node/bin/npm npm

### Install forever

Complete the following in order from the command line:

    cd /usr/local/bin
    sudo npm install forever -g
    sudo ln -s /usr/local/node/bin/forever forever

### Install git

Run the following from the command line:

    sudo apt-get install git-core

### Install Xorg

Complete the following in order from the command line:

    sudo apt-get install --no-install-recommends xserver-xorg
    sudo apt-get install --no-install-recommends xinit
    sudo apt-get install --no-install-recommends x11-xserver-utils


Install Pi Cap (OPTIONAL)
-------------------------

If the Pi will use the Pi Cap board, proceed with the following.  If the Pi Cap board is not already connected to the Pi, disconnect the Pi from power and carefully connect the board via the Pi's header.  From the command line run:

    sudo apt-get install picap
    picap-setup

Select _yes_ to everything except the installation of Node.js (which we already installed).  Reboot when prompted.

To test the picap at any time, run the following from the command line:

    picap-intro


Install Matrioshka software
---------------------------

From the command line, clone this repository in the pi user's home folder:

    cd
    git clone https://github.com/Sensorica/matrioshka-pi.git

Install the reelyActive packages:

    cd matrioshka-pi/reelyactive
    npm install

To make the reelyActive software and Pi Cap run on boot, edit /etc/rc.local:

    sudo vi /etc/rc.local

and add the following line above exit 0:

    forever start /home/pi/matrioshka-pi/reelyactive/server.js
    sleep 10; /home/pi/matrioshka-pi/picap/run &

