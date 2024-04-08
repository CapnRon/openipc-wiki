Wireless settings
---

### Additional packages

- Following packages are needed for the HI3516EV300 CamHi module.
- The example build configuration is: `hi3516ev300_lite_defconfig`

```shell
BR2_PACKAGE_WIRELESS_CONFIGURATION=y
BR2_PACKAGE_WPA_SUPPLICANT_AP_SUPPORT=y
BR2_PACKAGE_MT7601U_AP_OPENIPC=y
```

---

### Initial configuration

- The wireless settings are provided based on the chipset:

```diff
+SOC=$(fw_printenv -n soc)

# HI3516EV300 CamHi
+if [ "$SOC" == "hi3516ev300" ]; then
	devmem 0x100C0080 32 0x530
```

---

### Adopt custom settings

- The adapter configuration is located at: `/wireless-configuration/files/script/adapter`
- It is possible to convert existing wlan0 settings to the new wireless configuration:

```diff
auto wlan0
iface wlan0 inet dhcp
+	pre-up devmem 0x100C0080 32 0x530
+	pre-up echo 7 > /sys/class/gpio/export
+	pre-up echo out > /sys/class/gpio/gpio7/direction
+	pre-up echo 0 > /sys/class/gpio/gpio7/value
+	pre-up modprobe mt7601u
	pre-up wpa_passphrase "SSID" "password" >/tmp/wpa_supplicant.conf
	pre-up sed -i '2i \\tscan_ssid=1' /tmp/wpa_supplicant.conf
	pre-up sleep 3
	pre-up wpa_supplicant -B -D nl80211 -i wlan0 -c/tmp/wpa_supplicant.conf
	post-down killall -q wpa_supplicant
	post-down echo 1 > /sys/class/gpio/gpio7/value
+	post-down echo 7 > /sys/class/gpio/unexport
```

```diff
# HI3516EV300 CamHi
if [ "$SOC" == "hi3516ev300" ]; then
+	devmem 0x100C0080 32 0x530
+	echo 7 > /sys/class/gpio/export
+	echo out > /sys/class/gpio/gpio7/direction
+	echo 0 > /sys/class/gpio/gpio7/value
+	echo 7 > /sys/class/gpio/unexport
+	modprobe mt7601usta
fi
```

---

### Enter wireless credentials

- For the initial setup, the device will create an access point with the name OpenIPC and password 12345678.
- After connecting to the device, credentials can be changed with the wireless script:

```shell
wireless setup [SSID] [PASS]
```

- Additional settings are:

```shell
wireless connect
wireless reset
wireless show
```
