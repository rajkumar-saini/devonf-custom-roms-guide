# Android 16 ROM Build Guide for Motorola Moto G73 5G (`devonf`)

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/rajkumar-saini/device_motorola_devonf)
[![Android Version](https://img.shields.io/badge/Android-16-blue)](https://developer.android.com/about/versions/16)
[![Device](https://img.shields.io/badge/Device-Moto%20G73%205G-orange)](https://www.motorola.com/us/smartphones-moto-g-5g/p)

## ⚠️ Important Disclaimer

**READ THIS CAREFULLY BEFORE PROCEEDING:**

- Flashing custom ROMs **WILL VOID YOUR WARRANTY**
- There's a risk of **PERMANENTLY BRICKING** your device
- **BACKUP ALL IMPORTANT DATA** before starting
- Ensure you have the correct **stock firmware** for recovery
- This guide is for **ADVANCED USERS ONLY**
- The author is **NOT RESPONSIBLE** for any damage to your device

---

## 📋 Table of Contents

1. [⚠️ Important Disclaimer](#️-important-disclaimer)
2. [🔧 Prerequisites](#-prerequisites)
3. [💻 Hardware Requirements](#-hardware-requirements)
4. [🛠️ Environment Setup](#️-environment-setup)
5. [📦 Repo Sync & Source Setup](#-repo-sync--source-setup)
6. [📱 Device-Specific Trees](#-device-specific-trees)
7. [🔨 Build Process](#-build-process)
8. [📤 Post-Build Steps](#-post-build-steps)
9. [🧪 Testing & Validation](#-testing--validation)
10. [🔧 Troubleshooting](#-troubleshooting)
11. [❓ FAQ](#-faq)
12. [🤝 Contributing](#-contributing)
13. [🙏 Credits & Acknowledgments](#-credits--acknowledgments)
14. [📄 License](#-license)

---

## 🔧 Prerequisites

### Knowledge Requirements
- Basic understanding of Android development
- Familiarity with Linux command line
- Experience with Git and version control
- Understanding of Android build system

### Device Requirements
- **Unlocked bootloader** (critical!)
- **ADB and Fastboot** properly configured
- **Stock ROM backup** for recovery purposes

---

## 💻 Hardware Requirements

### Minimum Requirements
|   Component  | Recommended | Optimal |
|--------------|-------------|---------|
|   **CPU**    |   8+ cores  |   16+   |
|   **RAM**    |    32 GB    |  64 GB  |
|  **Storage** |  500 GB SSD | 1TB SSD |
| **Internet** |   50+ Mbps  |100+ Mb/s|

### Cloud VM Recommendations
```bash
# Google Cloud Platform
  --machine-type=c2d-highmem-8 \
  --boot-disk-size=1000GB
```

### Build Time Estimates
- **First build**: 6-8 hours (depending on hardware)
- **Incremental builds**: 60-90 minutes
- **Clean builds**: 2-4 hours

---

## 🛠️ Environment Setup

### 1. System Update & Dependencies

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install essential build dependencies
sudo apt-get install -y \
  git-core gnupg rsync flex bison build-essential zip curl \
  zlib1g-dev zram-tools libc6-dev-i386 git-lfs tmux ccache \
  x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev \
  libxml2-utils xsltproc unzip fontconfig python3-pip \
  openjdk-11-jdk bc bison build-essential ccache curl \
  flex g++-multilib gcc-multilib git gnupg gperf \
  imagemagick lib32ncurses5-dev lib32readline-dev \
  lib32z1-dev liblz4-tool libncurses5 libncurses5-dev \
  libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop \
  pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```

### 2. Memory Optimization

#### Enable ZRAM (Compressed RAM)
```bash
echo -e "ALGO=zstd\nPERCENT=80" | sudo tee /etc/default/zramswap
sudo systemctl restart zramswap.service
sudo systemctl enable zramswap.service
```

#### Setup Additional Swap
```bash
# Create 25GB swap file
sudo fallocate -l 25G /swapfile_extend_25GB
sudo chmod 600 /swapfile_extend_25GB
sudo mkswap /swapfile_extend_25GB
sudo swapon /swapfile_extend_25GB

# Make permanent
echo '/swapfile_extend_25GB none swap sw 0 0' | sudo tee -a /etc/fstab
```

#### Optimize System Parameters
```bash
# Add to /etc/sysctl.conf for better build performance
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
echo 'vm.dirty_ratio=15' | sudo tee -a /etc/sysctl.conf
echo 'vm.dirty_background_ratio=5' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 3. Git and SSH Configuration

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "287562898+rajkumar-saini@users.noreply.github.com" -f ~/.ssh/id_ed25519

# Start SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Display public key (add this to GitHub/GitLab)
cat ~/.ssh/id_ed25519.pub
```

### 4. Configure ccache (Build Cache)

```bash
# Set ccache environment variables
export USE_CCACHE=1
export CCACHE_EXEC=$(which ccache)

# Configure ccache settings
ccache -M 100G  # Set cache size to 100GB
ccache --set-config=compression=true
ccache --set-config=hash_dir=true
ccache --set-config=sloppiness=file_macro,time_macros
ccache --set-config=max_files=0

# Create ccache directory
rm -rf ~/.cache/ccache
mkdir -p ~/.cache/ccache/tmp

# Add to ~/.bashrc for persistence
echo 'export USE_CCACHE=1' >> ~/.bashrc
echo 'export CCACHE_EXEC=$(which ccache)' >> ~/.bashrc
```

### 5. Android Build Environment

```bash
# Clone and run Android build environment setup
cd ~/
git clone https://github.com/akhilnarang/scripts
chmod +x scripts/setup/android_build_env.sh
./scripts/setup/android_build_env.sh
```

### 6. Install repo tool

```bash
# Create bin directory and install repo
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

# Add to PATH
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 7. Git Identity Configuration

```bash
git config --global user.name "Rajkumar Saini"
git config --global user.email "287562898+rajkumar-saini@users.noreply.github.com"
git lfs install

# Start tmux session for long-running builds
tmux
```

---

## 📦 Repo Sync & Source Setup

### 1. Initialize Repository

```bash
# Create ROM directory
mkdir -p ~/customRom && cd ~/customRom
```
```
# Initialize repo (replace with your ROM's manifest)
repo init -u <ROM_MANIFEST_URL> -b <BRANCH_NAME> --git-lfs --depth=1

# Sync source code (this will take time!)
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

## 📱 Device-Specific Trees

### Clone Device Trees

```bash
# Navigate to ROM directory
cd ~/customRom

# Device Tree (main device configuration)
git clone git@github.com:rajkumar-saini/device_motorola_devonf.git -b sixteen-qpr2 device/motorola/devonf

# Vendor Tree (proprietary blobs)
git clone git@github.com:rajkumar-saini/vendor_motorola_devonf.git -b sixteen-qpr2 vendor/motorola/devonf

# Kernel Source
git clone git@github.com:rajkumar-saini/device_motorola_devonf-kernel.git -b sixteen-qpr2 device/motorola/devonf-kernel

# Additional Trees
git clone git@github.com:rajkumar-saini/vendor_motorola_devonf-motcamera.git -b sixteen-qpr2 vendor/motorola/devonf-motcamera

# MediaTek-specific repositories
git clone https://github.com/yaap/device_mediatek_sepolicy_vndr.git -b sixteen device/mediatek/sepolicy_vndr
git clone https://github.com/yaap/hardware_mediatek.git -b sixteen hardware/mediatek
```

### Optional: ROM Signing Keys

```bash
# Setup Signing Keys
git clone -b main https://github.com/LineageOS/scripts.git /tmp/lineage-scripts
mkdir -p vendor/lineage-priv
mv /tmp/lineage-scripts/lineage-priv-template vendor/lineage-priv/keys
rm -rf /tmp/lineage-scripts

cd vendor/lineage-priv/keys
./keys.sh
cd ../../..

# Add to device.mk
echo '-include vendor/lineage-priv/keys/keys.mk' >> device/motorola/devonf/device.mk
```

---

## 🔨 Build Process

### 1. Pre-build Checks

```bash
# Verify all trees are present
ls -la device/motorola/devonf/
ls -la vendor/motorola/devonf/
ls -la device/motorola/devonf-kernel/

# Check available disk space (minimum 200GB free)
df -h

# Verify ccache is working
ccache -s
```

### 2. Initialize Build Environment

```bash
cd ~/customRom

# Source build environment
source build/envsetup.sh

# List available lunch targets
lunch

# Select your target (replace <rom> with actual ROM name)
lunch <rom>_devonf-bp4a-user

# Set build variables
export RELAX_USES_LIBRARY_CHECK=true
export ALLOW_MISSING_DEPENDENCIES=true
export TARGET_DISABLE_EPPE=true
export TARGET_ENABLE_BLUR=true
```

### 3. Start Build

```bash
# Start the build process
mka bacon -j$(nproc --all)

# Alternative build commands
# mka bootimage    # Build only boot image
# mka systemimage  # Build only system image
# mka vendorimage  # Build only vendor image
```

### Build Variants

| Variant | Description | Use Case |
|---------|-------------|----------|
| **user** | Production build | End users, stable release |
| **userdebug** | Debug-enabled user build | Testing, debugging |
| **eng** | Engineering build | Development only |

### 4. Build Cleaning Options

```bash
# Full clean (removes everything)
make clobber

# Clean specific modules
make clean-<module_name>

# Partial clean (keeps some build artifacts)
make installclean

# Clean only output directory
rm -rf out/
```

---

## 📤 Post-Build Steps

### 1. Locate Build Output

```bash
# ROM ZIP location
ls -la ~/customRom/out/target/product/devonf/*.zip

# Individual images
ls -la ~/customRom/out/target/product/devonf/*.img

# Build information
cat ~/customRom/out/target/product/devonf/build_fingerprint.txt
```

### 2. Verify Build Integrity

```bash
# Check ZIP file integrity
unzip -t ~/customRom/out/target/product/devonf/*-UNOFFICIAL-devonf.zip

# Verify file sizes (ROM should be 1-3GB typically)
du -h ~/customRom/out/target/product/devonf/*.zip
```

### 3. Upload ROM

#### Using GoFile Upload Script
```bash
git clone https://github.com/Sushrut1101/GoFile-Upload.git
cd GoFile-Upload
chmod +x upload.sh
./upload.sh ~/customRom/out/target/product/devonf/*-UNOFFICIAL-devonf.zip
```

#### Alternative Upload Methods
```bash
# Using rclone (configure first)
rclone copy ~/customRom/out/target/product/devonf/*.zip gdrive:ROMs/

# Using scp to your server
scp ~/customRom/out/target/product/devonf/*.zip user@server:/path/to/uploads/
```

---

## 🧪 Testing & Validation

### Pre-Flash Checklist
- [ ] Bootloader is unlocked
- [ ] Stock ROM backup is created
- [ ] ADB/Fastboot is working
- [ ] Battery is >50% charged

### Flash Instructions

#### Method 1: Fastboot Flash (Recommended)
```bash
# Boot to fastboot mode
adb reboot bootloader
fastboot reboot fastboot

# Flash individual partitions
fastboot flash vendor_boot vendor_boot.img

# Boot to bootloader mode
fastboot reboot bootloader

# Reboot to recovery
fastboot reboot recovery

# Sideload ROM
adb sideload *-UNOFFICIAL-devonf.zip
```

#### Method 2: Recovery Flash using SDCard
```bash
# Boot to recovery
adb reboot recovery

# Copy ROM to device
adb push *-UNOFFICIAL-devonf.zip /sdcard/

# Flash via recovery interface
# Navigate to Install > Select ROM ZIP in sdcard > Swipe to flash
```

---

## 🔧 Troubleshooting

### Common Build Errors

#### Out of Memory Errors
```bash
# Increase swap space
sudo fallocate -l 50G /swapfile_extra
sudo mkswap /swapfile_extra
sudo swapon /swapfile_extra
```

#### Missing Dependencies
```bash
# Install missing packages
sudo apt-get install -y <missing_package>

# Update build environment
./scripts/setup/android_build_env.sh
```

#### Repo Sync Issues
```bash
# Force sync with cleanup
repo sync --force-sync --force-remove-dirty -j$(nproc --all)

# Reset to clean state
repo forall -c 'git reset --hard'
repo forall -c 'git clean -fdx'
```

### Device Issues

#### Device Not Detected After Flash
```bash
# Check device connection
lsusb | grep -i motorola

# Try different USB port/cable
# Manually boot to recovery:
# Power + Volume Down, then select Recovery
```

#### Bootloop Issues
```bash
# Collect logs
adb logcat -d > logcat.log
adb shell dmesg > dmesg.log

# Check for crash dumps
adb pull /sys/fs/pstore/
adb pull /proc/last_kmsg

# Emergency recovery
fastboot erase userdata
fastboot erase metadata
fastboot reboot recovery
```
---

## ❓ FAQ

### Q: How long does the first build take?
**A:** Typically 6-8 hours depending on your hardware. Subsequent builds are much faster (30-60 minutes).

### Q: Can I build on Windows?
**A:** Not recommended. Use WSL2 or a Linux VM/cloud instance for best results.

### Q: My build failed with "ninja: build stopped"
**A:** Usually indicates out of memory or disk space. Check resources and try with fewer parallel jobs.

### Q: How do I update my ROM build?
**A:** Run `repo sync` to get latest sources, then rebuild. Incremental builds are much faster.

### Q: Can I customize the ROM before building?
**A:** Yes! Modify device trees, add/remove apps, change system properties, etc.

---

## 🤝 Contributing

### Reporting Issues
- Use GitHub Issues for bug reports
- Include full build logs
- Mention ROM version and device tree commits

### Contributing Code
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

### Device Tree Contributions
- Follow Android coding standards
- Test on actual hardware
- Document any changes
- Update commit messages properly

---

## 🙏 Credits & Acknowledgments

### Core Contributors
- **[Rajkumar Saini](https://github.com/rajkumar-saini)** - Device maintainer
- **[Kushagra Jain](https://github.com/kjdev16)** - Base Guide
- **[Akhil Narang](https://github.com/akhilnarang)** - Build environment scripts
- **[@cyberknight777](https://t.me/cyberknight777)** - Kernel development
- **[@sarthakray2002](https://t.me/sarthakroy2002)** - Device tree assistance
- **[AdarshOs](https://t.me/Adarsh0s)** - Device Tree Maintainer

### Resources & References
- [Android Open Source Project](https://source.android.com/)
- [LineageOS Wiki](https://wiki.lineageos.org/)
- [Android ROM List](https://github.com/musabcel/android_rom_list)
- [XDA Developers](https://forum.xda-developers.com/)

### Special Thanks
- Android community for continuous support
- All beta testers and users

---

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Happy Building! 🚀**

*Made with ❤️ by the Android development community*
</div>
