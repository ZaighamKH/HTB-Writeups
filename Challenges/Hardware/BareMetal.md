# Challenge Description:
Concerned about the integrity of devices produced at a remote fabrication plant, management has ordered a review of our production line. This revealed many things, including a weird embedded device in one of our serial networks. In attempting to remove it, we accidentally triggered a hardware failsafe, which resulted in the device stopping working. However, luckily we extracted the firmware prior to doing so. We need to find out what it did to the slave device it was tapped into, can you help us? The microcontroller of the device appears to be an atmega328p.

# Walk-Through:
As apparent from the description this an embedded firmware reversing challenge. The target is an atmega328p and a single file is provided called "extracted_firmware.hex". Our first step is to get the datasheet for atmega328p and get information on the "extracted_firmware.hex" file using the following command:
```bash
(kali㉿kali)-[~/HTB/BareMetalChallenge]
└─$  file extracted_firmware.hex 
extracted_firmware.hex: ASCII text, with CRLF line terminators
```
We can see that it is an intel hex file. To disassemble the file I first need to convert the hex file into a binary file. Based on the datasheet I also know that out target chip uses an AVR 8 architecuture, so we use the "avr-objcopy" to convert the ihex file into binary.
```bash
┌──(kali㉿kali)-[~/HTB/BareMetalChallenge]
└─$ avr-objcopy -I ihex extracted_firmware.hex -O binary extracted_firmware.bin
```
Now we can start to disassemble the binary file to understand what the device was doing. We can choose from many reversing tools such as GHidra, IDA, radare2 etc. My choice was radare2 to reverse and analyse the binary file. 
Once opened through radare2 we can see that there are 2 functions that radre2 detects
```bash
[0x00000068]> afl
0x00000068    3 2182 -> 24   entry0
0x00000080    4 2154         fcn.00000080
```
Upon further inspection the function at 0x00000080 called fcn.00000080 seems to be what we are looking for.
```bash
[0x00000080]> pdf
Do you want to print 1083 lines? (y/N) y
            ;-- r29:
            ;-- sph:
            ; CALL XREF from entry0 @ 0x74
┌ 2154: fcn.00000080 ();
│           0x00000080      8ab1           in r24, 0x0a                ; IO UCSRB: USART Control and Status Register B.
│           0x00000082      806e           ori r24, 0xe0
│           0x00000084      8ab9           out 0x0a, r24               ; IO UCSRB: USART Control and Status Register B.
│           0x00000086      5498           cbi 0x0a, 4                 ; IO UCSRB: USART Control and Status Register B.
│           0x00000088      5f9a           sbi 0x0b, 7                 ; IO UCSRA: USART Control and Status Register A.
│           ; CODE XREF from fcn.00000080 @ 0x8e8
│       ┌─> 0x0000008a      5f98           cbi 0x0b, 7                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x0000008c      5d98           cbi 0x0b, 5                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x0000008e      5e9a           sbi 0x0b, 6                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x00000090      5e98           cbi 0x0b, 6                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x00000092      5d9a           sbi 0x0b, 5                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x00000094      5e9a           sbi 0x0b, 6                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x00000096      5e98           cbi 0x0b, 6                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x00000098      5d9a           sbi 0x0b, 5                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x0000009a      5e9a           sbi 0x0b, 6                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x0000009c      5e98           cbi 0x0b, 6                 ; IO UCSRA: USART Control and Status Register A.
│       ╎   0x0000009e      5d9a           sbi 0x0b, 5                 ; IO UCSRA: USART Control and Status Register A.
[snip]
```
The code seems to first intialise UCSRB and UCSRA which are both USART control and status register. After initialisation the 5th and 6th bit of the UCSRA are constantly being toggled in a pattern.

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/1551c4c9-caca-47a8-8161-c9c096c69e0b)

From the datasheet we can tell the the pattern of sbi and cbi for 6th bit shows when transmit is completed, so we are now concerened with the pattern on bit 5 as it does not follow a pattern like bit 6. This could represent the data that was transmitted. As pin 5 is set or cleared every third line, we create a simple python script to create a binary value where the value is 0 when pin 5 is cleared and the value is 1 when pin is set. First the output data is cleaned to have just cbi and sbi in every line, this method was chosen over having just pin numbers as cbi and sbi are unique making it easy to format. We simply replace any line that contains "cbi" by just "cbi" and the same for "sbi". Following commands was used for formatting:
```bash
grep -E "(cbi|sbi)" raw_output.txt > formatted_output.txt
sed -i 's/.*\(cbi\|sbi\).*/\1/' formatted_output.txt
```
So our text file looks like
```bash formatted_output.txt
──(kali㉿kali)-[~/HTB/BareMetalChallenge]
└─$ cat formatted_output.txt      
cbi
sbi
cbi
sbi
sbi
[snip]
```
The python script used is:
```python
with open("formatted_output.txt") as file:
    bitstream = ''
    #Go through each line and check if it is the third line
    #If it is the third line then check if it starts with c or s (cbi or sbi)
    for count, line in enumerate(file):
        mod = count % 3
        print(f"[{count}] / {mod}: {line}",end='')
        if(mod == 0):
            bit = '0' if line[0] == 'c' else '1'
            bitstream += bit
    
    print(f"\ntotal bits = {len(bitstream)}")
    
    #convert the bitstream string to int	
    n = int(bitstream, 2)
    #convert the bitstream to corresponding text
    x = binascii.unhexlify('%x' % n)
    
 
    print(x)
```

Finally when the script is run we obtain the flag as hypthoesised.
```bash
set_flag:HTB{817......}
```



