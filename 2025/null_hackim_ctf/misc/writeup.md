## Ancient Paper

We are given the following image:

![](ancient-paper.jpg)

Its a digit card. Each column its a character and each role its like the bits that form the character.
I had to learn from the following links:
- https://gen5.info/$/LU0NS2XPG8MDVCVZS
- https://codeincluded.blogspot.com/2012/07/punchcard-reader-software.html
- https://www.youtube.com/watch?v=KG2M4ttzBnY

Then we did it manually.

## USBnet

For this challenge we have pcap with USB traffic. Doing a quick research to extract data from the packets i found a writeup that gave me the answer: https://github.com/g4ngli0s/CTF/blob/master/AlexCTF2017/Fore3_usb_probing.md.

![](usb_pcap.png)

Basically we run the following commands: 

Extract the leftover capture data: `tshark -r ~/Downloads/usbnet.pcapng -Y 'usb.capdata and usb.device_address==22' -T fields -e usb.capdata > raw`

Decode the hex to a file: `xxd -r -p raw output2.bin`

If we do strings we get PLTE and IDAT which are from a png image. Use binwalk to extract png: `binwalk -D 'png image:png' output2.bin`

Get flag from QR code image.
