# What is SLOG or ZIL? And how does data get written onto pools?

## async without SLOG:
A async write, goes into ARC (which is in RAM) into TXGs (transaction groups).
By default, the TXGs have the to goal to collect around 5 seconds and then write that to the pool. I won't use the word cache here, since it is not traditional cache. Yes, I it can speed up writes, since it can collect many writes into one sequential one.
That part of ARC isn't read cache, so that part is NOT shrinkable. ZFS calls this zfs_dirty_data, because it is dirty write data.

This is sometimes important in the Proxmox context. You can fill your RAM with VMs and bet on ARC making room if needed, but it can't shrink that TXGs. So multiply you NIC speed with 5 and you have roughly the amount that isn't shrinkable. Add some cushing to that, this is the amount of RAM you should mentally reserve for ZFS.

## sync write without SLOG:
A sync write is written to TXG and to ZIL at the same time. Since we don't have a SLOG, ZIL is on the pool. Or onto special vdev (we ignore for simplicity).

So again, we write the data to TXG in ARC (RAM) and ZIL (pool) at the same time.
ZFS reports back that the write finished, after it was written to ZIL.
After the TXG is written onto the pool, the data in ZIL gets discarded.
So in a normal operation, ZIL is never ever read from! Never.

Only when the system halts, crashed, looses power, this changes.
During the import process for a zpool, ZFS checks the ZIL for any dirty writes. If it finds some , it will replay them from the ZIL, aggregating them into TXG, and committing the TXG to the pool. So your pool import during boot will take a little bit longer, but everything is fine.

## What happens if you add a SLOG?
ZIL moves from the pool, to that SLOG device. Nothing more. Huge implications though.

## With the knowlege from above applied, what does that mean?
For async, nothing. Async never touched ZIL to begin with.
For sync? ZIL is now on SLOG. So imagine you do a sync write.
The write goes into your ZIL which is on your hopefully extremly fast SLOG. And at the same time into TXG. It will be pretty fast be finished on ZIL and ZFS will report back that the write is finieshed. Even though in reality, it isn't. It is in TXG in RAM. But in case something goes wrong, it can replay it from SLOG. That is the only situation where a read from SLOG happens.
Like above, like for async, the write gets written from TXG onto your pool and after that gets discarded on ZIL/SLOG.

## What makes a good SLOG?
One with fast sync writes. Which is probably one with PLP. Why?
A normal drive can't make use of its DRAM cache for sync writes. Because in case of a power failure, that DRAM data (just like RAM) will be lost.
A PLP drive is a liar. Instead of actually wait until the data is written onto NAND, it lies.
Even thoug the data is dirty in DRAM, the drive will say "yeah, yeah I have written it down, don't you worry!".
But it hasn't. In case something goes wrong, it can use it's condensators to finish the write onto NAND. This approach makes PLP so much faster for sync writes than normal drives.
It also explains why, against popular homelabbers believe, PLP drives are NOT more secure. They cover their ass for a lie, while normal drives simply don't lie*.

With that in mind, one could argue that you don't need to mirror SLOG. Since it is is only read from after a halt. So you would have to have a system halt AND your SLOG do give up to loose data. On the other hand, people that care about speeding up sync writes, probably don't care much about the extra costs of a mirror.

Some AFAIK current defaults:
zfs_txg_timeout: 5s
zfs_dirty_data_max: 10% of physical RAM



* there have been ADATA phision controller firmware bugs in around 2018 or something, where consumer drives lied about sync status. It is extremly rare.
