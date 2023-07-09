# TerrainTronics Local Wireless Network Control
### Last Updated June 21st 2023

ESP-Now is a wireless communication protocol that is made by Espressive, the makers of ESP8266 and ESP32.
It doesn't require WiFi etc and is designed for it's own micro network.
Each node is communicated with using it's MAC address. A Controller communicates with Workers by knowing the Workers MAC address - so we need some way for Workers to communicate their MAC addresses to the controller in the first place. This could be something like InfraRed command from the worker, to the controller 

The ESP=Now protocol that is proposed to be used 

Communication can be for up to 250Bytes. Below is a proposal for how these should be used in TerrainTronics ecosystem. In TT ecosystem, the plan so far is for the data to travel one way, from Controller to Worker. ESP-Now does handle acknowledgement of receipt.

Worker MAC address shall be added to the Controller either using USB-Serial command, Firmware or using IR TX (Worker) to IR-RCVR (Controller) 

Commands to be supported:

- Initialize
- Info/Config Info - share which worker number the MAC address is.
- Trigger/Stop MP3
- Set Volume
- IR Message/Bridge
- RGB LED
- 

The general command structure is detailed below. The structure is described as A (Byte 0), B (Byte 1) all the way through the alphabet - that's 26 bytes described, the rest can be A1,A2 etc...
Whilst burning 250 bytes of transmission is wasteful, it's so quick on the ESP8266 that it's easier to let all instructions have 250bytes, then parse them in software based on the Command, Worker Address and Payload

| Comment | Byte 0 | Byte 1  |  Byte 2-250  | 
|--|--|--|--|
||  **Command** |**Worker Address** (see note)* |**Data Payload**  |
| Initialize| 0x00|0x00 (Global) or Specific (e.g. 0x01)| No Payload.
| System Info| 0x01|0x00 (Global) or Specific (e.g. 0x01)| See specific section
|MP3 Control|0x02|0x00 (Global) or Specific (e.g. 0x01)|See specific section|
| Set Volume| 0x03|0x00 (Global) or Specific (e.g. 0x01)| 0x00 -> 0xFF (single Byte payload)
| IR Message/Bridge| 0x04|0x00 (Global) or Specific (e.g. 0x01)| See specific section
| RGB-LED| 0x05|0x00 (Global) or Specific (e.g. 0x01)| See specific section
| Haptic ADSR| 0x05|0x00 (Global) or Specific (e.g. 0x01)| See specific section


* Worker Address (0x00 is "Global") - this is NOT the MAC address to talk to. All devices will receive this, and individual workers will need to know which worker Address they are. (i.e. Info-dump mode - devices will find out what address they are based on their MAC, e.g. **00-B0-D0-63-C2-26** = Device 5.


## Initialize

| Comment | Byte 0 | Byte 1  |  Byte 2-250  | 
|--|--|--|--|
||  **Command** |**Worker Address** (see note)* |**Data Payload**  |
| Initialize| 0x00|0x00 (Global) or Specific (e.g. 0x01)| No Payload.

This command is used to tell the worker (global or specific) to reset all outputs (LED's OFF, Stop Music etc)
**Use Case Description** Need to switch off all lights, disable outputs etc.

## System Info

| Comment | Byte 0 | Byte 1  |  Byte 2-250  | 
|--|--|--|--|
||  **Command** |**Worker Address** (see note)* |**Data Payload**  |
| System Info| 0x01|Specific (e.g. 0x01)| B Bytes of Mac Address, then zero's

**Use Case Description** All connected Workers need to decode this message. First they'll check the mac address and compare it to their own. If there is a match, then the slave.
**Example Description** 0x01, 0x03, 0x00, 0xB0, 0xD0, 0x63, 0xC2, 0x26, 0x...
System Info, Worker #3 should be whichever one is using MAC 00-B0-D0-63-C2-26

## MP3 Sound Effect Control

| Comment | Byte 0 | Byte 1  |  Byte 2-250  | 
|--|--|--|--|
||  **Command** |**Worker Address** (see note)* |**Data Payload**  |
|MP3 Control|0x02|0x00 (Global) or Specific (e.g. 0x01)| 2byte payload - first byte = output channel, second byte 0x00 = Stop, Any other 8bit value = which sample to play)|

**Use Care Description** If a connected worker is using a DFPlayer MP3 Module, this command tells the worker which sample to play from the SD Card. This could be used for sound effect or haptics effects etc. 
This uses a 2 byte payload, the first is the channel to communicate with (e.g. sound effects on 0, haptics on 1 etc)
**Example Instruction:** 0x02, 0x00,0x01,0x03, 0x00...
MP3 Instruction, Global Message for all recievers, Channel 1 (Haptic MP3), Play file 3. 
*0'xs after that means nothing.*

## Set Volume Control

| Comment | Byte 0 | Byte 1  |  Byte 2-250  | 
|--|--|--|--|
||  **Command** |**Worker Address** (see note)* |**Data Payload**  |
| Set Volume| 0x03|0x00 (Global) or Specific (e.g. 0x01)| 0xXY (Channel), 0xYZ ->0x00-0xFF Volume between 0 to 255. (two Byte payload)

**Use Care Description** This scales the output of the signal, depending on the channel used. This uses a 2 byte payload, the first is the channel to communicate with (e.g. sound effects on 0, haptics on 1 etc). The second is a volume control between 0 and 255. Linear vs Log, down to the output etc.
**Example Instruction:** 0x03, 0x02,0x03,0xFF, 0x00...
VOLUME Instruction, Worker Address 2, Output Channel 3 (Haptic MP3), Maximum Volume. 
*0'xs after that means nothing.*


## IR Message / Bridge

| Comment | Byte 0 | Byte 1  |  Byte 2-250  | 
|--|--|--|--|
||  **Command** |**Worker Address** (see note)* |**Data Payload**  |
| Set Volume| 0x04|0x00 (Global) or Specific (e.g. 0x01)| ~~0xXY (Channel), 0xYZ ->0x00-0xFF Volume between 0 to 255. (two Byte payload)~~

**Use Case Description** Worker unit is placed near a standard consumer product that has IR control (e.g. an IR controlled light fixture). From the Controller, you want to tell the Worker to "send this IR message out of your IR transmitter!".
***

> (WORK TODO -  What Parameters are needed? Protocol, Address, Command?)

***
**Example Instruction:** ~~0x03, 0x02,0x03,0xFF, 0x00...
VOLUME Instruction, Worker Address 2, Output Channel 3 (Haptic MP3), Maximum Volume. 
*0'xs after that means nothing.*~~



> Written with [StackEdit](https://stackedit.io/).
