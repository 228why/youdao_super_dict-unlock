# NetEase Youdao Super Dictionary — System Unlock & Customization Guide (macOS) and Adding Lawnchair Launcher Entry to the Native System

---

## ⚠️ Disclaimer

**Please read the following carefully before proceeding:**

1. **Personal devices only**: All operations described in this guide are intended solely for devices legally owned by the user. Use on others' devices or for commercial purposes is strictly prohibited.

2. **Proceed at your own risk**: Unlocking the Bootloader, flashing a third-party Recovery (TWRP), and gaining Root access are low-level system modifications that may result in a bricked device, data loss, or voided warranty. You assume full responsibility for any consequences.

3. **Software license agreement**: This guide involves reverse-engineering and modifying a pre-installed APK. Such actions may conflict with the End User License Agreement (EULA) of the relevant software. This guide is intended for technical research and documentation purposes only, and does not encourage using modified software in any way that violates license agreements.

4. **No warranty**: The content of this guide is provided "as is". The author makes no guarantees regarding the outcome of any operations described herein, and accepts no liability for any loss or damage resulting from following this guide.

5. **Applicable law**: Laws regarding device unlocking and reverse engineering vary by country and region. Please familiarize yourself with and comply with the laws of your jurisdiction before proceeding.

6. **Technical purpose statement**: This guide is published in the open-source community for the purpose of documenting and sharing technical research on specific hardware devices, for reference and learning by users with similar needs.

---

> **This is the complete macOS solution. For Windows, refer to https://www.bilibili.com/opus/914651317631189049. If you only need to add the Lawnchair Launcher entry, skip to Step 7 and beyond.**
>
> Compatible models: YDD011 / YDD012
> Compatible firmware versions: 1.2.7+ (including 1.2.8)
> Operating system: macOS (Intel / Apple Silicon both supported)
> Difficulty: Intermediate — some command-line experience required

---

## Device Information

| Item | Details |
|------|---------|
| Model | YDD011 / YDD012 |
| Firmware Version | 1.2.7 / 1.2.8 |
| Android Version | 8.1.0 |
| Chipset | MediaTek MT6739 |
| Architecture | ARM64 |

---

## End Result

- An "Android Desktop" entry is added to the dictionary's home screen, launching Lawnchair Launcher directly
- Magisk root access is permanently retained
- ADB remains permanently available, allowing installation of any APK at any time
- The dictionary's original translation functionality is fully preserved
- System apps such as Calculator and Calendar remain unchanged

---

## Required Materials

### Hardware
- USB-C cable (must support data transfer, not charge-only)
- USB OTG adapter + USB mouse (required for TWRP operation — the dictionary has only one USB-C port)

### Files (download in advance)

| File | Download Link |
|------|--------------|
| TWRP (YDD011-specific) | https://github.com/axycrio/Action-Recovery-builder/releases/download/2322650745/recovery.img |
| Magisk APK | https://github.com/topjohnwu/Magisk/releases/latest |
| fkyoudao module | https://github.com/ianchb/fkyoudao/releases/download/1/Fkyoudao_1.zip |
| Lawnchair 12 APK | https://github.com/LawnchairLauncher/lawnchair/releases (download the latest version) |

---

## Physical Button Reference

This guide refers to three physical buttons on the device:
- **Read-along key**: Used to navigate options in the boot menu
- **Assistant key**: Used to confirm selections
- **Power key**: Powers the device on/off

---

## Part 1: Environment Setup

### 1.1 Install Homebrew (skip if already installed)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 1.2 Install Android Platform Tools

```bash
brew install android-platform-tools
```

This installs the `adb` and `fastboot` commands.

### 1.3 Install MTK Client dependencies

```bash
brew install macfuse libusb
```

### 1.4 Install MTK Client

Since GitHub may be inaccessible in some regions, install manually:

1. Visit `https://github.com/bkerler/mtkclient` in your browser
2. Click the green **Code** button → **Download ZIP**
3. Extract the archive, then run in Terminal:

```bash
cd ~/Downloads/mtkclient-main
pip3 install -r requirements.txt
pip3 install .
```

Verify the installation:

```bash
mtk --help
```

If the help message appears, the installation was successful.

### 1.5 Install apktool (used in later steps)

```bash
brew install apktool
```

---

## Part 2: Unlock Bootloader

### 2.1 Background

The YDD011 uses a MediaTek MT6739 chipset. This chip supports direct partition access via DA (Download Agent) mode. Since the device's SBC/DAA security flags are both set to False, no authentication files are required. MTK Client can force-write the seccfg partition to unlock the Bootloader **without erasing user data**.

### 2.2 Enter deep flash mode and unlock

1. **Fully power off** the dictionary
2. Run the following command in Terminal (run the command first, then plug in):

```bash
mtk da seccfg unlock
```

3. Once the terminal begins scanning (a row of dots appears), hold down the **Read-along key + Assistant key** simultaneously and plug the USB cable into the dictionary
4. Wait for detection. On success, the output will read:

```
DaHandler - Successfully wrote seccfg.
```

> **macOS note:** If the device is not detected, try both orders: run the command then plug in, or plug in then run the command. The key is to complete the connection while the terminal is actively scanning.

### 2.3 Verify the unlock

Restart the dictionary (unplug the cable, then hold the power key). During boot, an **Orange State** warning screen will appear — this is Android's normal notification that the Bootloader has been unlocked. The device will automatically proceed to the system after a few seconds.

---

## Part 3: Flash TWRP

### 3.1 Enter Fastboot mode

1. Power off the dictionary
2. Hold the **Read-along key** without releasing
3. Press the power key to boot
4. A boot mode selection menu appears on screen
5. Continue pressing the Read-along key to navigate to **Fastboot Mode**
6. Press the **Assistant key** to confirm
7. The screen displays "Fastboot Mode..."

### 3.2 Flash TWRP

```bash
fastboot flash recovery ~/Downloads/recovery.img
```

Output of `Finished. Total time: X.XXXs` indicates success.

### 3.3 Boot into TWRP (macOS-specific steps)

> **Important:** On macOS, the `fastboot reboot recovery` command will return an error because the dictionary disconnects from USB immediately after receiving the command, before macOS can receive the return value. However, the command has already been successfully sent and the dictionary will reboot normally.
>
> Additionally, the YDD011 firmware includes a partition protection mechanism that restores the recovery partition to the stock version on every normal boot. Therefore, you must boot into TWRP immediately after flashing — do not allow a normal boot cycle in between.

Steps:

1. Run the flash command:
```bash
fastboot flash recovery ~/Downloads/recovery.img && fastboot reboot
```

2. The moment `Finished` appears in the terminal, **immediately** hold down the **Read-along key** without releasing
3. When the boot mode menu appears, select **Recovery Mode** and press the Assistant key to confirm
4. TWRP will load

> **Tip:** If you miss the timing, the dictionary will boot normally and the recovery partition will be restored. Simply repeat the steps — it takes a few tries. Speed is critical the moment `Finished` appears.

### 3.4 Navigating TWRP (mouse required)

After entering TWRP, connect the OTG adapter + mouse to the dictionary:

1. When prompted, select **Keep Read Only**
2. You can tap the language option to switch to your preferred language

---

## Part 4: Flash Magisk

### 4.1 Prepare the file

Rename the Magisk APK to .zip:

```bash
cp ~/Downloads/Magisk-v*.apk ~/Downloads/Magisk.zip
```

### 4.2 Flash via ADB Sideload

In TWRP:
1. Tap **Advanced**
2. Tap **ADB Sideload**
3. Swipe the slider to confirm

Unplug the mouse, connect the USB cable, then run in Terminal:

```bash
adb sideload ~/Downloads/Magisk.zip
```

`Total xfer: X.XXx` indicates success.

---

## Part 5: Flash the fkyoudao Module

### 5.1 What it does

fkyoudao is a Magisk module that automatically runs a script 15 seconds after boot. It launches the native Android Settings (`com.android.settings`) and enables USB debugging (ADB) and MTP file transfer.

### 5.2 Flash steps

From the TWRP main menu:
1. Reconnect the mouse
2. Tap **Reboot** → **Recovery** to re-enter TWRP
3. Go to **Advanced** → **ADB Sideload** → swipe the slider
4. Unplug the mouse, connect the USB cable, then run:

```bash
adb sideload ~/Downloads/Fkyoudao_1.zip
```

5. When complete, select **Reboot System**

### 5.3 Post-boot setup

After rebooting, wait approximately 15 seconds. The native Android Settings screen will appear automatically.

In Settings:
1. Find **About tablet** or **About device**
2. Tap **Build number** 7 times to enable Developer options
3. Go back to Settings and open **Developer options**
4. Enable **USB debugging**

Connect the USB cable. The dictionary will display an "Allow USB debugging?" dialog — tap **Allow**.

Verify the connection:

```bash
adb devices
# Example output: 4AA1300000107730   device
```

---

## Part 6: Install Full Magisk and Obtain Root

### 6.1 Push and install full Magisk

```bash
cp ~/Downloads/Magisk-v*.apk ~/Downloads/Magisk.apk
adb push ~/Downloads/Magisk.apk /sdcard/Magisk.apk
adb shell pm install /sdcard/Magisk.apk
```

### 6.2 Activate Root

Launch Magisk:

```bash
adb shell monkey -p com.topjohnwu.magisk 1
```

In the Magisk interface, tap **Direct Install**, then tap **Reboot** when complete.

After rebooting, root access is fully active. Verify:

```bash
adb shell su -c "id"
# After tapping Allow in the authorization dialog, output should be: uid=0(root)
```

---

## Part 7: Install Lawnchair

```bash
adb push ~/Downloads/Lawnchair*.apk /sdcard/lawnchair.apk
adb shell pm install /sdcard/lawnchair.apk
```

---

## Part 8: Modify the Dictionary Home Screen (Core Steps)

This is the core innovation of this guide: reverse-engineering the dictionary APK to locate and modify the app launch logic, adding an entry to the home screen that launches Lawnchair.

### 8.1 Reverse engineering analysis

**Pull the dictionary APK:**

```bash
adb pull /system/priv-app/HardwareDict/HardwareDict.apk ~/Downloads/HardwareDict.apk
```

**Decompile the APK:**

```bash
apktool d ~/Downloads/HardwareDict.apk -o ~/Downloads/HardwareDict_smali
```

**Findings:**

The dictionary home screen loads a local HTML/JS page via WebView (`file:///android_asset/updatable/webfront/`). The app list data comes from a local SQLite database called `apps_db`.

When an app is tapped, JavaScript calls the Java-side `HomeLinkHelper.appLink()` method via JSBridge. This method uses a **hardcoded switch-case structure** that determines which Activity to launch based on the string value of the `link` field:

```java
// HomeLinkHelper.java (decompiled)
case "thirdapp://crackingenglish":
    startActivityInNewTaskSafely(activity,
        "com.youdao.crackingenglish",
        "com.youdao.crackingenglish.SpeakMainActivity");
    break;
case "thirdapp://calculator":
    startActivityInNewTaskSafely(activity,
        "com.android.calculator2",
        "com.android.calculator2.Calculator");
    break;
// and so on...
```

**Key issues:**
1. The `appPackageName` field in the database is entirely ignored — only the `link` field matters
2. The system filters out all apps declaring the `HOME` category (i.e. Launcher-type apps) when processing `thirdapp://` links, so simply adding Lawnchair's package name does not work
3. Any `link` value not present in the hardcoded list is silently ignored by the switch-case — tapping produces no response

**Solution:** Redirect the launch target of `thirdapp://crackingenglish` (Youdao Oral English) from its original Activity to Lawnchair, and insert an "Android Desktop" entry in the database that uses this link.

### 8.2 Modify the smali code

Locate the lines to modify:

```bash
grep -n "crackingenglish\|SpeakMain" ~/Downloads/HardwareDict_smali/smali_classes2/com/youdao/hardware/dict/home/presenter/helper/HomeLinkHelper.smali
# Output:
# 503:    const-string p1, "com.youdao.crackingenglish"
# 505:    const-string v0, "com.youdao.crackingenglish.SpeakMainActivity"
```

Replace with Lawnchair:

```bash
sed -i '' '503s/com.youdao.crackingenglish/app.lawnchair/' \
  ~/Downloads/HardwareDict_smali/smali_classes2/com/youdao/hardware/dict/home/presenter/helper/HomeLinkHelper.smali

sed -i '' '505s/com.youdao.crackingenglish.SpeakMainActivity/app.lawnchair.LawnchairLauncher/' \
  ~/Downloads/HardwareDict_smali/smali_classes2/com/youdao/hardware/dict/home/presenter/helper/HomeLinkHelper.smali
```

Verify the changes:

```bash
grep -n "crackingenglish\|lawnchair" \
  ~/Downloads/HardwareDict_smali/smali_classes2/com/youdao/hardware/dict/home/presenter/helper/HomeLinkHelper.smali
# Expected output:
# 503:    const-string p1, "app.lawnchair"
# 505:    const-string v0, "app.lawnchair.LawnchairLauncher"
```

### 8.3 Repackage and sign

```bash
# Repackage
apktool b ~/Downloads/HardwareDict_smali -o ~/Downloads/HardwareDict_modified.apk

# Generate a signing key
keytool -genkey -v -keystore ~/Downloads/test.keystore -alias test \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -storepass testtest -keypass testtest \
  -dname "CN=test,OU=test,O=test,L=test,ST=test,C=CN"

# Sign the APK
jarsigner -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore ~/Downloads/test.keystore \
  -storepass testtest -keypass testtest \
  ~/Downloads/HardwareDict_modified.apk test
```

### 8.4 Replace the system APK

```bash
adb push ~/Downloads/HardwareDict_modified.apk /sdcard/HardwareDict_modified.apk
adb shell su -c "mount -o rw,remount /system"
adb shell su -c "cp /sdcard/HardwareDict_modified.apk /system/priv-app/HardwareDict/HardwareDict.apk"
adb shell su -c "chmod 644 /system/priv-app/HardwareDict/HardwareDict.apk"
```

### 8.5 Modify the app database

Pull the database (WAL files must be pulled together):

```bash
adb shell su -c "cp /data/data/com.youdao.hardware.dict/databases/apps_db /sdcard/apps_db"
adb shell su -c "cp /data/data/com.youdao.hardware.dict/databases/apps_db-wal /sdcard/apps_db-wal"
adb shell su -c "cp /data/data/com.youdao.hardware.dict/databases/apps_db-shm /sdcard/apps_db-shm"
adb pull /sdcard/apps_db ~/Downloads/apps_db
adb pull /sdcard/apps_db-wal ~/Downloads/apps_db-wal
adb pull /sdcard/apps_db-shm ~/Downloads/apps_db-shm
```

> **Note:** The database uses WAL (Write-Ahead Logging) mode. The actual data resides in the `-wal` file. All three files must be pulled together to access the complete data.

Use Python on your Mac to edit the database (no `sqlite3` command is available on the device):

```python
import sqlite3

conn = sqlite3.connect('/Users/your_username/Downloads/apps_db')
# Merge WAL data
conn.execute('PRAGMA wal_checkpoint(TRUNCATE)')

cursor = conn.cursor()

# View existing entries
cursor.execute('SELECT * FROM third_apps')
for row in cursor.fetchall():
    print(row)

# Insert the Android Desktop entry
cursor.execute("""
    INSERT INTO third_apps 
    (appName, link, style, isVisible, canHide, iconText, appPackageName) 
    VALUES ('Android Desktop', 'thirdapp://crackingenglish', 1, 1, 1, 'lawnchair', 'app.lawnchair')
""")
conn.commit()
conn.close()
```

Push the database back to the device and reboot:

```bash
adb push ~/Downloads/apps_db /sdcard/apps_db
adb shell su -c "cp /sdcard/apps_db /data/data/com.youdao.hardware.dict/databases/apps_db"
adb shell su -c "rm -f /data/data/com.youdao.hardware.dict/databases/apps_db-wal"
adb shell su -c "rm -f /data/data/com.youdao.hardware.dict/databases/apps_db-shm"
adb shell reboot
```

---

## Part 9: Verification and Wrap-up

After rebooting, navigate to the second page of the app list on the dictionary home screen and tap **Android Desktop** — Lawnchair will launch successfully.

### Installing additional apps

With ADB access, you can install any APK (a navigation bar app is recommended to compensate for the missing system gesture bar):

```bash
adb push app.apk /sdcard/app.apk
adb shell pm install /sdcard/app.apk
```

---

## Part 10: Troubleshooting

**Q: MTK Client reports `Unable to find libfuse`**

```bash
brew install macfuse
```

**Q: MTK Client reports `No backend available`**

```bash
brew install libusb
```

**Q: `mtk da seccfg unlock` keeps scanning but never detects the device**

Try both orders: run the command then plug in, or plug in then run the command. Make sure the dictionary is completely powered off.

**Q: Can't enter TWRP — always boots into stock Recovery**

This happens because the system restores the recovery partition during a normal boot. Repeat the flash + reboot sequence, and press the Read-along key the instant `Finished` appears.

**Q: `adb sideload` says `no devices/emulators found`**

In TWRP, you must swipe the Sideload slider first, then connect the USB cable. The mouse and USB cable cannot be connected at the same time — the dictionary has only one USB-C port.

**Q: `fastboot reboot recovery` throws an error**

This is a known macOS behavior: the USB disconnect happens before macOS can receive the return value. The command has already been sent successfully. Ignore the error and manually navigate to Recovery using the physical keys.

---

## Acknowledgements

- Original Bilibili tutorial author **ChengHB**: bilibili.com/opus/914651317631189049
- TWRP port author **axycrio**: https://github.com/axycrio/twrp_device_et669_64_bsp
- fkyoudao module author **ianchb**: https://github.com/ianchb/fkyoudao/releases/download/1/Fkyoudao_1.zip
- MTK Client author **bkerler**

> Note: This document was generated with the assistance of Claude AI and may contain errors. Please review carefully. @https://github.com/228why
