# Raspberry Pi Bitcoin Core Node 0.15.1+

This is a basic guide is for setting a Bitcoin Core 0.15.1+ node on a headless (no monitor / keyboard)
raspberry pi. Most of the work is in setting up the raspberry pi.
These are the steps that worked for me, your setup might be slightly different but hopefully this will
guide you in the right direction.

## Requirements
- SD Card with at least 4GB
- An external hard drive with at least 500GB / 1TB depending on how long you
expect to have your node running until the blockchain grows too large.
- A PSU that can give enough power to the raspberry pi. If the red light is
blinking then this is a good sign you haven't got enough juice.

## Install OS
Look for the latest image of [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/) and use [Etcher](https://etcher.io/) to burn it on the SD Card.
You'll need to make some changes before loading the card on the raspberry pi.

## Mounting SD Card on Mac OS

If you're working on Linux computer you should already see two partitions for the
SD card. On a Mac there's a bit more work, I've only been able to do this through VirtualBox:

- Start downloading a copy of the latest Ubuntu LTS (you'll need this later).
- On a terminal, find the mount point of the card with `mount`. I.e. /dev/disk2.
- Double check by ejecting the card and running the command again, verifying it is gone.
- Create a vmdk file by running `sudo VBoxManage internalcommands createrawvmdk -filename "~/sd-card.vmdk" -rawdisk /dev/disk2`.
- Set permissions to this file with `chmod 777 ~/sd-card.vmdk`.
- Set permissions to the mount `chmod 777 /dev/disk2` (You'll have to do this every time you unplug / plug it in).
- In VirtualBox, create a new machine and configure it for Linux (tip: add a bit more RAM and another CPU core or two to make this it a bit smoother to run)-
- Go to the settings for the machine you just created and under "Storage" create a SATA controller that points to that .vmdk file.
- Run the machine and select the Ubuntu image you downloaded earlier.
- Choose "Try Ubuntu" and when the OS loads you should be able to see the partitions under /media-

## Connectivity
By default, a raspbian installation doesn't have ethernet and wifi connectivity
setup. If you wish to have a wifi dongle setup as well as ethernet:

1. Put the ssh and wpa-supplicant.conf files in the /boot partition. (Replace the SSID and password with the one for your network).
2. In the other partition replace /etc/network/interfaces with the interfaces file provided here.
3. The SD card is ready now, eject it, plug it into the raspberry pi along with the ethernet cable / wifi dongle and boot it up.
4. Check your router's DHCP for the IP that was assigned to your device.
5. SSH into it with the user "pi" and password "raspberry".

If you want to have a static IP you can either do this on the router by assigning it to the raspberry pi's MAC address, or you
can set it up in the interfaces file.

## External Hard Drive
This will be the device where the node will store the blockchain data.
Once you've SSH'd in the device, connect your external hard drive.

1. Format your drive with either FAT32 of exFAT (I chose exFAT).
2. Put any random file in the drive (this is so we can check it is properly mounted later).
3. Connect the external hard drive to one of the free usb ports on the raspberry pi.
4. Update and install exfat dependencies if you've chosen exFAT.
```
sudo apt-get update
sudo apt-get install exfat-fuse
```
5. List the partitions `sudo lsblk -o UUID,NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,MODEL` and identify the name of the disk for your drive, i.e. "sda1".
6. Run `sudo blkid` to get the location of the disk partition i.e. `/dev/sda1`. Also get the UUID of the drive i.e. `5C24-1453`, you'll need this later if you want to auto-mount.
7. Create a target folder i.e. `sudo mkdir /mnt/usb_drive`.
8. Mount the drive: `sudo mount /dev/sda1 /mnt/usb_drive`.
9. Check that the drive is mounted by looking for the file you put in before: `ls /mnt/usb_drive`

To auto-mount the drive:

- Edit the fstab file and include this line `UUID=5C24-1453 /mnt/PIHDD exfat defaults,auto,umask=000,users,rw 0 0` (Use your UUID from before!)

## Install Bitcoin Core
1. Go to the [bitcoin core download page](https://bitcoin.org/en/download) and grab the link for ARM 32 bit.
2. Run `wget https://bitcoin.org/bin/bitcoin-core-0.15.1/bitcoin-0.15.1-arm-linux-gnueabihf.tar.gz` (Use the link you got).
3. Verify the signatures!
4. Unpack the file `tar xzf bitcoin-0.15.1-arm-linux-gnueabihf.tar.gz`.
5. Symlink the binaries `sudo ln -s ~/bitcoin-0.15.1/bin/* ~/bin/.`, then `source .profile`.

## Run it!
1. Create a folder for the bitcoin data in your drive `mkdir /mnt/usb_drive/.bitcoin`.
2. Add the bitcoin.conf file included in the `.bitcoin` folder you just created.
3. For convenience, lets create some aliases in ~/.profile.
```
alias bd = "bitcoind -datadir=/usr/usb_drive/.bitcoin -daemon"
alias bcli = "bitcoin-cli -datadir=/usr/usb_drive/.bitcoin"
```
4. Source the profile `source ~/.profile`.
5. Run the daemon `bd` - This will start your node!
6. Verify it's working `bcli getinfo`.

Congratulations, your node is now on its way to being part of the network!
Now go ahead and [explore the APIs](https://bitcoin.org/en/developer-reference#bitcoin-core-apis)!
