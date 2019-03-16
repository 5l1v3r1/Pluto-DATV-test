ADALM-Pluto and DATV

This is not a tutorial to install Pluto from scratch ! Just detailed notes following my 3-days workshop.  
You should have installed your Pluto tools and have it tested enough : SDRangel, GNUradio... have drivers/packages installed to manage IIO device : libiio, gr-iio, libad9361 ...  
Be sure to have "Industrial IO" listed as block section in GNUradio, with FMCOMM and PlutoSDR sub-sections.  
May also work using SoapySDR or osmocom.  

IMPORTANT REMINDER :  
Always use a bandpass filter. No filter = harmonics.  
You must have a license to transmit, except on very few frequencies.  
You can cause serious trouble by transmitting on unauthorized frequencies !  
 This is your own responsability !  
Outside allowed spectrum, use dummy load, or make test inside a Faraday cage, or in a deep tunnel under mountains :)  



Setup DATV RX environment on Linux: 
===================================

My choice goes  to SDRangel. 
However to enable DATV plugin (Linux only) I had to compile SDRangel from sources. 
Information : https://github.com/f4exb/sdrangel/tree/master/plugins/channelrx/demoddatv  

![image](https://user-images.githubusercontent.com/26578895/48941520-24c7d980-ef1c-11e8-88ac-23e613ba854e.png)


I played one day first using RPiDATV from @F5OEO_evariste  : https://github.com/F5OEO/rpidatv  
  
    
Receiving DATV using VLC and LEANDVB :
======================================

More infos here : http://www.pabr.org/radio/leandvb/leandvb.en.html  

    rtl_sdr -f 435008000 -s 2400000 -g 37 - | ./leandvb --gui --anf 0 --sr 500e3 --cr 1/2 --drift --tune 7e3  --drift  | cvlc -  
  
vlc can be replaced by mplayer, depending of the codec. However result is better using VLC
  
![image](https://user-images.githubusercontent.com/26578895/49224506-e0d44900-f3e1-11e8-901c-2d40c6fd0609.png)
  
  
  
RX setup using an old DVB-S FtA receiver :  
====================================

Made my tests using an old DVB-S receiver : a METRONIC "Touch Box 5"  
Erased all channels, favorites. Deselected satellites, transponders.  
Created a new "DATV" satellite, with some new transponders onboard this fake sat. 
For each transponder : freq 10720, don't care on polarity, and symbol rate : 1000, 1200, and 1500 kS/s  
![image](https://user-images.githubusercontent.com/26578895/48941174-c2baa480-ef1a-11e8-81e6-bddc9e49f6e9.png)  

Send your signal from the Pluto (see below), then perform a channel scan once.  

That's it, from now channel 1 will receive at 1000kS/s, channel 2 1200 kS/s, and channel 3 1500kS/s, all on the same frequency.  

Note : using DVB-S receiver you can only receive MPEG-2 . 
MPEG-4 is for DVB-S2 mode (correct me if I'm wrong) however using SDRangel you can decode both MPEG2 and MPEG4 TS streams.  

Thus you may have to convert mp4 video file to MPEG-2.  
Transcode video to MPEG2 .ts format (can be improved by RTFM):  

        ffmpeg -re -i my_file.mp4 -vcodec mpeg2video -s 360x288 -r 25 -b:v 1M -acodec mp2fixed -strict -2 -b:a 128k -f mpegts test3.ts  


GNURADIO setup :
================
The two provided files as examples (in "scripts" directory) are working in the same way, however not using same DVB-S blocks.
You can change symbol-rate "in-the-fly" : 333, 500, 1000, 1200, 1500 KS/s. 

dvbs_tx.grc :  

To run dvbs_tx.grc you have to install DVBS blocks from here  : https://github.com/drmpeg/gr-dvbs  
Once installed it should appear in GNUradio in "dvbs" blocks : 

![image](https://user-images.githubusercontent.com/26578895/48941069-550e7880-ef1a-11e8-845d-9b060eef682a.png)  


dvbs_tx2.grc :  

Blocks in use are native on GNUradio, under "Digital Television" blocks. Just run the script.  
This example comes from @csete Alex, here : https://myriadrf.org/blog/digital-video-transmission-using-limesdr-gnu-radio/


Transmit video file using GNURADIO :
================================

Copy the .ts files from "samples" folder to your Pluto USB Mass Storage (the files in the folder, not the folder itself)

*** From gnuradio-companion (GUI)

Open dvbs_tx.grc or dvbstx_grc2 and run it.  


![image](https://user-images.githubusercontent.com/26578895/48941283-33fa5780-ef1b-11e8-82a0-6e6ba0d305ee.png)  


Default freq is set to 970MHz, allowing use of a DVB-S receiver without LNB (DVBS RX set to 10.720 Ghz)

*** Using python
Just run files generated by GNUradio. 

Files : dvbs_tx.py python
python dvbs_tx.py
python python

-  Troubleshooting :

** Can't find the .ts files :
    GNUradio will try to open .ts files from : /media/<username>/PlutoSDR folder.
    in case of trouble, open GNUradio .GRC files from "scripts" folders:
    modify the path for "FILE SOURCE" block to your own needs to access video files.

 (Python : edit file, modify path ...)



Transmit video file directly from Pluto using shell and LEANDVB/LEANTRX (GNUradio or python not needed) 
===============================================================================================

This is based on the nice work of F4DAV and PABR team.  Nothing new.  

note 1: to transmit DATV from Pluto, leansdr/leantrx must be installed (at least running) on the pluto: 
 Follow instructions from here : http://www.pabr.org/radio/leantrx/leantrx.en.html 
or very good alternative : reflash at your own risk your Pluto using Plutoweb firmware from unixpunk/ImDroided team : https://github.com/unixpunk/PlutoWeb  

note 2: lot of variants are possible: you can also copy the file using SCP/SFTP protocol, or use  runme0.sh script from external USB storage.  


Copy MPEG2-lalinea.ts file to the USB_gadget volume (Pluto USB Mass Storage). 
Eject the gadget volume. Will auto-remount by itself after few seconds.  
Connect via SSH (or serial) to pluto shell then type following commands : 

	mkdir /gadget
	losetup /dev/loop7 /opt/vfat.img -o 512
	mount /dev/loop7 /gadget
	leandvbtx --cr 1/2  --s16 < /gadget/MPEG2-lalinea.ts    | leaniiotx -f 970000000 --bufsize 32768 --nbufs  32 --bw 3e6 -s 1e6 -v

Will stream video at 970 MHZ, BW 1MHz, symbrate 500kS/s, QPSK, CR 1/2  

Variant :  

	leandvbtx --cr 1/2  --s16 < /gadget/MPEG2-lalinea.ts   | leaniiotx -f 970000000 --bufsize 32768 --nbufs  32 --bw 2e6 -s 666e3 -v  

--> 970 MHZ, BW 500kHz, 333kS/s, QPSK, CR 1/2  


Transmit RPi live-webcam on Pluto (using RPi and avc2ts from F5OEO) : 
=====================================================================

Thanks to F5OEO for suggesting this idea, and helping to debug.

- video source from RPi : Picam or USB webcam, desktop.
- Pluto is listening for video-TS multicast sent by RPi running avc2ts.


On the Pi side, install avc2ts : https://github.com/F5OEO/avc2ts  
Installation is simple but can take a long time ! Do not interrupt.  
Do not confuse with avc2ts utility included with RPiDATV, this one is not compatible.  
Still on the Pi, install mnc : https://github.com/marascio/mnc  

Pluto : download mnc tool (compiled binary) from [here](https://github.com/LamaBleu/Pluto-DATV-test/raw/master/mnc/mnc).
      : copy the mnc binary to /bin or /usr/sbin, make it executable  


From shell on Pi run following command to start Picam streaming:

        ~/avc2ts/avc2ts -t 0 -m 403000 -b 300000 -x 640 -y 480 -f 20 -n 230.0.0.10:10000:0.0.0.0

Use : ~/avc2ts/avc2ts -t 3 .........  to stream USB cam.  
Adapt settings !

On the pluto side, to transmit on 437 MHz, run following command : 

        mnc -l -p 10000 230.0.0.10 | leandvbtx -f 4 --fill --cr 7/8  --s16  | leaniiotx -f 437000000 --bufsize 32768 --nbufs  32 --bw 3e6 -s 1e6 -v


This was tested using PiZero + Picam, and standalone Pluto powered by battery-pack and connected via WiFi, making a 100% mobile DATV transmitter ([demo video](https://mega.nz/#!m4hViISQ!M9kxJHDA3iXiK_Q7rS3hh-zflXgVD2A8wAFy8ql4I78))

Note : Pluto-firmware including mnc executable and more is available for download, please follow instructions from here : https://github.com/LamaBleu/plutoscripts





Credits :  
LEANTRX/LEANSDR : PABR team and F4DAV : http://www.pabr.org/radio/leantrx/leantrx.en.html  
[rpidatv](https://github.com/F5OEO/rpidatv) : F5OEO (tks Evariste for the video sample)  



Enjoy 
@fonera_cork - LamaBleu 11/2018




