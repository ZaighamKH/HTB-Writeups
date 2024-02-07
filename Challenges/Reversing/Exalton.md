# Challenge Description
Can you find the password?

# Walkthrough
The challenge has only one file called exalton, which upon execution asks for a password, which I assume we have to find as the challenge suggests.

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/c39e7e4a-87fb-41e7-a19f-8d3f9159b9b4)

First I run the file command to get basic information about the file. Then I try the strings commmand which resulted in nothing useful. My next step is disassemble the file using Ghidra which gives the following results: 

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/247e3e84-1251-4652-84da-50b04a77b227)

I cant see any main function or anthing useful which makes me think that the file might be packed using some tool like UPX. So I run hexdump and my assumption is proven correct.
```bash
┌──(kali㉿kali)-[~/HTB/Reversing/Exatlon]
└─$ hexdump -C exatlon_v1 
00000000  7f 45 4c 46 02 01 01 03  00 00 00 00 00 00 00 00  |.ELF............|
00000010  02 00 3e 00 01 00 00 00  d0 16 49 00 00 00 00 00  |..>.......I.....|
[snip]
000ad360  00 00 40 ff 00 00 00 00  55 50 58 21 00 00 00 00  |..@.....UPX!....|
000ad370  55 50 58 21 0d 16 08 09  0d d9 14 a3 b4 c6 c1 05  |UPX!............|
000ad380  b8 05 07 00 06 b3 01 00  c8 9b 21 00 49 22 00 a1  |..........!.I"..|
000ad390  f4 00 00 00                                       |....|
```
We can see some mentions of upx so I go ahead and unpack the file using upx with the following command:
```bash
┌──(kali㉿kali)-[~/HTB/Reversing/Exatlon]
└─$ upx -d exatlon_v1 -o unpacked_exalton_v1
```
Now I use Ghidra again to disassemble the unpacked file and there is a lot more information as expected. Especially the main function is now available.

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/2afd6861-bb92-4681-b069-04c1cf1810e0)

In the main function I can see the output to get password and call to the exalton function. I can also see an array after the exalton function which seems to be the encoded password.

I set a breakpoint for when it enters the exalton function, at 0x404d29, and when prompted for password, I use just H. Since flags are always in the form HTB{}, the first letter should be H and should correspond with the first value we saw in the array, i.e. 1152. 

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/b52e8dd5-4575-4b5e-a965-1d3d8900420c)

My guess is correct, at r10 we have the value hex value which corresponds to 1152 in ASCII. (R10 was chosen since just register R8 and R10 change with different values used in password, and only R10 has some connection to the array.)

Now I know that I need to decode the array to get the flag. I tried to find if the encryption was a basic pattern by finding the encrypted values for F, G, I, J and I found a basic pattern. Each number had a difference of 16, so it must be bit shifting. My next step is to go into the exalton function to see if I can find anything and I can clearly see that the character is bit shifted by 4. 

![image](https://github.com/ZaighamKH/HTB-Writeups/assets/119772901/06570985-e342-449d-b4a1-a17395c96896)

Now I just create a simple python script and run it to get the flag. The python script is:
```python
encoded_str = [1152, 1344, 1056, 1968, 1728, 816, 1648, 784, 1584, 816, 1728, 1520, 1840, 1664, 784, 1632, 1856, 1520, 1728, 816, 1632, 1856, 1520, 784, 1760, 1840, 1824, 816, 1584, 1856, 784, 1776, 1760, 528, 528, 2000]
str_value=""
for x in encoded_str:
        buffer=x >> 4
        str_value=str_value+chr(buffer)
print(str_value)
```

After running the script I get the flag finally.
```bash
HTB{l3g1.......!!}
```



