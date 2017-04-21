# cyanogenmod-stagefright
Stagefright on CyanogenMod 12 (Android 5.0.1) on Samsung Galaxy S3 GT-9301I

None of exploits worked for me

Wanted to test Stagefright :)

Great resources.

https://googleprojectzero.blogspot.de/2015/09/stagefrightened.html

https://github.com/NorthBit/Metaphor

I mostly work with G0 exploit.

Got a Samsung Galaxy S3 GT-9301I phone to play with .... figured out i play with customer rom (debug build) so I can change things and experiment ... decided to use Cyanogenmod ... got the source ca 20 GB of source code :) compiled it over night ... than rooted and flashed my mobile 

Started with g0 ... decided to disable ASLR from the beginning, unfortunately it didnt work out of the box ... problem ... think to me .... different ROP chain .... 

Solving problem by building the rop gadget into libc.so and recompiling

Still does not work

Looking at it close, it has no right to work !!!

ASLR... and heap spraying is making problems

Vtable is totally off ... way before the buffer .... after long time figure out that my rom is using dlmalloc ... 

Changed it to jemalloc

Recompiled and flashed the mobile


Still does not work ... but HEY at least vtable pointer is there after the buffer and I am able to overwrite it ....

So why it does not work .... well the values to overwrite vtable seem wrong in G0 exploit for my Cyanogen build

Adjusted them ....



WHOOOO HOOO ... overwrote vtable and function pointer of readAt ... and the ROP chain seem to work ... but it stucks on executing shellcode wanted to write a file .... problem seem to be SELinux (seems that mediaserver is sandboxed). Disabled it.

https://github.com/marcinguy/cyanogenmod-stagefright/blob/master/exploit1.py


New challenge .... make it work without modifying libc, so use different rop chain .... 


Tools I used to find gadgets 

Ropper, ROPGadget, xrop ... .

xrop seem to find a lot more than other ... wrote a custom chain....


WHOOOO HOOO ... it works 

https://github.com/marcinguy/cyanogenmod-stagefright/blob/master/exploit2.py


New challenge ... exploit stock Samsung ROM Android 4.4.2 

Currently stuck at it ... seems like Samsung remove libc symbols and modifies a lot ... dont see the memcpy call that overwrites 




