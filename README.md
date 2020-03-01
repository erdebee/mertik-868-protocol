# mertik-868-protocol
An effort to reverse engineer the wireless signal for Mertik Maxitrol 868MHz controlled fireplace.
This system is sometimes also referred to by Mertik as the "Symax Remote Controller"

The below research has been done with -for now- just 1 remote.
These are the specs of the remote:

Mertik Maxitrol 12 buttons "Symax" Remote
European version
Freq. 868.1 Mhz
Product Nr. B6R-H8TV21PBD

## Signal interpretation
In Ultimate Radio Hacker, the following settings lead to a correct interpretation of the signals:

Frequency: 868.0MHz
Modulation: FSK (Actually its GFSK, but FSK works the same in URH)
Noise 0,08
Center 0,25 (sometimes a little lower or higher, should be in the middle of the preamble signal freq.)
Samples per symbol: 62 (at 2MSps / 2MHz = 31 nanoseconds per symbol)
Fault tolerance: 5
Bits: 1 ( Plain FSK )

## Behaviour
First 5 times the command is being sent with ~50ms intervals
Then seemingly a response is being sent with an "ACK" message type. There is however no participant ID showing that this is actually a response

## Protocol
After trimming the initial low signal (3 to 5 zero's), then the following structure is found in every message:

| **Purpose**       | **Position** | **Length** | **Example**                                                 |
|-------------------|--------------|------------|-------------------------------------------------------------|
|      Preamble     |    001-279   |  278 Bits  |                      010101....1010101                      |
| Sync / device ID? |    280-335   |   7 Bytes  | 00000010000000010000000110101000000011101011000111110111000 |
|    Message type   |    336-343   |   1 byte   |                           00010110                          |
|       Zero?       |    344-351   |   1 byte   |                           00000000                          |
|  Sequence Number  |    352-359   |   1 byte   |                           10110010                          |
|    UNKNOWN/TBD #1 |    360-431   |   9 bytes  |010001111110100100000111001101000000100011000001000000001000000011111111|
|    UNKNOWN/TBD #2 |    432-439   |   1 byte   |                           11000100                          |
|     Checksum.     |    440-447   |   1 byte   |                         CRC Checksum                        |
|     Postamble     |    448-465   |   2 bytes  |                     01010101...01010101                     |

### Preamble
This signal is used to help a receiver to configure its timing, and channel centering

### Sync / device ID
I assume that this part identifies the devices, both the remote and the fireplace always seem to send this same values for this block.

### Message type
| Message Type       | Decimal rep | Binary rep |
|--------------------|-------------|------------|
| Ack                | 9           | 00001001   |
| Secondary fire off | 11          | 00001011   |
| Secondary fire on  | 13          | 00001101   |
| Fire min           | 22          | 00010110   |
| Fire max           | 23          | 00010111   |
| Lights on          | 24          | 00011000   |
| Lights off         | 25          | 00011001   |

### Zero's
This part seems to always be zero - could be a return code / error code byte, or something else?

### Sequence number
This is a ever incremental/looping sequence number, each instance of the message, even when its repeated, has an increment of 1

### Unknown / TBD #1
Some bits and bytes here change by random sometimes per message / message type. I think that most of this information represents the current absolute state. More research is necesarry.

### Unknown / TBD #2
The first 3 bits here change every message, seems totally random, the other 5 bits often seem to have some correlation with the 5 repetitions of the request / response. This could very well have something to do with the current absolute state of the remote and the fireplace.

### Checksum 
The first byte of the CRC Checksum with the following algorithm: 

Parameters: 
| Setting      | Value       |
|--------------|-------------|
| Range        | 280-439     |
| Polynomial   | 0x8005      |
| Start/init   | 0x2e81      |
| Refin/refout | false/false |
| xorout       | 0x0000      |
| residue      | 0x0000      |
| check        | 0x879d      |
