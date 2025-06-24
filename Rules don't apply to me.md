Are you fed up with all the preachy naysayers in the TrueNAS forum?  
Are you a rebel that was born free and is not afraid to take risks?  
Do you find it sometimes annoying that you always know better then everybody else?  

**Then you are exactly in the right place!**  


Let's go through some rules these nerds mumble in their stupid forums:  

> Follow hardware requirements

Phah, follow hardware requierements, don't use RAID controllers, don't do that, I hear them whining. To follow hardware requierements, 
I would have to read. Andrew Tate said that reading makes you stupid and is for loosers. 

> Put VMs on mirrors not RAIDZ

Bro, you know I am the top G with flashy cars and all, but I don't have that kind of money! Using mirrors? Only 50%?
No way! I have a 40TB Jellyfin VM!

> Don't use blockstorage for files. Use datasets for your Jellyfin files 

Nah, mounting a share in Linux is too much of hassle. I rather have my files stored in the VM.

> By using blockstorage on RAIDZ, by using blockstorage instead of datasets, you run into many issues. Read up on how volblocksize is a static number while record size is a max value. Read up on pool geometry.

Bro, I already told you that reading is for loosers and now you expect me to do math? Hell no!

> If you are to lazy to read up on it, just follow these simple rules: Don’t use block storage for files! Instead, use datasets for files. Separate your files from VM data. Your VM data is stored in zvols and block storage. For block storage, use NVMe mirrors (or at least SSDs). Don't use RAIDZ1. Never use RAIDZ2 for block storage. Use RAIDZ2 only for large files that are sequentially read and written.

I am a rebel, rules don't apply to me!

> You won't get the storage efficiency you think you will get with RAIDZ and blockstorage

Pfff idiot. I simply set the volblocksize to 64k. Problem solved. 

> 64k volblocksize will lead to read and write amplification and fragmentation

That is a problem for future me. Right now it work. If it becomes dogslow a few years from now or if my resilver takes forever, I will come back to the forum to whine about it. But mostly to shit on how bad ZFS is. 

> Keep it simple stupid. For example don't use QCOW2 on ZFS. Just follow the defaults unless you really know what you do

I thought you nerds are so clever. Don't you know that in theory, even stuff like nested virtualization can work. I run a 40TB SMB share from Windows Server. That server is run by TrueNAS. TrueNAS itself is a VM of Proxmox. Don't worry bro, I used passtrough for the disks.  
If something goes wrong, I can still come into your nerd forum. Have fun troubleshooting that setup!



