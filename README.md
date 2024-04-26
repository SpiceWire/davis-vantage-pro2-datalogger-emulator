# Davis Vantage Pro2 Datalogger Emulator
How to flash a microcontroller to emulate a datalogger. 

Several years ago I received a Davis Vantage Pro2 weather station as a gift. It sat unused in my basement. I recently found that a local astronomy
group could use an on-site weather station to report realtime hyperlocal weather conditions to members. (e.g. "I want to view the comet but there's
scattered rain. Is it raining at the astronomy site right now?") I decided I could help by donating my weather station. It reports weather, right?

Sort of. The outdoor Davis weather sensors send data wirelessly to the indoor Davis console which displays it. An additional $225(USD) [datalogger](https://www.davisinstruments.com/collections/data-collection-1/products/weatherlink-windows-usb), which would be plugged into the bottom of the console, is required to 
export any data from the console for logging or sharing. A $400 USD newer model console could also do the task, maybe. Either way, it would cost $4-$8 [per
mobile device per year](https://www.davisinstruments.com/pages/weatherlink-app) to access the data through Davis software, which would be an unreasonalbe ongoing expense for a non-profit amateur astronomy group. I elected to find a 
low-cost workaround so that we could access and share the weather data.

## Access denied
I did a quick online search, found the pinouts for the console-datalogger plug, connected a 3.3 volt Serial-to-USB converter to the RX/TX pins 
on the bottom of the console, and used [PuTTY](https://www.putty.org/) for serial communication.  I referenced the console's [serial protocols](https://cdn.shopify.com/s/files/1/0515/5992/3873/files/VantageSerialProtocolDocs_v261.pdf?v=1614399559) and sent a serial message of "TEST". The console
replied with "NO".  In this case, "NO" did not mean "Normally Open", "Not On", "Next Operating" or even any oxide of nitrogen. It meant "NO", as in
"NO means NO."  As in, "NOt mentioned in the documentation."  My console refused to speak with its lawful owner. Grrr. 

## Access allowed
A deeper online search lead me realize that Davis had used security measures to prevent users from accessing the data collected by their own equipment. Also,
community collaboration bypassed these measures. [DeKay](http://madscientistlabs.blogspot.com/2011/01/davis-weatherlink-software-not-required.html) worked
at length to understand the security protocols.  [Watson](https://www.wxforum.net/index.php?topic=18110.msg200376#msg200376) provided a table of
values that could be flashed to a suitable chip in order to emulate the Davis datalogger.  Others provided printed circuit board plans for download or fabbing.

## Physical Descriptions:
The physical port of the console consists of 20 male pins spaced in two rows, 2 mm between each pin. This is a more narrow spacing than many PCB boards (2.54 mm) used in prototyping. Broadly, the Davis console has two separate circuits traveling through the port's pins. As designed, one "authentication circuit" queries a chip with the Davis security code flashed on it. On booting up, this circuit would tell the console "Yes, I'm an official Davis datalogger,"  and the console would use the second circuit for communication with the logger.  

## Parts I ordered and/or tried to use (see Procedures)
Atmel ATTiny85 SMD chip  
Female 2x10 2mm pitch header  
[Custom PCB](https://oshpark.com/shared_projects/M0mczaXC)  
PL2303 USB-UART cable  
100 nF capacitor

## Parts I actually used
Atmel ATTiny85 DIP chip  
Random wire harness I already owned  
3.3 volt Serial-to-USB converter I already owned  

## What you'll probably need
Hirose DF11 series connector [DF11-20DS-2C](https://www.mouser.com/ProductDetail/Hirose-Connector/DF11-20DS-2C?qs=Ux3WWAnHpjDwSYTyawRgYw%3D%3D)  


## Procedures 
Many different chips meet the requirements to emulate the logger's security chip, and many users found success using Atmel's ATTiny85 microcontroller chip. 
I ordered a [PCB board](https://oshpark.com/shared_projects/M0mczaXC) but was not able to source a surface-mount ATTiny85 chip at a reasonable cost to me. I used a larger 2.45 mm DIP ATTiny85 instead. The PCB board had the proper circuit and was very good quality, but the 2x10 female socket I used failed to engage the male pins, even when completely seated. Instead of trying several female socket types, I found I already owned a female 16 pin 2 mm pitch plug, already wired, and the other side of the harness had a 2.54 pitch plug that fit my 3.3 volt Serial-to-USB converter. The plug engaged the pins well, and the four male pins on the 2x10 header that were not engaged by the 2x8 female Molex were not essential to the circuits I needed.

I used the davis_random HEX file provided by [night303](https://www.wxforum.net/index.php?topic=18110.275) to flash the ATTiny85 chip. An Arduino Nano (CH340 chip clone) operated as the [In-circuit Serial Programmer (ISP)](https://docs.arduino.cc/built-in-examples/arduino-isp/ArduinoISP) with the recommended "heartbeat" circuit. To flash the chip, I  downloaded [AVRDUDE](https://github.com/avrdudes/avrdude) into its own folder, included the HEX file in the same folder, and used the following commands in the folder's command line interface: 
1) avrdude -F -c arduino -b 19200 -p t85 -P COM4 -n   
2) avrdude -F -c arduino -b 19200 -p t85 -P COM4 -U efuse:w:0xFF:m -U hfuse:w:0xDF:m -U lfuse:w:0xC1:m -U lock:w:0xFF:m
3) avrdude -F -c arduino -b 19200 -p t85 -P COM4 -U flash:w:davis_random.hex

The "COM4" specified my active COM port at the time I flashed the chip. The "t85" parameter specified the ATTiny85 chip. I did not use the chip or methods described in the [epic writeup by Torkel](https://meteo.annoyingdesigns.com/DavisSPI.pdf) because I did not own the equipment used. But thank you, Torkel, for your research and contributions.

## Outcome
The ATTiny85 was flashed successfully and I wrote backend and frontend code to view, store and search weather data.

The astronomy group? A simple VP2 weather station does not gather data on cloud cover. They really needed an expensive ceilometer (which measures cloud height) instead. So I'm keeping it.



