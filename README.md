# Headless Pi

A shell script to configure an SDCard for a headless after imaging but before
first boot. 

If you play around with Raspberry Pi computers long enough you will get to
the point when you have more than one Pi on a network and one of them headless.
In these cases it is helpful to set up an number of things on a new headless
Pi before first boot.

## Prerequisites

You will need, of course, a Pi, an SDCard, a machine to image the SDCard on
and the big one for many of the things this script does, a Linux/Unix system to
mount the SDCard on to run the script (which can also be the machine you
image the SDCard on). This is because the Pi root filesystem is EXT3. If you
really must use Windows there are programs you can you can use to access the
root filesystem. This is beyond the scope of the project. You can use a
Raspberry Pi for imaging and running the script if you have the equipment to
mount an SDCard by USB.

## Limitations

All the changes to the root file system require super user privileges so if doing more tha setting
up SSH and WiFi you will need to run `headless` from `sudo`, and of course have those privileges on
the system. This does not change the identity of the *user running the script* as far as `headless`
is concerned.

The script is designed to be run on a fresh *Raspberry Pi OS* image. Re-running 
the script on an image may misconfigure the system. This is especially true of
changes to the root filesystem and when the configuration file has been changed.

## What Does it Do?

The script uses a configuration file, which will be discussed later, these
are the things that the script will do for you with proper configuration:
* Enable the SSH Daemon;
* Configure WiFi (if provided the correct config file);
* Change the hostname;
* Change the user name from `pi` to something else;
* Extract a tar file to the user home directory, inteded to set up SSH credentials;
* Copy the `id_rsa.pub` from the user running the script to the Pi user `.ssh/authorized_keys` file
and the root `.ssh/authorized_keys` file;
* Install my Debian repository on the system;
* Change the Pi system time zone.

## What Will it Do?

What it will do depends on the content of the configuration file. Each Pi that you are
managing with the script should have its own configuration file. Here is a sample
configuration (the `mypi` file included):

```
#
# Configuration for mypi host
#
PI_HOSTNAME="mypi"
PI_USERNAME="notpi"

# Configure SSH Daemon
SSHD="yes"

# Configure WIFI from file
WIFI="wpa_supplicant.conf"

# Add current user ~/.ssh/id_rsa.pub to authorized_keys for PI_USERNAME and root.
AUTH_KEYS="yes"

# Add VE3YSH Repository to system.
AUTH_REPO="yes"

# Set the system time zone see /usr/share/zoneinfo
PI_TIME_ZONE="America/Toronto"
```

### SSHD

If set to `yes` the SSH Daemon will be enabled by creating a file called `ssh` in the `boot` partition.

### WIFI

If set to the name of the file that resides in the same directory as the `headless` script the file will
be copied to `wpa_supplicant.conf` in the `boot` partition. The format is not checked. Assuming you have
provided a well formed and correct `wpa_supplicant.conf` file this will enable the Pi to connect to a 
WiFi network.

### PI_HOSTNAME

If defined to a non-empty value, the value will be used as the hostname by writing it in `/etc/hostname`
and `/etc/hosts` on the `rootfs` partition.

### PI_USERNAME

Controls how the user configuration. If defined to a non-empty value that is not `pi` all of the user
configuration changes will be made if other conditions are met:
* Change the user name from `pi` to something else;
* Extract a tar file to the user home directory, inteded to set up SSH credentials;
* Copy the `id_rsa.pub` from the user running the script to the Pi user `.ssh/authorized_keys` file
and the root `.ssh/authorized_keys` file.

#### Other Conditions

The tar file to be created must reside in the same directory as the `headless` script and must have
a name made up of the configuration file name (in this case `mypi`) and the PI_USERNAME (in this case `notpi`):

```mypi_notpi.ssh.tgz```

To copy `id_rsa.pub` as described the `AUTH_KEYS` variable must be set to `yes`, and the `id_rsa.pub` 
file must exist the the home directory of the account running the script.

And as previously mentioned if `PI_USERNAME` is set to `pi` the user name is not changed.

### AUTH_REPO

If set to yes a sources list file will be created directing the configured Pi to look to my Debian
repository for packages and their updates:

```
deb [trusted=yes] https://apt.fury.io/ve3ysh/ /
```

### PI_TIME_ZONE

If set to a non-empty value will change the link on the configured Pi `/etc/localtime` to the
specified time zone. The value must specify a directory path that exists under `/usr/share/zoneinfo` on
a RaspberryPi or similar system.
