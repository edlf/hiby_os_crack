
# Unbricking guide

## Generic Guide for R1/R3 Devices

If you can boot the recovery mode (Vol+ and power key), you don’t need to follow this guide. Just place the upt file in the root of your SD and recover using the recovery mode.

### What you need
- Cloner (cloner-2.5.80.2-windows_alpha or cloner-2.5.80.2-ubuntu_alpha)
- Cloner driver (cloner-win32-driver) only for windows)
- A recovery image (the upt file is missing u-boot, the recovery kernel and rootfs so that is not enough). They are provided in this repo. Both have been updated to work with the 2.5.80.2 version of the cloner.
- Know the bootrom key combo (for example the R1 uses the next track key)

### How to recovery your hiby in case of a brick

1. Make sure your player is off, hold the next track button and plug it
    to your PC. For windows PCs you need to install the cloner driver.
    If your player doesn’t show up you can try to hold the power button
    and the next track button till it shows up.

2. If you succeeded there should be a libusb-win32 device section in your device manager with an "usb cloner device".

3. Next, you will need to unpack the cloner, and open the cloner.exe. In the bottom right corner there’s a combo box that allows you to select the language, pick English.

4. Now click "Load Image" and select the recovery image you downloaded before.

5. Just press "Start", if all went well your device should now start to be flashed. It takes 1 minute or so. After its done your player will reboot and you are done!

## Additional Notes

### Adapting existing recovery packages

If you have a recovery package that doesn’t work with your cloner version, eg you are facing a SoC detection error for example, this can easily be fixed.

You need to modify the recovery package with 7z and replace the files in the firmwares/x1600 folder with the ones that come with the cloner tool you are trying to use (config.cfg, spl.bin, uboot.bin), these files are also located in the firmwares/x1600 folder in the cloner folder.

### Preloading a custom firmware
You can replace the image/kernel.bin and image/rootfs.squashfs files with the ones from your favourite custom firmware. So you will boot straight to your chosen firmware.

### Advanced settings
For advanced users, you can unlock advanced options by setting the lock level to 0 and typing the following password "!@#". This exposes a lot of potentially dangerous options, if you break your device you get to keep the pieces.
