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

    device *device* set-property Powered on
    adapter *adapter* set-property Powered on

With everything ready you can start scanning the networks around you:

    station *device* scan

And list all of them:

    station *device* get-networks

Finally you can connect to a network with the following command:

    station *device* connect *SSID*

## Peparing the hard drive

We will proceed now to create the necessary partitions to install our system:

### UEFI
| Partition | type | size |
| ----------- | ----------- | ----------- |
| boot | EFI system | 512MiB approx. |
| root | linux file system | Disk size - boot - swap | 
| swap (optional) | linux swap | RAM size approx. |

### BIOS
| Partition | type | size |
| ----------- | ----------- | ----------- |
| root | linux file system | Disk size - swap | 
| swap (optional) | linux swap | RAM size approx. |

we will do so with the following command which will load a guided partition tool

     cfdisk


