# Gustavo's Arch Linux

## Acquire an installation image

The first step is downloading the ISO file:

1. Visit the web page <https://archlinux.org/download/>.
2. Choose a mirror.
3. Download the file `archlinux-x86_64.iso`.
4. (Optional) Download the files `archlinux-x86_64.iso.sig` and `b2sums.txt`.

On Linux, we can download the files on the command line using `wget`. To install it run:

On Arch Linux:

```
sudo pacman -S wget
```

On Ubuntu:

```
sudo apt install wget
```

On Mac OS:

```
brew install wget
```

Then edit the variable `MIRROR` accordingly and execute:

```
MIRROR="http://linorg.usp.br/archlinux/iso/latest"
wget $MIRROR/archlinux-x86_64.iso -O archlinux-x86_64.iso
wget $MIRROR/archlinux-x86_64.iso.sig -O archlinux-x86_64.iso.sig
wget $MIRROR/b2sums.txt -O b2sums.txt
```

(The `-O` option is necessary to overwrite older versions of the files if they already exist.)

## Verify the ISO file (optional)

This is step is optional (but recommended). We will verify if the ISO file is not damaged and was not modified. To do this, we will use `gpg` and `b2sum`. Both programs should already be installed on Linux because they are part of the core utilities. On Mac OS, you may need to install `b2sum` (`gpg` should be available). To install it run:

On Mac OS:

```
brew install coreutils
```

The following commands check the b2sum and verify the signature of the ISO file:

```
b2sum -c b2sums.txt
gpg --keyserver-options auto-key-retrieve --verify archlinux-x86_64.iso.sig
```

We will get an error message corresponding to the files that we did not download. Ignore these messages. The relevant output lines are the following:

```
archlinux-x86_64.iso: OK
gpg: Good signature from "Pierre Schmitz <pierre@archlinux.org>" [unknown]
```

## Prepare an installation medium

### On Linux

Find out the name of your USB drive with `lsblk`. Edit the variable `USBDRIVE` accordingly.

Make sure that the drive is not mounted. Copy the ISO file to the USB drive using `dd`:

```
USBDRIVE="/dev/sdb"
dd bs=4M if=archlinux-x86_64.iso of=$USBDRIVE conv=fsync oflag=direct status=progress
```

The above command has to be executed as superuser.

### On Mac OS or Windows

See instructions in <https://wiki.archlinux.org/title/USB_flash_installation_medium>.

## Verify the boot mode

Boot the system with the USB drive where you wrote the ISO file.

After booting the live environment, execute

```
cat /sys/firmware/efi/fw_platform_size
```

If the command returns `64`, then the system is booted in 64-bit x64 UEFI mode.

## Connect to the network

Below, we describe how to connect to the network using `ip`, `iwctl` and `dhcpcd`.

Check the network connection with `ip`:

```
ip addr
```

Turn on network device:

```
ip link
ip link set <device> up
ip link show <device>
```

(Wireless) Connect to wi-fi with `iwctl`:

```
iwctl
device list
station <device> scan
station <device> get-networks
station <device> connect <network_name>
station <device> show
quit
```

Get a dynamic  IP address with `dhcpcd`:

```
dhcpcd <device>
```

Get a static IP address:

```
dhcpcd -S ip_address=150.164.25.215/24 \
	-S routers=150.164.25.254 \
	-S domain_name_servers=8.8.8.8 <device>
```

Test the internet connection:

```
ping -c 3 archlinux.org
```

## Update the system clock

Check clock:

```
timedatectl status
```

## Partition the disks

List partitions:

```
fdisk -l
```

EFI with GPT using gdisk:

```
gdisk /dev/sda
o
n
+400M
ef00
n
8300
w
```

EFI with GPT using fdisk:

```
fdisk /dev/sda
g
n
t 1
n
t 20
w
```

## Format the partitions

Execute

```
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

## Mount the file systems

Execute

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## Select the mirrors (optional)

Execute

```
COUNTRY="Brazil"
reflector -c $COUNTRY -a 6 --sort rate --save /etc/pacman.d/mirrorlist
```

## Install base system

Execute

```
pacstrap -K /mnt base linux linux-firmware
```

### Packages

```
base
linux
linux-firmware
```

## Generate fstab

Create

```
/etc/fstab
```

Add

```
genfstab -U /mnt >> /mnt/etc/fstab
```

## Change root

Change root into the new system:

```
arch-chroot /mnt
```

## Timezone

Set the time zone by creating

```
/etc/localtime
```

Add

```
TIMEZONE="America/Sao_Paulo"
ln -sf /usr/share/zoneinfo/$TIMEZONE /etc/localtime
```

Execute

```
hwclock --systohc
```

## Localization

Edit

```
/etc/locale.gen
```

Add

```
sed -i 's/^#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
locale-gen
```

Edit

```
/etc/locale.conf
```

Add

```
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```
