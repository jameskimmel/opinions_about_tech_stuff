Let’s assume a game running at 60 fps without frame gen.  
This would translate to 16ms latency.  
A new, real frame every 16ms.  
We assume that generating a fake frame also takes 16ms.  
We also assume that are fake frames are perfect and have no artifacts.  

T is the timeline.  
RF stands for real frame.  
FF stands for fake frame.  
 
Start: The GPU renders RF1 and it gets displayed.  

T16: The GPU finished RF2 after 16ms. But we are not showing it yet.  

T32: The GPU finished FF1 based on RF1 and RF2. But we are not showing it yet.  
T32: The GPU finished RF3. But we are not showing it yet.  
T32: Output of RF2.  

T40: Output of FF1.  

T48: Output of RF3.  
T48: The GPU finished FF2 based on RF2 and RF3. But we are not showing it yet.  
T48: The GPU finished RF4. But we are not showing it yet.  

T56: Output of FF2.  

T64: Output of RF4.  
T64: The GPU finished FF3 based on RF3 and RF4. But we are not showing it yet.  
T64: The GPU finished RF5. But we are not showing it yet.  

This cycle repeats forever.  

Now let’t take a look the two things that happened here.  

We have twice the framerate compared to the native experience!  
This of course helps with Moving Picture Response Time (MPRT).  
I am not going into why MPRT is better than GtG, [since there are good articles already on that topic](https://blurbusters.com/gtg-versus-mprt-frequently-asked-questions-about-display-pixel-response/).  

We added latency.    
For native, the RF2 gets displayed at the time T16.  
But with frame gen, RF2 was displayed at the time T32.  
We have added another 16ms of latency to a total of 32ms (instead of the 16ms native).  

Conclusion:  
For games like Counterstrike you probably do not want that.   
But for something like Baldurs Gate or Civ, you probably don’t care about the added latency and enjoy a clearer image.  
This isn’t the first time in gaming we have added latency to create a better image.  
V-Sync also adds latency but provides a better image by removing screen tearing.  
