## combo-pairing

[![Join the chat at https://gitter.im/gregorybel/combo-pairing](https://badges.gitter.im/gregorybel/combo-pairing.svg)](https://gitter.im/gregorybel/combo-pairing?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
Solutions to move away pairing issue on new android phone with Roche combo pump

The problem:
Since Android 4.1, google moves bt stack from bluez to broadcom. For unknown reasons this stack has a bug during pairing: device's name will be broadcasted with a \0 terminated character. The combo pump can't deal with that and that is the reason why combo can't find phone.

Concept to move away from this problem:
Pairing needs to be done only time and purpose is only to generate a so called linkkey which will be stored on both devices (phone and pump). Linkkey is generated using both mac addresses, pin code and some keys generated during pairing. So if we manage somehow to pair pump with one device and then move this linkkey to another device then pump will accept pairing from second device! This took my some time but it works! This is what I will explan in this readme.

What you need:
- one phone with android 4.1 or any LineageOS 14.1 (phone 1)
- Phone you want to use with your pump, what ever the Android version (phone 2)
- Attention: both phones need to be rooted!
- Both phones with ruffy installed: https://github.com/monkey-r/ruffy
- Developer mode and debug over USB activated
- Option: Titanium backup installed


Howto:
- Note bluetooh MAC address from phone 2 (Settings/About phone/State/Bluetooth address)
- Switch off Bluetooth on both phones
- On phone 1, modify Bluetooth MAC address to the one from phone 2
```
adb shell
su
vi /efs/bluetooth/bt_addr
'insert'
-> change MAC
:wq
```
- Reboot phone 1

- Enable Bluetooth on phone 1
- Check that phone 1 has Mac @ from phone 2
- Pair phone 1 with Combo pump as stated into ruffy
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
![alt text](http://i.imgur.com/IMmUu0g.png

- Disable BT on phone 1
- Kill ruffy
- Copy from phone 1 /data/data/org.monkey.d.ruffy.ruffy/shared_prefs/pumpdata.xml to same location on phone 2
- [OR] you may want to use Titanium Backup and backup data from ruffy, transfer backup to phone 2 and restore it
* When I do it, I use dropbox to upload/download backup file (only data)

- Connect phone 2 to PC:
```
adb shell
su
vi /data/misc/bluedroid/bt_config.conf
```

Past this after having modify COMBO_MAC_LOWER_CASE and Linkkey:
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

- reboot
- reenable BT on phone 2
- Into the BT settings, you should see a new device "SpiritCombo"
- Kill ruffy
- copy /data/data/org.monkey.d.ruffy.ruffy/shared_prefs/pumpdata.xml on phone 2 (if no yet done)
- Open ruffy and click "connect"
- Phone 2 should be connected to pump!

If not working, let me know on gitter!
