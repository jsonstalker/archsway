# **My** Arch Linux + Sway Installation Guide
<p align="center">
  <img src="./images/archlinux-logo-dark-1200dpi.png" height="100" />
  <img src="./images/sway-logo.png" height="100" />
</p
 
Sources:
* [Installation guide - ArchWiki](https://wiki.archlinux.org/title/Installation_guide)
* [Arch Linux: An 𝔼𝕟𝕔𝕣𝕪𝕡𝕥𝕖𝕕 Guide](https://youtu.be/kXqk91R4RwU?si=1yv3FowmuQ-yczeG)
* [How to Install Arch Linux: Step-by-Step Guide](https://youtu.be/FxeriGuJKTM?si=CjofZtAmX6EIGYZT)
## 1. Pre-installation
### 1.1. Download an installation image
An Arch Linux ISO file should be downloaded. It is recommended to download the Arch Linux ISO file from the [official Arch Linux download page](https://archlinux.org/download/) using BitTorrent, as suggested by the website.
### 1.2. Verify signature (optional but strongly recommended)
A PGP Signature file also can be downloaded for checking the integrity and authenticity of the ISO file. This file is located on the [Checksums and signatures](https://archlinux.org/download/#checksums) section of the page. Now download the [GnuPG](https://gnupg.org/download/index.html) software and install it. If you're on Windows you should download and install the [GPG4WIN](https://gpg4win.org/download.html) software. The PGP Signature file should be placed on the same directory as the ISO file. Now open a terminal in that directory and run the following command:
``` bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-_version_-x86_64.iso.sig
```
After running the command:
* Confirm “Good signature from…” appears.
* Compare the key fingerprint shown to the [official Arch Linux developer signing keys](https://archlinux.org/master-keys/).
* If it matches, the ISO is verified.
* If you see “BAD signature” or fingerprints do not match, do not use the ISO
### 1.3. Prepare an installation medium
Plug in a USB flash drive and write the ISO file to it using an app like [Balena Etcher](https://etcher.balena.io/). Now the flash drive is bootable.
### 1.4. Boot the live environment
Now reboot your system and open the UEFI (BIOS) firmware setup by pressing the respective key (for me it's F2[^1]). Disable **Secure Boot** (usually found in the Security tab). Also navigate to the boot tab and move the flash drive to the top of the boot order. Save the settings. Your PC will now boot into the live environment.

After a bit of time the installation bootloader will appear. Select *Arch Linux install medium* and press `Enter`. You will be logged in on virtual console.
### 1.5. Verify the boot mode
To verify the boot mode, check the UEFI bitness:
``` bash
cat /sys/firmware/efi/fw_platform_size
```
* If the command returns 64, the system is booted in UEFI mode and has a 64-bit x64 UEFI.
* If the command returns 32, the system is booted in UEFI mode and has a 32-bit IA32 UEFI. While this is supported, it will limit the boot loader choice to those that support mixed mode booting.
* If it returns No such file or directory, the system may be booted in BIOS (or CSM) mode.

If the system did not boot in the mode you desired (UEFI vs BIOS), refer to your motherboard's manual. 
### 1.6. Connect to the internet
If you're connected through a cable you might already have network connectivity. If you want to connect to a Wi-Fi, make sure the card is not blocked with rfkill. First run the `rfkill` command to check the current status:
```bash
rfkill
```
Expected output:
```text
ID TYPE      DEVICE      SOFT      HARD
 0 bluetooth hci0   unblocked unblocked
 1 wlan      phy0   unblocked unblocked
```
If it was *hard-blocked* toggle the hardware button to unblock it. If it was *soft-blocked* run the following command to unblock it:
```bash
rfkill unblock wlan
```
Now run the following command to get the interface name for the Wi-Fi:
```bash
ip addr show
```
In my case the name was `wlan0`. Now run the `iwctl` command. It will activate another command prompt that's dedicated to Wi-Fi:
```bash
iwctl
```
In the new command prompt run to view available Wi-Fi networks:
```bash
station wlan0 get-networks
```
Expected output:
```text
                                     Available networks
------------------------------------------------------------------------------------
        Network name                Security                  Signal
------------------------------------------------------------------------------------
        Narnia                      psk                      *****
        Dimension X                 psk                      *****
        Atlantis                    psk                      *****
        Galaxy                      psk                      *****
------------------------------------------------------------------------------------
```
Now exit the `iwctl` commnad prompt:
```bash
exit
```
Run the following command to connect to your desired Wi-Fi network. For example, to connect to *Narnia*:
```bash
iwctl --passphrase "1234" station wlan0 connect Narnia
```
Now verify the connection:
```bash
ping archlinux.org
```
### 1.7. Partition the disks
I created three partitions on my disk:
* EFI (boot) partition: Required for UEFI systems to store boot files.
* Swap partition: Used as virtual memory to supplement physical RAM.
* Linux filesystem (root) partition: The main partition where the operating system and applications are installed. I have encrypted this partition and I'll show you how to do it.

You can create additional partitions if you need—for example, a separate /home for user files or more specialized layouts depending on your workflow and storage needs.

First, you should find out the disk name with the following command. In my case it's _nvme0n1_:
```bash
lsblk
```
Now wipe the disk:
```bash
wipefs -a /dev/nvme0n1
```
Run the fdisk command on your disk to open fdisk command prompt for that disk:
```bash
fdisk /dev/nvme0n1
```
In the new command prompt write `g` and press `Enter` to create a new empty GPT partition table.

Create the EFI partition: 
1. Write `n` and press `Enter` to add a new partition.
2. Press `Enter` to accept the default partition number or enter number 1.
3. Press `Enter` to accept the default first sector.
4. For the Last Sector or Size write `+1G` and press `Enter`. If it asks you to remove the signature or not write `Y` and press `Enter`.
5. Now that the partiton is added, in the fdisk command prompt type `t` and press `Enter` to change the partition type.
6. Choose the partition by typing a number (number 1) or just `Enter` to choose the default partition.
7. It wants you to choose a name. Type `1` and press `Enter`.

Create the swap partition:
1. Write `n` and press `Enter` to add a new partition.
2. Press `Enter` to accept the default partition number or enter number 2.
3. Press `Enter` to accept the default first sector.
4. For the Last Sector or Size write `+4G` and press `Enter`. If it asks you to remove the signature or not write `Y` and press `Enter`.
5. In the fdisk command prompt type `t` and press `Enter` to change the partition type.
6. Choose the partition by typing a number (number 2) or just `Enter` to choose the default partition.
7. It wants you to choose a name. Type `19` and press `Enter`.

Create the Linux filesystem (root) partition:

1. Write `n` and press `Enter` to add a new partition.
2. Press `Enter` to accept the default partition number (number 3).
3. Press `Enter` to accept the default first sector.
4. For the last sector or size, simply press Enter to allocate all remaining available space to this partition.
### 1.8. Format the partitions
The partitions that are created should be formatted. 

Format the EFI partition:
```bash
mkfs.fat -F32 /dev/nvme0n1p1
```
Format the swap partition run:
```bash
mkswap /dev/nvme0n1p2
```
**Do not format the root partition immediately**. Instead, encrypt it first with LUKS, open the encrypted container, and then format it (typically with ext4):
1. Encrypt:
```bash
cryptsetup luksFormat /dev/nvme0n1p3
```
After entering the command it asks if your're sure you want to overwrite the data on _nvme0n1_ or not. Write `YES` and hit `Enter`.

Now enter a passphrase for your partition and press `Enter`.

2. Open:
```bash
cryptsetup open /dev/nvme0n1p3 cryptroot #the cryptroot name is optional
```
3. Format:
```bash
mkfs.ext4 /dev/mapper/cryptroot
```
### 1.9. Mount the file systems
1. Mount the unlocked (decrypted) root partition:
```bash
mount /dev/mapper/cryptroot /mnt
```
2. Create the boot mount point:
```bash
mkdir -p /mnt/boot
```
3. Mount the EFI partition:
```bash
mount /dev/nvme0n1p1/ /mnt/boot
```
4. Activate the swap partition:
```bash
swapon /dev/nvme0n1p2
```
5. Run `lsblk` and check your partitions.
## 2. Installation
Now that the system is prepared, you need to install the necessary files onto your new root partition. Use the `pacstrap` command to download and install the base system into the /mnt directory from Arch Linux repositories:
```bash
pacstrap /mnt base linux linux-firmware intel-ucode e2fsprogs dosfstools grub cryptsetup efibootmgr sudo base-devel linux-headers linux-lts linux-lts-headers sof-firmware networkmanager nano nvim man-db man-pages
```
* Replace `intel-ucode` with `amd-ucode` if you have an AMD processor.
* This command includes both standard and LTS kernels, essential system tools, editors, documentation, and bootloader packages.
## 3. Configure the system
### 3.1. Fstab
To get needed file systems (like the one used for the boot directory /boot) mounted on startup, generate an fstab file:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
### 3.2 Chroot
To directly interact with the new system's environment, tools, and configurations for the next steps as if you were booted into it, change root into the new system:
```bash
arch-chroot /mnt
```
### 3.3. Time
For human convenience (e.g. showing the correct local time or handling Daylight Saving Time), set the time zone:
```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
* You can find your region and city by listing the directories in `/usr/share/zoneinfo`. Use the `ls` command to browse and locate the appropriate timezone (for example, `ls /usr/share/zoneinfo/Europe`).
Run `hwclock` to generate /etc/adjtime:
```bash
hwclock --systohc
```
To check that the date is set correctly run:
```bash
date
```
### 3.4. Localization
Edit `/etc/locale.gen`, uncomment `en_US.UTF-8 UTF-8` and any other needed UTF-8 locales, then generate them with:
```bash
locale-gen
```
Create `/etc/locale.conf` and set the system language:
```text
LANG=en_US.UTF-8
```
### 3.5. Network configuration
Set your system hostname by creating `/etc/hostname` with your chosen name:
```bash
echo yourhostname > /etc/hostname
```
For local resolution, add your hostname to `/etc/hosts`:
```text
127.0.0.1   localhost
::1         localhost
127.0.1.1   yourhostname
```
This ensures your system has a consistent and identifiable name on the network.
### 3.6. Adding user and setting password
Run the following command:
```bash
EDITOR=nano visudo
```
Go all the way to the bottom and look for the line that says `# %wheel ALL=(ALL:ALL) ALL` and uncomment it. Press `Ctrl` + `O` and press `Enter` and then press `Ctrl` + `X` to exit the editor.

Now add a user:
```bash
useradd -m -G wheel -s /bin/bash john
```
Set a password for it:
```bash
passwd john
```
### 3.7. Edit mkinitcpio and Regenerate Initramfs
Run the command to edit the mkinitcpio configuration:
```bash
nano /etc/mkinitcpio.conf
```
Scroll down to find the `HOOKS` line, which should look like this:
```text
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```
Modify this line by adding the encrypt hook immediately after the `block` hook, like so:
```text
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```
This tells mkinitcpio to include support for unlocking encrypted partitions during boot.

After saving the file, regenerate the initramfs to apply changes:
```bash
mkinitcpio -P
```
### 3.8. Setting Up GRUB
Run the following command to install GRUB for a UEFI system:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=[BOOTLOADER_ID] --removable /dev/nvme0n1
```
* Replace [BOOTLOADER_ID] with a name like GRUB.
* Change /dev/nvme0n1 if your main disk differs.

Get the UUID for your encrypted partition and the decrypted root:
```bash
blkid -o value -s UUID /dev/nvme0n1p3         # Encrypted partition UUID
blkid -o value -s UUID /dev/mapper/cryptroot  # Decrypted (opened) root UUID
```
> [!TIP]
> Review the output in the terminal for accuracy.

Send the UUIDs to the end of your GRUB config file:
```bash
blkid -o value -s UUID /dev/nvme0n1p3 >> /etc/default/grub
blkid -o value -s UUID /dev/mapper/cryptroot >> /etc/default/grub
```
Edit the GRUB Configuration:
```bash
nano /etc/default/grub
```
Scroll to the bottom. Copy the two UUIDs you appended.

Locate the line starting with:
```text
GRUB_CMDLINE_LINUX_DEFAULT=
```
Directly after the word `quiet` (still inside the quotes), paste and edit into this format (all on one line):
```text
cryptdevice=UUID=ENCRYPTED_UUID:cryptroot root=UUID=DECRYPTED_UUID
```
Example:
```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet cryptdevice=UUID=abcd1234-ef56-...:cryptroot root=UUID=dcba4321-fe65-..."
```
Remove the UUID lines from the end of the file to keep things tidy.

Save changes (`Ctrl` + `O`, `Enter`), then exit (`Ctrl` + `X`).

Regenerate GRUB configuration and apply the changes with:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
### 3.9 Enable NetworkManager on Boot 
Run:
```bash
systemctl enable NetworkManager
```
## 4. Reboot
Exit the chroot environment by typing `exit` or pressing `Ctrl` + `D`. Then type `reboot` and press `Enter`.

When your system starts, the GRUB menu will appear.
Select *Arch Linux* and press `Enter`.

You will be prompted to enter the password for your encrypted volume.
Enter the password to unlock the encrypted root partition.

After that, the system will boot to the login screen (TTY).
Here, enter your username and password to log in.

To avoid typing your username and password every time you boot, you can enable automatic login on your virtual console. Open the `getty@.service` systemd unit file for editing:
```bash
sudo nano /lib/systemd/system/getty@.service
```
Locate the line starting with `ExecStart=`. Modify the line by removing everything from `-o` to `\\u`, then add `-a <your-username>` (replace <your-username> with your actual username, e.g., john). The line should look like this:
```bash
ExecStart=-/sbin/agetty -a john --noclear - $TERM
```
Reboot your machine to activate the changes. After restarting, you'll be logged in automatically on the console:
```bash
reboot
```
## 5. Post-installation
### 5.1. Installing and configuring Sway
To install Sway and recommended tools, run:
```bash
sudo pacman -S sway swaylock swaybg wofi kitty
```
Notes:
* Whenever you update Sway, it's a good idea to also update any related Wayland utilities (such as wlroots) to ensure compatibility.
* When prompted to select a default font during installation, choose Noto Sans—it offers broad language support and excellent readability.

Follow the instructions in the [Arch Linux NVIDIA drivers installation guide repository](https://github.com/korvahannu/arch-nvidia-drivers-installation-guide) to properly install the proprietary NVIDIA drivers for your system.

To automatically start Sway when you log into a TTY, edit your `.bash_profile`, open `.bash_profile` in the Nano editor:
```bash
nano ~/.bash_profile
```
Add the following code at the end of the file:
```text
if [ -z "$WAYLAND_DISPLAY" ] && [ -n "$XDG_VTNR" ] && [ "$XDG_VTNR" -eq 1 ] ; then
	exec sway --unsupported-gpu
fi
```
Save and exit Nano:
* Press `Ctrl` + `O`, then `Enter` to save.
* Press `Ctrl` + `X` to exit.

After reboot, signing in on TTY1 will automatically launch your Sway session.

#### 5.1.1. Set Kitty as the Default Terminal

For setting kitty as the default terminal in Sway create the Sway configuration directory (if it doesn’t already exist):
```bash
mkdir -p ~/.config/sway 
```
Copy the default Sway config file to your user directory:
```bash
cp /etc/sway/config ~/.config/sway/config
```
Edit the configuration file with Nano:
```bash
nano ~/.config/sway/config
```
Locate the line that sets the terminal, typically:
```text
set $term foot
```
Replace `foot` with `kitty` so it reads:
```text
set $term kitty
```
Save your changes:
* Press `Ctrl` + `O`, then `Enter` to write the file.
* Press `Ctrl` + `X` to exit Nano.

Reload Sway’s configuration (no need to log out or restart Sway completely):
* Press `Super` + `Shift` + `C` (the Super key is usually the Windows key).

Now, when you press `Super` + `Enter` in Sway, Kitty will be launched as your default terminal.

#### 5.1.2. Set Wofi as the Default Application Launcher

For setting Wofi as the default app launcher edit the Sway config file:
```bash
nano ~/.config/sway/config
```
Find the line that defines the application launcher, which usually looks like this:
```text
set $menu wmenu --run
```
Replace `wmenu --run` with `wofi --show drun` so it reads:
```text
set $menu wofi --show drun
```
Save your changes:
* Press `Ctrl` + `O`, then `Enter` to write the file.
* Press `Ctrl` + `X` to exit Nano.

To apply the changes, reload Sway’s configuration:
```text
Super + Shift + C
```
Now, when you press `Super` + `D` in Sway, Wofi will be launched as your default application launcher.
#### 5.1.3. Changing the Font in Sway
Install your desired font, for example, Fira Code:
```bash
sudo pacman -S ttf-fira-code
```
> On Arch-based systems, the ttf-fira-code package includes the Fira Code font.

Edit your Sway configuration file: 
```bash
nano ~/.config/sway/config
```
Add or modify the font setting by adding the following line:
```text
font pango:Fira Code 11
```
* This sets the font to Fira Code with size 11 using Pango font rendering (recommended for better font support).

Save the file (`Ctrl` + `O`, then `Enter`) and exit (`Ctrl` + `X`).

Reload Sway’s configuration without restarting your session by pressing:
```text
Super + Shift + C
```
#### 5.1.4. Adding a Keyboard Layout to Sway 
Edit your Sway configuration file:
```bash
nano ~/.config/sway/config
```
Add or modify the input section to include your desired keyboard layouts and options. For example, to add German (de) alongside US English with `Alt` + `Shift` toggling:
```text
input * {
    xkb_layout us,de
    xkb_options grp:alt_shift_toggle
}
```
Save your changes (`Ctrl` + `O`, then `Enter`) and exit Nano (`Ctrl` + `X`).

Reload Sway’s configuration to apply the changes without restarting the session: Press `Super` + `Shift` + `C`
#### 5.1.5. (Optional) Minimize Window Title Bars in Sway
By default, Sway displays a border and title bar around windows. You can reduce this visual clutter by setting thinner borders.

Open your Sway configuration file for editing:
```bash
nano ~/.config/sway/config
```
Add the following lines to set a minimal 1-pixel border for normal and floating windows:
```text
default_border pixel 1
default_floating_border pixel 1
```
### 5.2. Connecting to Wi-Fi
List available Wi-Fi networks:
```bash
nmcli device wifi list
```
Connect to a Wi-Fi network:
```bash
nmcli device wifi connect "SSID" password "YourPassword"
```
* Replace "SSID" with the exact name of your Wi-Fi network.
* Replace "YourPassword" with the network password.

> The nmcli command-line tool is a part of NetworkManager that manages network connections, including Wi-Fi, via NetworkManager’s interface.
### 5.3. Installing and Configuring Audio with PipeWire 
Install the pipewire-pulse package:
```bash
sudo pacman -S pipewire-pulse
```
This package provides PulseAudio compatibility using PipeWire.

Enable and start the PipeWire user service:
```bash
systemctl --user enable --now pipewire.service
```
This command enables the PipeWire audio service to start automatically at login and starts it immediately.
### 5.4. Unzipping and Unraring Files via Command Line 
Install the unzip package (for handling .zip files):
```bash
sudo pacman -S unzip
```
Extract a .zip archive:
```bash
unzip CompressedFile.zip
```
Install the unrar package (for handling .rar files):
```bash
sudo pacman -S unrar
```
Extract a .rar archive:
```bash
unrar x CompressedFile.rar
```
>  Using `unrar x` extracts files with full paths. Alternatively, `unrar e` CompressedFile.rar extracts files to the current directory without recreating folder structure.
### 5.5. Removing the GRUB Menu Countdown
Open the GRUB configuration file with a text editor, for example Nano:
```bash
nano /etc/default/grub
```
Find the line starting with GRUB_TIMEOUT and change its value to -1:
```text
GRUB_TIMEOUT=-1
```
This setting makes GRUB wait indefinitely at the menu until you make a selection.

Save and exit the editor:
* Press `Ctrl` + `O`, then `Enter` to save.
* Press `Ctrl` + `X` to exit Nano.


Update GRUB to apply the changes:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Reboot your system to see the effect:
```bash
reboot
```
### 5.6. Configuring Wofi
Create the Wofi configuration directory:
```bash
mkdir -p ~/.config/wofi
```
Create the main configuration file:
```bash
touch ~/.config/wofi/config
```
Create a CSS stylesheet file for customizing appearance:
```bash
touch ~/.config/wofi/style.css
```
Now Link the stylesheet in the Wofi config. Open the config file with your editor, for example Nano:
```bash
nano ~/.config/wofi/config
```
Add the following line to link the stylesheet:
```bash
stylesheet=style.css
```
Save and exit (`Ctrl` + `O`, then `Enter`, followed by `Ctrl` + `X`).

Customize Wofi appearance and behavior:
* You can style Wofi by editing the `~/.config/wofi/style.css` file.
* Refer to the [official Wofi documentation](https://cloudninja.pw/docs/wofi.html) for detailed customization options.

Customize config options in ~/.config/wofi/config. Here are some common examples you can add or modify:
* Change the prompt text:
```text
prompt=Run:
```
* Hide the scrollbar:
```text
hide_scroll=true
```
* Set the launch mode (e.g., application launcher):
```text
mode=drun
```
### 5.7. Enabling Dark Mode for GTK3 and GTK4 Applications
For GTK3 Applications, create the configuration directory if it doesn’t exist:
```bash
mkdir -p ~/.config/gtk-3.0
```
Create or edit the settings.ini file:
```bash
nano ~/.config/gtk-3.0
```
Add the following content to enable dark mode:
```text
[Settings]
gtk-application-prefer-dark-theme=1
```
Save and exit (`Ctrl` + `O`, then `Enter`, followed by `Ctrl` + `X`).

For GTK4 Applications, create the configuration directory if it doesn’t exist:
```bash
mkdir -p ~/.config/gtk-4.0
```
Create or edit the settings.ini file:
```bash
nano ~/.config/gtk-4.0
```
Add the following content to prefer dark mode:
```text
[AdwStyleManager]
color-scheme=ADW_COLOR_SCHEME_PREFER_DARK
```
Save and exit (`Ctrl` + `O`, then `Enter`, followed by `Ctrl` + `X`).

### 5.8. Configuring Kitty
Create the Kitty configuration directory (if it doesn't exist):
```bash
mkdir -p ~/.config/kitty
```
Create or open the kitty.conf file for editing:
```bash
nano ~/.config/kitty/kitty.conf
```
Customize Kitty by adding your preferred settings in this file. For detailed options and examples, refer to the [official Kitty configuration documentation](https://sw.kovidgoyal.net/kitty/conf/).
### 5.9. Installing and Configuring rclone for Google Drive 
Update your system and install rclone:
```bash
sudo pacman -Syu
sudo pacman -S rclone
```
Set up your Google Drive remote with rclone. Open the rclone configuration interface:
```bash
rclone config
```
Follow the prompts to add a new remote for Google Drive (Refer to the [official rclone config guide](https://rclone.org/commands/rclone_config/) for detailed, up-to-date setup instructions.).

Create a mount point for Google Drive:
```bash
mkdir -p gdrive
```
For Automatic mount Google Drive at login in Sway edit your Sway configuration file:
```bash
nano ~/.config/sway/config
```
Add this line to ensure your Google Drive is mounted on every Sway session start:
```text
exec rclone mount --vfs-cache-mode writes gdrive: ~/gdrive
```
* Replace `gdrive` with your actual remote name if you used a different one.
* `--vfs-cache-mode writes` ensures compatibility and reliable file operations with cloud storage.

(Optional) Mount Google Drive manually if you don’t want automatic mounting or want to mount it on-demand, simply run:
```bash
rclone mount --vfs-cache-mode writes gdrive: ~/gdrive
```
> [!TIP]
> I personally use this setup to access my Obsidian notes stored on Google Drive—simply open `~/gdrive` in Obsidian to work directly with your cloud-synced files.

> **Recommendation:**
> Whenever possible, consider using AppImages for desktop applications—they are convenient, cross-distro compatible, and can be removed without leaving traces on your system.
[^1]: Common [BIOS keys](https://www.tomshardware.com/reviews/bios-keys-to-access-your-firmware,5732.html) by brand:  
    | Manufacturer                | Key(s)                                           |
    |-----------------------------|--------------------------------------------------|
    | **ASRock**                  | `F2` or `DEL`                                        |
    | **ASUS**                    | `F2` (all PCs), `F2` or `DEL` (Motherboards)           |
    | **Acer**                    | `F2` or `DEL`                                        |
    | **Dell**                    | `F2` or `F12`                                        |
    | **ECS**                     | `DEL`                                              |
    | **Gigabyte / Aorus**        | `F2` or `DEL`                                        |
    | **HP**                      | `F10`                                              |
    | **Lenovo (Consumer Laptops)**| `F2` or `Fn` + `F2`                                   |
    | **Lenovo (Desktops)**       | `F1`                                               |
    | **Lenovo (ThinkPads)**      | `Enter`, then `F1`                                   |
    | **MSI**                     | `DEL` (Motherboards and PCs)                       |
    | **Microsoft Surface Tablets**| Press and hold Volume Up                        |
    | **Origin PC**               | `F2`                                               |
    | **Samsung**                 | `F2`                                               |
    | **Toshiba**                 | `F2`                                               |
    | **Zotac**                   | `DEL`                                              |
