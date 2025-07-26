*Source* : https://tryhackme.com/room/mbrandgptanalysis
MBR (Master Boot Record) and GPT (GUID Partition Table) are different partitioning schemes that act act as a map for all of the partitions.

The MBR/GPT is located in the very first sector of the disk and contains information about the structure and partitions of the disk.

# Boot Process

The boot process starts by initializing the system's hardware components, loading the operating system into memory, and finally allowing the user to interact with the system. 
![[Pasted image 20250714093306.png]]


## Power-On the System

```
Push the power button which sends electrical signals to the motherboard and initializes all the components. 
```
![[Pasted image 20250714093652.png]]  
[[https://tryhackme-images.s3.amazonaws.com/user-uploads/6645aa8c024f7893371eb7ac/room-content/6645aa8c024f7893371eb7ac-1737110666101.svg]]

## Power-On-Self-Test (POST)

```
The BIOS/UEFI then starts a Power-On-Self-Test to ensure all the system’s hardware components are working fine.
```

## Locate the Bootable Device

```
the BIOS/UEFI locates bootable devices, such as SSDs, HDDs, or USBs, with the operating system installed. Once the bootable device is located, it starts reading this device.
```

*Question*: Which firmware supports a GPT partitioning scheme? 
Re: UEFI

*Question*: Which device has the operating system to boot the system?
Re: Bootable Device

## What if MBR ?

```
The Master Boot Record (MBR) takes up 512 bytes of space at the very first sector of the disk and its signature is represented by **55 AA** which mark the end of the MBR code.
```

These 512 bytes of the MBR are further divided into three portions.

![[Pasted image 20250714101312.png]] 

The structure of the MBR can be seen below
![[Pasted image 20250714101342.png]]

## Bootloader Code (Bytes 0-445)

```
This Bootloader code contains the **Initial Bootloader**. The initial bootloader is the first thing that executes in the MBR. This initial bootloader code has a primary purpose of finding the bootable partition from the **partition table** present on the MBR.

It can be disassembled into assembly language
```

## Partitions Table (Bytes 446-509)

```
This table contains the details of all the partitions present on the disk.
```

An MBR disk has a total of 4 partitions, and each partition is represented by 16 bytes in the partition table.

![[Pasted image 20250714101936.png]]

The screenshot below shows the first partition from the partition table of the MBR. All the bytes have some specific meaning.

![[Pasted image 20250714102228.png]]

The table below shows the fields represented by these bytes.

| Bytes Position | Bytes Length | Bytes       | Fiels Name           |
| -------------- | ------------ | ----------- | -------------------- |
| 0              | 1            | 80          | Boot Indicator       |
| 1-3            | 3            | 20 21 00    | Starting CHS Address |
| 4              | 1            | 07          | Partition Type       |
| 5-7            | 3            | FE FF FF    | Ending CHS Addres    |
| 8-11           | 4            | 00 08 00 00 | Starting LBA Address |
| 12-15          | 4            | 00 B0 23 03 | Number of Sectors    |

### Boot Indicator

```
This byte tells you whether the partition is bootable or not.

It can only have two values *80* (bootable) or *OO* (not bootable)
```

### Starting CHS Address

```
Cylinder Head Sector (CHS) is the 3 bytes that tell you where this partition is starting from on the disk.

It will give you the starting physical address of the partition, such as the cylinder, head, and sector number.

```

### Partition Type

```
Every partition uses a filesystem such as NTFS, FAT32, etc. This byte indicates the filesystem of the partition. 

In our case 07 means NTFS partition

```

[Resources](https://www.writeblocked.org/resources/MBR_GPT_cheatsheet.pdf) to learn about the bytes used for other file systems.

### Ending CHS Address

```
The last 3 bytes at the end of the CHS Address indicates the physical location where the partition ends on the disk.

```

### Starting LBA Address

```
Logical Block Addressing (LBA) is the logical address that indicates the start of the partition.

Its give us the logical address of the partition rather than the physical address (CHS address).

```

The starting LBA address bytes are stored in the little-endian format in which the Least Significant Byte (LSB) is first, and the Most Significant Byte (MSB) is last. *So, we must first reverse these bytes.*

The next step is to convert these into decimal format.

In our case, we have **00 00 08 00** as LBA address bytes, in decimal values ->*2028* and if we multiply it by the size of the sector, which is 512 bytes
 *`2048 * 512 = 1,048,576  `* is the exact value where the partition is stored.
Same also for the size of partition.

## MBR Signature (Bytes 510-511)
```
These two bytes `55 AA` are also known as a **Magic Number** and they indicate that the MBR has been ended now.

```