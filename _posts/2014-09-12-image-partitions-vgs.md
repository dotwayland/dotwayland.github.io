---
layout: post
title:  "Imaging Partitions from Volume Groups"
date:   2014-09-14 00:00:01
categories: jekyll update
---

To access individual partitions you must use a write-able image, so only use a COPY of the original image, not the actual initial image.

Use `mmls` to determine the initial partitions.

```
DOS Partition Table

Offset Sector: 0

Units are in 512-byte sectors

Slot   Start       End         Length       Description

00: -----   0000000000   0000000000   0000000001   Primary Table (#0)

01: -----   0000000001   0000000062   0000000062   Unallocated

02: 00:00   0000000063   0000240974   0000240912   Linux RAID (0xFD)

03: 00:01   0000240975   0488375999   0488135025   Linux RAID (0xFD)

04: -----   0488376000   0488397167   0000021168   Unallocated
```

We already know that slot 00:00 is the boot partition and is running ext3. What we’re interested in is slot 00:01 which is a volume group. We will need to mount this locally using losetup. This will involve requiring an offset number. The listing above provides the offset to the beginning of the volume in 512-byte sectors. You will need to multiply the sectors by 512 to get the byte offset.

`losetup /dev/loop0 data001.bin -o 123379200`

Now that it’s mounted we perform a pvscan. Hopefully we’ll see that it had scanned our loop device to find a volume.

```
PV /dev/loop0       VG vg00   lvm2 [232.75 GB / 108.75 GB free]
```

It has found the 'vg00’ volume group. We can now run `lvdisplay` to list the partitions.

```
--- Logical volume ---
  LV Name                /dev/vg00/root
  VG Name                vg00
  LV UUID                d7vKzZ-y6BJ-ypEA-0mTy-NMqa-6CkT-wejIHQI
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                1.00 GB
  Current LE             32
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0

  --- Logical volume ---
  LV Name                /dev/vg00/home
  VG Name                vg00
  LV UUID                I7deX9v-FA0c-MwzE-zlmN-SBQb-cjje-I7deX9v
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                1.00 GB
  Current LE             32
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1

  --- Logical volume ---
  LV Name                /dev/vg00/services
  VG Name                vg00
  LV UUID                d7vKzZ-y6BJ-Uinn-dj94-0mTy-837a-wejIHQI
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                50.00 GB
  Current LE             1600
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:2

  --- Logical volume ---
  LV Name                /dev/vg00/usr
  VG Name                vg00
  LV UUID                d7vKzZ-UgTj-0JZx-7cKH-NxDc-aF83-Md1GTE
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                4.00 GB
  Current LE             128
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:3

  --- Logical volume ---
  LV Name                /dev/vg00/var
  VG Name                vg00
  LV UUID                r2uIPW-N057-yLMO-nH3X-OyGt-XfBf-xs22Iv
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                2.00 GB
  Current LE             64
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:4

  --- Logical volume ---
  LV Name                /dev/vg00/swap
  VG Name                vg00
  LV UUID                V6IVEW-Bc8O-5dx3-0ccF-r1qn-EJDv-d2Egr1
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                16.00 GB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:5
```

We can now start creating images of the individual partitions.

`dcfldd if=/dev/vg00/root of=test.root.bin hash=md5,sha1 md5log=test.root.md5 sha1log=test.root.sha1`

This method should work for systems with volume groups across multiple images.

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
