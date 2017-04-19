# rpi-mass-image-writer

Raspberry Pi that writes to many USB drives / SD cards at once. Hardware interface available to select the stored disk image to copy as well as to shutdown the device. 

Disk images are transferred to the Raspberry Pi via a Samba shared folder. The selected image is then written in parallel to all the drives at once using the ```dd```, ```pv``` and ```tee``` commands 

![Screen](/photos/console-and-hub.jpg)

![Screen](/photos/sd-cards-in-adapters.JPG)



# How to use?
1. Login to shared folder - Windows: navigate to `\\hostname\images`
2. Transfer image files to the shared folder.
3. Plug in all drives to write to.
4. Press left button to enumerate all images and drives.
5. Use up/down buttons to select image.
6. Press the right button to start writing to drives. Press again to halt writing.
7. Press the select button (extreme left) to shutdown device properly to prevent data corruption. The words "Shutting down" will remain even after the device has completely shutdown, so just wait for the activity light to turn off before removing power.

# Hardware
1. Raspberry Pi 3 Model B. Older models will work, you will need to edit which i2c bus is used in Adafruit_I2C.py
2. Adafruit i2c 16x2 LCD Pi Plate with keypad
3. USB hubs (not all hubs are supported well, use the ones from this [list](http://elinux.org/RPi_Powered_USB_Hubs))
4. USB SD card adapters

# Setting up
On a fresh install of ARCH Linux, login as root. If you logged in as alarm, execute `su root` to switch users. Password is `root`
Download and install packages:
`pacman -Syu python2 i2c-tools samba pv git`

If an error occurs, remove the offending file eg: I had to `rm /etc/ssl/certs/ca-certificates.crt` and reattempt installing the packages.
Enable i2c: (source: https://wiki.archlinux.org/index.php/Raspberry_Pi#I2C)

Use nano, append `/boot/config.txt` with `dtparam=i2c_arm=on`

Configure the `i2c-dev` and `i2c-bcm2708` modules to be loaded at boot:
Append `/etc/modules-load.d/raspberrypi.conf` with:
```
i2c-dev
i2c-bcm2708
```

Clone the repo
```
git clone https://github.com/michaelruppe/rpi-mass-image-writer.git
```

## I2C configuration
In `Adafruit_I2C.py`, I've uncommented line 35 and comment line 36 to hard-code the i2c bus to use i2c bus \#1 (raspberry pi 3 B)
ie: `self.bus = smbus.SMBus(1);` Selects to use bus \#1

Run the app to make sure all is well:
```bash
cd rpi-mass-image-writer
python2 writer.py
```

LCD should turn on

## Setting up samba
### The network shared directory to store images. 
There is already a default conf file, so executing the following command creates a new conf file – ie don’t be concerned if you are writing a completely new file.
`nano /etc/samba/smb.conf`
Add/Modify the following lines to your `smb.conf` file
```
[global]
workgroup = WORKGROUP
server string = SD Writer
security = user
log file = /var/log/samba/%m.log
max log size = 50
dns proxy = no

[images]
path = /root/rpi-mass-image-writer/images
writable = yes
```

### Create a user for samba
```bash
smbpasswd -a root
systemctl enable smbd
```

I chose “root” as the password, which is why the login to access the images directory over the network is user:root, pass:root
Change the hostname
```
hostnamectl set-hostname newHostName
```


## Starting at boot
```
cp /root/rpi-mass-image-writer/writer.service /etc/systemd/system/
systemctl enable writer.service
reboot
```

## Expanding the filesystem

From the command line or a terminal window enter the following
`sudo fdisk /dev/mmcblk0`
then type `p` to list the partition table
you should see two/three partitions. if you look in the last column labeled System you should have
1.	W95 FAT32
2.	Linux
3.	Linux Swap (I didn’t have this, that’s ok)
make a note of the start number for Linux partiton (in this case, partition 2), you will need this later. Though it will likely still be on the screen (just in case).
next type `d` to delete a partition.
You will then be prompted for the number of the partition you want to delete. In the case above you want to delete both the Linux and Linux swap (if it exists) partitions.
So type `2`
then type `d` again and then type `3` to delete the swap partition.
Now we will resize the main partition
type `n` to create a new partition.
This new partition needs to be a primary partition so type `p`.
Next enter `2` when prompted for a partition number.
You will now be prompted for the first sector for the new partition. Enter the start number from the earlier step (the Linux partition)
Next you will be prompted for the last sector you can just hit enter to accept the default which will utilize the remaining disk space.

If a warning appears about removing the signature, select Yes

Type `w` to save the changes you have made.
`reboot` the system.
Once the system has rebooted and you are back at the commandline enter the following command:
`sudo resize2fs /dev/mmcblk0p2`
Note: this can take a long time (depending on the card size and speed) be patient and let it finish so you do not mess up the file system and have to start from scratch.
Once complete reboot the system again.
You can now verify that the system is using the full capacity of the SD Card by entering the following command:
`df -h`



# References
1. [The original Github Repo](https://github.com/algoaccess/rpi-mass-image-writer)
2. [Open Source Image Duplicator](https://github.com/rockandscissor/osid)
3. [Adafruit Char Plate LCD](https://learn.adafruit.com/adafruit-16x2-character-lcd-plus-keypad-for-raspberry-pi/overview)
4. [i2c setup on Arch Linux](http://cfedk.host.cs.st-andrews.ac.uk/site/?q=2013-pi)
5. [Rpi 2 Model B 3D case top](http://www.thingiverse.com/thing:588608)
6. [Rpi 2 Model B 3D case bottom](http://www.thingiverse.com/thing:582366)
7. [Enable i2c on Arch Linux](http://archlinuxarm.org/forum/viewtopic.php?f=31&t=8330)
8. [dd to multiple drives](https://joshhead.wordpress.com/2011/08/04/multiple-output-files-with-dd-utility/)
