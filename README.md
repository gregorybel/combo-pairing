## combo-pairing
Solutions to move away pairing issue on new android phone with combo pump

The problem:
Since Android 4.1, google move bt stack from bluez to broadcom. For unknown reasons this stack has a bug during pairing: device name will be broadcasted wil a \0 terminated character. The combo pump can't deal with that and that is the reason why combo can#t find phone.

Concept to move away from this problem:
Pairing needs to be done only time and purpose is only to generate a so called linkkey which will be stored on both devices (phone and pump). Linkkey is generated using both mac adresses, pin code and some keys generated during pairing. So if we manage somehow to pair pump with one device and then move this linkkey to another device then pump will accept pairing from second device! This took my some time but it works! This is what I will explan in this readme.

What you need:
- one phone with android 4.1
- another phone, what ever the Android version
- Attention: both phone need to be rooted
- Both phone with ruffy installed
- Developer mode and debug over USB activated
- Option: Titanium backup





