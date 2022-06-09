## Setting up RTK GPS on a Raspberry Pi

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
Next, let's talk about the kinds of gps devices available for talking to the satellite systems.  Since the American GPS system was first early chips only talked to it.  Then when GLONASS came online chips came out that supported both GPS and GLONASS, even simultaneously.  Newer chips support GPS, GLONASS, BeiDou and Galileo and some of the smaller system.  The most popular devices are based on chips created by a company named [U-Blox](https://www.u-blox.com/).  They have lots of different GPS chips for various purposes and with varying capabilities.  Their latest chips are series 10.  You can still find series 6, 7, and 8 out there.  The series 8 chips are much better than 6 or 7; they support more kinds of satellites and more simultaneous satellite connections.  I'm using a device with a series 9 chipset, the ZED F9P; it is a big improvement over series 8 because it supports yet more satellite systems, more simultaneous connections and most importantly, it supports a second frequency band (L2) for higher precision measurements.  Newer is better.  Ask any baby.

U-Blox isn't the only game in town; you can find inexpensive GPS modules that use other chipsets, often from China.  In many ways these can work interchangeably with U-Blox based boards because they all support the a standard way of reading data from the satellites, call NMEA sentences.  This is a line-oriented text protocol.  So if your software can parse NMEA sentences then it can work with most gps modules.  

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

The points are within A 2.5 meter range in each direction.  So that is better than the first figure, which was done with a non-active antenna and no base plane.

Next, this is 1000 readings with a U-Blox ZED F9P, an active GPS/GNSS antenna and a good ground plane mounted on a tripod, the same setup as the last set of readings.  
<p align="center">
  <img src="img/fp9_tripod_base_1000_samples.png" style="height: 75%; width: 75%;" alt="U-Blox ZED F9P position readings over time while staying in place; 1 sample per second for 1000 seconds" />
  <p align="center">U-Blox ZED F9P position readings over time while staying in place; 1 sample per second for 1000 seconds</p>
</p>

The points are within 2.5 meter range vertically and 1.25 meters horizontally.  So that is a little better than the NEO-N8M.  This shows the series 9 chipset is superior to the series 8.  But still, this is not accurate enough for navigating a short RC course where the lane may be less than 2 meters wide.  Also, we can't take 100 or 1000 readings at each position in order to get an average; we are racing!  We need multiple readings per second so we can see where the car is as it moves.  Crap, we are our of luck I guess.  But wait, there is more!

### Kinematifanatics
What if you could make readings that were more like 2 centimeters CEP?  What if you could do that 10 times per second while you were moving?  What if it was cheap as hell?  Well, forget that last one, but the other two are possible by using RTK gps.  [Sparkfun](https://learn.sparkfun.com/tutorials/what-is-gps-rtk/all) describes RTK gps, "RTK is short for real time kinematics. A GPS receiver capable of RTK takes in the normal signals from the Global Navigation Satellite Systems. On top of these signals an RTK receiver takes in an RTCM correction stream and then calculates your location with 1cm accuracy in real time."

What are these magical 'corrections' they speak of and who conjures them?  Corrections are created by Earth-bound base stations and then transmitted to the gps receiver.  Think about corrections this way; a gps receiver uses the time stamps and orbital positions it gets from the satellites to triangulate it's otherwise unknown position on the Earth.  But the base stations already know their position; their position has been very, very, very, very carefully measured.  Base stations already know where they are, but they can also talk to the satellites and do the triangulation thing.  If the triangulation calculation yields an incorrect position, then base station can calculate the difference between where it knows it is and where triangulation says it is; that is a correction.  The base station can then send that offset to our gps receiver, then our gps receiver can apply the offset after it triangulates its position using the satellites, thus making the result much more accurate.  This works if you are pretty close to the base station that is broadcasting the corrections (say within about 35km) and if you apply very recent corrections.  This make sense if you think about the things that cause drift; if you are close to the base station then you are able to see the same satellites and you are subject to similar atmospheric conditions.  You need recent corrections so that not much drift has occurred between calculating the correction and applying the correction. 

For reals, the way corrections are calculated is much more complicated than how I have explained it, but you get the idea.

### Please correct me
There are many organizations that maintain base stations and provide corrections over the Internet.  The corrections are typically broadcast over the internet using a protocol called [NTRIP](https://www.ardusimple.com/rtk-explained/); in this case the server is called an NTRIP Caster.  The actual corrections are commonly in a format called RTCM3. You typically must sign up for a free account to get access to the NTRIP caster servers.  I have at least 4 base stations in the [UNAVCO](https://www.unavco.org/instrumentation/networks/status/all/realtime) network near me.  There is also a network maintained by individuals good enough to share their gps data for free; [RTK2Go](http://rtk2go.com/).  Resources are changing and improving all the time; check this global list of organizations that host NTRIP servers; [rtcm-ntrip.org](http://rtcm-ntrip.org/home.html).

You basically find an organization that covers your country or region and lookup their list of servers and locations.  If they have one with 35km of you then you are in luck; sign up with them for an account to get access to the servers, then configure your NTRIP client sofware using the information they provide (I tell more about client software below).


### All our base are belong to you

What if you don't have a fixed base station near enough to you?  That's simple; create your own.  It's possible to use a second RTK gps receiver as a base station.  One issue with that is that your base station does not have a very, very, very, very carefully measured position.  However, you can let the base station sit there for a while and get lots and lots and lots of readings so it calculate a very good estimate of its position.  Good enough for our needs.

[Sparkfun](https://learn.sparkfun.com/tutorials/setting-up-a-rover-base-rtk-system/all) has a very detailed walkthrough on how to setup your own RTK base station.  


### Client Tell
Ok, so we have a source of RTCM3 corrections, maybe sent over the Internet using the NTRIP protocol, but our gps receiver doesn't know anything about the Internet; it has serial ports (how 1980s) for communication.  So how do we get the RTCM3 corrections to the gps receiver?

Well we first need an NTRIP client to receive the RTCM corrections, then we need to get those corrections to the gps receiver board.  

Our Donkeycar has a RaspberryPi (or a Jetson Nano) and that has Wi-Fi.  So if the car is close enough to a Wi-Fi hotspot then it can use an NTRIP client to read the RTCM3 corrections from the base station over the internet, then send the corrections to the gps receiver over a serial connection.  If the car is not consistently close enough to a Wi-Fi hotspot (which it probably won't be), then you could use cellular data; I taped my phone to my car and put it into hotspot mode, then connected the RaspberryPi Wi-Fi to the phone's hotspot so it could access the Internet while it was driving. 

A RaspberryPi also has Bluetooth and so does our phone.  So another way to get corrections using your phone is to get an NTRIP client mobile app on the phone and use it to get the corrections from a base station, then transmit the corrections to the RaspberryPi using Bluetooth serial.  The RaspberryPi would then forward the corrections to the gps receiver over a serial connection.  

But what if you have to use your own base station?  That might actually be the easiest scenario, but the most expensive.  You need a second RTK gps device to act as the base station; a device that can create RTCM3 corrections.  Then we still have the problem of getting the corrections from the base station to the gps receiver.  We could go through the RaspberryPi; maybe connect a Bluetooth radio to the serial output on the base station and have it received by the RaspberryPi's Bluetooth, then the RaspberryPi can forward it on to the gps receiver via a serial connection.  Alternatively we can cut the RaspberryPi out of the picture altogether; we can connect a radio transmitter of some sort to the base station's serial port; the gps receiver transmits the RTCM3 connections to the serial port and so out to the radio.  On the car we then need a compatible radio connected t the gps receiver's serial port, so the corrections are received by the radio and input to the gps receiver via the serial port.  That is a common way to do this.  There are various radio technologies that could be used, but the best is LoRa.  LoRa radios have long range, which is good for us because it can cover a large track.  Further, a LoRa transmitter can be configured in broadcast mode so many LoRa receivers can listen for the data at the same time.  That works really well at a track with multiple competitors.

### The cold hard facts
Ok, this is the section (maybe sections) where I'll give detailed instructions on how I accomplished getting RTK gps to work on a Donkeycar.  Hopefully it will help you avoid a bunch of the experimentation I had to go through and make things faster for you.


#### RTKLIB
RTKLIB is an open source software package written by Tomoji Takasu, that can use GNSS raw data to run real-time or post-processing solutions to accurately determine relative position using differential information from two receivers (RTK/PPK).  This has way more capability than we will be using.  We really only need one of it's command line utilities; str2str.  str2str can connect to an NTRIP server and then write the corrections to up to 3 outputs.  I can convert formats if that is necessary; so it could take in RTCM2 corrections, but output RTCM3 corrections.

- http://www.rtklib.com/
- [RTKLIB github](https://github.com/tomojitakasu/RTKLIB)
- [RTKLIB Demo5; rtklibexplorer RTKLIB fork](https://github.com/rtklibexplorer/RTKLIB)

#### Lefebure NTRIP Client for Android
Lefebure offers a free NTRIP client and NMEA data logger for Android. Your phone connects to the NTRIP server and then sends the RTCM corrections via a Bluetooth SPP connection to the gps receiver if the receiver has an attached bluetooth radio or to the RaspberryPi, which would then forward to the gps receiver via RTKLIB str2str.  

- [Lefebure](http://lefebure.com/software/android-ntripclient/) 
- [Google Play Store](https://play.google.com/store/apps/details?id=com.lefebure.ntripclient)

### Thanks, but where is the real info?
ArduSimple makes gps boards and has excellent content.  
- https://www.ardusimple.com/rtk-explained/

Sparkfun makes lots of great stuff, including gps boards.  They are known for their excellent content explaining how to use their products.
- https://learn.sparkfun.com/tutorials/what-is-gps-rtk



