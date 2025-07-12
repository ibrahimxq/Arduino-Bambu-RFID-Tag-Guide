#Bambu Lab RFID Tag Guide With Arduino
This guide provides a basic overview of how to decrypt and read your tags using an Arduino RC522 sensor.

View Collection of Tags

I you are trying to find a cost-effective way to decrypt and read or write on your Bambu RFID Tags, you can use the RC522 sensor for Arduinos and ESP32 (not tested). This code is still in progress, so contributions would be pretty great to help me improve this repository.

The only way I found out to decrypt and read or write on the Bambu RFID tags is first find out the B keys and then using them to write the default keys (FFFFFFFFFFFF) which is recommended because if you are planning to use the code in Examples in Arduino IDE, what i have seen is that it can't dump the data until you have the right keys in the tag. After that, you can use the DumpInfo code in the Examples in Arduino IDE.
