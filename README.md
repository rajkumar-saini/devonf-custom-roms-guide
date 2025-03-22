# Android ROM Build Setup Guide for `devonf`

## Caution: This guide assumes that you know what you are doing and what flashing custom ROMs can do to your device. Proceed at your own risk. Flashing custom ROMs may void your warranty, brick your device, or result in data loss. Ensure you have backed up all important data before proceeding.

## Table of Contents

- [Introduction](#introduction)
- [Hardware Requirements](#hardware-requirements)
- [Phase 1: Environment Setup](#phase-1-environment-setup)
- [Phase 2: Device-Specific Setup](#phase-2-device-specific-setup)
- [Phase 3: Build Process](#phase-3-build-process)
- [Phase 4: Post-Build](#phase-4-post-build)
- [Troubleshooting](#troubleshooting)
- [Credits](#credits)

## Introduction

This guide provides detailed instructions for building custom Android ROMs (specifically Android 15-based) for your device. The example device used is the Motorola Moto G73 5G (devonf).

## Hardware Requirements

### Recommended Specifications

- **CPU**: 8+ cores
- **RAM**: 64GB+ recommended
- **Storage**: 300GB minimum, 500GB+ recommended
- **Internet**: High-speed connection (50+ Mbps) for source code download

### VM Configuration (Google Cloud Example)

- **Operating System**: Debian
- **Instance Type**: T2D
- **CPU**: 8 cores
- **RAM**: 32 GB
- **Storage**: 500 GB (standard persistent disk)
- **Firewall**: Enable all options

**Note**: A full build typically takes 2-4 hours depending on your hardware.

## Phase 1: Environment Setup

Check this link for further guide on environment setup: [Lineage Build Guide](https://wiki.lineageos.org/devices/bacon/build/).

### 1. Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Required Packages

```bash
sudo apt-get install git-core gnupg rsync flex bison build-essential zip curl zlib1g-dev zram-tools libc6-dev-i386 git-lfs tmux ccache x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
sudo apt install zram-tools
git-lfs install
```

### 3. Configure ZRAM

```bash
echo -e "ALGO=zstd\nPERCENT=80" | sudo tee -a /etc/default/zramswap
```

### 4. Setup SSH Keys

```bash
ssh-keygen -t ed25519 -C "kjdev00016@gmail.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

Copy the key displayed.

#### Add SSH Key to Git Services

1. Open [GitHub SSH Keys](https://github.com/settings/keys)
2. Open [Gitlab SSH Keys](https://gitlab.com/-/user_settings/ssh_keys)
3. Click **New SSH Key**
4. Paste the copied key and save

### 5. Configure Swap File

```bash
sudo fallocate -l 25G /swapfile_extend_25GB
ls -l /swapfile_extend_25GB
sudo chmod 600 /swapfile_extend_25GB
ls -l /swapfile_extend_25GB
sudo mkswap /swapfile_extend_25GB
sudo swapon /swapfile_extend_25GB
```

### 6. Install Build Dependencies

```bash
cd ~/
git clone https://github.com/akhilnarang/scripts
./scripts/setup/android_build_env.sh
```

This script installs necessary packages for building Android including OpenJDK, build tools, and other dependencies.

### 7. Install Repo Command

```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

The repo tool manages multiple Git repositories for Android development.

### 8. Configure Git

```bash
git config --global user.email "kjdev00016@gmail.com"
git config --global user.name "Kushagra Jain"
tmux
```

### 9. Set Environment Variables

```bash
rm -rf /home/kjnewrom/.cache/ccache/tmp
mkdir -p /home/kjnewrom/.cache/ccache/tmp
export RELAX_USES_LIBRARY_CHECK=true
export USE_CCACHE=1 && ccache -M 50G && export CONFIG_STATE_NOTIFIER=y && export SELINUX_IGNORE_NEVERALLOWS=true
export CCACHE_EXEC=/usr/bin/ccache
```

## Phase 2: Device-Specific Setup

### 10. Choose a ROM and Initialize Source
```bash
mkdir -p ~/customRom
cd ~/customRom
```
```
repo init -u <ROM_MANIFEST_URL> -b <BRANCH_NAME> --git-lfs
repo sync
```
This step downloads Android source code (~100GB) and may take several hours depending on your internet speed.

### 11. Clone Device-Specific Repositories
For Motorola Moto G73 5G (devonf):

```bash
#### Device Tree
git clone git@github.com:kjdev16/android_device_motorola_devonf.git -b fifteen-qpr2 device/motorola/devonf

# Vendor Tree
git clone git@github.com:kjdev16/android_vendor_motorola_devonf.git fifteen-qpr2 vendor/motorola/devonf

# Kernel
git clone git@github.com:kjdev16/android_device_motorola_devonf-kernel.git -b fifteen device/motorola/devonf-kernel

# Vendor MotCamera
git clone git@gitlab.com:devonf1/android_vendor_motorola_devonf-motcamera.git vendor/motorola/devonf-motcamera

# MediaTek SEPolicy Vendor
git clone https://github.com/yaap/device_mediatek_sepolicy_vndr.git device/mediatek/sepolicy_vndr

# MediaTek SEPolicy
git clone https://github.com/LineageOS/android_device_mediatek_sepolicy.git -b lineage-17.1 device/mediatek/sepolicy

# MediaTek Hardware
git clone https://github.com/yaap/hardware_mediatek.git hardware/mediatek
```

### 12. Private Keys for signing
**Note:** *Check the ROM for Official Signing script*
```bash
git clone git@github.com:kjdev16/devonf-keys-vendor-lineage-priv.git vendor/lineage-priv
```

### 13. ROM Signing Setup (Optional / If above step not done)
```bash
wget https://raw.githubusercontent.com/306bobby-android/crDroid-build-signed-script/main/create-signed-env.sh \
  && chmod +x create-signed-env.sh \
  && ./create-signed-env.sh
```
This creates signing keys for your custom ROM.

### 14. Modify Makefile (If Using Signing Keys)
Edit `device.mk` or similar file and add this line at the end:
```bash
-include vendor/lineage-priv/keys/keys.mk
```

## Phase 3: Build Process

## **Note**: All commands displayed below depends on various ROMs, use as per declared in their manifest.
### 15. Initial Build
```bash
source build/envsetup.sh
lunch <rom>_devonf-ap4a-user
mka bacon
```
This step compiles the ROM and may take 2-4 hours depending on hardware.

Also, build `userdebug` version so as to take logs for any issues during flashing.
```bash
source build/envsetup.sh
lunch <rom>_devonf-ap4a-userdebug
mka bacon
```

### 16. Cleaning Builds

#### Clean Build (For Major Changes)
```bash
make clobber
```
Removes all build output files, forcing a complete rebuild.

#### Dirty Build (For Minor Changes) (Also, if you shift from GAPPS variant to Vanilla build variant or vice versa)
```bash
make installclean
```
Removes only installed binaries, preserving most compiled objects.

## Phase 4: Post-Build

### 17. Locate Build Output
The completed ROM zip file will be in:
`
~/customRom/out/target/product/devonf/*-YYYYMMDD-UNOFFICIAL-devonf.zip
`

### 18. Generate download link
```
git clone https://github.com/Sushrut1101/GoFile-Upload.git
cd GoFile-Upload
chmod +x *
```
Upload using:
`
./upload.sh ~/customRom/out/target/product/devonf/*-YYYYMMDD-UNOFFICIAL-devonf.zip
`

### 19. Install ROM on Device
1. Download the rom from the displayed gofile link
2. Extract the payload.bin
3. Use `payload-dumper-go` to extract `vendor_boot.img` ```payload-dumper-go ./payload.bin```
4. **Caution**: Unlock your device's bootloader before proceeding!!!
5. Boot the device into fastboot mode ```adb reboot fastboot```
6. Flash the `vendor_boot.img` ```fastboot flash vendor_boot ./vendor_boot.img```
7. Boot device into recovery mode ```fastboot reboot recovery```
8. Transfer ROM zip to device via ADB:
   ```adb sideload "/path/to/custom_rom.zip"```
9. After installation, select "Format Data" or perform
   ```
   fastboot erase userdata
   fastboot erase metadata
   ```
10. Reboot system

## Troubleshooting
### Common Build Errors
#### Missing Dependencies
**Error**: `error: vendor/xxx/xxx.mk: No such file or directory`
**Solution**: Make sure all required repositories are cloned in the correct locations.

### Common Flash Issues
#### 1. Device not detecting after flashing
1. Disconnect your data cable
2. Reboot to recovery again
3. Connect your data cable
4. Try again

#### 2. Bootlooping
Take logs in case of bootloop and send to your ROM developer
Use these commands after enabling `adb` in recovery.
1. For simple log use:
`adb logcat -d > logcat.log`

2. If you need dmesg (kernel logs) then:
`adb shell dmesg > dmesg.log`

3. For getting console ramoops:
`adb pull /sys/fs/pstore`

4. For last kmsg:
`adb pull /proc/last_kmsg`

5. In Recovery if didn't boot
`adb shell dd if=/dev/block/by-name/expdb of=/tmp/expdb.img`
`adb shell strings /tmp/expdb.img > expdb.txt`

6. Extended boot logs
`adb root`
`adb pull /metadata/boot_log.txt`

## Credits
- [Akhil Narang](https://github.com/akhilnarang) for build environment scripts
- [Android ROM List](https://github.com/musabcel/android_rom_list)
- [@cyberknight777](https://t.me/cyberknight777) & [@sarthakray2002](https://t.me/sarthakroy2002) for trees.
- [AdarshOs](https://t.me/Adarsh0s) for all the guidance.
- Finally, Thanks to all Project Matrixx admins for helping me guide to increase speeds for build.
