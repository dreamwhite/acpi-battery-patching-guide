# How to correctly patch battery status on macOS

Before deep going inside ACPI patching techniques for achieving working battery status, I'd like to give a brief introduction:

> Because the battery hardware in PCs is not compatible with Apple SMbus hardware, we use ACPI to access battery state when running OS X on laptops.</br>
> Generally the use of [ACPIBatteryManager.kext](https://github.com/RehabMan/OS-X-ACPI-Battery-Driver) may help reading the battery status.</br>
> Sadly, later releases of `AppleACPIPlatform` are unable to correctly access fields within the EC (embedded controller). This causes problems for ACPIBatteryManager as the various ACPI methods for battery fail (`_BIF`, `_STA`, `_BST`, etc).</br>
> Despite it's possible to use an older version of AppleACPIPlatform, it's safer and more vanilla using the latest version bundled with the operating system and adapt your DSDT to be comply with the limitations of AppleACPIPlatform.</br>
> More specifically, any fields in the EC larger than 8-bit, must be changed to be accessed 8-bits at one time. This includes 16, 32, 64, and larger fields.

This section has been gently taken from [RehabMan's thread on tonymacx86](https://www.tonymacx86.com/threads/guide-how-to-patch-dsdt-for-working-battery-status.116102/).

Please note that, despite the guide explicitly talks about Clover, the patching criteria is the same as on OpenCore, even with the hot-patching technique, which we'll cover in another guide. 

//TODO: Add link to the hot-patching guide

## How to start then?

The requirements are just a few:

- A copy of your DSDT extracted using OpenCore's SysReport feature
- [MaciASL](https://github.com/acidanthera/maciasl/releases/latest)
- [Hex Fiend](https://hexfiend.com)
- Lot and lot of patience. No need to hurry, as you may make mistakes

Let's go (pray for at least 5 times a day and hope that everything goes well haha).

Let's start by opening with MaciASL your `DSDT.aml`:

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_open.png)

Then, press `⌘+F` and search `EmbeddedControl`

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_ec_search_result.png)

If you don't get any result, then your battery status isn't handled by `EmbeddedControl` and I don't have any idea on how to do it :/

//BUG: e.g. Surfaces laptops use _SAN method for handling the battery status reporting and it's not easy to patch it :/

//TODO: Please note that this is just an example, so feel free to change the content as you want, by highlighting the Operation Region

Else, if you found a result, like I did, write down `OperationRegion` line and the `Field` line one.

In our case:

```
            OperationRegion (ERAM, EmbeddedControl, Zero, 0xFF)
            Field (ERAM, ByteAcc, NoLock, Preserve)
```
It's really important to take note of the field name as the patching fields depends in fact on `Field (ERAM)` (or whatever it's called). We'll need it later.

Then let's proceed by reading the EC fields contained inside `Field (ERAM)`:

<details><summary>EC Fields</summary>
<pre>
{
                SMPR,   8, 
                SMST,   8, 
                SMAD,   8, 
                SMCM,   8, 
                SMD0,   256, 
                BCNT,   8, 
                SMAA,   8, 
                Offset (0x40), 
                SW2S,   1, 
                    ,   2, 
                ACCC,   1, 
                TRPM,   1, 
                Offset (0x41), 
                W7OS,   1, 
                QWOS,   1, 
                    ,   1, 
                SUSE,   1, 
                RFLG,   1, 
                    ,   1, 
                    ,   1, 
                Offset (0x42), 
                    ,   5, 
                UBOS,   1, 
                Offset (0x43), 
                    ,   1, 
                    ,   1, 
                ACPS,   1, 
                ACKY,   1, 
                GFXT,   1, 
                    ,   1, 
                    ,   1, 
                Offset (0x44), 
                    ,   7, 
                DSMB,   1, 
                GMSE,   1, 
                    ,   1, 
                QUAD,   1, 
                Offset (0x46), 
                Offset (0x47), 
                ADC4,   8, 
                ADC5,   8, 
                Offset (0x4C), 
                STRM,   8, 
                Offset (0x4E), 
                LIDE,   1, 
                Offset (0x50), 
                    ,   5, 
                DPTL,   1, 
                    ,   1, 
                DPTE,   1, 
                Offset (0x52), 
                ECLS,   1, 
                Offset (0x55), 
                EC45,   8, 
                Offset (0x58), 
                RTMP,   8, 
                ADC6,   8, 
                Offset (0x5E), 
                TMIC,   8, 
                Offset (0x61), 
                SHPM,   8, 
                ECTH,   8, 
                ECTL,   8, 
                Offset (0x67), 
                LDDG,   1, 
                    ,   1, 
                GC6R,   1, 
                IGC6,   1, 
                Offset (0x68), 
                    ,   3, 
                PLGS,   1, 
                Offset (0x69), 
                    ,   6, 
                BTVD,   1, 
                Offset (0x6C), 
                GWKR,   8, 
                Offset (0x70), 
                BADC,   16, 
                BFCC,   16, 
                BVLB,   8, 
                BVHB,   8, 
                BDVO,   8, 
                Offset (0x7F), 
                ECTB,   1, 
                Offset (0x82), 
                MBST,   8, 
                MCUR,   16, 
                MBRM,   16, 
                MBCV,   16, 
                VGAV,   8, 
                FGM2,   8, 
                FGM3,   8, 
                Offset (0x8D), 
                    ,   5, 
                MBFC,   1, 
                Offset (0x92), 
                Offset (0x93), 
                Offset (0x94), 
                GSSU,   1, 
                GSMS,   1, 
                Offset (0x95), 
                MMST,   4, 
                DMST,   4, 
                Offset (0xA0), 
                QBHK,   8, 
                Offset (0xA2), 
                QBBB,   8, 
                Offset (0xA4), 
                MBTS,   1, 
                MBTF,   1, 
                    ,   4, 
                AD47,   1, 
                BACR,   1, 
                MBTC,   1, 
                    ,   2, 
                MBNH,   1, 
                Offset (0xA6), 
                MBDC,   8, 
                Offset (0xA8), 
                EWDT,   1, 
                CWDT,   1, 
                LWDT,   1, 
                AWDT,   1, 
                Offset (0xAA), 
                    ,   1, 
                SMSZ,   1, 
                    ,   5, 
                RCDS,   1, 
                Offset (0xAD), 
                SADP,   8, 
                Offset (0xB2), 
                RPM1,   8, 
                RPM2,   8, 
                Offset (0xB7), 
                Offset (0xB8), 
                Offset (0xBA), 
                Offset (0xBB), 
                Offset (0xBC), 
                Offset (0xC1), 
                DPPC,   8, 
                Offset (0xC8), 
                    ,   1, 
                CVTS,   1, 
                Offset (0xC9), 
                TPVN,   8, 
                Offset (0xCE), 
                NVDX,   8, 
                ECDX,   8, 
                EBPL,   1, 
                Offset (0xD2), 
                    ,   7, 
                DLYE,   1, 
                Offset (0xD4), 
                PSHD,   8, 
                PSLD,   8, 
                DBPL,   8, 
                STSP,   8, 
                Offset (0xDA), 
                PSIN,   8, 
                PSKB,   1, 
                PSTP,   1, 
                    ,   1, 
                PWOL,   1, 
                RTCE,   1, 
                Offset (0xE0), 
                DLYT,   8, 
                DLY2,   8, 
                Offset (0xE5), 
                GP12,   8, 
                SFHK,   8, 
                Offset (0xE9), 
                DTMT,   8, 
                PL12,   8, 
                ETMT,   8, 
                Offset (0xF2), 
                ZPOD,   1, 
                    ,   4, 
                WLPW,   1, 
                WLPS,   1, 
                ENPA,   1, 
                Offset (0xF4), 
                SFAN,   8, 
                Offset (0xF8), 
                BAAE,   1, 
                S3WA,   1, 
                BNAC,   1, 
                    ,   1, 
                EFS3,   1, 
                S3WK,   1, 
                RSAL,   1
            }
</pre>
</details>

As you can see, this list is really long so we're gonna strip it a little bit with the following criteria:

- count the fields whose size is greater than 8
- exclude (just for the moment) `Offset` keyword lines

After stripping you should have a result like the spoiler below:

<details><summary>Stripped EC fields</summary>
<pre>
{
                SMD0,   256, 
                BADC,   16, 
                BFCC,   16, 
                MCUR,   16, 
                MBRM,   16, 
                MBCV,   16, 
}

</pre>
</details>

As you can see there are only 5 16bits fields and only one 256 bit variable.
The next step is to examine the items in the `Field` definition, looking for items which are larger than 8-bit.

Let's search `Field (ERAM)` (or whatever is called the previously identified field) and again take note of the fields which are larger than 8-bit:

<details><summary>New found fields</summary>
<pre>

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                SMW0,   16
            }

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD0,   64
            }

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD1,   128
            }

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD2,   192
            }

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD3,   256
            }
</pre>
</details></br>
Now after taking note of the fields let's search them manually on MaciASL and if we find a reference, we need to write down and patch it.

//BUG: Please note that SMD0 variable mustn't be considered as generally it's found on I2C devices and can break its functionality on macOS and possibly on other OSs.

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_found_ec_field.png)

Below a list of found references
<details><summary>Found references</summary>
<pre>

                BADC,   16, 
                BFCC,   16, 
                MCUR,   16, 
                MBRM,   16, 
                MBCV,   16, 
                SMW0,   16,
                FLD0,   64,
                FLD1,   128,
                FLD2,   192,
                FLD3,   256

</pre>
</details>
</br>

As RehabMan says in the patching guide:

> When fields larger than 32-bit are accessed, they are accessed as type Buffer.</br>
> Fields 32-bit or under are accessed as Integers.</br>
> This is important to realize as you change the code.<br>
> Buffers are a bit more work to change.<br>
> Also, realize this code is "reading" from the EC.<br>
> Reads and writes must be handled differently.


Again please note that this list is strictly linked to the laptop model and BIOS version we used. Don't blind-patch ++

So let's start by first patching 16-bits fields.

## 16-bits field patching

Let's take for example `BADC` field which has 1 reference in the DSDT (we're not going to count the comments >_>):

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_badc_ec_field.png)

For each 16-bits field we should use a utility method called `B1B2` which can do the work for us:

open MaciASL patch menu, copy and paste this patch and click on apply

```
into method label B1B2 remove_entry;
into definitionblock code_regex . insert
begin
Method (B1B2, 2, NotSerialized) { Return(Or(Arg0, ShiftLeft(Arg1, 8))) }\n
end;
```

Basically, what this method does is taking two arguments (low-byte, high-byte), and returns a 16-bit value from the two 8-bit values.

In other words: imagine you have a cracker (food biscuit) and split it onto two equal parts. If you reattach them it's still a cracker. Simple, nah?

Anyways, after this ridiculous explanation, let's rename the original `BADC` field onto two 8-bit fields.</br>
Please note that the two fields MUST have different names.</br>
In this case we'll use this convention for `BADC` field:

- remove the first letter (`B`) and add a `0` for the low-byte field, and a `1` for the high-byte field.

So we'll have splitted the original `BADC` field onto two 8-bit fields called `ADC0` and `ADC1`.

See the below image for reference:

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_split_ec_fields.png)

After doing this, click on `Compile` button of MaciASL and, since we found a reference to the field, and look for the errors.

Here we found an error:

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_patched_ec_fields_error.png)

Click on it and look for the field call:

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_badc_ec_field.png)

Let's change it as it follows:

since `BADC` doesn't exist more, we're going to call `B1B2` method as it follows:

![](.assets/images/battery-acpi/hard-patching/maciasl_dsdt_patched_ec_fields_fixed_reference.png)

It's important that you respect the previous field call: 

//TODO: improve the explanation of the call

Before it was called with `^^PCI0.LPCB.EC0.BADC`, but now, since we splitted it onto two 8-bit fields, we call it via `B1B2(^^PCI0.LPCB.EC0.ADC0, ^^PCI0.LPCB.EC0.ADC1)`.

Iterate this process for each 16-bit fields.

Please note that variable naming has strict rules which you can consult here

//TODO: give variable naming rules by looking on ACPICA 6.4


## 32-bit fields patching

As well as we did for 16-bit fields which we splitted onto 2 8-bit fields, we're going to split 32-bit onto 4 8-bit fields.

This time instead, we're going to use another utility method called `B1B4`:

open MaciASL patch menu, copy and paste this patch and click on apply

```
into method label B1B4 remove_entry;
into definitionblock code_regex . insert
begin
Method (B1B4, 4, NotSerialized)\n
{\n
    Store(Arg3, Local0)\n
    Or(Arg2, ShiftLeft(Local0, 8), Local0)\n
    Or(Arg1, ShiftLeft(Local0, 8), Local0)\n
    Or(Arg0, ShiftLeft(Local0, 8), Local0)\n
    Return(Local0)\n
}\n
end;
```

Despite on our DSDT.aml sample there's no 32-bit field, basically you need to split it onto 4 8-bit fields.

e.g.

`ABCD, 32` will be splitted onto `ABC0, 8, ABC1, 8, ABC2, 8, ABC3,8` and whenever there's a reference of the field, just call it with `B1B4(ABC0, ABC1, ABC2, ABC3)`.

## What about fields larger than 32 bits?

In our provided DSDT.aml example there are a few fields whose size is larger than 32:

<details><summary>More than 32-bit size larger fields</summary>
<pre>
            
            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD0,   64
            }

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD1,   128
            }

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD2,   192
            }

            Field (ERAM, ByteAcc, NoLock, Preserve)
            {
                Offset (0x04), 
                FLD3,   256
            }
</pre>
</details>
N.B. we omitted `SMD0` as it's commonly found on trackpad device configuration and we don't wanna mess things up :P

For those fields, after we checked that they're referenced inside DSDT.aml, we'll want to check if they're threated as read-only fields or they're written onto too.

Below the found references:

<details><summary>Found references</summary>
<pre>

                        If ((Local3 < 0x09))
                        {
                            Local2 = FLD0 /* \_SB_.PCI0.LPCB.EC0_.FLD0 */
                        }
                        ElseIf ((Local3 < 0x11))
                        {
                            Local2 = FLD1 /* \_SB_.PCI0.LPCB.EC0_.FLD1 */
                        }
                        ElseIf ((Local3 < 0x19))
                        {
                            Local2 = FLD2 /* \_SB_.PCI0.LPCB.EC0_.FLD2 */
                        }
                        Else
                        {
                            Local2 = FLD3 /* \_SB_.PCI0.LPCB.EC0_.FLD3 */
                        }
</pre>
</details>

In this case those fields are used as read-only fields as there's no assignment like `FLD3 = something`.

For handling those fields we'll need to get our hands a little bit dirty: since splitting those fields onto 8-bits will be a tedious process and will make lot of confusion, we'll use another utility method called `RECB` which will do the work for us.

What this method does basically is taking 2 arguments: the field-offset and it's size.
Please note that the field size is expressed in bits, while the field offset in bytes, so you'll need to convert the bits in byte by dividing for 8.

Let's make a quick example with `FLD0` variable: if we give a quick look at its code, there's an offset of `0x04` bytes and then a field of `0x40` (`64/8` in hex) is created.
Moving on the only found reference, we can see that the value of our field will be assigned to a `Local2` variable. As we said before this is a read-only operation, thus we can proceed as it follows:

- rename `FLD0` onto another name (e.g. `LD0A`; double check if it's not used in the DSDT)
- replace the reference by calling `RECB` method with the following syntax `RECB(<offset>, <field_size divided by 8>)`: `Local2 = RECB (0x04, 0x40)`

For the fields who don't have a `Offset(<size>)` declaration right before its declaration, we've gently took the section from RehabMan post:

<details><summary>Expand this section</summary>
<pre>

    Field (ECF2, ByteAcc, Lock, Preserve)
    {
            Offset (0x10), <------ This is the offset declaration. The next field declared right after it will have an offset of 0x10
       BDN0,   56,     //!!0x10
            Offset (0x18)
       BME0,   8,
            Offset (0x20), <-------- Offset of 0x20
       BMN0,   32,     //0x20
       BMN2,   8,     //0x24 <------ In this case we'll have to sum 0x20 to 32/8 (0x4) and we'll have 0x24
       BMN4,   88,    //0x25 <------- In this case we'll have to sum 0x24 to 8/8 (0x1) and we'll have 0x25
       BCT0,   128,     //!! 0x30
       BDN1,   56,     //!! 0x40
            Offset (0x48),
       BME1,   8,
            Offset (0x50),
       BMN1,   32,     //0x50
       BMN3,   8,     //0x54
       BMN5,   88,     //0x55
       BCT1,   128,     //!!0x60

</pre>
</details>

And what about write fields (like `ABCD = something`)?
Let's suppose that `ABCD` is a 256-bit field and we found a reference like the above example. In this case we'll use another utility method called `WECB` (write EC buffer).
Open MaciASL patch menu, copy and paste this patch and click on apply
 
```
into method label WE1B parent_label H_EC remove_entry;
into method label WECB parent_label H_EC remove_entry;
into device label H_EC insert
begin
Method (WE1B, 2, NotSerialized)\n
{\n
    OperationRegion(ERAM, EmbeddedControl, Arg0, 1)\n
    Field(ERAM, ByteAcc, NoLock, Preserve) { BYTE, 8 }\n
    Store(Arg1, BYTE)\n
}\n
Method (WECB, 3, Serialized)\n
// Arg0 - offset in bytes from zero-based EC\n
// Arg1 - size of buffer in bits\n
// Arg2 - value to write\n
{\n
    ShiftRight(Add(Arg1,7), 3, Arg1)\n
    Name(TEMP, Buffer(Arg1) { })\n
    Store(Arg2, TEMP)\n
    Add(Arg0, Arg1, Arg1)\n
    Store(0, Local0)\n
    While (LLess(Arg0, Arg1))\n
    {\n
        WE1B(Arg0, DerefOf(Index(TEMP, Local0)))\n
        Increment(Arg0)\n
        Increment(Local0)\n
    }\n
}\n
end;
```

Then replace each reference with the following syntax:

`WECB(<field_offset>,<field_size>, <store_value>)`

e.g. if `ABCD` field is declared after a `Offset(0x40)` offset, and is large 256 bits and stores a value, you'll write something among these lines:

`WECB(0x40, 0x20, <store_value>)`

What we learn so? Well I know that this topic is still hot and you may have lot of difficult while understanding it and you'll probably won't accomplish anything in a few hours. Dedicate it the time you need understanding this.
As you can see, the whole logic is just a bunch of additions and multiplication/divisions. Try if you can reach the same numbers we got :)
Iterate this whole process for each field whose size is larger than 32-bit (e.g. 56,64,128,256 and so on)


## Finish!

After finishing this nightmare, save the patched DSDT.aml onto `OC/ACPI` folder and declare the DSDT onto your config. Double check that `SMCBatteryManager.kext` is present both in `Kexts` folder and in `config.plist/Kernel/Add` section.
If everything went well you've successfully patched your battery using the static patching method.
If you wanna harm yourself again, check out our hot-patching guide. It's a little bit more tedious, but if you managed to get a working battery status using the static patching method, the road will be downhill :")

## Credits

* **Apple** for macOS
* **Acidanthera** for MaciASL
* **RehabMan** for DSDT patching guide
* **ridiculous_fish** for Hex Fiend
* **every other people that contributed to the hackintosh world :haha:**



