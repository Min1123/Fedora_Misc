# Overview
--------
Short instructions for making a Fedora 42 disk image that can boot a serial console on a Star64 board from Pine64 from the Generic Server disk image
Star64 has firmware blobs for 


# Images Needed
-------------
661136d2c464a12d00cfe2508bacc9bb88a721cfe1b2f68e1b202113659c8e61  star64-image-minimal-star64.wic.bz2 (https://github.com/Fishwaldo/meta-pine64/releases/tag/v2.1)

56d17d7dbda2d2cd2dbe1720f63d3c2b8aeedcd14afc37a482d2ee4e5247da84  Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw.xz (https://dl.fedoraproject.org/pub/alt/risc-v/release/42/Server/riscv64/images/Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw.xz)



# Partition Tables
----------------

## Initial Fishwaldo
Device      Start     End Sectors   Size Type

sf.img1      4096    8191    4096     2M HiFive BBL

sf.img2      8192   16383    8192     4M HiFive FSBL

sf.img3     16384  793803  777420 379,6M Microsoft basic data

sf.img4    794624 4801763 4007140   1,9G Linux filesystem

## Initial Fedora
Device                                                               Start      End  Sectors  Size Type

./Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw1    2048  1026047  1024000  500M EFI System

./Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw2 1026048  3074047  2048000 1000M Linux extended boot

./Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw3 3074048 20971486 17897439  8,5G Linux root (RISC-V-64)

## Combined table
Device       Start      End  Sectors  Size Type

sf.img1       4096     8191     4096    2M HiFive BBL

sf.img2       8192    16383     8192    4M HiFive FSBL

sf.img3      16384    18431     2048    1M Microsoft reserved

sf.img4      18432  1042431  1024000  500M EFI System

sf.img5    1042432  3090431  2048000 1000M Linux extended boot

sf.img6    3090432 20987870 17897439  8,5G Linux root (RISC-V-64)


# Making the Image
----------------

```
# Extract the BBL and FSBL from the Fishwaldo image
bzcat star64-image-minimal-star64.wic.bz2 | dd count=16384 of=sf.img

# Add the Fedora image to the end
xzcat ~/Downloads/Fedora-Server-Host-Generic-42.20250414-8635a3a5bfcd.riscv64.raw.xz >> sf.img

# Regenerate the partitions for the new image
sudo parted -a none -s sf.img -- mklabel gpt unit s mkpart primary 4096 8191 mkpart primary 8192 16383 mkpart primary 16384 18431 mkpart primary 18432 1042431 mkpart primary 1042432 3090431 mkpart primary 3090432 20987870 type 1 2E54B353-1271-4842-806F-E436D6AF6985 type 2 5B193300-FC78-40CD-8002-E86C45580B47 type 3 E3C9E316-0B5C-4DB8-817D-F92DF00215AE type 4 C12A7328-F81F-11D2-BA4B-00A0C93EC93B type 5 BC13C2FF-59E6-4262-A352-B275FD6F7172 type 6 72EC70A6-CF74-40E6-BD49-4BDA08E8F224

# Mount the boot partition and add the devicetree to the grub.cfg
mkdir mnt
sudo mount -o offset=$((512*1042432)) sf.img mnt
echo -ne "devicetree /dtb/starfive/jh7110-pine64-star64.dtb\n" |  sudo tee -a mnt/loader/entries/5654299c6cb54f958a607c5d2e38622c-6.13.0-0.rc4.36.0.riscv64.fc42.riscv64.conf
```

