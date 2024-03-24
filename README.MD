# Project Title

Encrypt Raspberry PI 5 NVMe. PI boot form MicroSD card.

## Description

Encrypting an NVMe drive on a Raspberry Pi is a good way to protect your data in case the drive is lost or stolen. You can use the Linux built-in encryption tools to achieve this. Here's a step-by-step guide to encrypt your NVMe drive on a Raspberry Pi:

## Update and remote:

### Check for updates:
`apt update`

### Check for upgrades:
`apt upgrade`

### Configure SSH & VNC:
`raspi-config`

### Check for firmware update:
`rpi-eepram-update`

## edit `sudo nano /boot/firmware/config.txt`

`dtparam=pciex1`

`dtparam=pciex1_gen=3`

Or use the below one line command to add the lines:

`sudo grep -q "^dtparam=pciex1" /boot/firmware/config.txt || echo "dtparam=pciex1" | sudo tee -a /boot/firmware/config.txt > /dev/null; sudo grep -q "^dtparam=pciex1_gen=3" /boot/firmware/config.txt || echo "dtparam=pciex1_gen=3" | sudo tee -a /boot/firmware/config.txt > /dev/null`


## Install Required Packages:
`apt-get install cryptsetup`

## Partition the NVMe Drive: 
If the NVMe drive is not already partitioned, you can use a partitioning tool such as fdisk or parted to create partitions on the drive.

`sudo fdisk /dev/nvme0n1`

Run fdisk Command: After running sudo fdisk /dev/nvme0n1, you'll be presented with a command-line interface within fdisk.

Create a New Partition: Here are the steps to create a new partition:

Type n and press Enter to create a new partition.
You'll be asked whether you want to create a primary or extended partition. For a typical setup, choose p for primary.
Next, you'll be prompted to specify the partition number. You can usually accept the default value, which is the next available number.
Then, you'll need to specify the starting and ending sectors for the partition. You can typically accept the default values to use the entire disk.
After specifying the ending sector, fdisk will confirm the creation of the partition. If everything looks correct, type w and press Enter to write the changes to the disk.

# Find the new created partition:
`sudo lsblk`
You will find it under nvme0n1, let's say it is nvme0n1p1

## Encrypt the Partition: 
Once you have a partition set aside for encryption, you can encrypt it using cryptsetup. Replace /dev/nvme0n1p1 with the partition you want to encrypt, which you found is lsblk in the above step.

`sudo cryptsetup luksFormat /dev/nvme0n1p1`

You'll be prompted to confirm the action and set a passphrase. Make sure to choose a strong passphrase and remember it, as you'll need it to access the encrypted data.

## Open the Encrypted Volume: 
After encrypting the partition, you need to open it to create a mapping to a device that can be used like any other block device.
You can replace `encrypted_nvme` with the name you want. the replaced name will be on /dev/mapper/

`sudo cryptsetup luksOpen /dev/nvme0n1p1 encrypted_nvme`

## Format the Encrypted Volume: 
Now that the encrypted volume is open, you can format it with the file system of your choice. For example, to format it with ext4:
`sudo mkfs.ext4 /dev/mapper/encrypted_nvme`

## Mount the Encrypted Volume: 
Finally, you can mount the encrypted volume like any other partition.
`sudo mkdir /mnt/encrypted_nvme
sudo mount /dev/mapper/encrypted_nvme /mnt/encrypted_nvme`

You can now use /mnt/encrypted_nvme to access the encrypted volume.

Remember, whenever you want to access the data on the encrypted volume, you'll need to open the encrypted volume using `cryptsetup luksOpen` and provide the passphrase you set during encryption.

## Verify:

To verify if a partition is encrypted, you can examine its header to see if it contains the LUKS (Linux Unified Key Setup) header. This header is a distinctive marker indicating that the partition is encrypted using LUKS.
You can use the cryptsetup command with the luksDump option to check if a partition contains a LUKS header. Here's how to do it:
Open a terminal on your Raspberry Pi.
Run the following command, replacing /dev/nvme0n1p1 with the partition you want to check:

`sudo cryptsetup luksDump /dev/nvme0n1p1`

This command will display detailed information about the LUKS header, including the encryption algorithm, key slots, and other metadata.
Look for output indicating that the device contains a LUKS header. If the command returns information about the LUKS header, it means that the partition is encrypted.
If the partition is encrypted, you'll see output similar to the following:

`LUKS header information for /dev/nvme0n1p1

Version:        2
Cipher name:    aes
Cipher mode:    xts-plain64
Hash spec:      sha256
...`

## Permission issue:

If you can't write on the disk, you may need permission to the user or group. The below example if the username is pi and the group is pi.
You can find the current user `whoami` or `id -u -n` and group `groups` and apply the below command.

`sudo chown pi:pi /mnt/nvme`


