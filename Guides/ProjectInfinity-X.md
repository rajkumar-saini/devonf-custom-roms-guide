# Project Infinity X Device Tree Bringup & Setup Signing Keys

## Rename Device Makefile
```bash
mv device/motorola/devonf/lineage_devonf.mk device/motorola/devonf/infinity_devonf.mk
```
## Update Build Configuration
#### Replace all `lineage` references with `infinity`:
```bash
nano device/motorola/devonf/AndroidProducts.mk
```
#### Replace all `lineage` references with `infinity` and add the flags mentioned below:
```bash
nano device/motorola/devonf/infinity_devonf.mk
```
```bash
# InfinityOS Custom Flags
INFINITY_MAINTAINER := "Dark\u00A0Devil"

# Whether the device supports Fingerprint On Display
TARGET_HAS_UDFPS := false #(Default: false)

# Whether Including Google Apps
WITH_GAPPS := true  #(Default: true)
```
### Update Specs Configuration
```bash
nano device/motorola/devonf/configs/properties/system.prop
```
and add:
```bash
# InfinityX Phone Specs Properties
ro.product.marketname=Moto G73 5G
ro.infinity.soc=Mediatek Dimensity 930 6nm
ro.infinity.camera=50MP (f/1.8) + 8MP (UW) + 16MP
ro.infinity.battery=6000mAh Li-P (non-removable)
ro.infinity.display=6.5FHD+ (2400x1080), 120Hz
```
## Clone InfinityX Signing Keys
```bash
git clone git@github.com:rajkumar-saini/vendor_infinity-priv_keys.git -b main vendor_infinity-priv_keys
```
### Generate InfinityX Signing Keys For FIrst Time
```bash
# If you are building for the first time
git clone https://github.com/ProjectInfinity-X/vendor_infinity-priv_keys-template vendor/infinity-priv/keys
cd vendor/infinity-priv/keys
./keys.sh
cd ../../..
```
## Credits
Thanks to [**Ayu Kashyap**](https://github.com/DevAyu-Codes)

