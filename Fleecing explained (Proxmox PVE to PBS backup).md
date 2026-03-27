PVE VMs can suffer time outs and extreme slowdowns when the upload or the PBS destination is slow.
But why?

When the backup is running, and the VM wants to do a write, the write will cause a system slowdown, because it has to go to PBS first, before the local VM disk can be written to.
But why does the write have to go to the PBS first?

If we look at a timeline, we start the backup at time_0.
We want to backup the data, that the system had at exactly time_0.

The backup started, and we are now at time_1.
The VM tries to write to block_1.
Block_1 we already backed up to PBS, so it can directly write it to the local VM disk.
No slowdown here.

We are now at time_2.
The VM tries to write to block_2. That block is not yet in PBS.
Hypothetically, if we would write that block_2 with the new write from the VM, and at time_8 the backup reads that changed block and writes it to the PBS, the backup would not be consistent. The backup would be from time_0 for block_1 and time_2 for block_2.

So in reality, we do is this at time_2:
The VM wants to write block_2. We say "ok but since block_2 is not yet in the PBS yet, we first need to write the state of time_0 to the PBS!"
We backup block_2 to the PBS and then let the VM do the write.
That is a slowdown, since we need to wait for PBS.

How does fleecing fit into this?
Well instead of waiting to the high latency or low bandwith, or low PBS performance
connection to PBS to finish the write of block_2, we write block_2 (at time 0) to the fleecing device. That way PVE can come back at time_8 and read the block_2 from the fleecing device instead of our already changed VM disk.
The VM still has to wait until the write to the fleecing device was done, but since this is a local and fast SSD, the wait time potentially will be way smaller than to PBS.

PS: I have clients with copper and slow uploads. While a SSD of course is always better, even a single slow 3,5" HDD as fleecing device, can be faster than a remote PBS.
 
