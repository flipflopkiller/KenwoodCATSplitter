# KenwoodCATSplitter

In the last few months I have been quite active in the early morning on 160 and 80m bands. I use a remote station I have in Ostia Antica, about 15 km from my home QTH. In this rural site there are no limits for the antennas size like we have in city environment.
The transceiver is a Kenwood TS-480HX connected via serial port to a PC running the still very good Kenwood software ARCP. 
<p align="center">
<img src="https://github.com/flipflopkiller/KenwoodCATSplitter/assets/92851744/50e56090-f26f-48e2-a214-00bb76293c94" width="600" />
</p>
Everything works fine for CW but for FT8 the software JTDX cannot be connected via CAT to the transceiver because its serial port is already busy with ARCP.
Till now JTDX was set to “Transceiver None” in its settings and the band changes were made manually on ARCP (RTX) first, then on JTDX. If I forgot to change the band on JTDX, the QSOs are automatically logged on wrong band and I spend a lot of time to fix them. Another annoying problem in using JTDX without CAT control is the frequent TX audio level adjustment needed to maintain constant the output power while the TX tone frequency is close to the sides of the tx filter. This can be easily solved by setting the “Split Operation” parameter to RIG or FAKE IT in JTDX settings window, if one has a working CAT…

So I looked up how to split the serial port to make two programs work together and stay synchronized with the rig. I thought it would be easy with socat for Linux and com0com for Windows. The issue arose when ARCP connected to the transceiver. The Kenwood software sets the AI2 mode, and when the AI2 command is sent, the transceiver outputs any parameter variation on the serial port without being asked (no polls). This results in a large amount of data sent over the port. JTDX doesn’t handle this type of traffic and fails to connect to the radio.
The idea is to filter the traffic directed to JTDX so that it only receives the answers to its commands without the extra data generated for ARCP.
It worked!


Below programs and syntax. It might be helpful for someone

    1- the TS-480 is connected via RS232 to a physical port on the PC /dev/ttyS0
    
    2- the program ser2net opens the serial port /dev/ttyS0 and creates a TCP socket server on port 10000
    
    3- the first virtual serial port is created to feed the ARCP control software that runs in wine.
    wine looks for the serial ports in the folder /home/user/.wine/dosdevices/ so the command
    /usr/bin/socat PTY,link=/home/user/.wine/dosdevices/com4,raw,echo=0 TCP:localhost:10000
    creates the virtual port com4 in that folder linked to the actual port /dev/ttyS0
    
    4- the second virtual serial port for WSJTX/JTDX needs to be filtered, so the command is a bit more complicated.
      A- a FIFO special file must be created with the command mkfifo jtdxfifo
      B- create a text file filter.sh in /home/user/ with the following content inside
      /usr/bin/stdbuf -i0 -o0 tr ';' '\n' | /usr/bin/grep --line-buffered -e IF -e FA -e FB -e MD -e FW -e AI | /usr/bin/stdbuf -i0 -o0 tr '\n' ';'
      then make it executable with chmod 775 filter.sh
      C- exec this long command to create the virtual filtered port ttyTS480 for WSJTX/JTDX
      /usr/bin/socat - "TCP:localhost:10000" 0<jtdxfifo | /usr/bin/socat - "exec:/home/user/filter.sh" | /usr/bin/socat - PTY,link=/home/user/ttyTS480,raw,echo=0 1>jtdxfifo
    
    launch ARCP480 and connect it to the port com4
    launch JTDX and configure the filtered radio port

<p align="center">
<img src="https://github.com/flipflopkiller/KenwoodCATSplitter/assets/92851744/910fe7b4-9f44-4c14-a065-e8dc97b57086" width="600" />
</p>





    
