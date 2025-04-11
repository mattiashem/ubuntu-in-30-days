# i3 Window Manager Installation and Configuration

Below are the commands and steps to install, configure, and use the i3 Window Manager on Ubuntu. This guide covers installation, screen configuration, setting up multiple monitors, and some extra useful commands.

## Install i3 Window Manager

To install i3 on Ubuntu, open a terminal and run the following command:

```bash
sudo apt install i3
```

This command installs i3 Window Manager along with essential tools. After installation, log out from your current session, select i3 as the window manager, and log in again.

## Basic i3 Usage

### Open Firefox or Brave

To open Firefox or Brave in i3, press `Win + d` (Windows key + d), type the name of the browser (e.g., Firefox), and press Enter. The window will tile beside the terminal, as i3 is a tiling window manager.

### Open a New Terminal

To open a new terminal window, press `Win + ↵` (Windows key + Enter).

## Screen Configuration for i3

If you're using multiple screens or need to adjust screen settings, follow these steps:

1. Navigate to the `.config` directory:

    ```bash
    cd .config
    ```

2. Create the `.XResources` file (note the `.` at the beginning):

    ```bash
    nano .XResources
    ```

3. Add the following settings:

    ```bash
    Xft.dpi: 220
    Xcursor*size: 30
    Xcursor.size: 30
    ```

4. Apply the values by running:

    ```bash
    xrdb .XResources
    ```

5. Reload the window manager with the following key combination:  
    `Win + ⇧ Shift + r` (Windows key + Shift + r).

The settings above work for a Dell XPS with a 4k screen but may need to be adjusted for different screen resolutions.

## Screen Resolution and Multi-Screen Setup

If you need to change the screen resolution or set up additional screens, use the following commands.

1. To check your connected screens:

    ```bash
    xrandr
    ```

2. To set the output of the screen to auto (replace `<enter_screen_name>` with the actual screen name):

    ```bash
    xrandr --output <enter_screen_name> --auto
    ```

3. To set the resolution of the screen (replace `<enter_screen_name>` with the actual screen name):

    ```bash
    xrandr --output <enter_screen_name> --mode 3840x2160 --right-of <enter_screen_name>
    ```

4. To turn off the external screen:

    ```bash
    xrandr --output eDP-1 --auto --output DP-1 --off
    ```

### Create Scripts for Multi-Screen Setup

#### Only Laptop Screen Script

```bash
#!/bin/bash
xrandr --output eDP-1 --auto --output DP-1 --off
```

#### Office Setup Script

```bash
#!/bin/bash
xrandr --output DP-2 --mode 3840x2160 --right-of eDP-1
```

Add these scripts to your `.config` folder and save them with meaningful names.

### Save i3 Settings on Boot

To ensure your settings are applied when i3 starts, add the following line to your i3 configuration file:

```bash
exec --no-startup-id xrdb ~/.config/.XResources
```

## Extra Commands for i3 Configuration

Below are some extra useful commands for managing audio and taking screenshots.

### Open Sound Control (Pavucontrol)

To open the sound control panel, use the following command:

```bash
bindsym $mod+XF86AudioMute exec pavucontrol
```

### Take Screenshots

To take a screenshot using the `gnome-screenshot` tool, add the following commands to your i3 configuration file:

```bash
bindsym Print exec gnome-screenshot
bindsym Control+Print exec gnome-screenshot -i
```

This will allow you to capture the screen or selected area using the Print key.

---

# Custom Shortcuts and Configuration for i3

Here are the commands and steps to customize shortcuts, set backgrounds, configure locking, and more on i3.

## Custom Shortcuts

To add custom shortcuts, open the `i3` config file in your preferred text editor:

```bash
gedit .config/i3/config
```

### Change Mod Key

By default, the Windows key is set as the Mod key. To change this to another key, modify the following line:

```bash
set $mod Mod4
```

For example, to use the Control key instead:

```bash
set $mod Control
```

## Background Image

1. First, download an image to use as a background and save it in your `.config/background` folder.
2. Install the necessary package to set the background:

    ```bash
    sudo apt-get install feh
    ```

3. Add the following line at the bottom of your i3 config file to set the background:

    ```bash
    exec --no-startup-id feh --bg-fill ~/.config/background/image.jpg
    ```

4. Restart i3 by pressing `Mod + Shift + r` to apply the changes.

## Lock Screen

1. Install the required packages for locking the screen:

    ```bash
    sudo apt-get install xautolock i3lock
    ```

2. Add these lines to your i3 config file to configure the lock screen:

    ```bash
    exec xautolock -time 15 -locker 'i3lock -i ~/.config/background/lock.png' &
    bindsym $mod+l exec i3lock -i ~/.config/background/lock.png
    ```

The first command ensures the screen locks after 15 minutes, showing a custom lock screen. The second command binds the lock screen to `Mod + l`.

3. Save and reload i3. Test the lock screen by pressing `Mod + l`.

## Extra Configurations

Add the following to your i3 config to start network and Bluetooth applets in the bottom bar:

```bash
exec --no-startup-id nm-applet
exec --no-startup-id blueman-applet
```

If you do not have Bluetooth, you can remove the second line. Install the Bluetooth applet with:

```bash
sudo apt-get install blueman
```

## Autostart Applications in Specific Workspaces

Set up your workspace and autostart applications by naming your workspaces:

```bash
set $ws1 "1:com"
set $ws2 "2:term"
set $ws3 "3:web"
set $ws4 "4:code"
set $ws5 "5:media"
set $ws6 "6:vm"
set $ws7 "7:misc"
```

Autostart applications on specific workspaces:

```bash
# Start terminal on workspace 2
exec --no-startup-id i3-msg 'workspace 2:term; exec i3-sensible-terminal'

# Start Brave browser on workspace 3
exec i3-msg 'workspace 3:web; exec /snap/bin/brave'

# Start Slack and Discord on workspace 1
exec i3-msg 'workspace 1:com; exec /snap/bin/slack'
exec i3-msg 'workspace 1:com; exec /snap/bin/discord'

# Start VSCode on workspace 4
exec i3-msg 'workspace 4:code; exec /snap/bin/code'
```

### Move Windows to Specific Workspaces

To ensure apps open in the correct workspace, add these lines:

```bash
for_window [class="Slack"] move to workspace 1:com
for_window [class="discord"] move to workspace 1:com
for_window [class="Brave-browser"] move to workspace 3:web
```

## Communication Tools

To install and use Slack, Teams, and Discord:

```bash
sudo apt-get install slack
sudo apt-get install teams
sudo apt-get install discord
```

## Video Streaming Tools

### Install VLC

```bash
sudo apt-get install vlc
```

To stream video from an IP camera:

```bash
vlc rtsp://10.100.0.90:554/s2
```

### Install OBS Studio for Live Streaming

1. Add the OBS PPA:

    ```bash
    sudo add-apt-repository ppa:obsproject/obs-studio
    sudo apt update
    ```

2. Install OBS Studio:

    ```bash
    sudo apt install ffmpeg obs-studio
    ```

3. Start OBS Studio and configure your streaming settings.

## Webcam

To install the Cheese webcam app (if not installed):

```bash
sudo apt-get install cheese
```

## Syncing Files

For syncing files between Google Drive and Ubuntu, use `Odrive`:

```bash
https://flathub.org/apps/details/io.github.liberodark.OpenDrive
```

Install Dropbox:

```bash
https://www.dropbox.com/install-linux
```

Install Mega:

```bash
https://mega.io/desktop
```

For syncing files between devices, use Resilio:

```bash
https://www.resilio.com/
```

## Git Setup

Install Git:

```bash
sudo apt-get install git
```

Generate an SSH key:

```bash
ssh-keygen
```

Add the SSH key to GitHub:

```bash
cat .ssh/id_rsa.pub
```

Clone a Git repository:

```bash
git clone https://github.com/bpbpublications/Ubuntu-Linux-in-30-days.git
```

## Install Visual Studio Code (VSCode)

To install VSCode, download the `.deb` package from the official website or use the following command if you have a `.deb` file:

```bash
sudo dpkg -i <name of the file you downloaded>
```

