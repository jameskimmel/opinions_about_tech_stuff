Apple switched to APFS for their machines. APFS is a copy-on-write filesystem.  
This has possible technical implications. 

- Backing up to another APFS destination (like a Time Machine backup disk) could offer huge performance improvements. We could see almost instant backups, just like we see with Snapshots in ZFS. So what performance do we see with APFS?  
- In the beginning of APFS, there were lots of reports about how HDDs would perform very poorly and APFS was only suitable for SSDs. It is unclear to me if this was just an initial performance bug or if this has to do with SMR drives behaving poorly with CoW.  

So let’s find out. To have a comparison, I ran multiple tests.  
First is the initial backup time. This isn't really important IMHO, since this is only done once. 
Second run is a second run without any changes.   
This is a pretty important metric, since this simulates the normal behavior of your daily usage. Time Machine backups every hour, so this metric is pretty important.  
The third run is with the disk being almost full (only 20GB left). For that, I will fill up the drive with random dates.  
Some SSDs have some kind of pseudo SLC cache, and some SMR HDDs in combination with CoW suffer when not having free space.  
With that third test, we get a more realistic result of how Time Machine will behave a year down the road when the backup got filled. What this test won't show is how fragmentation affects performance.
I don't know of a good way to simulate fragmentation, and I am also not sure if that even is a big issue for Time Machine or if it mostly writes sequentially. Just like with ZFS, there are no tools to defrag APFS.  
In all these tests, I will wait 60 minutes between tests. The reason for that is that SSDs and HDDs sometimes do some internal work to speed things up. In real life, we also have 60 minutes between backups where the disks have nothing else to do. 
For good measure, I will throw in my current setup that backs up over SMB to a NAS.  
All tests will be run with an encrypted Time Machine setup.  


System to back up:  
macMini with a 500GB SSD. The data to back up is roughly 100GB. macOS is 15.4.1.

Old TimeMachine setup:  
My old setup was a TimeMachine backup of my TrueNAS system over 10GBit/s SMB. The TrueNAS system has a 48TB pool (8 drives in a RAIDZ2 and a special vdev mirror for metadata performance). 
Initial backup time -> unknown  
Second run without any changes -> 20min  
Third run with a 90% full disk -> can't simulate that  
Conclusion -> I don't know why it takes so long. SMB is a complicated beast, maybe there are some bottlenecks there. TrueNAS shows no CPU usage, no disk busy, nothing. Same goes for the macMini.  
So I don't know where the bottleneck is, I just know it is very slow but also easy on the system. 

HDD:  
Lacie Porsche Design 2TB SSD 2.5" HDD with USB 3.0.  
Initial backup time -> 21min  
Second run without any changes -> 1min 22s  
Third run with a 90% full disk -> 1min 30s  
Conclusion -> Even a cheap and slow HDD is way better than my old SMB solution. It is perfectly fine performance-wise, although 1min is not nothing when started every hour. 

SSD:  
Kingston XS1000 1TB SSD with USB 3.2 Gen 2
Initial backup time -> 6min 27s  
Second run without any changes -> 23s  
Third run with a 90% full disk -> 25s  
Conclusion -> This is it! This is the perfect solution for a TimeMachine backup!

**TLDR: TimeMachine backups to local APFS disks are way faster than to a network drive.**  
I haven't found any documentation on if APFS is even capable of sending snapshots, so it could be that Time Machine is not using them. That would explain the rather poor performance compared to something like ZFS.
