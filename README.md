# Bambu Lab RFID Tag Guide With Arduino
This guide provides a basic overview of how to decrypt and read your tags using an Arduino RC522 sensor.

[View Collection of Tags](https://github.com/queengooborg/Bambu-Lab-RFID-Library)

I you are trying to find a cost-effective way to decrypt and read or write on your Bambu RFID Tags, you can use the RC522 sensor for Arduinos and ESP32 `(not tested)`. This code is still in progress, so contributions would be pretty great to help me improve this repository.

The only way I found out to decrypt and read or write on the Bambu RFID tags is first find out the B keys and then using them to write the default keys `(FFFFFFFFFFFF)` which is recommended because if you are planning to use the code in `Examples` in `Arduino IDE`, what i have seen is that it can't dump the data until you have the right keys in the tag. After that, you can use the `DumpInfo` code in the `Examples` in `Arduino IDE`.



# Hacking a Bambu Lab Tag and readout of its data
Here is where you have the way to derive and read the data of the tags.
### Deriving the keys
A way to derive the keys from the UID of an RFID tag was discovered, which unlocked the ability to scan and scrape RFID tag data without sniffing, as well as with other devices like the Flipper Zero. A script is included in the repository to derive the keys from the UID of a tag. This script is from the [Bambu-Research-Group / RFID-Tag-Guide](https://github.com/Bambu-Research-Group/RFID-Tag-Guide)


````
# -*- coding: utf-8 -*-

# Python script to generate keys for Bambu Lab RFID tags
# Created for https://github.com/Bambu-Research-Group/RFID-Tag-Guide
# Written by thekakester (https://github.com/thekakester) and Vinyl Da.i'gyu-Kazotetsu (www.queengoob.org), 2024

import sys
from Cryptodome.Protocol.KDF import HKDF
from Cryptodome.Hash import SHA256

if not sys.version_info >= (3, 6):
   print("Python 3.6 or higher is required!")
   exit(-1)

def kdf(uid):
    master = bytes([0x9a,0x75,0x9c,0xf2,0xc4,0xf7,0xca,0xff,0x22,0x2c,0xb9,0x76,0x9b,0x41,0xbc,0x96])
    return HKDF(uid, 6, master, SHA256, 16, context=b"RFID-B\0")

if __name__ == '__main__':
    uid = bytes.fromhex(sys.argv[1])
    keys = kdf(uid)

    output = [a.hex().upper() for a in keys]
    print("\n".join(output))

    output = [a.hex().upper() for a in keys]
    print("\n".join(output))
````

To use this code, open up a terminal and type this command in the same directory as the Python script: `python3 deriveKeys.py [UID] > ./keys.dic`

For example: `python3 deriveKeys.py 7AD43F1C > ./keys.dic`



Now to use it, just use the Bambu-RFID-Write.ino file to write the default keys to the RFID Tag.
