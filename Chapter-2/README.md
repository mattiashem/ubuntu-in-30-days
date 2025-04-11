## 


# Verify Computer Device

When Ubuntu is installed, it automatically loads the drivers for various devices like network cards, video cameras, etc. However, sometimes Ubuntu might face issues setting up a device, and the following commands can help troubleshoot and find a resolution. 

From the output of the commands, you can search the internet for the device ID and find Ubuntu guides on how to use the device with Ubuntu.

## Commands

### List All USB Devices

To list all USB devices connected to your Ubuntu machine, run the following command:

```bash
lsusb -vvv
```

The output will display detailed information about all detected USB devices connected to the computer.

### List All PCI Devices

To list all PCI devices connected to your Ubuntu system, use the following command:

```bash
lspci -vvv
```

This command provides information about internal devices like network cards, sound cards, and more. If there are errors with devices such as the sound card or network card, this command will help you see what is detected.

## Verify Devices

After running these commands, you can verify if your devices like soundcards, microphones, or cameras are showing up properly. If there are issues, further troubleshooting steps can be done by searching the device ID online and consulting Ubuntu guides.