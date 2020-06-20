# Introducing the Packet Squirrel!  

from [Hak5 2307](https://www.youtube.com/watch?v=fYdFNFTSoy4)

- more appealing to white hats; like a LAN tap

Or a dropbox that you can pivot and MitM

## Default Payloads

- Default payloads for
    1. tcpdump and
    1. DNS spoofing (MitM) and
    1. OpenVPN (remote access)
    1. Arming Mode

Low Power so can run off battery "for days"

Button to allow user interaction.

USB port is for a flash drive to dump data to.

- Ethernet IN goes to the computer.
- Ethernet OUT goes to the network.

 Press the button when finished capturing, remove the USB, and read the pcap files offline.

## Custom Payloads

If you stick a payload on a flash drive and plug it in, the Squirrel will run that payload on boot.

Payloads on USB disks should be stored in /payloads/ in corresponding switch1, switch2 and switch3 folders, e.g. /payloads/switch2

So in the switch2 folder you would have a file called payload.sh (or payload.py, or payload.txt) This file can set the mode (e.g. NETMODE) for the Squirrel, the LEDs and then call any other scripts or read any config files to set things up.  

This is where you would put the payloads; on the USB.
