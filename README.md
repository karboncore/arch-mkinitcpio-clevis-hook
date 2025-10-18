# Introduction

A simple mkinitcpio hook to unlock LUKS devices on boot using TPM and `clevis`.

Inspired by [arch-mkinitcpio-clevis-hook](https://github.com/kishorviswanathan/arch-mkinitcpio-clevis-hook?tab=readme-ov-file), with the following new features added:
* Multi-device encrypted root partitions are supported - unlock several devices at boot by using multiple cryptdevice= kernel parameters
* Fall back to passphrase prompt when TPM key is invalid or absent
* Offer to regenerate clevis/TPM keys when needed
* Allow mounting with the discard option

Tested on Artix Linux and should work on Arch and Arch derivatives.

# Installing

## Manual Method

1. Install the following packages.
    ```sh
    sudo pacman --needed -S clevis tpm2-tools luksmeta libpwquality
    ```
2. Add a `clevis` binding to your LUKS device; for example,
    ```sh
    sudo clevis luks bind -d /dev/nvme0n1p1 tpm2 '{"pcr_ids":"0,2,5,8"}'
    ```
    For an explanation of the different PCR registers you can choose from, see [Accessing PCR registers](https://wiki.archlinux.org/title/Trusted_Platform_Module#Accessing_PCR_registers).
3. Install the `clevis` hook.
    ```sh
    sudo ./install.sh
    sudo nano /etc/mkinitcpio.conf
    # Edit the hooks and add clevis before the 'encrypt' hook. Eg:
    # HOOKS=(.. clevis encrypt ..)
    ```
4. Regenerate the `initramfs` image.
    ```sh
    sudo mkinitcpio -P
    ```
5. Add kernel parameters with the format `cryptdevice=DEVICE:NAME:OPTIONS` for each device to be unlocked. For example,
    ```sh
    cryptdevice=PARTUUID=22ea7485-1688-48ef-83c8-452feb21d18b:root:discard
    ```
   Options may be left blank and currently the only supported option is discard.
6. Reboot.

7. Once happy, you can consider adding disablehooks=encrypt to your kernel command line. However keep encrypt in the mkinitcpio.conf hooks array to ensure that all dependencies get included.

# Updating

When the PCR registers change, typically due to changed BIOS settings or kernel options, the TPM key will become invalid. On next boot you will be prompted to unlock via passphrase, and the hook will offer to regenerate the TPM key, at your discretion.

# Troubleshooting

Please report any issues and also let me know if this works for you!
