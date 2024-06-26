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
