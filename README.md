# Combo pairing with incompatible Android version

[![Join the chat at https://gitter.im/gregorybel/combo-pairing](https://badges.gitter.im/gregorybel/combo-pairing.svg)](https://gitter.im/gregorybel/combo-pairing?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
Solutions to move away pairing issue on new android phone with Roche combo pump

## The problem and solution approach:
Since Android 4.1, google moves bt stack from bluez to broadcom. After some research, I found that this stack has a bug during pairing: device's name will be broadcasted with a \0 terminated character. The combo pump can't deal with that and that is the reason why combo can't find phone.

Concept to move away from this problem:
Pairing needs to be done only time and purpose is only to generate a so called linkkey which will be stored on both devices (phone and pump). Linkkey is generated using both mac addresses, pin code and some keys generated during pairing. So if we manage somehow to pair pump with one device and then move this linkkey to another device then pump will accept pairing from second device! This took my some time but it works! This is what I will explan in this readme.

## What you need:
- one phone with android 4.1 or any LineageOS 14.1 (tested with a Samsung or Huawei/Honor phone) (phone 1)
- Phone you want to use with your pump, what ever the Android version (phone 2)
- Attention: both phones need to be rooted!
- Both phones with ruffy installed: https://github.com/monkey-r/ruffy
- Developer mode and debug over USB activated
- Option: Titanium backup installed


Note: before continuing you might want to backup your data first: https://github.com/gregorybel/combo-pairing/blob/master/Backup.md


## Howto:

### Preparation
- Note bluetooh MAC address from phone 2 (Settings/About phone/State/Bluetooth address)
- Note Bluetooth MAC address from phone 1 too (if you want to set it back later)
- Switch off Bluetooth on both phones

### Clone Bluetooth MAC address of phone 2 onto phone 1
#### Modify Bluetooth MAC address on a Samsung phone 1
- On phone 1, modify Bluetooth MAC address to the one from phone 2
- Optional: change BT name on phone 1 to the same BT name you have on phone 2 (to change the name, BT has to be on)
```
adb shell
su
vi /efs/bluetooth/bt_addr
'insert'
-> change MAC
:wq
```

  or 
  - copy the File with a root Filemanager (like total commander) direct on the Smartphone to SDCard, modify the MAC there, and copy it back to the old place (this is related due to linux authorisation restriction that copy is possible direct, but not modifying). This way is may in the case that no vi is available in the adb shell.

- Reboot phone 1
- Enable Bluetooth on phone 1


#### Modify Bluetooth MAC address on a Huawei phone 1

- enable Bluetooth
- Since Huawei phones do not have vi or any other cli editor installed, the following steps will copy the relevant files to a folder that is readable via a file browser on your computer with the phone atatched via USB:
```
adb shell
su
mkdir /mnt/sdcard/Bluetooth/original
cp /data/misc/bluedroid/* /mnt/sdcard/Bluetooth/original
exit
exit
```

- Reboot phone (to make sure the new files are visible in the file browser on the computer)
- Disable Bluetooth
- Enable Usb storage access and copy files from directory `Bluetooth/original` to local computer
- Open `btmac` in editor that supports unix line breaks and modify Bluetooth MAC address to Bluetooth mac address of phone 2. Be sure to use all capital letters and save changes
* open `bt_config.conf` in editor that supports unix line breaks. Within the `[Adapter]` section,  modify the line `Address = 12:ab:34:cd:e5:6f` to the Bluetooth MAC address (this time all non-capital letters) of phone 2 and save
- With your computer, create new directory `modified` under Bluetoooth on the smartphone
- copy the modified files `btmac` and `bt_config.conf` to directory `Bluetooth/modified`

```
adb shell
su
cp /mnt/sdcard/Bluetooth/modified/btmac /data/misc/bluedroid/ 
cp /mnt/sdcard/Bluetooth/modified/bt_config.conf /data/misc/bluedroid/ 
exit
exit
```

- enable Bluetooth
- Do **NOT** reboot the phone now, as this will reset the Bluetooth MAC address

#### Modifying Bluetooth MAC address on a non-Samsung or non-Huawei phone 1 - No tested solution yet
- The bluetooth MAC address is stored at different locations on Android phones. Unfortuantely no common location exists that works for all phones. 
- For phones with other partition-style please consult Google to find a proper way of bluetooth MAC spoofing for the specific phone. For Lineageos 14.1 standard free modules did not work (like xposed). Maybe paid versions are better, but no knowledge up to now. For Android 4.1 it may work.

### Pair Combo with modified phone 1 and save pairing configuration
- Check that phone 1 has Mac @ from phone 2 (Settings/About phone/State/Bluetooth address)
- To avoid problems caused by two phones having the same Bluetooth MAC address, turn off phone 2
- Pair phone 1 with Combo pump as stated into ruffy

#### a) Android 4.1:
  - Note Combo MAC address (displayed into ruffy log window right at the beginning)
  - Open terminal
  ```
  adb shell
  su
  cat /data/misc/bluetoothd/[new MAC address]/linkkeys
  XX:XX:XX:XX:XX:XX 1D44B76C0CCDD88357073475C7D13B6D 4 0
  ```

  16 bytes number is the linkkey, note the linkkey corresponding to Combo MAC address:
  ```
  1D44B76C0CCDD88357073475C7D13B6D
  ```

  Make lower case:
  ```
  1d44b76c0ccdd88357073475c7d13b6d
  ```

  Swap bytewise reverse order:
  ```
  6d3bd1c77534075783d8cd0c6cb7441d
  ```
  ![alt text](http://i.imgur.com/IMmUu0g.png)

#### b) Lineageos 14.1:
  - Output the linkkey directly from the file /data/misc/bluedroid/bt_config.conf
  ```
  adb shell
  su
  cat /data/misc/bluedroid/bt_config.conf
  ```

- Copy the complete section that contains the line `Name = SpiritCombo`, you will need it later on phone 2 (reference point X). The relevant section  starts with a blank line and the Combo Bluetooth MAC address in brackets and usually ends with another blank line or the end of the file.
- Disable BT on phone 1 or switch-off phone 1 and make sure it is not around the pump
- On Samsung phone, write back original mac address
- On Huawei phones, just reboot phone 1 to get back to the original bluetooth MAC address

### Copy pairing configuration to phone 2
- Kill ruffy
- Copy from phone 1 the complete directory /data/data/org.monkey.d.ruffy.ruffy/ to same location on phone 2
  Note: make sure the directory and its contents on phone 2 is NOT root owned, to check do ```ls -la /data/data/org.monkey.d.ruffy.ruffy``` , if yes do ```chown app_111:app_111 /data/data/org.monkey.d.ruffy.ruffy/*``` (or whatever the app owner is)
- [OR] you may want to use Titanium Backup and backup data from ruffy, transfer backup to phone 2 and restore it
* When I do it, I use dropbox to upload/download backup file (only data)
- [OR] you may want to look: https://github.com/gregorybel/combo-pairing/blob/master/Backup.md

- Connect phone 2 to PC:
```
adb shell
su
vi /data/misc/bluedroid/bt_config.conf
```

On Huawei phones with no vi editor available, re-apply the approach described above to copy the file to your local computer, edit it there and copy it back onto the phone.

Past this after having modified COMBO_MAC_LOWER_CASE and Linkkey:
```
[XX:XX:XX:XX:XX:XX]
Timestamp = 1476801455
DevClass = 001f00
DevType = 1
AddrType = 0
Manufacturer = 0
LmpVer = 0
LmpSubVer = 0
Name = SpiritCombo
LinkKeyType = 0
PinLength = 16
LinkKey = 6d3bd1c77534075783d8cd0c6cb7441d
Service = 00001101-0000-1000-8000-00805f9b34fb 00000000-0000-1000-8000-00805f9b3
```

or on Android 5:

```
adb shell
su
vi /data/misc/bluedroid/bt_config.xml
```

Past this after having modified COMBO_MAC_LOWER_CASE and Linkkey:

```
<N1 Tag="00:0e:2f:xx:xx:xx">
        <N1 Tag="Timestamp" Type="int">1398527119</N1>
        <N2 Tag="Name" Type="string">SpiritCombo</N2>
        <N3 Tag="DevClass" Type="int">001f00</N3>
        <N4 Tag="DevType" Type="int">1</N4>
        <N5 Tag="AddrType" Type="int">0</N5>
        <N6 Tag="Manufacturer" Type="int">0</N6>
        <N7 Tag="LmpVer" Type="int">0</N7>
        <N8 Tag="LmpSubVer" Type="int">0</N8>
        <N9 Tag="LinkKeyType" Type="int">0</N9>
        <N10 Tag="PinLength" Type="int">16</N10>
        <N11 Tag="LinkKey" Type="binary">6d3bd1c77534075783d8cd0c6cb7441d</N11>
        <N12 Tag="Service" Type="string">00001101-0000-1000-8000-00805f9b34fb 00000000-0000-1000-8000-00805f9b3</N12>
 </N1>
 ```

on LOS 14.1 or any rooted Android 7.1.x device 
go to the end of the file and paste the content you copied from phone 1 from /data/misc/bluedroid/bt_config.conf (reference point X)

```
[XX:XX:XX:XX:XX:XX]
Timestamp = 1476801455
DevClass = 001f00
DevType = 1
AddrType = 0
Manufacturer = 0
LmpVer = 0
LmpSubVer = 0
Name = SpiritCombo
LinkKeyType = 0
PinLength = 16
LinkKey = 6d3bd1c77534075783d8cd0c6cb7441d
Service = 00001101-0000-1000-8000-00805f9b34fb 00000000-0000-1000-8000-00805f9b3
```

Exit vi and save: ':wq'

- dont enable BT yet or /data/misc/bluedroid/bt_config.conf will be overwriten 
- reboot phone 2

### Finalize Ruffy configuration on phone 2
- reenable BT on phone 2
- Into the BT settings, you should see a new device "SpiritCombo"
- Kill ruffy
- copy /data/data/org.monkey.d.ruffy.ruffy/shared_prefs/pumpdata.xml on phone 2 (if no yet done) and make sure the directory and its contents on phone 2 is NOT root owned, to check do ```ls -la /data/data/org.monkey.d.ruffy.ruffy``` , if yes do ```chown app_111:app_111 /data/data/org.monkey.d.ruffy.ruffy/*``` (or whatever the app owner is)
- Open ruffy and click "connect"
- if ruffy crashes then maybe pumpdata.xml and its parent directorys or just one directory still is owned by root 
- Phone 2 should be connected to pump!

If not working, let me know on gitter!
@gregorybel
