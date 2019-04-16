# Raspberry Pi Setup Guide (2019-04-16)
##  1. <a name='Overview'></a>Overview

<!-- vscode-markdown-toc -->
* 1. [Overview](#Overview)
* 2. [Before you start](#Beforeyoustart)
	* 2.1. [Everyone](#Everyone)
	* 2.2. [Windows Users](#WindowsUsers)
		* 2.2.1. [Text Editor](#TextEditor)
		* 2.2.2. [Verify Support for mDNS](#VerifySupportformDNS)
	* 2.3. [Linux Users](#LinuxUsers)
* 3. [Install Raspbian](#InstallRaspbian)
	* 3.1. [Write Raspbian to MicroSD](#WriteRaspbiantoMicroSD)
	* 3.2. [Update Configuration](#UpdateConfiguration)
		* 3.2.1. [Enable SSH](#EnableSSH)
* 4. [Initial Boot](#InitialBoot)
	* 4.1. [Choose a New Password](#ChooseaNewPassword)
	* 4.2. [Default Configuration](#DefaultConfiguration)
	* 4.3. [Update Hostname](#UpdateHostname)
* 5. [Enable Passwordless Login](#EnablePasswordlessLogin)
* 6. [Connect to Wifi](#ConnecttoWifi)
	* 6.1. [Configure the WPA Supplicant](#ConfiguretheWPASupplicant)
	* 6.2. [Testing and Troubleshooting](#TestingandTroubleshooting)
* 7. [Update Software Packages](#UpdateSoftwarePackages)
* 8. [Graceful Shutdown](#GracefulShutdown)
* 9. [Troubleshooting](#Troubleshooting)
	* 9.1. [Can't find _raspberrypi.local_ on Windows](#Cantfind_raspberrypi.local_onWindows)
	* 9.2. [Missing Network Drivers](#MissingNetworkDrivers)
	* 9.3. [SSH Connection Warning](#SSHConnectionWarning)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

##  2. <a name='Beforeyoustart'></a>Before you start
###  2.1. <a name='Everyone'></a>Everyone
Download and install **Etcher** from https://etcher.io. Etcher is among the easiest options for writing a Pi OS image to a microSD.


###  2.2. <a name='WindowsUsers'></a>Windows Users
####  2.2.1. <a name='TextEditor'></a>Text Editor
Among the many subtle differences between Windows and Unix-based machines is a difference in the control characters used to terminate lines in text files. In many cases, this difference will prevent Linux from parsing a file you've written and copied from Windows.

To avoid this issue, install a text editor that supports Unix-style line endings, e.g., __Visual Studio Code__, and use it exclusively for the labs and projects throughout this course.

####  2.2.2. <a name='VerifySupportformDNS'></a>Verify Support for mDNS
Please check the full version of Windows that you have installed by running `Get-ComputerInfo -Property Windows*` in a PowerShell console. After a moment, you'll receive a message describing your current Windows installation (as illustrated below). 

```
WindowsBuildLabEx              : 17763.1.amd64fre.rs5_release.180914-1434
WindowsCurrentVersion          : 6.3
WindowsEditionId               : Professional
WindowsInstallationType        : Client
WindowsInstallDateFromRegistry : 1/10/2019 8:10:04 AM
WindowsProductId               : 00330-80000-00000-AA819
WindowsProductName             : Windows 10 Pro
WindowsRegisteredOrganization  :
WindowsRegisteredOwner         : clementine
WindowsSystemRoot              : C:\WINDOWS
WindowsVersion                 : 1809
```

If the system reports that you are on Windows 10 and that the `WindowsVerion` is `1803` or newer, you may skip to the main tutorial. Otherwise, please proceed to download and install __Bonjour Print Services for Windows (current version 2.0.2)__ from Apple. 

Bonjour includes an implementation of _multicast DNS (mDNS). mDNS is a standard protocol that lets us locate devices and services (including our RaspberryPi's) on simple networks without any manual configuration or supporting infrastructure. Microsoft has only begun to embrace these standards, and this poses a problem since our initial network consists of nothing more than a point-to-point Ethernet connection.

###  2.3. <a name='LinuxUsers'></a>Linux Users
Using your default package manager (likely `yum` or `apt`), confirm that __Avahi mDNS__ services are installed (mDNS is typically part of the default distribution).

https://www.raspberrypi.org/documentation/linux/usage/commands.md

##  3. <a name='InstallRaspbian'></a>Install Raspbian
The Raspberry Pi is built with Linux distributions in mind. The official distribution, which we'll use in our labs is known as __Raspbian__ and is based on Debian Linux. If you're familiar at all with Ubuntu, you should be mostly at home working in Raspbian. Don't worry if Linux is not your jam. We'll provide plenty of guidance so that you can focus your energy on the network concepts.

###  3.1. <a name='WriteRaspbiantoMicroSD'></a>Write Raspbian to MicroSD
Download a current image of Raspbian from http://www.raspberrypi.org/downloads. 

We will use __Raspbian Stretch Lite__ for this course. This version of Raspbian is headless, meaning that it does not include a desktop environment. We’ll leverage SSH to do all of our configuration through the CLI.

Use Etcher to write the image you've downloaded to a microSD. Be aware that this process will overwrite any data that was already stored on the card. 

###  3.2. <a name='UpdateConfiguration'></a>Update Configuration
Etcher will eject the microSD once the image is completely rewritten. We want to edit some files on the SD, so you will need to briefly remove the card before inserting it again.

On macOS or Windows, you'll be limited to accessing the boot partition of the card. Use Explorer or Finder to locate and open the partition. 

You must complete the following step before the first boot.

####  3.2.1. <a name='EnableSSH'></a>Enable SSH
Due to security considerations, the newest versions of Raspian disable SSH by default, but it's easy to turn the feature on so that we can use it for initial setup. 

To enable SSH on the first boot, add an empty file named ssh to the boot volume In macOS, this looks like:

```bash
# Unix-based systems mount external storage to a path in the directory tree. For a freshly written Raspbian image, this path will be /Volumes/boot.

touch /Volumes/boot/ssh
```

If you’re running Windows, you can do this from PowerShell:

```powershell
# Windows mounts external storage to a drive letter. Replace E: with the letter assigned on your system.

New-Item -type File E:\ssh
```

Raspbian will check for this file during the first startup and proceed to configure the SSH _daemon_ to start automatically. The term daemon, by the way, is the name Unix operating systems use to describe a service that runs in the background (e.g., to respond to network requests). 

##  4. <a name='InitialBoot'></a>Initial Boot
It's time to boot the Pi for the first time. Close your editor and any windows that are open to the microSD so that you can eject the card gracefully from your OS. 

- Remove the microSD and insert into the card slot on your Pi.
- Connect power to the designated micro-USB port.
- Attach to your computer with an Ethernet cable and get ready to launch an SSH connection.
- From terminal or PowerShell, connect to the Pi for the first time by running the following command:

```bash
# The default password for pi is raspberry
ssh pi@raspberrypi.local
```

This command directs your local SSH client to connect to a network host named `raspberrypi.local` with the username `pi`. 

Using the mDNS service, your computer will resolve the hostname to an IP address SSH will ask you to accept the connection of an unknown device before presenting you with a password prompt. 

```
$ ssh pi@raspberrypi.local
The authenticity of host 'raspberrypi.local (fe80::1b9c:bcf2:acd6:bbbe%42)' can't be established.
ECDSA key fingerprint is SHA256:QqhpMybvctuIxV03xcnlANU3cxWM1JhvSYxloSd69Rw.
Are you sure you want to continue connecting (yes/no)?
```

If you've previously connected to a host with the name `raspberrypi.local`, you may also see the warning shown below (please refer to the troubleshooting section to resolve this error):

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

If you are running Windows and receive a message that Windows cannot resolve the name `raspberrypi.local`, please refer to the instructions at the top of this guide and ensure that you have installed _Bonjour Services_.

After confirming the prompt shown above, you will see another short message followed by a password prompt.

```
Warning: Permanently added 'raspberrypi.local,fe80::1b9c:bcf2:acd6:bbbe%42' (ECDSA) to the list of known hosts.
pi@raspberrypi.local's password:
```

The default password for the `pi` user is `raspberry`. Once login is complete, you should be greeted with a message similar to that shown here.


```
Linux raspberrypi 4.14.79-v7+ #1159 SMP Sun Nov 4 17:50:20 GMT 2018 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Linux raspberrypi 4.14.79-v7+ #1159 SMP Sun Nov 4 17:50:20 GMT 2018 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

Wi-Fi is disabled because the country is not set.
Use raspi-config to set the country before use.

pi@raspberrypi:~ $
```

###  4.1. <a name='ChooseaNewPassword'></a>Choose a New Password
Before proceeding further with setup, you should change the default password by entering the `passwd` at the command prompt. You will be guided through three prompts to enter the current password and then to update/confirm the new password.

_Notes: SSH is a favorite target of malicious attackers. Nothing makes their lives easier than devices that are configured with the default password for the type of system._

###  4.2. <a name='DefaultConfiguration'></a>Default Configuration
After you complete this step, I also recommend the following changes on every new Linux device:

```bash 
# Setting the timezone will ensure that log messages are displayed in local time, which is quite helpful for troubleshooting
# You can see a list of timezones by running `timedatectl list-timezones`

pi@raspberrypi:~ sudo timedatectl set-timezone America/Los_Angeles

# Configure the locale and keymap prevents common problems encountered when editing files and troubleshooting bricked devices ... Raspbian defaults to a British locale

# The first command will open the specified file in the nano text editor. You need to locate and uncomment the line that defines the en_US.UTF-8 locale.

pi@raspberrypi:~ sudo nano /etc/locale.gen

# After you edit this file, we need to re-generate locale information for the local system before we can update the current setting and updating the keyboard mapping.

pi@raspberrypi:~ sudo locale-gen
pi@raspberrypi:~ sudo localectl set-locale "LANG=en_US.UTF-8"
pi@raspberrypi:~ sudo localectl set-keymap us
```

_Note: We started each of the previous commands with the term `sudo`. When used in this way, `sudo` runs the remainder of the statement as the `root` super user, i.e., super-user-do._

_By default, the `pi` user is restricted in which parts of the system it can read and modify (Linux is heavily user-based in it's permissions model); however, the user is permitted to elevate itself to the admin level when it is necessary to perform privileged commands._

_As a general security precaution, don't `sudo` commands unless it's necessary. For now, we'll indicate when that's the case. In later tasks, you will need to make this determination on your own._

###  4.3. <a name='UpdateHostname'></a>Update Hostname
At first boot, we could locate our Pi on the network based on the default hostname of `raspberrypi.local`. This is fine in an isolated, point-to-point network, but it's a problem when we connect to shared networks. 

Use the `hostnamectl` command to set a unique name for your device.

```bash
# This little pi likes to be called Titan
pi@raspberrypi:~ $ sudo hostnamectl set-hostname titan
```

```bash
# Edit the following files and replace any remaining references to raspberrypi with your new hostname
pi@raspberrypi:~ $ sudo nano /etc/hostname
pi@raspberrypi:~ $ sudo nano /etc/hosts
```

In order for the _hostname_ changes to take full effect, reboot your pi by calling `sudo reboot now`. After 20 - 30 seconds your pi will be visible with the new name -- you will need to include the `.local` suffix, .e.g., `ssh pi@titan.local.`

##  5. <a name='EnablePasswordlessLogin'></a>Enable Passwordless Login
We can significantly simplify the process of managing our Pi’s by copying our public SSH keys into the `authorized_keys` file of the default pi user.

To prepare the Raspberry Pi, we need to create a remote directory under the `pi` user's home directory. 

If you're logged into your Pi already, you can create the directory using `mkdir ~/.ssh`. Type `exit` to terminate the connection and return to your local command prompt.

You can also use `ssh` to remotely execute the command without having to open a full SSH session.

```bash
# Make a .ssh configuration directory on your Pi and immediately exit
ssh pi@titan.local 'mkdir ~/.ssh'
```

With the directory in place, copy your public key (most likely `~/.ssh/id_ed25519.pub`) to the pi using the `scp` command.

```bash
scp $HOME/.ssh/id_ed25519.pub pi@raspberrypi.local:.ssh/authorized_keys
```

_Note: The second set of parameters in this command specifies the user (`pi`) the host (`titan.local`) and the location of the new file (`.ssh/authorized_keys`). Like `ssh`, `scp` is a user-based tool and will be executed relative to the home directory of the authenticated user (`/home/pi`). We can override the default by specifying an absolute path beginning with a `/`, e.g., `/home/pi/.ssh/authorized_keys`._

##  6. <a name='ConnecttoWifi'></a>Connect to Wifi
To complete this guide, you will need to establish Internet connectivity for your Pi. Since the Pi 3 has integrated wireless capabilities, we can solve the problem by connecting to local wifi. 

###  6.1. <a name='ConfiguretheWPASupplicant'></a>Configure the WPA Supplicant
Wireless settings for the Pi are controlled by a service called _wpa_supplicant_, which stores network connection settings inside `/etc/wpa_supplicant/wpa_supplicant.conf`. You can edit this directly on the Pi using the nano text editor (or vi for the daring). Alternatively, you can create the file on your local system and copy it into place on the Pi (as described later in this guide). 

For this project, you must configure a connection to the Eduroam network. It's also a good idea at this time to configure connections to your home network.

_Note: **Do not** associate your Pi with *University of Washington* unless given direct instructions to do so. Because the network requires a browser-based login, it has been a major source of trouble for former students._

Begin your `wpa_supplicant.conf` file with the following lines and then add one or more network blocks (including a connection to Eduroam) as shown below.

```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

# INSERT WPA2 ENTERPRISE CONFIG FOR EDUROAM
```

_Note: When working with configuration in Linux, it's best to assume that the syntax is case and whitespace sensitive unless otherwise specified. It's common for seemingly non-important changes to render the entire configuration invalid._

When you are finished updating `wpa_supplicant.conf` reconfigure the wireless interface by calling `wpa_cli -i wlan0 reconfigure`. The command should return `OK` after a few seconds. Check that you are attached to the wireless network  by calling `wpa_cli -i wlan0 status`.

```
pi@titan:~ $ wpa_cli -i wlan0 status

bssid=ac:a3:1e:eb:53:c1
freq=2462
ssid=eduroam
id=0
mode=station
pairwise_cipher=CCMP
group_cipher=CCMP
key_mgmt=WPA2/IEEE 802.1X/EAP
wpa_state=COMPLETED
ip_address=10.18.185.176
p2p_device_address=fa:f8:ee:8c:fe:62
address=b8:27:eb:db:fe:93
Supplicant PAE state=AUTHENTICATED
suppPortStatus=Authorized
EAP state=SUCCESS
selectedMethod=25 (EAP-PEAP)
EAP TLS cipher=ECDHE-RSA-AES256-GCM-SHA384
tls_session_reused=0
EAP-PEAPv0 Phase2 method=MSCHAPV2
eap_session_id=19865aef996591ffbd01643fbcf1b5111ef04dd3ed4f3387752c8cc0cf3b8c88b95c40bfc4915b284b3b91f60f5cb71074baaf1360d39f7a91530143dcb3c8e4ce
uuid=c4adf7c1-f863-5192-8179-f5d1d2e27fd7
```

###  6.2. <a name='TestingandTroubleshooting'></a>Testing and Troubleshooting
In addition to `wpa_cli`, there are several tools that can be used to check the current state of your wireless interface and determine whether it has received a valid configuration from DHCP.

```bash
# Show the current state of the wlan0 interface
pi@titan:~ $ ip link show wlan0

3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DORMANT group default qlen 1000
    link/ether b8:27:eb:db:fe:93 brd ff:ff:ff:ff:ff:ff

# Show address configuration for the wlan0 interface
pi@titan:~ $ ip addr show wlan0

3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:db:fe:93 brd ff:ff:ff:ff:ff:ff
    inet 10.18.185.176/15 brd 10.19.255.255 scope global wlan0
       valid_lft forever preferred_lft forever
    inet6 fe80::ce14:df50:b3f0:9c2a/64 scope link
       valid_lft forever preferred_lft forever
```

##  7. <a name='UpdateSoftwarePackages'></a>Update Software Packages
Let's finalize the initial setup by checking for updates to Raspbian and its default packages (this can take a few minutes on slow networks).

As a Debian based Linux distribution, Raspbian relies on `apt` for package management. The `apt update` command is used to determine whether there are new packages to download.

```bash
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade

# We've found that an additional update is sometimes needed after a dist-upgrade
sudo apt update

# Install some useful packages while you're at it
sudo apt install dnsutils
```

##  8. <a name='GracefulShutdown'></a>Graceful Shutdown
Never yank the power from your Pi when it's time to quit. This can lead to data corruption on the microSD (they aren't as resilient as the drives installed in your laptop). 

Instead, you should always issue a proper shutdown via SSH.

From an SSH connection, `run sudo poweroff`. If you ever need to reboot the Pi, call `sudo reboot now`.

Wait about 15 - 20 seconds to allow the Pi to complete it’s shutdown process before disconnecting the power.

##  9. <a name='Troubleshooting'></a>Troubleshooting

###  9.1. <a name='Cantfind_raspberrypi.local_onWindows'></a>Can't find _raspberrypi.local_ on Windows
Make sure that you've installed an mDNS client as described early in this document. Though mDNS is now a draft standard, Microsoft did not include a full client until the most recent versions of Windows 10 (confirmed on 1809). If you receive an error the first time you connect to the Raspberry Pi, it's likely that you're missing mDNS or had a problem with the Bonjour install.

If you are sure that mDNS is installed correctly, you will need to check the configuration of the Ethernet port or USB device on your local computer. 

Run `ipconfig` on Windows or `ifconfig` on other devices to confirm that the interface registers a network connection. The local IPv4 address for this interface should begin with `169.254.x.x`. If you do not see a valid connection, review the next section on missing drivers.

###  9.2. <a name='MissingNetworkDrivers'></a>Missing Network Drivers
If you are using a USB Ethernet port for the first time on your computer, it's possible that your first attempt to connect will be unsuccessful. 

Many of these issues are resolved by disconnecting the USB briefly and trying after you have plugged everything back in. If issues persist, search online to determine whether your computer is missing a driver needed for the specific model of Ethernet adapter.

###  9.3. <a name='SSHConnectionWarning'></a>SSH Connection Warning 
At some point, you are likely to receive the following warning when trying to launch `ssh`. In most cases, what you are seeing is expected behavior. 

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

The first time you connect to a host, SSH will bind a cryptographic signature to the hostname, e.g., `raspberrypi.local` and store it within `~/.ssh/known_hosts`. To prevent attacks, SSH warns us when the signature changes for a known host. 

In our case, it's likely that there is another explanation for the mismatched signature since SSH will create a new private key every time we set up or rebuild a Raspberry Pi.

Since this is not an attack scenario, resolve the conflict with `ssh-keygen -R raspberrypi.local`, which will remove the existing entry. 

_Note: The `ssh-keygen -R` feature is still missing in OpenSSH for Windows 10. If you are a Windows user, you will have to locate `known_hosts` within the `.ssh` configuration directory and remove the line indicated by the warning message._