# <img src="./arch_linux.png" width="30" /> Arch Linux + Sway Installation Guide
Sources:
* <a href="https://wiki.archlinux.org/title/Installation_guide" target="_blank">Installation guide - ArchWiki</a>
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
[^1]: Common BIOS keys by brand:  
    | Manufacturer                | Key(s)                                           |
    |-----------------------------|--------------------------------------------------|
    | **ASRock**                  | F2 or DEL                                        |
    | **ASUS**                    | F2 (all PCs), F2 or DEL (Motherboards)           |
    | **Acer**                    | F2 or DEL                                        |
    | **Dell**                    | F2 or F12                                        |
    | **ECS**                     | DEL                                              |
    | **Gigabyte / Aorus**        | F2 or DEL                                        |
    | **HP**                      | F10                                              |
    | **Lenovo (Consumer Laptops)**| F2 or Fn + F2                                   |
    | **Lenovo (Desktops)**       | F1                                               |
    | **Lenovo (ThinkPads)**      | Enter, then F1                                   |
    | **MSI**                     | DEL (Motherboards and PCs)                       |
    | **Microsoft Surface Tablets**| Press and hold Volume Up                        |
    | **Origin PC**               | F2                                               |
    | **Samsung**                 | F2                                               |
    | **Toshiba**                 | F2                                               |
    | **Zotac**                   | DEL                                              |
