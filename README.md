# rpi-mass-image-writer

**Full build and operating instructions at [Core Electronics](https://core-electronics.com.au/projects/mass-sd-card-image-writer)**

Raspberry Pi that writes to many USB drives / SD cards at once. Hardware interface available to select the stored disk image to copy as well as to shutdown the device. 

Disk images are transferred to the Raspberry Pi via a Samba shared folder. The selected image is then written in parallel to all the drives at once using the ```dd```, ```pv``` and ```tee``` commands 

![Screen](/photos/console-and-hub.jpg)

![Screen](/photos/sd-cards-in-adapters.JPG)



# How to use?
1. Login to shared folder - Windows: navigate to `\\[hostname]\images`
2. Transfer image files to the shared folder.
3. Plug in all drives to write to.
4. Press left button to enumerate all images and drives.
5. Use up/down buttons to select image.
6. Press the right button to start writing to drives. Press again to halt writing.
7. Press the select button (extreme left) to shutdown device properly to prevent data corruption. The words "Shutting down" will remain even after the device has completely shutdown, so just wait for the activity light to turn off before removing power.
