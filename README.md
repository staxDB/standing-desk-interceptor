# standing-desk-interceptor
I wanted to intercept the communication between my Flexispot E5B standing desk buttons. This is the progress on the project.
It should work on any other flexispot desk as well, as i suspect they use the same button interface with nearly the same protocol. But this needs testing!

# RJ45 Pins
```
1 - RES
2 - SWIM
3 - ?
4 - WAKE
5 - RX
6 - TX
7 - GND
8 - VCC
```

# Using SWIM
The SWIM Debug interface is on the RJ45 Port, but not possible to use, due to set Read Protection, which can only be disabled by overwriting the Chip itself.

# Protocol
Every Command starts with 0x9b and ends with 0x9d. The second byte is the length of the Message with the Endbyte included. It seems like the Byte after the length is some kind of `messagetype`, because it stays constant for button presses, height configuration and different messages. 

| Start Byte | Length | Type | Payload | Checksum | End Byte |
|---|---|---|---|---|---|
| 9b | 06 | 02 | 00 00 | 6c a1 | 9d |
| 9b | 07 | 12 | 06 06 7d | 38 b7 | 9d |
| 9b | 04 | 11 | | 7c c3| 9d |
| 9b | 04 | 15 | | bf c2 | 9d |

The Checksum is a CRC16 Modbus Checksum over the `Length+Type+Payload`

## Button presses
A button press has the message type `0x02` and can be used as a fixed command. The checksum is steady and doesn't have to be recalculated.

Those are the known commands from the button controller to the motor controller with their respective button names.
```
commands = {
    'reset': '\x9b\x06\x02\x00\x00\x6c\xa1\x9d',
    'Mod2':  '\x9b\x06\x02\x08\x00\xac\xa6\x9d',
    'Mod1':  '\x9b\x06\x02\x04\x00\xac\xa3\x9d',
    'down':  '\x9b\x06\x02\x02\x00\x0c\xa0\x9d',
    'up':    '\x9b\x06\x02\x01\x00\xfc\xa0\x9d'
}
```

## Height
The height has the MessageType `0x12` and the payload is a simple 7 Segment display output.

As an example we look at the height answer: `9b 07 12 06 06 7d 38 b7 9d` which represents `116`

| P1 | P2 | P3 |
|---|---|---|
| 06 | 06 | 7d |

If we convert this to a regular 7 Segment Display:

|||||||
|---|---|---|---|---|---|
||a|a|a|||
|f||||b||
|f||||b||
|f||||b||
||g|g|g|||
|e||||c||
|e||||c||
|e||||c||
||d|d|d||dp|

Converting each byte to is binary representation and mapping each bit to a segment, we can generate the following table:

| Name | a | b | c | d | e | f | g | dp |
|---|---|---|---|---|---|---|---|---|
|P1|0|1|1|0|0|0|0|0|
|P2|0|1|1|0|0|0|0|0|
|P3|1|0|1|1|1|1|1|0|

Mapping this to the example above we can display the number `116` as a 7-Segment output. (don't want to include pictures, just believe me or try it out)

## Unkown MessageTypes
We don't know about the MessageTypes `0x11` and `0x15`. They are send by the motorcontroller.
They seem to be constant across different tables and don't bear any payload.
