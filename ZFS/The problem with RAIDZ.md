# The problem with RAIDZ or why you probably won't get the storage efficiency you think you will get.
As a ZFS rookie, I struggled quite a bit to figure out what settings I should use for my Proxmox hypervisor. Hopefully, this will help other ZFS rookies. 
This text focuses on Proxmox, but it generally applies to all ZFS systems.

This entire document assumes an ashift of 12 (4K), which is the default for modern drives!

## TLDR
RAIDZ is only great for sequential reads and writes of large files (>1MB). An example would be a file server that mostly hosts big files.  
For VMs or iSCSI, RAIDZ will not give you the storage efficiency you think you will get, and it will also perform poorly. Use mirror instead. It's a pretty long text, but you can jump to the conclusion and the efficiency tables at the end.  

## Introduction and glossary
Before we start, some ZFS glossary. These are important to understand the examples later on.

### sector size:
older HDDs used to have a sector size of 512b, while newer HDDs have 4k sectors. SSDs can have even bigger sectors, but their firmware controllers are mostly tuned for 4k sectors. There are still enterprise HDDs that come with 512e, where the "e" stands for emulation. These are not 512b but 4k drives, they only emulate to be 512. For this whole text, I assume that we have drives with 4k sectors.

### ashift:
ashift sets the sector size, ZFS should use. ashift is a power of 2, so setting ashift=12 will result in 4k. Ashift must match your drive's sector size. Extremely likely this will be 12 and also automatically detected.

### dataset:
A dataset is inside a pool and is like a file system. There can be multiple datasets in the same pool, and each dataset has its own settings like compression, dedup, quota, and many more. They also can have child datasets that by default inherit the parent's settings. Datasets are useful to create a network share or create a mount point for local files. In Proxmox, datasets are mostly used locally for ISO images, container templates, and VZdump backup files.

### zvol:
zvols or ZFS volumes are also inside a pool. Rather than mounting as a file system, it exposes a block device under /dev/zvol/poolname/dataset. This allows backing up disks of virtual machines or making it available to other network hosts using iSCSI. In Proxmox, zvols are mostly used for disk images and containers.

### recordsize:
Recordsize applies to datasets. ZFS datasets use by default a recordsize of 128KB. It can be set between 512b to 16MB (1MB before openZFS v2.2).

### volblocksize:
Zvols have a volblocksize property that is analogous to recordsize.
Since openZFS v2.2 the default value is 16k. It used to be 8k.

### padding:
ZFS allocates space on RAIDZ vdevs in even multiples of p+1 sectors to prevent unusable-small gaps on the disk. p is the number of parity, so for RAIDZ1 this would be 1+1=2, for RAIDZ2 this would be 2+1=3,for RAIDZ3 this would be 3+1=4.
To avoid these gaps, ZFS will pad out all writes so they’re an even multiple of this p+1 value. Padding is not really writing data onto disks, it just leaves these sectors out.

With that technical stuff out of the way, let's look at real examples :)

## Dataset
Datasets only apply to ISOs, container templates, and VZDump and are not really affected by the RAIDZ problem. You can skip this chapter, but maybe it helps with understanding. 
Let's look at an example of a dataset with the default recordsize of 128k and how that would work. We assume that we want to store a file 128k in size (after compression).

For a 3-disk wide RAIDZ1, the total stripe width is 3.

One stripe has 2 data sectors and 1 parity sector. Each is 4k in size.  
So one stripe has 8k data sectors and a 4k parity sector.  
To store a 128k file, we need 128k / 4k = 32 data sectors.  
To store 32 data sectors, each stripe has 2 data sectors, so we need 16 stripes in total.  
Or you could also say that to store a 128k file, we need 128k / 8k data sectors = 16 stripes.  
Each of these stripes consists of two 4k data sectors and a 4k parity sector.  
In total, we store 128k data sectors (16 stripes * 8k data sectors) and 64k parity sectors (16 stripes * 4k parity sectors).  
Data sectors + parity sectors = total sectors  
128k + 64k = 192k.  
That means we write 192k to store 128k data.  
192k is 48 sectors (192 / 4).  
48 sectors is a multiple of 2, so there is no padding needed.  
128k / 192k = 66.66% storage efficiency.  

This is a best-case scenario. Just like one would expect from a 3-wide RAID5 or RAIDZ1, you "lose" a third of storage.  

Now, what happens if the file is smaller than the recordsize of 128? A 20k file?  

We do the same steps for our 20k file.  
To store 20k, we need 20k / 4k = 5 data sectors.  
To store 5 data sectors, each stripe has 2 data sectors, so we need 2.5 stripes in total.  
Half-data stripes are impossible. That is why we need 3 stripes.  
The first stripe has 8k data sectors and a 4k parity sector.  
Same for the second stripe, 8k data sectors and a 4k parity sector.  
The third stripe is special.  
We already saved 16k of data in the first two sectors, so we only need to save another 4k.  
That is why the third stripe has a 4k data sector and a 4k parity sector.  
In total, we store 20k data sectors (2 times 8k from the first two stripes and 4k from the third stripe) and 12k parity sectors (3 stripes with 4k).  
20k + 12k = 32k.  
That means we write 32k to store 20k data.  
32k is 8 sectors (32 / 4).  
8 sectors is a multiple of 2, so there is no padding needed.  

The efficiency has changed. If we calculate all together, we wrote 20k data sectors, 12k parity sectors.  
We wrote 32k to store a 20k file.  
20k / 32k = 62.5% storage efficiency.  
This is not what you intuitively would expect. We thought we would get 66.66%!  

We do the same steps for a 28k file.  
To store 28k, we need 28k / 4k = 7 data sectors.  
To store 7 data sectors, each stripe has 2 data sectors, so we need 3.5 stripes in total.  
Half-data stripes are impossible. That is why we need 4 stripes.  
The first three stripes have 8k data sectors and a 4k parity sector.  
The fourth stripe is special.  
We already saved 24k of data in the first two sectors, so we only need to save another 4k.  
That is why the fourth stripe has a 4k data sector and a 4k parity sector.  
In total, we store 28k data sectors (3 times 8k from the first three stripes and 4k from the fourth stripe) and 16k parity sectors (4 stripes with 4k).  
28k + 16k = 44k.  
That means we write 44k to store 28k data.  
44k is 11 sectors (44 / 4).  
11 sectors is not a multiple of 2, so there is padding needed.  
We need an extra 4k padding sector to get 12 sectors in total.  

The efficiency has changed again. If we calculate all together, we wrote 28k data sectors, 16k parity sectors, and one 4k padding sector.  
We wrote 48k to store a 28k file.  
28k / 48k = 58.33% storage efficiency.  
This is not what you intuitively would expect. We thought we would get 66.66%!  

What happens if we want to save a 4k file?  

We calculate the same thing for a 4k file.  
We simply store a 4k data sector on one disk and one parity sector on another disk. In total, we wrote a 4k data sector and a 4k parity sector.  
We wrote 8k in sectors to store a 4k file.  
4k / 8k = 50% storage efficiency.  

This is the same storage efficiency we would expect from a mirror!  

**Conclusion for datasets:**  
If you have a 3-wide RAIDZ1 and only write huge files like pictures, movies, and songs, the efficiency loss gets negligible. For 4k files, RAIDZ1 only offers the same storage efficiency as mirror.  

## ZVOL and volblocksize
For Proxmox we mostly don't use datasets though. We use VMs with RAW disks that are stored on a Zvol.  
For Zvols and their fixed volblocksize, it gets more complicated.  

In the early days, the default volblocksize was 8k and it was recommended to turn off compression. Until very recently (2024) Proxmox used 8k with compression as default.
Nowadays, it is recommended to enable compression and the current default is 16k since OpenZFS v2.2. Some people in the forum even recommend going as high as 64k on SSDs.

In theory, you want to have writes that exactly match your volblocksize.  
For MySQL or MariaDB, this would be 16k. But because you can't predict compression, and compression works very well for stuff like MySQL, you can't predict the size of the writes.  
A larger volblocksize is good for mostly sequential workloads and can gain compression efficiency.  
Smaller volblocksize is good for random workloads, has less IO amplification, and less fragmentation, but will use more metadata and have worse space efficiency.  
We look at the different volblocksizes and how they behave on different pools.  

### volblocksize 16k
This is the default size for openZFS since 2.2.  

#### RAIDZ1 with 3 drives
With 3 drives, we get a stripe that is 3 drives wide.  
Each stripe has two 4k data sectors (8k) and one 4k parity sector.  
For a volblock of 16k, we need two stripes, because one stripe stores 8k data and two stripes will store the needed 16k (16k/8k).  
Each stripe has two 4k data sectors, two stripes are in total 16k. 
Each stripe has one 4k parity sector, two stripes are in total 8k.  
That gets us to 24k in total to store 16k.  
24k is 6 sectors and that can be divided by 2 so there is no padding needed.  
Storage efficiency is 66.66%, as expected.  

#### RAIDZ1 with 4 drives
With 4 drives, we get a stripe 4 drives wide.  
Each stripe has three 4k data sectors (12k) and one 4k parity sector.  
For a volblock of 16k, we need 1.33 stripes (16k/12k).  
The first stripe has three 4k data sectors, in total 12k.  
The first stripe also has one 4k sector for parity. 
The second stripe has one 4k data sector.  
The second stripe also has one 4k sector for parity.  
In total, we have four 4k data sectors and two 4k parity sectors.  
That gets us to 24k in total to store 16k.  
24k is 6 sectors and that can be divided by 2 so there is no padding needed.  
We expected a storage efficiency of 75%, but only got 66.66%!

#### RAIDZ1 with 5 drives
With 5 drives, we get a stripe 5 drives wide.  
Each stripe has four 4k data sectors (16k) and one 4k parity sector.  
For a volblock of 16k, we need 1 stripe (16k/16k).  
In total, we have four 4k data sectors and one 4k parity sector.  
That gets us to 20k in total to store 16k.  
20k is 5 sectors and that can't be divided by 2 so there is an additional padding sector needed.  
That gets us to 24k in total to store 16k.  
We expected a storage efficiency of 80%, but only got 66.66%!

#### RAIDZ1 with 10 drives
With 10 drives, we get a stripe 10 drives wide.  
That 10 drives wide stripe in theory would get us 9 data sectors and one parity sector.  
A single stripe could thus hold 9 * 4k = 36k.  
But that is of no use to us, we only need 16k!  
ZFS will shorten the stripes.  
For a volblock of 16k, we need one stripe with 4 data sectors and one parity sector.  
In total, we have four 4k data sectors and one 4k parity sector.  
The stripe is only 5 drives wide.  
That gets us to 20k in total to store 16k.  
20k is 5 sectors and that can't be divided by 2 so there is an additional padding sector needed.  
That gets us to 24k in total to store 16k.  
We expected a storage efficiency of 90%, but only got 66.66%!  

**Notice something? No matter how wide we make the RAIDZ1, there are no efficiency gains beyond 5 drives. This is because we can't make the stripe any wider, no matter how wide we make your RAIDZ1. Because of padding, even going from 4 to 5 drives wide does not help with storage efficiency.**  

#### RAIDZ2 with 4 drives
With 4 drives, we get a stripe 4 drives wide.  
Each stripe has two 4k data sectors and two 4k parity sectors.  
For a volblock of 16k, we need two stripes (16k/8k = 2).  
That gets us to 32k in total to store 16k.  
32k is 8 sectors and that can't be divided by 3 so there is padding needed.  
We need another padding sector to get to 9 sectors total.  
9 sectors can be divided by 3.  
That gets us to 36k in total to store 16k.  
We expected a storage efficiency of 50%, but only got 44.44%!  
That is WORSE than mirror!  

#### RAIDZ2 with 5 drives  
With 5 drives, we get a stripe 5 drives wide.  
Each stripe has three 4k data sectors and two 4k parity sectors.  
For a volblock of 16k, we need 1.33 stripes (16k/12k).  
The first stripe has three 4k data sectors, in total 12k.  
The first stripe also has two 4k sectors for parity.  
The second stripe has one 4k data sector.  
The second stripe also has two 4k sectors for parity.  
In total, we have four 4k data sectors and four 4k parity sectors.  
That gets us to 32k in total to store 16k.  
32k is 8 sectors and that can't be divided by 3 so there is padding needed.  
We need another padding sector to get to 9 sectors total.  
9 sectors can be divided by 3.  
That gets us to 36k in total to store 16k.   
We expected a storage efficiency of 60%, but only got 44.44%!  
That is WORSE than mirror!

#### RAIDZ2 with 6 drives  
With 6 drives, we get a stripe 6 drives wide.  
Each stripe has four 4k data sectors and two 4k parity sectors.  
For a volblock of 16k, we need one stripe (16k/16k = 1).  
That gets us to 24k in total to store 16k.  
24k is 6 sectors and that can be divided by 3 so there is no padding needed.  
We expected a storage efficiency of 66.66%, and got 66.66%!  

#### RAIDZ2 with 10 drives
With 10 drives, we get a stripe 6 drives wide.   
This is because we don't need 10 drives to store 16k.  
It also behaves exactly as with 6 drives.   
Each stripe has four 4k data blocks and two 4k parity blocks.  
For a volblock of 16k, we need one stripe (16k/16k = 1).  
That gets us to 24k in total to store 16k.  
We expected a storage efficiency of 80%, but only got 66%!  

### volblocksize 64k
Some users in the forums recommend 64k on SSDs.  
I am no VM expert by any means, but there are still a lot of workloads that are smaller than 64k.  
I would recommend using volblocksize 64k with caution. You will could get huge read-write amplification and fragmentation.  

#### RAIDZ1 with 3 drives
With 3 drives, we get a stripe 3 drives wide.  
Each stripe has two 4k data sectors (8k) and one 4k parity sector.  
For a volblock of 64k, we need eight stripes (64k/8k = 8).  
Each stripe has two 4k data sectors, eight stripes are in total 64k. 
Each stripe has one 4k parity sector, eight stripes are in total 32k. 
That gets us to 96k in total to store 64k.  
96k is 24 sectors and that can be divided by 2 so there is no padding needed.  
Storage efficiency is 66.66%, as expected.   

#### RAIDZ1 with 4 drives
With 4 drives, we get a stripe 4 drives wide.  
But not all stripes have three 4k data sectors (12k) and one 4k parity sector.  
For a volblock of 64k, we need 5.33 stripes (64k/12k).  
Five stripes have three 4k data sectors, in total 60k.  
Five stripes also have one 4k sector for parity, in total 20k. 
The sixth stripe has one 4k data sector.  
The sixth stripe also has one 4k sector for parity.  
In total, we have sixteen 4k data sectors and six 4k parity sectors.  
That gets us to 88k in total to store 64k.  
88k is 22 sectors and that can be divided by 2 so there is no padding needed.  
We expected a storage efficiency of 75%, but only got 72.72%!

#### RAIDZ1 with 5 drives
With 5 drives, we get a stripe 5 drives wide.  
For a volblock of 64k, we need 4 stripes (64k/16k).  
Each stripe has four 4k data sectors and one 4k parity sector.  
In total, we have 16 4k data sectors and 4 4k parity sectors.  
That gets us to 80k in total to store 64k.  
80k is 20 sectors and that can be divided by 2 so there is no padding needed.  
Storage efficiency is 80%, as expected.  

#### RAIDZ1 with 6 drives
With 6 drives, we get a stripe 6 drives wide.  
But not all stripes have five 4k data sectors (20k) and one 4k parity sector.  
For a volblock of 64k, we need 3.2 stripes (64k/20k).  
Three stripes have five 4k data sectors, in total 60k.  
Three stripes also have one 4k sector for parity, in total 12k. 
The fourth stripe has one 4k data sector.  
The fourth stripe also has one 4k sector for parity.  
In total, we have 16 4k data sectors and four 4k parity sectors.  
That gets us to 80k in total to store 64k.  
80k is 20 sectors and that can be divided by 2 so there is no padding needed.  
We expected a storage efficiency of 83.33%, but only got 80%!
The same problem applies for 7 or 8 drives wide! 

#### RAIDZ1 with 9 drives
With 9 drives, we get a stripe 9 drives wide.  
For a volblock of 64k, we need two stripes (64k/32k). 
Each stripe has eight 4k data sectors and one 4k parity sector.  
In total, we have 16 4k data sectors and two 4k parity sectors.  
That gets us to 72k in total to store 64k.  
72k is 18 sectors and that can be divided by 2 so there is no padding needed.  
Storage efficiency is 88.88%, as expected.

#### RAIDZ1 with 10 drives
With 10 drives, we get a stripe 10 drives wide.  
But not all stripes have nine 4k data sectors (36k) and one 4k parity sector.  
For a volblock of 64k, we need 1.77 stripes (64k/36k). 
First stripe has nine 4k data sectors, in total 36k.  
First stripe also has one 4k sector for parity. 
The second stripe has seven 4k data sectors, in total 28k .  
The second stripe also has one 4k sector for parity.  
In total, we have 16 4k data sectors and two 4k parity sectors.  
That gets us to 72k in total to store 64k.  
72k is 18 sectors and that can be divided by 2 so there is no padding needed.  
We expected a storage efficiency of 90%, but only got 88.88%!
The same problem applies for all RAIDZ1 that are wider than 10 drives!  

#### RAIDZ2 with 6 drives
With 6 drives, we get a stripe 6 drives wide.  
Each stripe has four 4k data sectors and two 4k parity sectors.  
For a volblock of 64k, we need four stripes (64k/16k = 4).  
That gets us to 96k in total to store 64k.  
96k is 24 sectors and that can be divided by 3 so there is no padding needed.  
Storage efficiency is 66.66%, as expected.  

#### RAIDZ2 with 7 drives
With 7 drives, we get a stripe 7 drives wide.  
But not all stripes have five 4k data sectors (20k) and two 4k parity sectors.  
For a volblock of 64k, we need 3.2 stripes (64k/20k).  
Three stripes have five 4k data sectors, in total 60k.  
Three stripes also have two 4k sectors for parity, in total 24k. 
The fourth stripe has one 4k data sector.  
The fourth stripe also has two 4k sectors for parity.  
In total, we have 16 4k data sectors and eight 4k parity sectors.  
That gets us to 96k in total to store 64k.  
96k is 24 sectors and that can be divided by 3 so there is no padding needed.  
We expected a storage efficiency of 71.42%, but only got 66.66%!
Same is true for 8 or 9 wide RAIDZ2!  

#### RAIDZ2 with 10 drives
With 10 drives, we get a stripe 10 drives wide.  
Each stripe has eight 4k data sectors and two 4k parity sectors.  
For a volblock of 64k, we need two stripes (64k/32k = 2).  
That gets us to 80k in total to store 64k.  
80k is 20 sectors and that can't be divided by 3 so there is padding needed.  
We add a padding sector to get 21 sectors in total. 
That gets us to 84k in total to store 64k.  
We expected a storage efficiency of 80%, but only got 76.19%!
The same problem applies to all RAIDZ between 10 and 17 drives wide!  

## efficiency tables
Efficiency tables for different numbers of drives, with 16k or 64k volblocksize, and what efficiency you would naturally expect to get. When expectations match up, it is formatted in bold.  

## RAIDZ1

|          | 3 drives   | 4 drives | 5 drives | 6 drives | 7 drives | 8 drives | 9 drives   | 10 drives | 11 drives | 12 drives |
|----------|------------|----------|----------|----------|----------|----------|------------|-----------|-----------|-----------|
| 16k      | **66.66%** | 66.66%   | 66.66%   | 66.66%   | 66.66%   | 66.66%   | 66.66%     | 66.66%    | 66.66%    | 66.66%    |
| 64k      | **66.66%** | 72.72%   | **80%**  | 80%      | 80%      | 80%      | **88.88%** | 88.88%    | 88.88%    | 88.88%    |
| expected | **66.66%** | 75%      | **80%**  | 83.33%   | 85.71%   | 87.5%    | **88.88%** | 90%       | 90.90%    | 91.66%    |

## RAIDZ2
|          | 4 drives | 5 drives | 6 drives   | 7 drives | 8 drives | 9 drives | 10 drives | 11 drives | 12 drives | 13 drives | 14 drives | 15 drives | 16 drives | 18 drives  |
|----------|----------|----------|------------|----------|----------|----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|------------|
| 16k      | 44.44%   | 44.44%   | **66.66%** | 66%      | 66%      | 66%      | 66%       | 66%       | 66%       | 66%       | 66%       | 66%       | 66%       | 66%        |
| 64k      | 48.48%   | 53.33%   | **66.66%** | 66.66%   | 66.66%   | 66.66%   | 76.19%    | 76.19%    | 76.19%    | 76.19%    | 76.19%    | 76.19%    | 76.19%    | **88.88%** |
| expected | 50%      | 60%      | **66.66%** | 71.42%   | 75%      | 77.77%   | 80%       | 81.81%    | 83.33%    | 84.61%    | 85.71%    | 87.5%     | 88.23%    | **88.88%** |

## Conclusion
RAIDZ is different from traditional RAID and often has worse storage efficiency than expected.  
You can get better space efficiency and compression gains with a larger volblocksize, 
but it will come at the cost of read and write amplification and create higher fragmentation. 
Remember that all these variants will only write as fast as the slowest disk in the pool.
Mirrors have a worse storage efficiency but will offer twice the write performance with 4 drives and 4 times the write performance with 8 drives over a RAIDZ pool.  
**Use NVMe (or at least SSD) mirrors for Zvols**  
**Use RAIDZ for huge, sequentially written and read files in datasets**
