# API of Voltcraft 3600 BT

## Initialisation and authorisation  

Before you can push commands to the smart meter it is required to synchronize time and authorize by using a secret key. 

```
Write Request Handle 0x018

03 e3 07 05 0a 14 1d 26 53 03
|  |  |  |  |  |  |  |  |  +  Byte 9: Secret key (byte 1)
|  |  |  |  |  |  |  |  + Byte 8: Secret key (byte 2)
|  |  |  |  |  |  |  + Byte 7: Seconds in hex 
|  |  |  |  |  |  + Byte 6: Minute in hex
|  |  |  |  |  + Byte 5: Hour of day in hex   
|  |  |  |  + Byte 4: Day in month in hex
|  |  |  + Byte 3: Month in hex, where 01 is January and 0C is December
|  |  + Byte 2: Year in hex (byte 1)
|  + Byte 1: Year in hex (byte 2)
+ Byte 0: Command, "03" for setting time
```

See blog [Voltcraft SEM-3600BT, Who needs security?](https://pushstack.wordpress.com/2018/01/25/voltcraft-sem-3600bt-who-needs-security/) in order to understand how the security code is calculated.

## Enable / Disable command notifications

The device does not send responses until you subscribe for notifications.

```
Write Request Handle 0x019

01 00
|  + always "0"
+ 1 = Enable,  0 = disable
```

## Power
### Turn power on

```
Write Request Handle 0x018
04 01
```

### Turn power off
```
Write Request Handle 0x018
04 00
```

## Schedulers
### Request scheduler setting
```
Write Request 0x0018

0e 00 00 05
|  |  |  + always "05"
|  |  + always "00"
|  + ID of timer to query, 00 - 05
+ Command, "0e" for quering time
```

### Interprete scheduler notification
```
Notification handle = 0x0018
value:
0e 00 00 82 81 02 03 04
0e id 00 dd ah mm hh mm
|  |  |  |  |  |  |  + End minutes
|  |  |  |  |  |  + Bit 8: Action (1 = turn on, 0 = turn off)
|  |  |  |  |  |    Bit 1 - 7: End hours
|  |  |  |  |  + Start minutes
|  |  |  |  + Bit 8: Action (1 = turn on, 0 = turn off), 
|  |  |  |    Bit 1 - 7: Start hours
|  |  |  + Activate: Bit 1: Sun 
|  |  |              Bit 2: Mon 
|  |  |              Bit 3: Tue 
|  |  |              Bit 4: Wed 
|  |  |              Bit 5: Thu
|  |  |              Bit 6: Fri
|  |  |              Bit 7: Sat
|  |  |              Bit 8: Active (1=active, 0=inactive)
|  |  + always "00"
|  + ID of scheduler, 0 - 5
+ "0e" notification for scheduler
```

### Set scheduler
```
Write Request 0x0018
0c 00 00 82 81 02 03 04
0c id 00 dd ah mm hh mm
|  |  |  |  |  |  |  + End minutes
|  |  |  |  |  |  + Bit 8: Action (1 = turn on, 0 = turn off)
|  |  |  |  |  |    Bit 1 - 7: End hours
|  |  |  |  |  + Start minutes
|  |  |  |  + Bit 8: Action (1 = turn on, 0 = turn off), 
|  |  |  |    Bit 1 - 7: Start hours
|  |  |  + Activate: Bit 1: Sun 
|  |  |              Bit 2: Mon 
|  |  |              Bit 3: Tue 
|  |  |              Bit 4: Wed 
|  |  |              Bit 5: Thu
|  |  |              Bit 6: Fri
|  |  |              Bit 7: Sat
|  |  |              Bit 8: Active (1=active, 0=inactive)
|  |  + always "00"
|  + ID of scheduler, 0 - 5
+ "0c" command for setting scheduler
```

ah - action and hours in hex, hours 0 - 23
     turn on: 128 + hh
     turn off: hh

### Reset scheduler

```
Write Request 0x0018
0c id 00 00 00 00 00 00
```

## Countdown

### Set countdown
```
Write Request 0x0018
06 ah mm
|  |  + minutes in hex, 0 - 59 
|  + action and hours in hex, hours 0 - 23
|    Bit 8: 1 = turn on, 0 = turn off
|    Bit 1 - 7: Hours
+ "06" for set countdown command
```

## Reset countdown

```
Write Request 0x0018
06 00 00
```

### Interprete countdown notification

If countdown is running and you have subscribed for command notifications, you get permanent - probably every second - a notification like this: 

```
Notification handle = 0x0018
value: 
06 ah mm
|  |  + minutes in hex, 0 - 59 
|  + action and hours in hex, hours 0 - 23
|    Bit 8: 1 = turn on, 0 = turn off
|    Bit 1 - 7: Hours
+ "06" for set countdown command

```

## Overload

### Set overload

```
Write Request 0x0018
15 40 b0 04
15 aa ww ww
|  |  |  + Watt (high byte)
|  |  + Watt (low byte)
|  + Bit 8: 1 = turn off
|    Bit 7: 1 = buzzer
+ "15" for overload command
```

### Reset overload

```
Write Request 0x0018
15 00 00 00
```


### Request overload setting

```
Write Request Handle 0x018
16
```

### Interprete overload notification

```
Notification handle = 0x0018
value:
16 40 b0 04
16 aa ww ww
|  |  |  + Watt (high byte)
|  |  + Watt (low byte)
|  + Bit 8: 1 = turn off
|    Bit 7: 1 = buzzer
+ "16" for overload notification
```

## Standby 

### Set standby

```
Write Request 0x0018
13 aa ww ww
13 00 00 00
|  |  |  + low mark (high byte), here 3.4 Watt
|  |  + low mark (low byte), here 3.4 Watt
|  + Bit 8: 1 = turn off
```

### Reset standby

```
Write Request 0x0018
13 00 00 00
```


### Request standby settings

```
Write Request 0x0018
14
```

### Interprete standby notification

```
Notification handle = 0x0018
value:
14 40 b0 04
16 aa ww ww
|  |  |  + Watt (high byte)
|  |  + Watt (low byte)
|  + Bit 8: 1 = turn off
+ "14" for standby notification
```


## Realtime measurement

### Enable / Disable realtime measurement notifications

In order to get realtime measurements you must subscribe for notifications.

```
Write Request Handle 0x013

01 00
|  + always "00"
+ 1 = Enable,  0 = disable
```

### Interprete realtime measurement notification

```
Notification handle = 0x0012
value:
01 03 23 85 01 00 34 01 42 77 01 05 18 02 49 97
ss uv vv vv ua aa aa uw ww ww up pp pp uf ff ff

ss - State
0 = off
1 = on
2 = countdown

uv, ua, uw, up, uf define count of digits before comma

1 -> 0.000
2 -> 00.00
3 -> 000.0
4 -> 00000
5 -> 0.000
other: 0.0

v - Voltage
a - Ampere
w - Watts
p - Power factor
f - Frequency

```
**Note** Measured data is just in decimal instead of hex!

## Take snapshot w/o authorization

```
gatttool -b D0:39:72:BB:AE:EC --char-read -a 0x15
Characteristic value/descriptor: 23 62 82 03 67 84 00 04 28 04 92 89 05 00 24 00 00 00 00
                                 23 52 57 00 00 00 00 00 00 10 00 00 04 99 87 00 00 00 00
                                 V  V  V  A  A  A  W  W  W  PF PF PF  f ff ff
```
see https://wiki.volkszaehler.org/hardware/channels/meters/power/wittech_witenergy_e100


## Recorded measurements on hourly base

### Request stored measurement

```
01 00 00 05
|  |  |  + Amount of records from starting hour to future (range from 1 to 8)
|  |  | Start of timeframe, byte 2
|  + Start of timeframe, byte 1, shift hours back from now (0 = latest full hour not current!)
+ Command, "01" for quering data on houry base
```


### Interprete stored measerment notification

```
Notification handle = 0x0018 value: 01 00 00 07 14 00 00 00 00 00 00 00 00 00 20 00 20 00
                                    |  |   | |  7     6     5     4     3     2     1  
                                    |  |   | |  |     |     |     |     |     |     + R7: 32 Wh
                                    |  |   | |  |     |     |     |     |     + R6: 32 Wh
                                    |  |   | |  |     |     |     |     + R5: 0 Wh
                                    |  |   | |  |     |     |     + R4: 0 Wh
                                    |  |   | |  |     |     + R3: 0 Wh
                                    |  |   | |  |     + R2: 0 Wh
                                    |  |   | |  + R1: 20 Wh
                                    |  |   | + Avail. records in this notification
                                    |  +---+ Requested Timeframe (2 bytes)
                                    + "01" Notification for recorded data on hourly base
```

## Recorded measurements on minute base

### Request stored measurement

```
02 00 00 05
|  |  |  + Amount of records from starting minute to future (range from 1 to 8)
|  |  | Start of timeframe, byte 2
|  + Start of timeframe, byte 1, shift minutes back from now (0 = latest full minute not current!)
+ Command, "02" for quering data on minute base
```

### Interprete stored measerment notification

```
Notification handle = 0x0018 value: 02 00 00 07 14 00 00 00 00 00 00 00 00 00 20 00 20 00
                                    |  |   | |  7     6     5     4     3     2     1  
                                    |  |   | |  |     |     |     |     |     |     + R7: 32 Wh
                                    |  |   | |  |     |     |     |     |     + R6: 32 Wh
                                    |  |   | |  |     |     |     |     + R5: 0 Wh
                                    |  |   | |  |     |     |     + R4: 0 Wh
                                    |  |   | |  |     |     + R3: 0 Wh
                                    |  |   | |  |     + R2: 0 Wh
                                    |  |   | |  + R1: 20 Wh
                                    |  |   | + Avail. records in this notification
                                    |  +---+ Requested Timeframe (2 bytes)
                                    + "02" Notification for recorded data on minute base
```

### Request power-on time 
```
char-write-req 18 18

Notification handle = 0x0018 value: 18 01 00 00
                                    |  +-----+ 3 bytes of power on time in minutes
                                    + "18" poewr-on time notification
```

### Request total consumption 
```
char-write-req 18 17
Notification handle = 0x0018 value: 17 00 00 00 00 00 00 00 00 00 00 00 00 34 15 00 00
                                                                           +---------+ 4 bytes for consumption in Wh
```

### Reset recorded data 
```
char-write-req 18 19
```

## Device masterdata

```
char-read-uuid 00002a00-0000-1000-8000-00805f9b34fb
handle: 0x0003   value: 57 69 54 20 50 6f 77 65 72 20 4d 65 74 65 72
=> WiT Power Meter

char-read-uuid 00002a25-0000-1000-8000-00805f9b34fb
handle: 0x0020   value: 53 4e 3a 20 30 30 30 30 30 30 00
=> SN: 000000

char-read-uuid 00002a26-0000-1000-8000-00805f9b34fb
handle: 0x0022   value: 46 2f 57 3a 20 56 30 31 2e 33 32 00
=> F/W: V01.32

char-read-uuid 00002a27-0000-1000-8000-00805f9b34fb
handle: 0x0024   value: 48 2f 57 3a 20 56 30 30 2e 30 30 00
=> H/W: V00.00

char-read-uuid 00002a28-0000-1000-8000-00805f9b34fb
handle: 0x0026   value: 53 2f 57 3a 20 56 30 31 2e 31 33 00
=> S/W: V01.13

char-read-uuid 00002a29-0000-1000-8000-00805f9b34fb
handle: 0x0028   value: 57 69 74 74 65 63 68 20 43 6f 6d 70 61 6e 79 20 4c 74 64
=> Wittech Company Ltd
```