# Manual arch linux installation guide
Small guide on how to install arch manually into a UEFI or BIOS system
## Terminal preparation
Let's assume that you have already downloaded and stored an arch linux iso and have bootable media. I'll try to explain how to appropriately install both for a modern UEFI machine or a legacy BIOS.

Once the terminal is loaded you are executing the installer. Keep in mind that until we enter our new system everything we are doing is "external". We have a set of tools (like nano or vim, or even fonts like terminus) that are embedded into the installer but are not present in the new installation.

The first step that I always do is to change the keyboard layout (in this case to Spanish)

    loadkeys es

After that I usually change the font to terminus bold and chage it size accordingly to the DPI of the screen. In the case of the monitor that I currently use I have found the following to be the most adequate:

    setfont ter-128b

## Verify boot mode 
To verify which boot mode you are using you should enter the following command:

    cat /sys/firmware/efi/fw_platform_size

if it returns a number, you are on UEFI (32 or 64 bits). If it says that the file does not exist, you are on BIOS.
## Internet connection
Installing arch without a working internet connection is virtually impossible. If you are on a virtual machine you are probably spoofing an ethernet connection so you are all set. In any case you can check that your internet connection is working with the following command:

    ping -c 1 archlinux.org

Now, if you are installing arch locally in your machine, you'll make your life easier with an ethernet connection. Nonetheless, the bootable image comes with a wi-fi utility called iwd. To start it you just type:
    
    iwctl

You can list the wi-fi devices after iwd is started with:

    device list

if the device or adapter is not turned on you should use:

> device *device* set-property Powered on

> adapter *adapter* set-property Powered on

With everything ready you can start scanning the networks around you:

> station *device* scan

And list all of them:

> station *device* get-networks

Finally you can connect to a network with the following command:

> station *device* connect *SSID*

## Peparing the hard drive

We will proceed now to i) create the necessary partitions to install our system ii) format them and iii) mount them. Below you can find a table with all the required parameters.

### UEFI Partition guide
| Partition | type | size | file system | mount point |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| boot | EFI system | 512MiB approx. | FAT32 | /mnt/boot |
| root | linux file system | Disk size - boot - swap | Recommended ext4 or btrfs |/mnt |
| swap (optional) | linux swap | RAM size approx. | Not applicable | Not applicable |

### BIOS Partition guide
| Partition | type | size | file system | mount point |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| root | linux file system | Disk size - swap | Recommended ext4 or btrfs | /mnt | 
| swap (optional) | linux swap | RAM size approx. | Not applicable | Not applicable |

### Partitioning the disk
We will start by loading a guided partition tool:

     cfdisk

If you are in UEFI select GPT, in BIOS select DOS.

Then you should find your disk in a /dev/*yourdiskname* format i.e: dev/sda1. Using the UI create the partitions according to the table at the beginning of this section. After that select "Write" and then "Quit".

Let's check that everything is correctly partitioned with this command:

    lsblk -f

### Creating the file systems
As you will see, there is no file system yet in each of the partitions. We will create them now with the following commands:

**For FAT 32**

> mkfs.fat -F 32 /dev/*efi_system_partition*

**For btrfs**

> mkfs.btrfs /dev/*root_partition*

**For ext4**

> mkfs.ext4 /dev/*root_partition*

**For swap**

> mkswap /dev/*swap_partition*

Again, let's check that everything is correctly applied:

    lsblk -f

### Mounting the partitions

We will start by mounting the root volume into the default mount point (/mnt):

> mount /dev/*root_partition* /mnt

And now if you are in a UEFI system you can mount the EFI system partition with:

> mount --mkdir /dev/*efi_system_partition* /mnt/boot

Please note that this is only a suggestion since you can mount this partition into other mount points like the widely used /mnt/boot/efi

We will continue by enabling the swap:

> swapon /dev/*swap_partition*

## Minimal installation

We will leverage the pacstrap command (remember that we are still "external" to our OS so that's why we cannot use pacman) to create what I consider a minimal installation of arch linux:

    pacstrap -K /mnt base base-devel linux linux-firmware networkmanager wpa_supplicant vim grub sudo

Once the installation finishes we are going to generate the fstab file and place it where it is required in one command:

    genfstab -U /mnt >> /mnt/etc/fstab

## Configuring our new system

We are at last going to enter our new system:

    arch-chroot /mnt

### Localization
Our first step here is to generate the locales needed by modifiying the file:

  vim  /etc/locale.gen

In here we will uncomment (erase the #) of the locales that we need. In my case en_US.UTF-8 and es_ES.UTF-8, save and exit. After that we will execute the command:

    locale-gen

And now we will define the language of the system by creating the file:

    vim /etc/locale.conf

And adding here our desired language by writing:

    LANG=en_US.UTF-8

Even though the changes to the terminal that we did on our very first step are still present, the moment we reboot this machine both the keymap and the font will go back to its defaults. To prevent this we will first download the terminus font with:

    pacman -S terminus-font
    
Then create the file:

    vim /etc/vconsole.conf

and in here we will add our keymap and font of preference. In my case:

    KEYMAP=es
    FONT=ter-128b

### User management

Before we reboot into our new system we will create a hostnane by creating the following file:

    vim /etc/hostname

And writing in here your desired hostname (the name of your machine).

Subsequently we will set the root password with the command

    passwd

And now we will set up our user with the command:

> useradd -m -G wheel *username*

What this will do is not only create the new user but also generate their home folder (with the -m) and add them to the wheel (the usual administrator group) with the -G. We want our user to be able to use sudo so we will go into the file with the following command:

    visudo

In here we will uncomment the line:

    %wheel      ALL=(ALL:ALL) ALL

Exit and save.

Finally, lets generate a password for our new administrator and exit root for safety reasons.

> passwd *username*
> su *username*

We can check if the account truly has sudo permissions by executing the command:

    sudo whoami

If it returns *root*, the sudo configuration has been successful.

## Bootloader setup

As you might have noticed on our minimal installation we have included GRUB. GRUB will be our bootloader of choice and we will configure it now:

### UEFI

> sudo pacman -S efibootmgr

> grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

> grub-mkconfig -o /boot/grub/grub.cfg

### BIOS

> grub-install --target=i386-pc */dev/yourdisk*

> grub-mkconfig -o /boot/grub/grub.cfg

## First boot into Arch Linux

Once we are here we will just type 'exit' until we are back into the root@archiso and from there we will type 'reboot'. Do not forget to extract the installation USB/CD to prevent booting from it.

We will be greeted with a terminal (hopefully with our preferred settings applied) and we will login with our username and password.

You will notice that if you do a ping command we do not have internet. In order to fix this we need to type this commands (the first one will enable the service on every boot and the second one will start it).

    sudo systemctl enable NetworkManager.service

    sudo systemctl start NetworkManager.service
   
And lets replicate this step for the wpa_supplicant service

    sudo systemctl enable wpa_supplicant.service

    sudo systemctl start wpa supplicant.service


