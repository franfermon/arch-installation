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

**For FAT 32

> mkfs.fat -F 32 /dev/*efi_system_partition*

**For btrfs

> mkfs.btrfs /dev/*root_partition*

**For ext4

> mkfs.ext4 /dev/*root_partition*

**For swap

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

## Minimal installation packages

We will leverage the pacstrap command (remember that we are still "external" to our OS so that's why we cannot use pacman) to create what I consider a minimal installation of arch linux:

    pacstrap -K /mnt base base-devel linux linux-firmware networkmanager wpa_supplicant vim grub

