# <img src="./arch_linux.png" width="30" /> Arch Linux + Sway Installation Guide
Sources:
* [Installation guide - ArchWiki](https://wiki.archlinux.org/title/Installation_guide)
## 1. Pre-installation
### 1.1 Download an installation image
An Arch Linux ISO file should be downloaded. It is recommended to download the Arch Linux ISO file from the [official Arch Linux download page](https://archlinux.org/download/) using BitTorrent, as suggested by the website.
### 1.2 Verify signature (optional but strongly recommended)
A PGP Signature file also can be downloaded for checking the integrity and authenticity of the ISO file. This file is located on the [Checksums and signatures](https://archlinux.org/download/#checksums) section of the page. Now download the [GnuPG](https://gnupg.org/download/index.html) software and install it. If you're on Windows you should download and install the [GPG4WIN](https://gpg4win.org/download.html) software. The PGP Signature file should be placed on the same directory as the ISO file. Now open a terminal in that directory and run the following command:
``` bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-_version_-x86_64.iso.sig
```
After running the command:
* Confirm “Good signature from…” appears.
* Compare the key fingerprint shown to the [official Arch Linux developer signing keys](https://archlinux.org/master-keys/).
* If it matches, the ISO is verified.
* If you see “BAD signature” or fingerprints do not match, do not use the ISO
### 1.3 Prepare an installation medium
Plug in a USB flash drive and write the ISO file to it using an app like [Balena Etcher](https://etcher.balena.io/). Now the flash drive is bootable.
### 1.4 Boot the live environment
Now reboot your system and open the UEFI (BIOS) firmware setup by pressing the respective key (for me it's F2[^1]). Disable **Secure Boot** (usually found in the Security tab). Also navigate to the boot tab and move the flash drive to the top of the boot order. Save the settings. Your PC will now boot into the live environment.

After a bit of time the installation bootloader will appear. Select *Arch Linux install medium* and press `Enter`. You will be logged in on virtual console.
### 1.5 Verify the boot mode
To verify the boot mode, check the UEFI bitness:
``` bash
cat /sys/firmware/efi/fw_platform_size
```
* If the command returns 64, the system is booted in UEFI mode and has a 64-bit x64 UEFI.
* If the command returns 32, the system is booted in UEFI mode and has a 32-bit IA32 UEFI. While this is supported, it will limit the boot loader choice to those that support mixed mode booting.
* If it returns No such file or directory, the system may be booted in BIOS (or CSM) mode.

If the system did not boot in the mode you desired (UEFI vs BIOS), refer to your motherboard's manual. 
### 1.6 Connect to the internet
If you're connected through a cable you might already have network connectivity. If you want to connect to a Wi-Fi, make sure the card is not blocked with rfkill. First run the `rfkill` command to check the current status:
```bash
rfkill
```
Expected output:
```bash
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
In my case the name was `wlan0`. Now run the `iwctl` command. It will activate another command prompt that's dedicated to Wi-Fi.:
```bash
iwctl
```
In the new command prompt run to view available Wi-Fi networks:
```bash
station wlan0 get-networks
```
Expected output:
```bash
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
