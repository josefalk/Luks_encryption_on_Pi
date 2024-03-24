# Project Title

Encrypt Raspberry PI 5 NVMe. PI boot form MicroSD card.

## Description

Encrypting an NVMe drive on a Raspberry Pi is a good way to protect your data in case the drive is lost or stolen. You can use the Linux built-in encryption tools to achieve this. Here's a step-by-step guide to encrypt your NVMe drive on a Raspberry Pi:

## Update and remote:

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

## edit `config.txt`

```
sudo nano /boot/firmware/config.txt
```

`dtparam=pciex1`

`dtparam=pciex1_gen=3`

Or use the below one line command to add the lines:

```
sudo grep -q "^dtparam=pciex1" /boot/firmware/config.txt || echo "dtparam=pciex1" | sudo tee -a /boot/firmware/config.txt > /dev/null; sudo grep -q "^dtparam=pciex1_gen=3" /boot/firmware/config.txt || echo "dtparam=pciex1_gen=3" | sudo tee -a /boot/firmware/config.txt > /dev/null
```


## Install Required Packages:
```
sudo apt-get install cryptsetup
```

## Partition the NVMe Drive: 
If the NVMe drive is not already partitioned, you can use a partitioning tool such as fdisk or parted to create partitions on the drive.

## Encrypt Partition:

```
sudo cryptsetup luksFormat /dev/nvme0n1p3
```


## Open the Encrypted partition:
```
sudo cryptsetup open /dev/nvme0n1p3 encrypted_home
```

## Create File System:
```
sudo mkfs.ext4 /dev/mapper/encrypted_home
```

##Mount the Encrypted Partition:

Create a directory for mounting and mount the encrypted partition to your home directory:
```
sudo mkdir /mnt/encrypted_home
sudo mount /dev/mapper/encrypted_home /mnt/encrypted_home

```

## Configure Automatic Mounting:

To automatically mount the encrypted partition on boot, you'll need to add an entry to `/etc/crypttab`:

```
sudo nano /etc/crypttab
```
```
/dev/mapper/encrypted_home /home ext4 defaults 0 2
```

## Unmount and Test:
```
sudo umount /mnt/encrypted_home
```
```
sudo cryptsetup close encrypted_home
```
```
sudo mount -a
```

## Verify:

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


## Permission issue:

If you can't write on the disk, you may need permission to the user or group. The below example if the username is pi and the group is pi.
You can find the current user `whoami` or `id -u -n` and group `groups` and apply the below command.

```
sudo chown pi:pi /mnt/nvme
```


