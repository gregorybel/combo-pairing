## combo-pairing

[![Join the chat at https://gitter.im/combo-pairing/Lobby](https://badges.gitter.im/combo-pairing/Lobby.svg)](https://gitter.im/combo-pairing/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
Solutions to move away pairing issue on new android phone with combo pump

The problem:
Since Android 4.1, google move bt stack from bluez to broadcom. For unknown reasons this stack has a bug during pairing: device name will be broadcasted wil a \0 terminated character. The combo pump can't deal with that and that is the reason why combo can#t find phone.

Concept to move away from this problem:
Pairing needs to be done only time and purpose is only to generate a so called linkkey which will be stored on both devices (phone and pump). Linkkey is generated using both mac adresses, pin code and some keys generated during pairing. So if we manage somehow to pair pump with one device and then move this linkkey to another device then pump will accept pairing from second device! This took my some time but it works! This is what I will explan in this readme.

What you need:
- one phone with android 4.1 (phone1)
- On this find install: [mac changer]
- Phone you want to sue with your pump, what ever the Android version (phone2)
- Attention: both phone need to be rooted
- Both phone with ruffy installed: https://github.com/monkey-r/ruffy
- Developer mode and debug over USB activated
- Option: Titanium backup


Howto:
- Note bluetooh mac adress from phone 2 (Settings/About phone/State/Blueooth adress)
- Swith off Bluetooth on both phones
- On phone 1, use MAC CHANGER and modify bluetooth mac adresse to the one fron phone 2
- Reboot phone 1

- Enable Bluetooth on phone 1
- Check that phone 1 has Mac @ from phone 2
- Pair phone 1 with combo pump as stated into ruffy
- Note combo mac adress (displayed into ruffy log window right at the beginning)
- Open terminal
adb shell
su
cat /data/misc/bluetoothd/[new mac adress]/linkkeys
XX:XX:XX:XX:XX:XX 1D44B76C0CCDD88357073475C7D13B6D 4 0

16 bytes number is the linkkey, note the linkkey corresponding to combo mac adress:
1D44B76C0CCDD88357073475C7D13B6D

Make lower case:
1d44b76c0ccdd88357073475c7d13b6d

Swap byte wise reverse order:
6d3bd1c77534075783d8cd0c6cb7441d

- Disable BT on phone 1
- Kill ruffy
- Copy from phone 1 /data/data/org.monkey.d.ruffy.ruffy/shared_prefs/pumpdata.xml to same location on phone 2
- [OR] you may want to use Titanium Backup and back data from ruffy, transfer backupo to phone 2 ans restore it
* When I do it, I use dropbox to updload/Download backup file

- Connect phone 2 to PC:
adb shell
su
vi /data/misc/bluedroid/bt_config.conf

Past this:
[COMBO_MAC_LOWER_CASE]
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

Exit vi and save: ':wq'

- reboot
- reenable BT on phone 2
- Into the BT stettings, you should see as device "SpiritCombo"
- Kill ruffy
- copy /data/data/org.monkey.d.ruffy.ruffy/shared_prefs/pumpdata.xml on phone 2 
- Open ruffy and click "connect"

If not working, let me know on gitter!
















