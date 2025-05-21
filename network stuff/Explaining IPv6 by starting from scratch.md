# Explaining IPv6 by starting from scratch
When reading online about IPv6, it becomes very apparent that there is a lot of misinformation and fear around IPv6. 
This is mostly based on either outdated or simply wrong knowledge. 

After discussing with many people online, I came to the conclusion that people are either too scared or too much stuck in their old IPv4 thinking, so they aren’t open to any arguments. 
That is why I want to try a different approach. 

Let’s start from scratch!
Let’s start with nothing and then work your way up to where we are now. 
That way it is hopefully easier for people to grasp the concepts of IPv4 and IPv6. 

## It is the year 2050
It is the year 2050 in our alternative multiverse and the internet has not been invented yet. 
Some smart folks invent IPv4 and IPv6. The internet is born. 
There are no bad actors on the internet.
That is why there are no firewalls in the year 2050!  

## John makes an internet subscription
He gets a router from his ISP. 
He connects that router to his Optical Termination Outlet (OTO).  
He gets one single IPv4. 
That IPv4 is 46.54.15.54.  
The router also gets a /48 prefix. 
That prefix is 1234:1234:1234::/48  

## John goes online
So far so good. Now he connects his MacBook Air over Wi-Fi
Now, for both IPv4 and IPv6 some things happen by default.

IPv4: 
- The router has a DHCPv4 server
- That server has a range from 192.168.1.2 to 192.168.254 
- John’s MacBook has the MAC address 00:05:02:41:45:57
- John’s MacBook asks for an IP
- The router responds with 192.168.1.2 and writes down the 11:05:02:41:45:57
- John’s MacBook has now the IP 192.168.1.2
- John’s MacBook also gets a gateway and DNS assigned.

**John’s MacBook is now ready to reach IPv4 internet!**

IPv6:
- John’s MacBook wants to use the link local IPv6 fe80:0000:0000:0000:0000:1105:0241:4557.
- John’s MacBook asks the network if there is already another device with fe80:0000:0000:0000:0000:1105:0241:4557.
- This is highly unlikely, but it is still better to be safe than sorry. In case this IP is already used, John’s MacBook would make up a new one. 
- We assume for now that there isn't another device with that IP already.

Great, now John’s MacBook has working IPv6. 
But that IPv6 is only working on the local network. 
It will not be routed and he can't access the internet with it. So we need more.  

RA:
- The router has RA (Router advertisement) running.
- That RA hands out all devices on the link local network, stuff about the network. 
- RA tells John’s MacBook about network mode, prefix, DNS servers, Gateways and so on. 
- John’s MacBook now knows that the prefix we have is 1234:1234:1234::/48, what DNS servers we use, what Gateway and so on.
- John’s MacBook decides to generate another IPv6 based on that information. 
- John’s MacBook creates the IPv6 1234:1234:1234:0000:0000:1105:0241:4557
- John’s MacBook asks the network if that IP is already in use
- Probably not, so John’s MacBook keeps that IP.

That whole process is called SLAAC. Stateless Address Autoconfiguration.

**John’s MacBook is now ready to reach IPv6 internet!**

This is awesome! 
**John now has a fully working dual stack (IPv4 & IPv6) internet connection.**

But there is a difference. IPv4 is slower than IPv6. Why that is the case, we will take a look later on. All you have to know for now is that IPv4 is slower than IPv6. 
That is why his MacBook (and basically anything else) decided to use happy eyeballs. Happy eyeballs means that devices will always prefer IPv6 over IPv4.

## John visits Netflix
Netflix is dual stack. When John is visiting netflix.com, it will be done over IPv6. 
IPv4 isn't used at all. I will repeat myself to make the point clear, **IPv4 is NOT used at all!**

If we stop right there and don't come up with other scenarios, you could argue that IPv4 and IPv6 are mostly the same.  
Sure, the handing out of the IP is a little bit different, but you won’t notice it anyway as a user.  
It all happens in the background. And sure, IPv6 is a little bit faster. But other than that? 
There is no difference. You could even argue that IPv4 has become totally meaningless and obsolete, and John could just turn it off.  

**Now let's take a look at use cases to find out the differences between IPv4 and IPv6.**  
Remember that all these scenarios happen in the alternative universe in the year 2050 without any bad actors and NOT in our timeline! 
Some things I made a little bit simpler to make the topic less complex. I will completely leave out IPv6 privacy extension, tracking over IP in general, shortening IPv6 by using :: and many other great details of IPv6.  

## Use case 1: John visits sarasblog.com:
John has a friend called Sara that writes her own blog about classic cars. 
Sara’s ISP is called OldBell. OldBell is a bunch of old network engineers that can't be bothered to implement IPv6. 
"We used IPv4 for the decades. I don't want to learn something new before I get into my pension." is a common mantra in the company OldBell. 
Because of that, Saras’ blog is only reachable over IPv4. 

John does not like to enter http://16.54.45.82 to get to Saras’ blog.
It is very hard to remember that number. 
That is why we invented DNS. So, instead, John types sarasblog.com into his browser. 
He does not know if sarasblog.com gets translated to, for example, http://16.54.45.82 or to http://[1654:4582:0000:0000:0000:0000:0000:0000]
Can you imagine having to enter that IPv6 by hand? That would be a nightmare! Thank god we have DNS!

Because of that, John does not even realize that he made a connection over IPv4 and not over IPv6. He doesn't enter IPs, he just enters names.
This is totally fine, but it also explains why John can't just turn off IPv4. Otherwise, he would be unable to reach the IPv4-only host sarasblog.com


## Use case 2: John installs a printer:
IPv4 option 1: 
The printer gets the IP 192.168.1.3. 
John installs the printer using that IP.
But there is a problem. That IP isn't static. If for any reason that IP changes, he would no longer be able to print.
So John gets into his router and tells the router that the DHCPv4 should always assign 192.168.1.3 to that printer. The router does this by writing down the MAC address of the printer: 41:45:57:11:01:01.
So far, so good. The only problem is that if John switches his router, that DHCPv4 reservation is also lost. 

IPv4 option 2: 
The printer can self-assign the static IP 192.168.1.3.
John installs the printer using that IP.
That IP is static. 
Problem is that now you have to test first if 192.168.1.3 is unused. Otherwise, you could create network collisions. 
The printer will also never ask for DHCP. So if he takes his printer to Sara’s home, and Sara is using the range 192.168.178.1 - 192.168.178.254, we can't easily connect to this printer and have to reset the network card. 

IPv6:
The printer self-assigns the IP fe80:0000:0000:0000:0000:4145:5711:0101
John installs the printer using that IP, but it is a little bit annoying to type in that IP.
That IP is static. 

All three options work, but aren't great. And I am too lazy to type in any IP. Let us use DNS instead.

IPv4 option 1:
The printer gets the hostname brotherprinter.home.arpa
John installs the printer using that hostname.

IPv4 option 2:
Since the printer never asks for DHCP, we have to go into the router’s GUI and add the hostname there.
John installs the printer using that hostname.

IPv6:
The printer gets the hostname brotherprinter.home.arpa
John installs the printer using that same hostname.


Ahh much better. No more annoying typing of IPs. Option 2 is trash though and made it even more annoying.
We rule that one out. 

DNS is nice, but there is a catch. We are now dependent on the DNS server. That sucks. Imagine your router rebooting or simply breaking down. Now you can't print from your MacBook to your Brother printer just because of that? Hell no. 
That is why Brother uses DNS during the installation to find out the fe80:0000:0000:0000:0000:4145:5711:0101 link local IPv6 of the printer, but then for the installation it uses fe80:0000:0000:0000:0000:4145:5711:0101.
That is the best of both worlds. That is why John could even use Wi-Fi Direct to connect to his printer and still use the same link local IPv6 IP.
(BTW this isn't a made-up scenario and at least real for HP printers).

**Clear win for IPv6!**

## Use case 3: John hosts his own blog:
John wants to host his own blog. Remember, it is the year 2050, we don't have firewalls yet.
He installs an Apache2 Webserver on his MacBook.
He wants his friend Sara to be able to visit his blog by inserting john.com into her browser.

That is why he creates an A record with his router’s IPv4 46.54.15.54 and an AAAA record with his MacBook’s IPv6 1234:1234:1234:0000:0000:1105:0241:4557.
Can you spot the problem already? Ask yourself the question, why do we assign for IPv4 the router’s IP and for IPv6 we assign the MacBook’s IP?

Well the problem is that you only got one IPv4 from your ISP. So devices in your network don't have their own public IPv4. 
Instead they got a link local only IPv4 from the routers DHCP server. For the MacBook this is 192.168.1.2. 

IPv4:
Let's look at the IPv4 problem from a visitor’s side. John’s friend Arnold wants to visit John’s blog. Arnold types into the URL http://john.com.
This gets translated to John’s router’s IPv4 address 46.54.15.54. So Arnold connects to John’s router. And the router has no idea what to do with that traffic. 

This is where NAT comes into play: Network Address Translation. 
We got to the router and created the NAT rule that we want to redirect the incoming traffic to 192.168.1.2. 
Great, problem solved, right? Not quite yet. 
Imagine John not only hosting the webpage but also a live webcam from his garden that has a wonderful view of Lake Thao. The webcam has the IP 192.168.1.4. 
How does the router now know if it should redirect the visitor to the webcam or the webpage?
It does so by using ports. We say that all traffic using port 80 (that is the default port of HTTP) should be redirected to the MacBook at 192.168.1.2. 
We also decide that all traffic on port 5000 should be redirected to the webcam at 192.168.1.4. 
As you can see, we can only have one thing on port 80, not two.
That sucks, because now we can't use http://johnswebcam.com! We have to use http://johnswebcam.com:5000 so it does not use the default port 80  
but we explicitly set it to port 5000. Urgghhh that is ugly!  

Uff, what a complicated mess! And it comes with so many disadvantages. NAT on your router hinders performance. And for every visitor, we have to add another entry   
to our NAT table. It could be that we even run out of RAM and NAT totally breaks down! All that mess, simply because we only got one IPv4 for our router. 

IPv6:
John’s friend Arnold wants to visit John’s blog. Arnold types into the URL http://john.com.
This gets translated to John’s router IPv6 1234:1234:1234:0000:0000:1105:0241:4557. So Arnold directly connects to John’s MacBook with the webpage.
http://johnswebcam.com on the other hand gets translated to http://[1234:1234:1234:0000:0000:1111:1111:1111] which is the IPv6 of the webcam.

Done! That is it. See how simple that is?

**Clear win for IPv6!**


## Use case 4: John does not get a public IPv4.
We write the year 2060. Unfortunately, the two ISPs OldBell and ModernTelco have run out of IPv4 to assign to their customers. 
That is why John no longer gets the IPv4 46.54.15.54 for himself. Instead, he has to share that IP. 
His ISP ModernTelco is implementing carrier-grade NAT or CG-NAT. 
This means that his ISP is basically doing to him what his John’s router is doing to its clients; putting them behind NAT.
John gets the IP 10.10.10.1 and his neighbor Marie gets 10.10.10.10.2. 
Both are behind a router that has the IP 46.54.15.54. 
So now both of them share that IP. This comes with many problems. 
First of all, performance is very bad. From the internet to John’s MacBook, we now have to traverse two routers or two times NAT. 
Another problem is that Marie got a virus and because of that is DDoSing classiccars.com. 
The server classiccars.com is not amused about the DDoS and blocks the IP 46.54.15.54.
classiccars.com does and can't know that behind 46.54.15.54 there are multiple users. 
As a result, John can now no longer access classiccars.com. 
He has become collateral damage. 

But worst of all, his website no longer works. Let's look at it again from a visitor’s point of view. 
John’s friend Arnold wants to visit John’s blog. Arnold types into the URL http://john.com.
This gets translated to the ISP router’s IPv4 46.54.15.54. 
So Arnold connects to John’s ISP router. And the router has no idea what to do with that traffic. It can't. How should it now if it has to redirect that traffic to John 10.10.10.1 or his neighbor 10.10.10.2, Marie?
ModernISP offers no interface to enter NAT based on port. And even if ModernISP would offer that, how would they decide if John or Marie gets port 80?

Self-hosting for John simply became impossible!!!

And for IPv6? Well, even in the year 2060, we still have plenty. John still gets a /48 prefix from ModernISP (which roughly translates to 1,208,925,819,614,629,174,706,176 IPs).

Let that sink in for a moment. In the year 2060, John gets zero, none, nada, nothing, or simply 0 public IPv4 IPs, while he gets 1,208,925,819,614,629,174,706,176 public IPv6 IPs.


## Does John have a static IPv4 or static IPv6 prefix?
Now that John has john.com and johnswebcam.com running, he has a potential problem. What if any of these IPs are not static?
This isn't really a technical discussion, more of a marketing one. Simply because it has nothing to do with technology.
So what is the most common case?

For IPv4, you are lucky if you even get a public IPv4. And if you get one, it will most likely not be static. Sometimes you can buy a static IPv4 for something like $20 a month or get a very expensive business line that has one or even more included.  
For IPv6, RIPE recommends a static /48 prefix, or at least /56. So even normal home users should get at least a static /56. 

Again, this isn't something technical and your ISP may differ. But in general, it is more likely for you to get a better deal on IPv6 than on IPv4.

In either case, John has to make sure that the internal IPv4 (192.168.1.2) stays static and that the IPv6 prefix and suffix stay static. 
 
Or alternatively use some kind of DynDNS.  

## Use case 5: John wants to access his cam from his internal network.

For IPv4, this is again a PITA. 
johnswebcam.com gets translated to 46.54.15.54, which his router probably can't handle. And even if it can, it is unnecessary to contact the router when he wants to access something from his own network. 
So instead, he creates an override rule on his router so that the router’s DNS does not respond with 46.54.15.54 but 192.168.1.4 when he enters johnswebcam.com locally. 

For IPv6, there is no difference between internal or external IP. The camera’s IP simply is always 1234:1234:1234:0000:0000:1111:1111:1111. So there is no need for DNS override rules. 


## In 2070, evil internet users arise.

John bought a Synology NAS in 2070. He forgot to set up a new admin password. So the NAS still uses the default credentials admin and the password admin.
The NAS runs with the IP 192.168.1.10 and 1234:1234:1234:0000:0000:222:2222:2222 

Since John has not created any NAT rules yet, there is simply no route to the NAS. So he can't get attacked over IPv4.
But attackers can attack the NAS over 1234:1234:1234:0000:0000:222:2222:2222.
But there is a caveat. There are so many IPv6 addresses, attackers can't simply brute force scan them. It is simply impossible. 
But maybe John already created the johnsnas.com record. Then attackers can easily find out. 

Well, that is a problem! IPv6 is less secure!
We have to do something!


## Here comes the firewall

We invent the firewall in 2070. By default, all incoming connections are blocked. No matter if IPv4 or IPv6.
If we really want to open something incoming, we have to manually do it.

Boom! All of a sudden, IPv6 is as secure as IPv4. Block all incoming by default. Done. 
NAT has lost all security "advantages"!

## Use case 6: Marco wants to play CoD on his PS6

We now live in a firewall world. This has its problems. The newest CoD wants to be able to talk to his PS6 over Port 4500. 
Otherwise, it will show NAT strict. Hmm.... what could we do here?

IPv4: 
Well, one option would be to tell the user Marco to open up his port. But what if Marco does not know much about routers, let alone how to open up a port and do NAT?
We invent UPnP. Marco’s PS6 is using UPnP to tell the router that it should open up port 4500 for its new CoD game. Unfortunately, UPnP turns out to be a security nightmare.
In 2075, we mostly decide to turn it off. In 2080, UPnP is practically dead. 

IPv6: 
Remember the evil attackers we discussed earlier? How IPv6 won't get scanned, but attackers could find out over AAAA records? Well, that does not really apply here. 
Since Marco’s PS6 does not need an AAAA record, it only needs some open ports for CoD. 

Here is a crazy idea: What if we open up by default all incoming IPv6 connections on the router? Again, there are no port scans anyway. And the average home user does not have an AAAA record. Marco does not have any AAAA records. And if he does,he/she is knowledgeable to change back the default to block all incoming again. And even if someone is able to find out Marco’s PS6 IP, the PS6 itself also has a firewall that only allows port 4500. So there is no practical downside. 
But as an upside, CoD now runs perfectly. Problem solved!  
But you know what, since we want to be extra cautious, we won't allow by default incoming traffic on potentially dangerous ports like SSH, RDP, HTTP, HTTPS. 

**BTW, this is again not a made-up scenario in a different universe.** This is real life. The biggest ISP in Switzerland, Swisscom, did exactly that for consumer routers. They changed the router’s default. It used to be "strict" (block all incoming) and is now "normal" (block all incoming IPv4, allow all incoming IPv6, but with the exception of some "dangerous" ports). It simply isn't a problem.  
