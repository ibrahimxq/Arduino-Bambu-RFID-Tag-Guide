# Bambu Lab Experimental RFID Tag Guide With Arduino
This experimental guide provides a basic overview of how to decrypt and read your tags using an Arduino RC522 sensor.

[View Collection of Tags](https://github.com/queengooborg/Bambu-Lab-RFID-Library)

## Requirements

- A computer running macOS or Linux, or a Windows computer.
- Python 3.6 or higher
- Bambu Lab Filament spool **or** the related tags
- Arduino IDE
- RC522 RFID sensor
- Arduino or ESP32 **(not tested)**

I you are trying to find a cost-effective way to decrypt and read or write on your Bambu RFID Tags, you can use the RC522 sensor for Arduinos and ESP32 **(not tested)**. This code is still in progress, so contributions would help improve this repository.

The only way I found out to decrypt and read or write on the Bambu RFID tags is first find out the B keys and then using them to write the default keys `(FFFFFFFFFFFF)` which is recommended because if you are planning to use the code in **Examples** in **Arduino IDE**, what i have seen is that it can't dump the data until you have the right keys in the tag. After writing the default keys, you can use the `DumpInfo.ino` code in the **Examples** in the **Arduino IDE** to dump the data.



# Hacking a Bambu Lab Tag and readout its data
Here is where you have the way to derive and read the data of the tags.
### Deriving the keys
A way to derive the keys from the UID of an RFID tag was discovered, which unlocked the ability to scan and scrape RFID tag data without sniffing. TO find the UID of the Tag, you can use the [DumpInfo.ino](DumpInfo.ino) to find the UID. To derive the keys, a script is included in the repository to derive the keys from the UID of a tag. This script is originally from the [Bambu-Research-Group / RFID-Tag-Guide](https://github.com/Bambu-Research-Group/RFID-Tag-Guide)


````python
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

Link to script: [Key Derivation.py](deriveKeys.py)

To use this code, open up a terminal and type this command in the same directory as the Python script: `python3 deriveKeys.py [UID] > ./keys.dic`

For example: `python3 deriveKeys.py 7AD43F1C > ./keys.dic`.
It will save the keys to a .dic file, which you can open with a text editor and see the keys.



Now to use it, use the [Bambu-RFID-Write.ino](Bambu-RFID-Write.ino) file to write the default keys to the RFID Tag.
In the file, you can see where you need to change the key to authenticate with the card. What it does is it replaces the `878787` with `FF0780` in the last Block of every Sector, and resets permissions, and it writes `FFFFFFFFFFFF078069FFFFFFFFFFFFFF` to the last Block of every Sector and resets permissions and sets A- and B-Keys to FFFFFFFFFFFFF (writes on sector trailers).
### Guide, use at your own risk! 

At the start of the code, you can see where you need to change the key:
````c++
#include <SPI.h>
#include <MFRC522.h>

#define RST_PIN 9
#define SS_PIN 10

MFRC522 rfid(SS_PIN, RST_PIN);

// Replace with your current working Key B
MFRC522::MIFARE_Key keyB = {
  .keyByte = {0x85, 0xA0, 0x56, 0xBD, 0x52, 0xCD}
};
````

Then you have where you have to change the block, you have to change the 3 to: 7, 11, 15, 19, 23, 27, 31, 35, 39, 43, 47, 51, 55, 59, 63:
````c++
}

void loop() {
  if (!rfid.PICC_IsNewCardPresent() || !rfid.PICC_ReadCardSerial()) {
    return;
  }

  byte trailerBlock = 3; // Block 3 is the trailer of sector 0

  // Authenticate with Key B
  MFRC522::StatusCode status = rfid.PCD_Authenticate(
    MFRC522::PICC_CMD_MF_AUTH_KEY_B,
    trailerBlock,
    &keyB,
    &(rfid.uid)
  );
````


Remember to change the key for each sector trailer, or it will give you an error.


After you have run and written the new keys, you can dump out its data using the [DumpInfo.ino](DumpInfo.ino) and it will successfully dump. After that, you can use the Tag without any problems while dumping, reading, and writing.
