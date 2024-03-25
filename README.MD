# Luks Encryption on NVMe Partition in PI5:

Encrypt Home Directory in Raspberry PI 5 NVMe.

The setup ment to have a sperate partition for the home directory in the same NVMe disk. 

## Description

Encrypting an NVMe drive on a Raspberry Pi is a good way to protect your data in case the drive is lost or stolen. You can use the Linux built-in encryption tools to achieve this. Here's a step-by-step guide to encrypt your NVMe drive on a Raspberry Pi.

To avoid removing the NVMe every time you install PI OS on it, I prepared the SD card, Install PI OS in it, I insert it (Pi will boot from SD by default) and then I can install PI OS on NVMe drive and partition it in the same time.

### Prepare and partition the NVMe:

You can prepare the NVMe from USB reader of course, It will be similar steps.



![Image](image1.png)
![Image](image2.png)

After that reboot and remove the SD card, the PI will automatically boot from NVMe.

### Check for updates:
```
sudo apt update
```

### Check for upgrades:
```
sudo apt upgrade
```

### Configure SSH & VNC:
```
sudo raspi-config
```

### Check for firmware update:
```
sudo rpi-eepram-update
```

### edit `config.txt` to overclock PCIe from Gen2 to Gen3

```
sudo nano /boot/firmware/config.txt
```

`dtparam=pciex1`

`dtparam=pciex1_gen=3`

Or use the below one line command to add the lines:

```
sudo grep -q "^dtparam=pciex1" /boot/firmware/config.txt || echo "dtparam=pciex1" | sudo tee -a /boot/firmware/config.txt > /dev/null; sudo grep -q "^dtparam=pciex1_gen=3" /boot/firmware/config.txt || echo "dtparam=pciex1_gen=3" | sudo tee -a /boot/firmware/config.txt > /dev/null
```

Below is a Benchmark test before/after overclicking the PCIe form Gen2 to Gen3.
Benchmark test using https://pibenchmarks.com/
```
sudo curl https://raw.githubusercontent.com/TheRemote/PiBenchmarks/master/Storage.sh | sudo bash
```
Photos show comparison before and after overclocking the PCIe form Gen2 to Gen3.

![Image](image3.png)
![Image](image4.png)

## Install Required Packages:
```
sudo apt-get install cryptsetup
```

## Partition the NVMe Drive: 
If the NVMe drive is not already partitioned, you can use a partitioning tool such as fdisk or parted to create partitions on the drive.

### Encrypt Partition:

```
sudo cryptsetup luksFormat /dev/nvme0n1p3
```


### Open the Encrypted partition:
```
sudo cryptsetup open /dev/nvme0n1p3 encrypted_home
```

### Create File System:
```
sudo mkfs.ext4 /dev/mapper/encrypted_home
```

##Mount the Encrypted Partition:

Create a directory for mounting and mount the encrypted partition to your home directory:
```
sudo mkdir /mnt/encrypted_home
sudo mount /dev/mapper/encrypted_home /mnt/encrypted_home

```

### Configure Automatic Mounting:

To automatically mount the encrypted partition on boot, you'll need to add an entry to `/etc/crypttab`:

## Update /etc/cryptab
```
sudo nano /etc/crypttab
```
```
/dev/mapper/encrypted_home /home ext4 defaults 0 2
```

## Update /etc/fstab:
Next, you need to update the /etc/fstab file to mount the encrypted partition on boot. Open the file:
```
sudo nano /etc/fstab
```
```
/dev/mapper/encrypted_home /home ext4 defaults 0 2
```
## Permission & Create Home directory:
```
sudo mkdir pi
```
```
sudo chown pi:pi /home/pi
```
Create a file and save it to test permissions.

### Unmount and Test:
```
sudo umount /mnt/encrypted_home
```
```
sudo cryptsetup close encrypted_home
```
```
sudo mount -a
```
## Reboot:

You will be asked for the password. if PI stuck on flash screen, press f12 to enter the password.

### Verify:

To verify if a partition is encrypted, you can examine its header to see if it contains the LUKS (Linux Unified Key Setup) header. This header is a distinctive marker indicating that the partition is encrypted using LUKS.
You can use the cryptsetup command with the luksDump option to check if a partition contains a LUKS header. Here's how to do it:
Open a terminal on your Raspberry Pi.
Run the following command, replacing /dev/nvme0n1p1 with the partition you want to check:

```
sudo cryptsetup luksDump /dev/nvme0n1p3
```

This command will display detailed information about the LUKS header, including the encryption algorithm, key slots, and other metadata.
Look for output indicating that the device contains a LUKS header. If the command returns information about the LUKS header, it means that the partition is encrypted.
If the partition is encrypted, you'll see output similar to the following:

LUKS header information for /dev/nvme0n1p1

Version:        2
Cipher name:    aes
Cipher mode:    xts-plain64
Hash spec:      sha256


Check Mount Options:
First, check the mount options for your home directory to see if it's mounted from an encrypted partition. You can do this by running the mount command:

```
mount | grep '/home'
```
If your home directory is mounted from an encrypted partition, 
you should see output indicating that it's using a device mapper device (e.g., `/dev/mapper/encrypted_home`).

Check Filesystem Type:
You can also check the filesystem type of your home directory to verify if it's an encrypted filesystem. Run:

```
df -T /home
```
If the filesystem type is crypt, ext4, or any other encrypted filesystem type, it indicates that your home directory is encrypted.

Check `/etc/fstab` and `/etc/crypttab`:
Review the /etc/fstab file to see if there's an entry for your home directory that points to an encrypted device. Additionally, check the /etc/crypttab file to verify the mapping name and device associated with the encrypted partition.

Check Encrypted Volume Status:
You can also check the status of the encrypted volume using cryptsetup. Run:
```
sudo cryptsetup status encrypted_home
```


## Key Storage:

If you want to store the key so you don't enter the password every time:

Create a key file, Let's say it is on /etc/password
write the password in it.

### Edit /etc/crypttab:
Ensure that the key file is correctly specified in the `/etc/crypttab` file. Open `sudo nano /etc/crypttab` and verify that the entry for your encrypted volume specifies the correct key file. It should look something like this:

```
encrypted_home  /dev/nvme0n1p3  /etc/password  luks
```
### Rebuild the Initramfs:
After making changes to `/etc/crypttab`, you might need to rebuild the initramfs to apply the changes. You can do this using the following command:
```
sudo update-initramfs -u
```
This command rebuilds the initramfs with the updated configuration from `/etc/crypttab`.

### Verify Key File Permissions:
Ensure that the permissions of the key file are set correctly so that it's accessible during boot. The key file should be readable by the root user only. You can set the correct permissions using the following command:

```
sudo chmod 400 /etc/password
```
### Test Manually (Optional):
Try manually unlocking the encrypted volume using the key file to see if it works outside of the boot process. You can use the following command:
```
sudo cryptsetup luksOpen --key-file /etc/password /dev/nvme0n1p3 encrypted_home
```
If this command successfully unlocks the volume, then the key file is valid, and the issue may lie in the boot process configuration.





