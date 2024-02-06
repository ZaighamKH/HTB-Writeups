# Challenge Description
We accessed the embedded device's asynchronous serial debugging interface while it was operational and captured some messages that were being transmitted over it. Can you decode them?

# Walkthrough
Once you unzip the provided file, you are given a "debugging_interface_signal.sal" file. When you run the file command for more details on the file, it is apparent that it is another zip file.
```bash
┌──(kali㉿kali)-[~/HTB/Hardware/Intro to Hardware Hacking/DebuggingInterfaceChallenge]
└─$ file debugging_interface_signal.sal 
debugging_interface_signal.sal: Zip archive data, at least v2.0 to extract, compression method=deflate
```
Once unzipped, you are presented with two files, "digital-0.bin" and "meta.json". I run the strings command on the digital-0.bin file to see if it has any information of interest.
```bash
┌──(kali㉿kali)-[~/HTB/Hardware/Intro to Hardware Hacking/DebuggingInterfaceChallenge]
└─$ strings digital-0.bin 
<SALEAE>
ffffff
~L@Y
[snip]
```
Only one thing looks interesting, its the mention of [SALEAE] in the first line. SALEAE is a logic analyzer used to decode various communication protocols such as I2C, SPI etc.
So my next step is to open the files in SALEAE logic analyzer.

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/e0285438-2f11-4f68-9c82-76d1ed1d1995)

On the left there is an option of different analyzers. From the challenge description I know its an asynchronous serial signal so I choose the Async Serial analyzer.

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/86d26e42-5df3-4ed8-80da-45a455b500db)

The only thing we need is the bit rate in Bit/s. There is a simple solution to find this out. From the zoomed in signal below, we can see the bit rate mentioned in the bos on the left.
Which shows the widht (bitrate) as 31.23 kHz which is equal to 31230 Bit/s.

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/2bbea02f-3f03-46e2-9da2-8e708f9c1bb0)

Once we input the bitrate and set the output to ASCII, the flag can be seen clearly above the signal or in the terminal on the left, like in the refrence image below.

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/2eb000ad-52e9-4a81-a1cb-2957df09128c)



