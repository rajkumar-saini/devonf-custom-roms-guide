## LineageOS Device Tree Bringup & Setup Signing Keys

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
