# arch-installation
Small guide on how to install arch manually into a UEFI or BIOS system
## first steps
Let's assume that you have already downloaded and stored an arch linux iso. You should create a virtual machine and start the installer. I'll try to explain how to appropriately install both for a modern UEFI machine or a legacy BIOS.

Once the terminal is loaded you are executing the installer. Keep in mind that until we enter our new system everything we are doing is "external". We have a set of tools (like nano or vim, or even fonts like terminus) that are embedded into the installer but are not present in the new installation.

The first step that I always do is to change the keyboard layout (in this case to Spanish)

{
loadkeys es
}

After that I usually change the font to terminus bold and chage it size accordingly to the DPI of the screen. In the case of the monitor that I currently use I have found the following to be the most adequate:

{setfont ter-128b} 
