# cyanogenmod-stagefright
Stagefright on CyanogenMod 12 (Android 5.0.1) on Samsung Galaxy S3 Neo+ GT-9301I

Here is the "patient" ....

![SamsungS3Neo](/image.png)


Let's begin the operation ... just for laughs ... hahahahah

![debug](/debug.gif)




I know this animated GIF looks cool, but seriously, let's get back to the point ... I think it is x86 anyway, we wanna do some ARM

None of exploits worked for me

Wanted to test Stagefright :)

Great resources.

https://googleprojectzero.blogspot.com/2015/09/stagefrightened.html

https://github.com/NorthBit/Metaphor

I mostly worked with G0 exploit.

https://www.exploit-db.com/exploits/38226/

Got a Samsung Galaxy S3  Neo+ GT-9301I phone to play with .... figured out I play with custom rom (user debug build) so I can change things and experiment ... decided to use Cyanogenmod ... got the source ca. 20 GB of source code :) compiled it over night ... than rooted and flashed my mobile 

Here is the ready ROM:

https://forum.xda-developers.com/galaxy-s3-neo/orig-development/rom-cyanogenmod-12-s3-neo-t2983109

or if you want to compile from source:

repo init -u git://github.com/CyanogenMod/android.git -b cm-12.0

After that you have to create manifest in .repo/local_manifest.xml
For cm 12.0 for S3 Neo it should look like this :

```xml

<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="CyanogenMod/android_hardware_samsung.git" path="hardware/samsung" remote="github" revision="cm-12.0" />
  <project name="CyanogenMod/android_hardware_qcom_fm.git" path="hardware/qcom/fm" remote="github" revision="cm-12.0" />
  <project name="CyanogenMod/android_device_qcom_common.git" path="device/qcom/common" remote="github" revision="cm-12.0" />
  <project name="MSM8226-samsung/android_device_samsung_s3ve3g.git" path="device/samsung/s3ve3g" remote="github" revision="cm-12.0" />
  <project name="MSM8226-samsung/android_vendor_samsung_s3ve3g.git" path="vendor/samsung/s3ve3g" remote="github" revision="cm-12.0" />
  <project name="MSM8226-samsung/android_kernel_samsung_s3ve3g.git" path="kernel/samsung/s3ve3g" remote="github" revision="cm-12.0" />
  <project name="CyanogenMod/android_device_samsung_qcom-common.git" path="device/samsung/qcom-common" remote="github" revision="cm-12.0" />
  <project name="MSM8226-samsung/android_device_samsung_msm8226-common.git" path="device/samsung/msm8226-common" remote="github" revision="cm-12.0" />
</manifest>
```

repo sync

now following commands will start build (be in folder where you synced cm ) : 

. build/envsetup.sh

lunch - then you have to find cm_s3ve3g-userdebug and enter its number 

time mka bacon


and then you should wait 3-6 hours for build to finish ( depends on your PC configuration )



Used Termux (and installed GDB from there). Could use GDB directly on the mobile, no need to use gdbserver .... 

Started with g0 ... decided to disable ASLR from the beginning, unfortunately it didnt work out of the box ... problem ... think to me .... different ROP chain .... 

Solving problem by building the rop gadget into libc.so and recompiling

Still does not work

Looking at it closer, it has no right to work !!!



Vtable is totally off ... way before the buffer .... after long time figure out that my rom is using dlmalloc ... 

Changed it to jemalloc

Recompiled and flashed the mobile


Still does not work ... but HEY at least vtable pointer is there after the buffer and I am able to overwrite it ....

So why it does not work .... well the values to overwrite vtable seem wrong in G0 exploit for my Cyanogen build

Adjusted them .... + 8 bytes 

Now in the original version of the exploit (G0) I happened to land 28 bytes further (+28 offset) of vtable pointer in my sprayed heap, that should be readAt function address .... but in the original version NOPs were on the heap ... so I changed it to only 28 NOPs and started my ROP Chain/Stack right away

Here is the assembly, see at the bottom. It is branching to value of offset +28 bytes:

```asm
(gdb) disass 0xb66e1042
Dump of assembler code for function _ZN7android14MPEG4Extractor10parseChunkEPxi:
   0xb66e0ff8 <+0>: stmdb   sp!, {r4, r5, r6, r7, r8, r9, r10, r11, lr}
   0xb66e0ffc <+4>: vpush   {d8}
   0xb66e1000 <+8>: mov r6, r0
   0xb66e1002 <+10>:    ldr.w   r8, [pc, #708]  ; 0xb66e12c8 <_ZN7android14MPEG4Extractor10parseChunkEPxi+720>
   0xb66e1006 <+14>:    mov r9, r1
   0xb66e1008 <+16>:    ldr r3, [pc, #704]  ; (0xb66e12cc <_ZN7android14MPEG4Extractor10parseChunkEPxi+724>)
   0xb66e100a <+18>:    sub sp, #444    ; 0x1bc
   0xb66e100c <+20>:    add r8, pc
   0xb66e100e <+22>:    add r7, sp, #120    ; 0x78
   0xb66e1010 <+24>:    str r2, [sp, #52]   ; 0x34
   0xb66e1012 <+26>:    ldr.w   r5, [r8, r3]
   0xb66e1016 <+30>:    ldrd    r2, r3, [r1]
   0xb66e101a <+34>:    ldr r4, [sp, #52]   ; 0x34
   0xb66e101c <+36>:    ldr r1, [pc, #688]  ; (0xb66e12d0 <_ZN7android14MPEG4Extractor10parseChunkEPxi+728>)
   0xb66e101e <+38>:    ldr r0, [r5, #0]
   0xb66e1020 <+40>:    strd    r2, r3, [sp]
   0xb66e1024 <+44>:    ldr r2, [pc, #684]  ; (0xb66e12d4 <_ZN7android14MPEG4Extractor10parseChunkEPxi+732>)
   0xb66e1026 <+46>:    str r4, [sp, #8]
   0xb66e1028 <+48>:    add r1, pc
   0xb66e102a <+50>:    str r0, [sp, #436]  ; 0x1b4
   0xb66e102c <+52>:    movs    r0, #2
   0xb66e102e <+54>:    add r2, pc
   0xb66e1030 <+56>:    blx 0xb66bc038 <__android_log_print@plt>
   0xb66e1034 <+60>:    ldr r0, [r6, #80]   ; 0x50
   0xb66e1036 <+62>:    movs    r3, #8
   0xb66e1038 <+64>:    ldr r1, [r0, #0]
   0xb66e103a <+66>:    str r7, [sp, #0]
   0xb66e103c <+68>:    str r3, [sp, #4]
   0xb66e103e <+70>:    ldrd    r2, r3, [r9]
=> 0xb66e1042 <+74>:    ldr r4, [r1, #28]
   0xb66e1044 <+76>:    blx r4
```



WHOOOO HOOO ... overwrote vtable and function pointer of readAt to my ROP chain/stack ... and the ROP chain seem to work ... but it stucks on executing shellcode wanted to write a file .... problem seem to be SELinux (seems that mediaserver is sandboxed). Disabled it.

https://github.com/marcinguy/cyanogenmod-stagefright/blob/master/exploit1.py


New challenge .... make it work without modifying libc.so, so use different rop chain .... 


Tools I used to find gadgets 

Ropper, ROPGadget, xrop ... .

xrop seem to find a lot more than other ... wrote a custom chain....


WHOOOO HOOO ... it works 

https://github.com/marcinguy/cyanogenmod-stagefright/blob/master/exploit2.py


New challenge ... exploit stock Samsung ROM Android 4.4.2 

Had to compile gdbserver and/or GDB since Termux does not work on Android <=5

Currently stuck at it ... seems like Samsung remove libc symbols and modifies a lot ... dont see the memcpy call that overwrites 




