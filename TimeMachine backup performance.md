Have to completly rewrite this. 
Turns out SMB implementation of CORE was the problem. 



Apple switched to APFS for their machines. 
APFS is a copy-on-write filesystem.  

Does TimeMachine profit from the CoW nature of APFS?  
And how does a SMB share perform compared to a HDD or SSD?  
Let’s find out. I ran multiple tests for comparison.  

First is the initial backup time. This isn't really important IMHO, since this is only done once. 

Second run is a second run without any changes.   
This is a pretty important metric, since this simulates the normal behavior of your everyday usage. Time Machine backups every hour, so this metric is pretty important.  

The third run is with the disk being almost full (only 20GB left). For that, I will fill up the drive with random data.  
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

**TLDR: TimeMachine backups to an external disk is way faster than SMB!**  
SMB is very slow, way slower than external disks. There is not a huge difference between SSD or HDD. At least not initally with no fragmentation. It could be that HDDs suffer down the line. 

**External drives with APFS perform poor, considering their CoW nature**  
I haven't found any documentation on if APFS is even capable of sending snapshots, so it could be that Time Machine is not using them. That would explain the rather poor performance compared to other CoW filesystems like ZFS. 

**Performance on SMB is very poor**  
According to TrueNAS forums, it also seems like TimeMachine is using sync writes. On normal HDDs, this could have a very bad performance impact. 

**Why is performance so poor over SMB?**
Is really TimeMachine doing sync for everything the problem?  
I tried to hunt this down by creating a new and fresh TrueNAS SCALE installation with one single HDD as backup destination. All settings left to default, but turned sync off. By that, TrueNAS is now lying to my macMini about sync writes. 

The inital backup (with sync disabled) was done in 50min.  

Even after multiple runs, took roughly 30 seconds for a second run. Wait what? Now we are at speed levels of local disks?! What happened?
Well this time we used SCALE (which is based on Linux) instead of Core (which is an older version based on FreeBSD). 
Maybe something in the SMB implementation of SCALE is way better!.


What happens if we add 10'000 files with 50KB?

With sync off it took:
With sync onn it took:



Don't forget ARC hit ratio
