# mertik-868-protocol
An effort to reverse engineer the wireless signal for Mertik Maxitrol 868MHz controlled fireplace 


## Signal interpretation
In Ultimate Radio Hacker, the following settings lead to a correct interpretation of the signals:

Frequency: 868.0MHz
Modulation: FSK (Actually its GFSK, but FSK works the same in URH)
Noise 0,08
Center 0,25
Samples per symbol: 62 (at 2MSps / 2MHz = 31 nanoseconds per symbol)
Fault tolerance: 5
Bits: 1 ( Plain FSK )

## Behaviour
First 5 times the command is being sent with ~50ms intervals
Then seemingly a response is being sent with an "ACK" message type. There is however no participant ID showing that this is actually a response

## Protocol
Strip first 3~5 0's, then the following protocol:

- 001 TO 279 = Preamble consisting of repeating 0101 sequence 
- 280 TO 338 = Sync / device id? Always the same for my setup
- 339 TO 343 = Message type
- 344 TO 351 = Always zero?
- 352 TO 359 = Sequence number / counter, always increases and reloops
- 360 TO 431 {..... UNKNOWN/HAVENT CRACKED YET, HELP IS APPRECIATED, current fireplace state???......}
- 432 TO 447 = Checksum -> {Havent figured it out yet, but looks like 16 bit OR mask )
- 448 TO 465 = End-of-frame / postamble of repeating 0101 sequence 

Schematically:
|              | Preamble      | Sync / device ID?                                          | Message type | Zero?     | Sequence Number | UNKNOWN/TBD | Checksum??     | Postamble   |
|:------------ |:-------------:|:----------------------------------------------------------:|:------------:|:---------:|:---------------:|:-----------:|:--------------:|:-----------:|
| **Position** | 001-279       | 280-335                                                    | 336-343      | 344-351   | 352-359         | 360 - 431   | 432-447        | 448-465     |
| **Length**   | 278 Bits      | 7 Bytes                                                    | 1 byte       | 1 byte    | 1 byte          | 9 bytes.    | 2 bytes        | 2 bytes     |
| **Example**  | 0101....10101 |00000010000000010000000110101000000011101011000111110111000 | 00010110     | 00000000  | 10110010        |             | totally random | 0101..0101  |

