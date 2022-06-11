## Setting up RTK GPS on a Raspberry Pi

***In process: this content is being created.  It is not yet complete and it has not yet been tested as written.  If you see anything wrong or have a better/another way to do this please open an issue in the github repo.  Thanks.***

I want to race RC cars.  I don't want to actually drive them.  I'd probably suck at it.  I'm really pretty sure about that.  So they have to drive themselves.  There are a bunch a ways to get an RC to drive itself.  When you are outside gps is a good choice, but it has to be the right kind of gps and it must be setup the right way.  You also need software to read and interpret the data sent by the gps.  My automonous RC car uses the [DonkeyCar](https://www.donkeycar.com/) framework for small scale car racing, so I'm adding gps support to that framework.  An autonomous car also needs a computer to run the software.  DonkeyCar supports the Raspberry Pi or the Nvidia Jetson Nano as the brains of the car.  I've tried both, but one worked better than the other for this purpose.  More after this commercial break...

<p align="center">
  <img src="img/donkeycar.png" style="height: 50%; width: 50%;" alt="A DonkeyCar" />
  <p align="center">A DonkeyCar</p>
</p>

Just kidding, no one would sponsor me.  It turns out that the Raspberry Pi works a little better, but either can work.  I'll talk about both.

### Send in the Satellites
First lets talk about gps satellites.  The term GPS (Global Positioning System) is commonly used to refer to satellite positioning systems generally, but technically it refers to the to American system of positioning satellites provided as a service of the United States government.  Other systems have been added (generally also by governments).  The Russian government provides the GLONASS system; China provides the BeiDou system and the European Union provides the Galileo system.  There are other smaller regional systems, some based on a combination of satellites and land based installations.  Collectively these are often referred to as the Global Navigation Satellite Systems (GNSS) system.  The more the merrier; literally, if you get happy about coverage, accuracy and speed.  In this discussion I'll use lower case gps or GNSS to refer to these systems collectively and upper case GPS to refer specifically to the American system.

So the gps receiver (more below) gets radio signals from the gps satellites.  Each satellite is transmitting a very exact time and it's current position in space.  When the gps receiver gets a time and satellite position, it calculates how long it took the signal to reach the receiver and can then calculate the distance to the satellite.  With 3 satellites the receiver can triangulate it's position on the surface of the Earth.  The reality is a little more complicated because the receiver uses many satellites and the Earth is not flat (glad I got that off my mind), but you get the idea.

### NEO ZED Blox What?
Next, let's talk about the kinds of gps devices available for talking to the satellite systems.  Since the American GPS system was first early chips only talked to it.  Then when GLONASS came online chips came out that supported both GPS and GLONASS, even simultaneously.  Newer chips support GPS, GLONASS, BeiDou and Galileo and some of the smaller system.  The most popular devices are based on chips created by a company named [U-Blox](https://www.u-blox.com/).  They have lots of different GPS chips for various purposes and with varying capabilities.  Their latest chips are series 10.  You can still find series 6, 7, and 8 out there.  The series 8 chips are much better than 6 or 7; they support more kinds of satellites and more simultaneous satellite connections.  I'm using a device from Sparkfun with a series 9 chipset, the [SparkFun GPS-RTK-SMA Breakout - ZED-F9P (Qwiic)](https://www.sparkfun.com/products/16481).  The name really rolls off the tongue doesn't it?  Catchy.  Anyway, the ZED-F9P chipset is a big improvement over the NEO-8 series because it supports yet more satellite systems, more simultaneous connections and most importantly, it supports a second frequency band (L2) for higher precision measurements.  Newer is better.  Ask any baby.

<p align="center">
  <img src="https://cdn.sparkfun.com//assets/parts/1/5/3/5/2/16481-SparkFun_GPS-RTK-SMA_Breakout_-_ZED-F9P__Qwiic_-01a.jpg" style="height: 75%; width: 75%;" alt="SparkFun GPS-RTK-SMA Breakout - ZED-F9P (Qwiic)" />
  <p align="center"><a href="https://www.sparkfun.com/products/16481">SparkFun GPS-RTK-SMA Breakout - ZED-F9P (Qwiic)</a></p>
</p>

Other companies produce ZED-F9P based boards.  ArduSimple makes a number of different boards.  [This](https://www.mouser.com/ProductDetail/ArduSimple/AS-STARTKIT-BASIC-L1L2-NH-02?qs=GBLSl2Akiru%2FUw8RAcKyUw%3D%3D)  one includes the antenna and costs a little less than the SparkFun board without the antenna.

<p align="center">
  <img src="https://www.mouser.com/images/ardusimple/hd/AS-STARTKIT-BASIC-L1L2-NH-02_SPL.jpg" style="height: 50%; width: 50%;" alt="ArduSimple AS-STARTKIT-BASIC-L1L2-NH-02" />
  <p align="center"><a href="https://www.mouser.com/ProductDetail/ArduSimple/AS-STARTKIT-BASIC-L1L2-NH-02?qs=GBLSl2Akiru%2FUw8RAcKyUw%3D%3D">ArduSimple AS-STARTKIT-BASIC-L1L2-NH-02</a></p>
</p>

What is it with these product names?  Anyway, U-Blox isn't the only game in town; you can find inexpensive GPS modules that use other chipsets, often from China.  In many ways these can work interchangeably with U-Blox based boards because they all support the a standard way of reading data from the satellites, call NMEA sentences.  This is a line-oriented text protocol.  So if your software can parse NMEA sentences then it can work with most gps modules.  

U-Blox does have an ace up its' sleeve relative to other chipsets; it provides a piece of software called [U-Center](https://www.u-blox.com/en/product/u-center) that can be used to configure and operate U-Blox based devices.  It allows you to optimize the way the chipset works for how you want to use it, within limits.  U-Center is written for Microsoft Windows, but it can be run on Macintosh or Linux using the Wine emulation system.  I've successfully done this on Macintosh.  I'll add a section near the end that describes how to use U-Center and how to set it up on Macintosh.

<p align="center">
  <img src="img/u-center.png" style="height: 75%; width: 75%;" alt="U-Center" />
  <p align="center">U-Center</p>
</p>

### How accurate is gps?
Good question.  Here is the crappy answer: it depends.  Let's just assume you have a great view of the sky; then [GPSInsight](https://help.gpsinsight.com/deep-dive/how-accurate-is-gps-technology/) suggests that gps should yield an accuracy of 2meters CEP (Circular Error Probability); that means that 50% of readings will fall within a circle with a radius of 2m.  The other 50% will be outside.  This is a bit of a weird way to measure accuracy of gps; this is because CEP assumes a Guassian error distribution.  However, if you make a whole bunch of readings over a period of time while staying in one place, you will notice that the errors are not randomly distributed around a center; they are more like a random walk.  In fact, the is error not really random; much of it is caused by atmospheric changes; these are not really random, but instead cause the position to look like it is moving on a path over time, even though you stay in one place.  Here is an chart I created using the gps code I created for DonkeyCar.  It represents 1 sample per second for 100 seconds;

<p align="center">
  <img src="img/random_walk.png" style="height: 75%; width: 75%;" alt="Gps position readings over time while staying in place; 1 sample per second for 100 seconds" />
  <p align="center">Gps position readings over time while staying in place; 1 sample per second for 100 seconds</p>
</p>

These positions are in [UTM](https://en.wikipedia.org/wiki/Universal_Transverse_Mercator_coordinate_system) units; essentially the position in meters on the surface of the earth.  You can see the readings span about 5 meters vertically and 8.5 meters horizontally.  If we averaged all of these points we would get one measure of the center (the centroid); then we could find the 50 points closest to this center; that would be the CEP.

But the important point is that these are not randomly arranged around the center; instead the position reading is moving around over time like it is walking.  This is because of atmospheric affects, perturbations in satellite orbits and other affects that gradually change over time and have the overall effect of making the positiion reading 'drift'. 

Think about this if you are relying on gps readings in order to know where your autonomous car is located; beyond the fact that readings are +/- 4 meters from center, at any given time if you take a few readings it may appear you are quite near a given point, but that is a false assumption. If you keep taking readings and average them, your position will get more accurate, but not in the same way it would if the error was a Guassian distribution around a center.  

Here are 1000 readings taken with an U-Blox NEO M8N with an active GPS/GNSS antenna and a good ground plane mounted on a tripod (basically, a good setup; more on antennas and ground planes later).  

<p align="center">
  <img src="img/m8n_tripod_base_1000_samples.png" style="height: 75%; width: 75%;" alt="U-Blox NEO M8N position readings over time while staying in place; 1 sample per second for 1000 seconds" />
  <p align="center">U-Blox NEO M8N position readings over time while staying in place; 1 sample per second for 1000 seconds</p>
</p>

The points are within A 5 meter range in each direction.  So that is better than the first figure, which was done with a non-active antenna and no base plane.

Next, this is 1000 readings with a U-Blox ZED F9P, an active GPS/GNSS antenna and a good ground plane mounted on a tripod, the same setup as the last set of readings.  
<p align="center">
  <img src="img/fp9_tripod_base_1000_samples.png" style="height: 75%; width: 75%;" alt="U-Blox ZED F9P position readings over time while staying in place; 1 sample per second for 1000 seconds" />
  <p align="center">U-Blox ZED F9P position readings over time while staying in place; 1 sample per second for 1000 seconds</p>
</p>

The points are within 2.5 meter range vertically and 1.25 meters horizontally.  So that is a even better than the NEO-N8M.  This shows the series 9 chipset is superior to the series 8.  But still, this is not accurate enough for navigating a short RC course where the lane may be less than 2 meters wide.  Also, we can't take 100 or 1000 readings at each position in order to get an average; we are racing!  We need multiple readings per second so we can see where the car is as it moves.  Crap, we are our of luck I guess.  But wait, there is more!

### Kinematifanatics
What if you could make readings that were more like 2 centimeters CEP?  What if you could do that 10 times per second while you were moving?  What if it was cheap as hell?  Well, forget that last one, but the other two are possible by using RTK gps.  [Sparkfun](https://learn.sparkfun.com/tutorials/what-is-gps-rtk/all) describes RTK gps, "RTK is short for real time kinematics. A GPS receiver capable of RTK takes in the normal signals from the Global Navigation Satellite Systems. On top of these signals an RTK receiver takes in an RTCM correction stream and then calculates your location with 1cm accuracy in real time."

What are these magical 'corrections' they speak of and who conjures them?  Corrections are created by Earth-bound base stations and then transmitted to the gps receiver.  Think about corrections this way; a gps receiver uses the time stamps and orbital positions it gets from the satellites to triangulate it's otherwise unknown position on the Earth.  But the base stations already know their position; their position has been very, very, very, very carefully measured.  Base stations already know where they are, but they can also talk to the satellites and do the triangulation thing.  If the triangulation calculation yields an incorrect position, then base station can calculate the difference between where it knows it is and where triangulation says it is; that is a correction.  The base station can then send that offset to our gps receiver, then our gps receiver can apply the offset after it triangulates its position using the satellites, thus making the result much more accurate.  This works if you are pretty close to the base station that is broadcasting the corrections (say within about 35km) and if you apply very recent corrections.  This make sense if you think about the things that cause drift; if you are close to the base station then you are able to see the same satellites and you are subject to similar atmospheric conditions.  You need recent corrections so that not much drift has occurred between calculating the correction and applying the correction. 

For reals, the way corrections are calculated is much more complicated than how I have explained it, but you get the idea.

### Please correct me
There are many organizations that maintain base stations and provide corrections over the Internet.  The corrections are typically broadcast over the internet using a protocol called [NTRIP](https://www.ardusimple.com/rtk-explained/); in this case the server is called an NTRIP Caster.  The actual corrections are commonly in a format called RTCM3. You typically must sign up for a free account to get access to the NTRIP caster servers.  I have at least 4 base stations in the [UNAVCO](https://www.unavco.org/instrumentation/networks/status/all/realtime) network near me.  There is also a network maintained by individuals good enough to share their gps data for free; [RTK2Go](http://rtk2go.com/).  Resources are changing and improving all the time; check this global list of organizations that host NTRIP servers; [rtcm-ntrip.org](http://rtcm-ntrip.org/home.html).

You basically find an organization that covers your country or region and lookup their list of servers and locations.  If they have one with 35km of you then you are in luck; sign up with them for an account to get access to the servers, then configure your NTRIP client sofware using the information they provide (I tell more about client software below).


### All our base are belong to you

What if you don't have a fixed base station near enough to you?  That's simple; create your own.  It's possible to use a second RTK gps receiver as a base station.  One issue with that is that your base station does not have a very, very, very, very carefully measured position.  However, you can let the base station sit there for a while and get lots and lots and lots of readings so it calculates a very good estimate of its position.  Good enough for our needs.

[Sparkfun](https://learn.sparkfun.com/tutorials/setting-up-a-rover-base-rtk-system/all) has a very detailed walkthrough on how to setup your own RTK base station.  


### Client Tell
Ok, so we have a source of RTCM3 corrections, maybe sent over the Internet using the NTRIP protocol, but our gps receiver doesn't know anything about the Internet; it has serial ports (how 1980s) for communication.  How do we get the RTCM3 corrections to the gps receiver's serial ports?  Well we first need an NTRIP client to receive the RTCM corrections, then we write those corrections to a serial port on the gps receiver board either using a direction connection to the RaspberryPi/Jetson Nano or via a radio connected a ZED-F9P serial port.  

The ZED-F9P has several serial ports.  There is a serial port associated with the USB connection; this is very easy to use.  There are two other serial ports to which header pins can be soldered (so less easy).  Remember that picture of the Sparkfun-ZED-F9P-stupid-long-name above; look for the connections marked RX2/TX2; those are the receive/transmit pins respectively of the serial ports.  This is clearly explained in the [Sparkfun hookup guide](https://learn.sparkfun.com/tutorials/gps-rtk2-hookup-guide).  It also explains about antennas and ground plans and other setup stuff, including connecting to a corrections source, but we'll talk about that here as well.

So receiving the corrections; our Donkeycar has a RaspberryPi (or a Jetson Nano) and that has Wi-Fi.  So if the car is close enough to a Wi-Fi hotspot then it can use an NTRIP client to read the RTCM3 corrections from the base station over the internet, then send the corrections to the gps receiver over a serial connection.  If the car is not consistently close enough to a Wi-Fi hotspot (which it probably won't be), then you could use cellular data; I taped my phone to my car and put it into hotspot mode, then connected the RaspberryPi Wi-Fi to the phone's hotspot so it could access the Internet while it was driving. 

A RaspberryPi also has Bluetooth and so does our phone.  So another way to get corrections using your phone is to get an NTRIP client mobile app on the phone and use it to get the corrections from a base station, then transmit the corrections to the RaspberryPi using Bluetooth serial.  The RaspberryPi would then forward the corrections to the gps receiver over a serial connection.  

But what if you have to use your own base station rather than a publicly available NTRIP server?  That might actually be the easiest scenario, but the most expensive.  In that case need a second RTK gps device to act as the base station; a device that can create RTCM3 corrections.  Then we still have the problem of getting the corrections from the base station to the gps receiver.  We could go through the RaspberryPi; maybe connect a Bluetooth radio to the serial output on the base station and have it received by the RaspberryPi's Bluetooth, then the RaspberryPi can forward it on to the gps receiver via a serial connection.  Alternatively we can cut the RaspberryPi out of the picture altogether; we can connect a radio transmitter of some sort to the base station's serial port; the base station transmits the RTCM3 connections to the serial port and so out to the radio.  On the car we then need a compatible radio connected to the gps receiver's serial port on the car, so the corrections are received by the radio and input to the gps receiver via the serial port to which the radio is connected.  That is a common way to do this.  There are various radio technologies that could be used, but the best is LoRa.  LoRa radios have long range, which is good for us because it can cover a large track.  Further, a LoRa transmitter can be configured in broadcast mode so many LoRa receivers can listen for the data at the same time.  That works really well at a track with multiple competitors.

## The cold hard facts
Ok, this is the section (maybe sections) where I'll give detailed instructions on how I accomplished getting RTK gps to work on a Donkeycar.  Hopefully it will help you avoid a bunch of the experimentation I had to go through and make things faster for you.  First a pretty picture to set the mood.

<p align="center">
  <img src="img/donkeycar_gps.jpg" style="height: 75%; width: 75%;" alt="DonkeyCar with Sparkfun RTK GPS" />
  <p align="center">DonkeyCar with Sparkfun RTK GPS</p>
</p>

So much blue tape.  Anyway, the picture shows a DonkeyCar with a RaspberryPi connected to a Sparkfun ZED-F9P RTK GPS board via USB and GPIO serial.  In this case it is using an extra hardware backed UART (uart3) mapped to pins on the gpio bus.  This was required for reasons explained later.  So in this setup we want to get corrections into the RaspberryPi, either using RTKLIB or via an Android NTRIP client, and then send them via a serial port to the ZED-F9P. The following sections describe how to setup the software to do that.

### RTKLIB
RTKLIB is an open source software package written by Tomoji Takasu, that can use GNSS raw data to run real-time or post-processing solutions to accurately determine relative position using differential information from two receivers (RTK/PPK).  This has way more capability than we will be using.  We really only need one of it's command line utilities; str2str.  str2str can connect to an NTRIP server and then write the corrections to up to 3 outputs.  I can convert formats if that is necessary; so it could take in RTCM2 corrections, but output RTCM3 corrections.

- http://www.rtklib.com/
- [RTKLIB github](https://github.com/tomojitakasu/RTKLIB)
- [RTKLIB Demo5; rtklibexplorer RTKLIB fork](https://github.com/rtklibexplorer/RTKLIB)

#### Setup RTKLIB NTRIP client on RaspberryPi/Jetson Nano
We can build RTKLIB on the nano and use it to request corrections over the internet and send them to a serial port.  We only need the str2str command line application, so we will only build that.  

You must have the `GCC` compiler and the `make` build utility available.  These are pretty much always available on a Linux system.  However for MacOS you must install the XCode toolset; so first, open the Mac App Store and install Xcode for free. Then, open Xcode, go to Xcode menu (on the menu bar) > Preferences > Downloads, and install Command Line Tools. You will get commands like gcc, make, purge...  Windows is an inhospitable place for gcc, but you can give it a try.  Here is a way to install GCC; https://www.freecodecamp.org/news/how-to-install-c-and-cpp-compiler-on-windows/  Another ways to try this on windows is to install Linux Subsystem For Linux from the Windows store; then you have a nice linux shell as part of Windows.  

Once you have a working gcc and make, just clone the repo, checkout the 2.4.3 branch and build str2str.  The executable will end up in the gcc folder.

```
git clone https://github.com/tomojitakasu/RTKLIB.git --branch rtklib_2.4.3
cd RTKLIB/app/consapp/str2str/gcc
make
```

If you have the gps board connected to the computer then you can send the corrections via USB serial.  It is also possible to connect a serial port on the Sparkfun F9P SMA board directly to the serial port exposed on the RaspberryPi/Jetson gpio bus. In our final setup we will need to send corrections to one serial port and read NMEA sentences from another, so we will need both the USB and the gpio serial ports. The general format of the command to output RTCM3 a serial port is:
```
./str2str -in ntrip://USER:PASSWORD@CORS_IP:2101/MOUNT_POINT -b 1 -out serial://<device>:<baud>:8:n:1
```

RTKLIB manual is at http://www.rtklib.com/prog/manual_2.4.2.pdf  It has more details.

Once you have build str2str, try to use it to get RTCM corrections from an NTRIP server and output them to the console to make sure you account credentials are working.  In this case it is using a UNAVCO server in Sonoma California:

```
./str2str -in ntrip://USER:PASSWORD@rtgpsout.unavco.org:2101/P200_RTCM3
```

If that works then your credentials and server address are correct.  

See this JetsonHacks [video](https://jetsonhacks.com/2019/10/10/jetson-nano-uart/) for using the serial ports on the Jetson Nano GPIO header. To output to Jetson Nano's serial port you must have permissions to read/write the serial port.  Run this command to add your user to the dialout group.
 ```
 sudo usermod -aG dialout $(whoami)
 ```

The serial UART in the 40pin GPIO header is setup to allow login via serial by default.  If you intent to use that port for sending RTCM corrections to the gps board, you must stop that service before we can use the serial port for other purposes. 
```
systemctl stop nvgetty
```

To stop this from restarting on every boot, permanently disable after stopping you can do this.  Note that this kills the ability to `ssh` into the nano, so you may NOT want to make this permanent.

```
systemctl disable nvgetty
udevadm trigger
```

Now you can run str2str and give it the serial port as an output destination.  The Sparkfun F9P breakout board's RTCM correction port (UART2) defaults to 38400bps.  We need to make the serial port on the Nano that we are transmitting from match that baud rate.
```
sudo stty -F /dev/ttyTHS1 38400
```

Make sure you are in the gcc folder where we built the str2str application.  Run the application and specify the ntrip server as the input and the serial port as the output.  This example reads the corrections from a UNAVCO NTRIP server and sends them to a serial port using the default baud rate for the F9P serial ports.
```
./str2str -in ntrip://USER:PASSWORD@rtgpsout.unavco.org:2101/P200_RTCM3 -out serial://ttyTHS1:38400:8:n:1
```

As we will see later, it is possible to use U-Center to configure the F9P serial port to use a higher baud rate.  We will want to do that.  When we do that then we will also want the Nano's serial port to use that higher baud rate before connecting to str2str.  Here is a command to set the serial port to 115200 baud.  
```
sudo stty -F /dev/ttyTHS1 115200
```

Note that you do not have to do this for the USB serial port; it can autodetect the baud rate.  Just make sure the baud rate specified in the str2str command matches the baud rate of the F9P serial port.  For instance, if we set the F9P serial port to 230400 using U-Center and we want to send corrections to it from the computer's USB serial port:
```
./str2str -in ntrip://USER:PASSWORD@rtgpsout.unavco.org:2101/P200_RTCM3 -out serial://ttyUSB0:230400:8:n:1
```

Note that you could _also_ output to a file to capture the data or to help in debugging.  str2str can write to to up to 4 -out destinations.
```
./str2str -in ntrip://USER:PASSWORD@rtgpsout.unavco.org:2101/P200_RTCM3 -out ./rtcm.txt -out serial://ttyTHS1:38400:8:n:1 -p 38.23983 -122.45170 -25.074
```

Note the `-p` longitude latitude altitude arguments.  Normally a base station sends it's exact position as part of the RTCM stream.  Some servers might not OR you might be using your own server and need to specifiy it's exact location this way.

The output of str2str looks like this when it first starts
```
$ ./str2str -in ntrip://USER:PASSWORD@rtgpsout.unavco.org:2101/P200_RTCM3 -out serial://ttyTHS1:38400:8:n:1
stream server start
2022/05/06 22:53:40 [WC---]          0 B       0 bps (0) connecting... (1) /dev/ttyTHS1
2022/05/06 22:53:45 [CC---]       1477 B    2518 bps (0) rtgpsout.unavco.org/P200_RTCM3 (1) /dev/ttyTHS1
2022/05/06 22:53:50 [CC---]       3052 B    2514 bps (0) rtgpsout.unavco.org/P200_RTCM3 (1) /dev/ttyTHS1
2022/05/06 22:53:55 [CC---]       4627 B    2517 bps (0) rtgpsout.unavco.org/P200_RTCM3 (1) /dev/ttyTHS1

```
These are not the corrections; they are logging statements emitted by the running application. The `[WC---]` box is for the status of connections.  The first letter is for connection (0), the input connection, subsequent letters for for up to 4 outputs.  
- W = waiting for connection
- C = connected
- E = error

For both the RaspberryPi and the Jetson Nano there are 3 pins that will want to connect to the F9P if we want to use the gpio serial port to send corrections from the computer to the gps board:
- pin board 6 is ground
- pin board 8 txd; tty transmit
- pin board 10 is rxd; tty receive

Note that we have NOT connect any positive voltage.  There is no need; the F9P board will be powered when it is connection to the USB port, so we do not need to power it through the serial pins.

### Lefebure NTRIP Client for Android
Lefebure offers a free NTRIP client and NMEA data logger for Android. Your phone connects to the NTRIP server and then sends the RTCM corrections via a Bluetooth SPP connection to the gps receiver if the receiver has an attached bluetooth radio or to the RaspberryPi, which would then forward to the gps receiver via RTKLIB str2str.  

- [Lefebure](http://lefebure.com/software/android-ntripclient/) 
- [Google Play Store](https://play.google.com/store/apps/details?id=com.lefebure.ntripclient)

#### Configure Android NTRIP Client

- Install Lefebure NTRIP client for android from the android store or from http://lefebure.com/software/android-ntripclient/
- Configure the NTRIP source on Lefebure NTRIP client
    - Configure the NTRIP settings to connect to your NTRIP server.  Note that the interface wants an ip address, not a url.  You can usually get the ip address either from the orgnanization's website, or just run the `ping` command with the server's url and it will usually yield the ip address.
        ```
        $ ping rtgpsout.unavco.org
        PING saturn.unavco.org (69.44.86.36): 56 data bytes
        Request timeout for icmp_seq 0
        ```
    - See how it resolved `rtgpsout.unavco.org` to the ip address `69.44.86.36`. 
- Once you setup the caster, you can connect in 'test' mode (no receiver) to see if the client will connect.
- Configure the receiver settings.  You may need to do this _after_ pairing the RaspberryPi and the phone (steps 1, 2 and 3 below) so that the raspberrypi device can be selected in the `Bluetooth Device` field.
    - `Receiver Connection` should be 'External via Bluetooth'
    - `Bluetooth Device` should be `raspberrypi'
    - Leave the other fields to the defaults.
    

#### Pair the RaspberryPi and Android Phone over Bluetooth

The bluetooth pairing (steps 1, 2 and 3 below) needs to be done only once on the Raspberry Pi to set it up to connect to the Android Bluetooth.  The process was adapted from what I found in this video: https://www.youtube.com/watch?v=sY06F_sPef4

1. Setup Raspberry Pi for serial over bluetooth (SPP).  Add the SP profile to the Pi; edit `/etc/systemd/system/dbus-org.bluez.service`.  In the editor, 
add the compatibility flaf `-C` to the ExecStart line.  If necessary, add a second line that starts the serial port (SP) service. NOTE: if you don't want the serial port service to start on every reboot, then leave this line out, in which case you will need start the serial port service manually before connecting the phone (see step 4 below).  When done editing; save the file.  The two lines in your `dbus-org.bluez.service` configuration should like this when you are done editing.   When satified reboot the RaspberryPi so this change takes affect.
    ```
    ExecStart=/usr/lib/bluetooth/bluetoothd -C
    ExecStartPost=/usr/bin/sdptool add SP
    ```

2. Use the bluetoothctl command to pair and trust the phone
    - Put the Android phone into bluetooth pairing mode
    - Open a command prompt and start the bluetooth control application by entering `bluetoothctl`. You will be placed in the bluetoothctl shell.
    - Type `agent on` and enter to run the `agent on` command.
    - Run the `default-agent` command.
    - Run the `scan on` command to start looking for devices.  It may take a minute, but you should see the device appear; take note of the 48 bit MAC address that is displayed; it will look something like `00:4d:e9:79:eb:a6`
    - Run `scan off` once you have this information.
    - Run `pair <MAC>`, replacing <MAC> with the device’s MAC address.
    - Once paired, in the future you just have to connect, not pair.  So in the future just run `connect <MAC>` if the connection does not happen automatically.
    - Run `trust <MAC>` so we don't have to do this again in the future.

3. Test the pairing.
    - You should still be in the bluetoothctl shell. Run `paired-devices`. You should see the Android phone in the list of paired devices.
    - Run quit to exit the bluetoothctl shell.
    - You should now be back at the command prompt.

#### Send corrections from the Android NTRIP client to the RaspberryPi
1. Now that RPi and your phone are paired, connect the NTRIP app.
    - On the RaspberryPi, if you did not configure the serial port service to start on every reboot (see step 1 above), then start it now; 
        - Run `sudo sdptool add SP` at the command prompt.  You should get “Serial Port service registered”
    - Start the RaspberryPi listening for a connection on bluetooth serial so it will automatically connect when the Android phone is available.
        - Run `sudo rfcomm watch hci0`.
    - In the Lefebure NTRIP app, choose `Connect`.
    - On the raspberrypi you should see should it connect on a  bluetooth serial port (`/dev/rfcomm0` in the example below):
    ```
    pi@raspberrypi:~ $ sudo rfcomm watch hci0
    Waiting for connection on channel 1
    Connection from 00:3D:E8:89:EA:B6 to /dev/rfcomm0
    Press CTRL-C for hangup
    ```
   
2. Now we can run RTKLIB str2str in a separate command console to route the incoming NTRIP to the F9P (in the case below, via the second hardware serial port that has been enabled on the GPIO bus).
    ```
    ./str2str -in serial://rfcomm0:115200:8:n:1 -out ./rtcm.txt -out serial://ttyAMA1:115200:8:n:1
    ```

## The Hardware: An Unexpected Journey

<p align="center">
  <img src="img/one_does_not_simply_stream.jpg" style="height: 75%; width: 75%;" alt="One does not simply stream RTCM3 to GPIO serial." />
</p>

The plan at the start was to stream the RTCM3 corrections from the RaspberryPi or Jetson Nano's default GPIO serial port to the ZED-F9P's corrections port (F9P UART2).  So the first thing that was needed was to solder the headers to the F9P so we can make physical connections using dupont jumper wires.  I'm not going to go into that; there are a lot of videos on YouTube on how to solder headers and what you need to do it.  My only suggestion is to practice a LOT before you try to solder headers to your $300 gps board.  And have some flux ready.

Once that is done the wiring was connected from the computer's default GPIO serial port on board pins 8&10 to the corrections serial port on the F9P.  By default the ZED-F9P corrections port is configured to read RTCM3.  The reason I started this way was because 'default' is a good word.  Look at all those defaults.  Nice. Or so I thought.

Wiring Connections
- F9P-UART2 is the corrections port on the Sparkfun GPS-RTK-SMA Breakout ZED-F9P
  - This port is setup to input UBX, NMEA and RCTM3
  - This port is setup to output RCTM3
- F9P-USB port
  - This port is setup to input and output UBX, NMEA and RTCM3

GPIO on the 40 GPIO bus on the SBC (RaspberryPi or Jetson Nano)
- Board pin 6 is GND
- Board pin 8 is TX for uart2
  - On the Jetson Nano the default spio serial port is `/dev/ttyTHS1`.
  - On the RaspberryPi 4 the default gpio serial port is  `/dev/ttyAMA1`.
- Board pin 10 is RX for uart2

```
RPi/Nano            ZED-FP9   
-----------------------------
Pin-6        <--->  GND
Pin-8, TXD   <--->  UART2-RX
Pin-10, RXD  <--->  UART2-TX
USB          <--->  USB
```

Software setup (TLDR version)
Use u-center to set the FP9 UART-2 to 115200 baud
Set the RPi/Jetson Nano serial port to 115200 baud
`sudo stty -F /dev/ttyTHS1 115200`
Run str2str on the RPi to get corrections from UNAVCO or your NTRIP server and send them from the default gpio serial port to to the FP9 corrections port.
`./str2str -in ntrip://USERNAME:PASSWORD@rtgpsout.unavco.org:2101/P200_RTCM3 -out serial://ttyTHS1:115200:8:n:1`
Read NMEA messages on USB serial port, /dev/ttyACM0

The bad news is that with this setup I never get to float mode on the ZED-F9P using either the RaspberryPi or the Jetson Nano; the `RTK` LED on the Sparkfun ZED-F9P stayed solidly lit.  Let me explain (isn't that what this whole article is for?); the ZED-F9P goes through stages as it develops a more accurate position estimate.  This is indicated by an LED on the Sparkfun ZED-F9P board labeled `RTK`.  When it is solid it is not using any corrections and so the position estimate is at its least accurate.  When it starts using corrections, but before most of its satellites are corrected, it is in 'float' mode.  In this mode the `RTK` LED blinks.  Float mode is more accurate, but more importantly it indicates that RTCM3 corrections data is getting to the ZED-F9P.  If you have a good view of the sky and you never get to float mode it probably means the RTCM3 corrections are not getting to the gps board correctly.  Of course if you are in your basement then it doesn't matter if the corrections make it to the board; you can't see any satellites!  If you get to float mode then the next stage you are hoping for is 'fixed' mode.  In this mode the F9P can apply corrections to most of the satellites it can see and so its position estimate is optimal.  In 'fixed' mode the `RTK` LED goes off.

So the bad news is that sending RTCM3 corrections from the RPi or Nano using the default gpio serial port does not seem to work.  But there is some good news.  I got to 'fixed' on the Jetson Nano by streaming RTCM3 to the USB serial port and reading the NMEA from UART2.  Even better news, In 100 samples I got a range of 2.5 cm.  Notice the 'c' before that 'm'.  If it ain't broken, it's fixed and fixed is good.

This is what worked (note that I have increase the baud rate of the ZED-F9P corrections port to 115200 baud using U-Center, more on that later);
```
./str2str -in ntrip://USERNAME:PASSWORD@rtgpsout.unavco.org:2101/P200_RTCM3 -out serial://ttyACM0:115200:8:n:1
```
- Then read NMEA messages from `/dev/ttyTHS1` (or `/dev/ttyAMA0` on the RPi)
- On the Jetson Nano, remember to stop the `nvgetty` login service does it does interfere with the serial port.

So this is good if I am reading corrections over the internet; I can read them from the internet and send them to the ZED-F9P via USB serial, then I can read the super awesome accurate NMEA sentences using the gpio serial port.  However, in the field (like at a race track) I will not always be getting RTCM3 corrections over the internet; I might not have fixed internet.  I may have access to a local base station which means I will need a radio, like a LoRa radio, attached to a ZED-F9P serial port to receive corrections from the local base station.  It is common to connect the radio directly to the corrections serial port on the ZED-F9P, in which case I would not even need to send corrections from the Jetson Nano; I would get them directly from the radio.  That would allow me to read NMEA via USB.

However, if I am in a place where I want to use an NTRIP server, but I don't have good fixed internet, then I will need another option.  There are a few I could use;
Use an Android NTRIP client on my phone. pair my phone with the RaspberryPi or Jetson Nano via Bluetooth.  The Android NTRIP client receive the corrections over the mobile internet and then sends the corrections to the RPi/Nano over bluetooth serial.  Then we can forward the corrections, using str2str running on the RPi/Nano, over the USB serial to the ZED-F9P.  The RPi/Nano can read NMEA using the GPIO serial connected to the ZED-F9P corrections port.
We could get real hacky and tape our phone to the Donkeycar.  Turn on the phone's mobile hotspot, have the RPi/Nano connect to it via Wi-Fi, then stream corrections for you favorite NTRIP server via the mobile internet.  Beyond the hotspot setup all the software setup and hardware connections are the same as if streaming directly over fixed Wi-Fi..

##### Where art UART oh RPi? 
I have found that UART on the GPIO bus (gpio 14&15 exposed on board pins 8&10 respectively) to be unreliable; it is not based on a hardware UART and so is more prone to issues at higher speeds.  By default the RPi uses two UARTs; one for the bluetooth functionality and one for the GPIO serial port.  The BT uart uses underlying hardware and so can handle very high baud rates.  The UART exposed on the GPIO bus is less capable.  Luckily the RPi 4 actually has 6 hardware-based UARTS. We can expose these by using device-tree overlays.  The device tree is data that is loaded at boot time that tells Linux which devices are attached and so which drivers to load.  A device tree overlay (dtoverlay) is a way of adding to the device tree without having to create a whole new one.  RasberryPi OS has a bunch of available dtoverlays that can be used to turn on optional features.  We can list the ones that have to do with UARTs (see this RPi forum discussion)

$ dtoverlay -a | grep uart
  midi-uart0
  midi-uart1
  miniuart-bt
  uart0
  uart1
  uart2
  uart3
  uart4
  uart5



Add this in the /boot/config.txt in the [pi4] section (you will need to sudo) to enable uart3 using defaults, then reboot.

# Enable UART3 on gpio 4=tx & 5=rx, no flow control pins
dtoverlay=uart3


Once that is done, then the serial device /dev/ttyAMA1 can be used to control the UART exposed on gpio 4&5, which are exposed on the 40 pin bus on board pin 7 and board pin 29 respectively.

Connections when using the RaspberryPi UART3

```
RPi            <--->  FP9   
--------------       ----------
Pin-9  (GND)   <--->  GND
Pin-7  (GPIO4) <--->  UART2-RX
Pin-29 (GPIO5) <--->  UART2-TX
USB            <--->  USB
```

<p align="center">
  <img src="https://cdn.sparkfun.com/assets/learn_tutorials/1/5/9/5/GPIO.png" style="height: 75%; width: 75%;" alt="RaspberryPi GPIO" />
  <p align="center">RaspberryPi GPIO</p>
</p>


## Thanks, but where is the real info?
ArduSimple makes gps boards and has excellent content.  
- https://www.ardusimple.com/rtk-explained/

Sparkfun makes lots of great stuff, including gps boards.  They are known for their excellent content explaining how to use their products.
- https://learn.sparkfun.com/tutorials/what-is-gps-rtk



