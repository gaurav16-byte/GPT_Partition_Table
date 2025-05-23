# GPT (GUID Partition Table)

## 1. Summary

GPT (GUID Partition Table) is a standard for the layout of partition tables of a physical computer storage device. It is part of the Unified Extensible Firmware Interface (UEFI) standard. It has several advantages over Master Boot Record (MBR) partition tables, such as support for more than four primary partitions (128) and 64-bit rather than 32-bit logical block addresses (LBA) for blocks on a storage device. The larger LBA size supports larger disks (more than 2TB).

GPT uses universally unique identifiers (UUIDs), which are also known as globally unique identifiers (GUIDs), to identify partitions and partition types. Using LBAs (sectors) rather than Cylinder Head and Sectors (CHS) to divide the partitions makes them way faster in comparison to that in MBR schemes.

GPT consists of several sections:

| Section                     | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Protective MBR              | Master Boot Record used for compatibility with older BIOS-based systems.     |
| Primary GPT Header          | Contains metadata about the disk layout and partition table.                |
| Partition Entry Array       | Contains the details of each partition on the disk.                         |
| Backup GPT Header           | A redundant copy of the Primary GPT Header, located at the end of the disk. |
| Backup Partition Entry Array | A redundant copy of the Partition Entry Array, located at the end of the disk.|

<br>

**1. Protective MBR:** This section is located at the very beginning of the disk and serves the following purposes:

* *i.* It exists for compatibility with legacy systems that don't understand GPT.
* *ii.* Contains a single partition entry that spans the entire disk, marking it as in-use, so older tools don’t overwrite GPT disks by mistake.

**2. Primary GPT Header:** This holds critical metadata about the disk and partitions. It includes:

* *i.* Disk GUID: A unique identifier for the entire disk.
* *ii.* Pointer to the Partition Entry Array: A reference indicating where the partition table begins.
* *iii.* Checksum (CRC32): Ensures the integrity of the header data and allows detection of corruption.
* *iv.* Backup GPT Header Location: Identifies the location of the backup header at the end of the disk.

**3. Primary Partition Entry Array:** It contains detailed information about each partition on the disk. Key details stored here include:

* *i.* Partition Type GUID: Specifies the type of partition. Ex. EFI System Partition, Linux Filesystem, etc.
* *ii.* Partition GUID: A unique identifier for the specific partition.
* *iii.* Starting and Ending Addresses: Defines the location of the partition on the disk.
* *iv.* Attributes: Flags for the partition Ex. bootable, hidden etc.
* *v.* The number of entries in the array varies by operating system but is typically 128 entries on Windows.

**4. Backup Partition Entry Array:** The Backup Partition Entry Array mirrors the primary array to ensure redundancy and data recovery. It’s stored near the end of the disk, providing an alternative in case the primary array is damaged.

**5. Backup GPT Header:** Located at the very end of the disk, the Backup GPT Header is a replica of the primary GPT header. Its role is to:

* *i.* Facilitate recovery by providing key disk metadata.
* *ii.* Verify integrity with its own CRC32 checksum.
* *iii.* Restore corrupted data from the backup partition table.

## 2. Protective MBR

```
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 79 C3 BB B6 00 00 00 00
02 00 EE FE FF 33 01 00 00 00 FF FF FF FF 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 55 AA
```
This is what the Protective MBR looks like. Entirely filled with 0s, the FE on 4th last row and 4th column means that the current drive supports GPT partition scheme. The last two bytes 55AAh mean that this is an MBR sector.

## 3. Primary GPT Header

The GPT header starts right from the next byte (the start of sector 1) after the Protective MBR ends at ```55 AA``` (the end of sector 0). It acts as a blueprint of the partitions on the disk. All the bytes in the GPT header have a specific meaning. Below is a screenshot of the GPT header with these bytes highlighted with different colors. It has the whole 512 sector space but occupies the first 92 bytes of space, and after these 92 bytes, there would be all ```00``` bytes for padding purposes to complete the sector's total bytes.
<br>

The table of the byte distribution in the Header is as follows:

| Bytes Position | Bytes Length | Bytes                                  | Field Name                |
|----------------|--------------|----------------------------------------|---------------------------|
| 0-7            | 8            | 45 46 49 20 50 41 52 54               | Signature                 |
| 8-11           | 4            | 00 00 01 00                           | Revision                  |
| 12-15          | 4            | 5C 00 00 00                           | Header Size               |
| 16-19          | 4            | DF B3 48 24                           | CRC32 of Header           |
| 20-23          | 4            | 00 00 00 00                           | Reserved                  |
| 24-31          | 8            | 01 00 00 00 00 00 00 00               | Current LBA               |
| 32-39          | 8            | AF 12 9E 3B 00 00 00 00               | Backup LBA                |
| 40-47          | 8            | 22 00 00 00 00 00 00 00               | First Usable LBA          |
| 48-55          | 8            | 8E 12 9E 3B 00 00 00 00               | Last Usable LBA           |
| 56-71          | 16           | F5 F3 1B 19 A5 7D A6 4E A7 E2 75 71 A9 34 69 BD | Disk GUID                 |
| 72-79          | 8            | 02 00 00 00 00 00 00 00               | Partition Entry Array LBA |
| 80-83          | 4            | 80 00 00 00                           | Number of Partition Entries |
| 84-87          | 4            | 80 00 00 00                           | Size of Each Partition Entry |
| 88-91          | 4            | C5 C0 45 85                           | CRC32 of Partition Array  |

<br>

![Primary GPT Header](image.png)

1.	**SIGNATURE:** This is used to recognize the GPT Header. Where ```45 46 49 20 50 41 52 54``` translates to ```EFI Part``` and is always at the start of the GPT partition.
2.	**REVISION:** This represents the GPT Version. In this case it is ```00 00 01 00``` which means 1.0. 
3.	**HEADER SIZE:** Represents the size of the GPT Header. It is always set to ```5C 00 00 00``` which translates to 92 in decimal.
4.	**HEADER CRC32:** CRC32 checksum of the GPT header, If it is different from the original one, then it would mean that the GPT is tampered with or corrupted. This is calculated after calculating the partition array checksum. ```DF B3 48 24``` in this case.
5.	**RESERVED:** Reserved bytes that can be used in the future in case of any updates or future changes. ```00 00 00 00``` in this case.
6.	**CURRENT LBA:** Indicates the location of the GPT header ```01 00 00 00 00 00 00 00```, converting this into decimal would be 1, which means 1st sector.
7.	**BACKUP LBA:** This field shows where the backup GPT Header is stored in the hard drive. The exact sector number. ```AF 12 9E 3B 00 00 00 00``` converting this to decimal would result in 1,000,215,215.
8.	**FIRST USABLE LBA:** Indicates the first address from which the partition can start on the disk. ```22 00 00 00 00 00 00 00```. This translates to sector 34.
9.	**LAST USABLE LBA:** The last address to which the partitions can be written. In this case it is ```8E 12 9E 3B 00 00 00 00``` which means sector 1,000,215,182 is the last partition where any data of the hard drive lies. Any partition cannot occupy any space after their last usable LBA(Logical Block Address).
10.	**DISK GUID:** This represents the globally unique identifier of the disk. The GUID bytes in this case are ```F5 F3 1B 19 A5 7D A6 4E A7 E2 75 71 A9 34 69 BD```. This translates to ```191BF3F5-7DA5-4EA6-A7E2-7571A93469BD```. The proper translation from hexadecimal to GUID will be explained in a later section.

11.	**PARTITION ENTRY ARRAY LBA:** Start of the partition entry array. Translating ```02 00 00 00 00 00 00 00``` to decimal means the entry partition starts from sector 2.
12.	**NUMBER OF PARTITION ENTRIES:** Number of partitions on the disk. ```80 00 00 00``` which means 128 partitions.
13.	**SIZE OF EACH PARTITION:** Size occupied by each partition entry array. It is set to ```80 00 00 00``` which means 128 bytes of partition data.
14.	**CRC32 PARTITION ARRAY:** CRC32 checksum of the whole partition entry array. ```C5 C0 45 85``` this is the CRC32 checksum of the entire partition array of 32 sectors.

Rest of the bytes in sector 1 are filled with zeroes from byte 93 to 512.

## 4. Partition Entry Array

*	Sector 0 was occupied by the Protective MBR.
*	Sector 1 was occupied by the Primary GPT Header.
*	Sector 2 onwards will be occupied by the Partition Entry Array.
* There are a total of 128 partitions on a GPT disk, and this partition entry array contains information about all these partitions.
*	Each partition entry is represented by 128 bytes.
*	So, there are a total of 32 sectors that will have the partition information. If the number of partitions is less than the 128 partitions, then the rest of the 128*x bytes will be filled with zeroes.

The partition tables in a single sector look like this:

![4 partitions in a single sector](image-1.png)

The partition table would be like:

| Bytes Position | Bytes Length | Bytes                                             | Field Name            |
|----------------|--------------|---------------------------------------------------|-----------------------|
| 0-15           | 16           | 28 73 2A C1 1F F8 D2 11 BA 4B 00 A0 C9 3E C9 3B | Partition Type GUID   |
| 16-31          | 16           | EE 7E 30 99 70 E1 98 42 B5 5E E8 21 8E AD 24 B5 | Unique Partition GUID |
| 32-39          | 8            | 00 08 00 00 00 00 00 00                           | Starting LBA          |
| 40-47          | 8            | FF 47 1F 00 00 00 00 00                           | Ending LBA            |
| 48-55          | 8            | 00 00 00 00 00 00 00 80                           | Attributes            |
| 56-127         | 72           | 45 00 46 00 49 00 20 00 73 00 79 00 73 00 74 ...    | Partition Name        |

Partition entry array byte distribution:

![artition entry array byte distribution](image-2.png)

All the group of bytes in the partition entry array of the GPT have specific meanings:

1.	**PARTITION TYPE GUID:** The GUID indicates the partition type. They are stored with the mixed little-endian format. To fix this we can:
```28 73 2A C1 1F F8 D2 11 BA 4B 00 A0 C9 3E C9 3B```
* Reverse first 4 bytes: ```C1 2A 73 28```
* Reverse next 2 bytes: ```F8 1F```
* Reverse next 2 bytes: ```11 D2```
* Keep next 2 bytes as it is: ```BA 4B```
* Keep next 6 bytes as it is: ```00 A0 C9 3E C9 3B```
* This GUID appears as: ```C12A7328- F81F-11D2-BA4B-00A0C93EC93B```

2.	**UNIQUE PARTITION GUID:** Unique Partition GUID is used to distinguish partitions on a disk. It is a unique GUID that is given to all the partitions on the disk. They follow the same conversion steps as partition type GUID. 
Similarly, for this GUID: ```EE 7E 30 99 70 E1 98 42 B5 5E E8 21 8E AD 24 B5```

* This unique GUID appears as: ```99307EEE-E170-4298-B55E-E8218EAD24B5```

3.	**STARTING LBA:** Indicates the sector from where this partition starts on the disk. The bytes in this place look like ```00 08 00 00 00 00 00 00```. All this data is in little-endian. After converting this hexadecimal data to decimal, we get 2048. This means that this partition starts from sector number ```2048```.

4.	**ENDING LBA:** Indicates the sector at which this partition is ending on the disk. The bytes in this place look like ```FF 47 1F 00 00 00 00 00```. Again, all this data is in little-endian which when converted from hexadecimal to decimal, results to ```2050047```. Meaning, that the partition ends at sector 2050047.

5.	**ATTRIBUTES:** Flags that tell whether the partition is bootable, hidden or normal. The bytes are like ```00 00 00 00 00 00 00 00```. These 8 bytes are a total of 64 bits. ```1 byte = 8 bits```. The attributes are further broken down into:

| Bit(s)  | Usage                                                                 |
|---------|-----------------------------------------------------------------------|
| 0       | Partition is required by the platform to function properly.          |
| 1       | EFI firmware should ignore the content and not attempt to read.      |
| 2       | Marks the partition as bootable in legacy BIOS systems.             |
| 3-47    | Reserved for future use.                                            |
| 48      | Marks the partition as read-only (prevents modifications).         |
| 49      | Indicates that the partition is a shadow copy or backup.            |
| 50      | Hides the partition from the operating system (invisible to users). |
| 51      | Prevents the operating system from automatically mounting.          |
| 52-63   | Reserved for use by the partition type or future extensions.        |

6.	**PARTITION NAME:** It represents the name of the partition in string format and is UTF-16 encoded. When the data of the last 72 bytes is passed through a hexadecimal to a text convertor, the data we get for this particular partition is:
```EFI system partition```

## 4. Backup Partition Entry Array

The Backup Partition Entry Array mirrors the primary array to ensure redundancy and data recovery. It’s stored near the end of the disk, providing an alternative in case the primary array is damaged. The calculation and everything to this is also similar to that of the Primary Partition array.

## 5. Backup GPT Header
This is located at the very end of the disk, the Backup GPT Header is a replica of the primary GPT header. Again for the backup GPT Header too the calculation and all the byte distribution is the same. So, no need to cover the same thing over and over again.

## Code Output

```
This is a GPT Drive

Signature: EFI PART
Header Length: 92 bytes
Header Checksum: 2448B3DF
Current sector: 1
Backup header sector: 1000215215
First drive sector: 34
Last drive sector: 1000215182
Disk GUID: 191BF3F5-7DA5-4EA6-A7....
Partition Entry sector: 2
Entries: 128
Size of each entry: 128
Partition Checksum: 8545C0C5

Header Checksum match, therefore everything is intact.
Partition checksums match, therefore partition array is intact.

Details of Partition 1:
Partition type GUID: C12A7328-F81F-11D2-BA4B...
Unique Partition GUID: 99307EEE-E170-4298-B55E...
Starting Sector: 2048
Ending Sector: 2050047
Attributes: 0000000000000000
Partition Name: EFI system partition

Details of Partition 2:
Partition type GUID: E3C9E316-0B5C-4DB8-817D...
Unique Partition GUID: C82BFD99-9279-4B95-AE43...
Starting Sector: 2050048
Ending Sector: 2312191
Attributes: 0000000000000000
Partition Name: Microsoft reserved partition

Details of Partition 3:
Partition type GUID: DE94BBA4-06D1-4D40-A16A...
Unique Partition GUID: AEFF9399-38F0-43F0-8C02...
Starting Sector: 2312192
Ending Sector: 4360191
Attributes: 8000000000000001
Partition Name: Basic data partition

Details of Partition 4:
Partition type GUID: EBD0A0A2-B9E5-4433-87C0...
Unique Partition GUID: 4BBA5413-FA2F-480D-BF58...
Starting Sector: 4360192
Ending Sector: 1000214527
Attributes: 0000000000000000
Partition Name: Basic data partition
```