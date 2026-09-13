## Rising Revived OSS Device Tree Bringup & Setup Signing Keys

**Add Build flags in `lineage_devonf.mk`**
```bash
nano device/motorola/devonf/lineage_devonf.mk
```
```bash
# Lunch banner maintainer variable
RISING_MAINTAINER="Dark Devil"

# Chipset/Maintainer properties (ro.rising.chipset/ro.rising.maintainer) 
# Set RISING_MAINTAINER for version control 
# (Optional if builder is setting properties via init_<device>.cpp)
PRODUCT_BUILD_PROP_OVERRIDES += \
    RisingChipset="MediaTek Dimensity 930" \
    RisingMaintainer="Dark Devil"

RISING_MAINTAINER := "Dark Devil"

# Disable/enable blur support, false by default
TARGET_ENABLE_BLUR := true

# Whether to ship aperture camera, false by default
PRODUCT_NO_CAMERA := false

# Whether to ship lawnchair launcher, false by default
TARGET_PREBUILT_LAWNCHAIR_LAUNCHER := false

# GMS build flags, false by default
# ship with GMS packages, replaces default AOSP packages with Google manufactured packages.
WITH_GMS := true

# These flags needs WITH_GMS set to true
# Whether to ship pixel launcher and set it as default launcher, false by default
TARGET_DEFAULT_PIXEL_LAUNCHER := true 

# Whether to ship prebuilt Google Dialer and Messages, false by default
TARGET_INCLUDE_GOOGLE_DIALER := true
```

**Fix for vendor_camera_prop missing**
Paste `vendor_restricted_prop(vendor_camera_prop);` after `#Camera`
```bash
nano device/motorola/cancunf/sepolicy/vendor/property.te
```

**Clone Keys and Generate**
```bash
git clone -b main https://github.com/LineageOS/scripts.git /tmp/lineage-scripts
mkdir -p vendor/lineage-priv
mv /tmp/lineage-scripts/lineage-priv-template vendor/lineage-priv/keys
rm -rf /tmp/lineage-scripts
cd vendor/lineage-priv/keys
./keys.sh
cd ../../..
```

## Credits
Thanks to [**Ayu Kashyap**](https://github.com/DevAyu-Codes)
