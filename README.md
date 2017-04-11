# Into The Packets
Most of the vulnerabilities and exploits we've examined so far, such as XSS or
CSRF, operate at the request level, in the realm of HTTP(S). Even something
like SQLI, which directly targets specific middleware, is only really possible
because of improper exposure at the request level. Since HTTP is the
application-layer protocol that enables webapps, this makes sense. But HTTP
itself is enabled by a lower-level protocol, TCP/IP, at what's called the
transport layer. As the request/response is to HTTP, the network packet is to
TCP/IP.

## Topics in Focus
* NetSec Crash Course
* Firewalls
* Intrusion Detection Systems
* Risk Assessment
* Penetration Testing
* Threat Monitoring
* Incident Response

## Staying Motivated When You're In Over Your Head
![Image](http://i.imgur.com/TuOpc3E.gif)

There are a more than few challenges facing the introduction of NetSec. Coming
from web application security, we're now dropping into a lower level of the
stack than we've been working with, so the territory will most likely be much
less familiar. And that territory is huge: one could build an entire career on
some of these topics. If our goal is proficiency when it comes to web security,
we can't really hope to achieve any more than an informed level of awareness
about topics like packet analysis and intrusion detection.

But we can't let that stop us, either: even a very basic level of knowledge
about these topics can be quite valuable, both as engineers and as budding
security researchers. Even just knowing the jargon goes a long way (newbie: "Is
there script or something that can tell me when someone is scanning my computer
machine?"; informed newbie: "What's a good IDS for detecting web app
footprinting?"). Having a little bit of experience with some of these topics
and tools can give you a lot of context when considering possible threats and
defensive measures.

But in a real sense, we're building one of the most valuable skills there is:
diving into subject matter that seems hopelessly over our heads. It is perhaps
this quality more than any other which marks those who excel in security: a
certain fearlessness about swimming beyond their depth. Even the security pros
who are acknowledged experts in specialized topics had to start somewhere, but
it seems that an unusual amount of them are also conversant in topics far
outside their specializations.

As engineers, we're taught the value of comprehensive learning, starting with
the fundamentals and progressing deliberately toward mastery, and we develop a
healthy respect for the depth of true expertise. As hackers, we are often
called to set all that healthy respect aside, acquire knowledge tactically,
apply techniques surgically, and forge our own path, often in defiance of
convention, standards and practices.

## Milestone 0: Networking Toolbox
We'll start with a rundown of the most basic net tools you should be familiar
with. Most of these aren't hacking tools or even utilities specific to
security; they're basic tools that every programmer should know how to use.
These are so well documented we won't cover usage here, but you should make
sure you understand at least the basic use cases of each and be able to answer
all the challenge questions. These lists aren't by any means comprehensive, nor
are the tools on them equally important.

> The following tools are standard on most unix/linux systems, as well as
> macOS; anything that isn't already available should be easily installable via
> a package manager like apt or brew. If you're using Windows as your host and
> can't install one of these try using your Kali Linux container.

### Utilities

Very basic, single-serving utilities for discovering information about a network

> `ifconfig`

* Facts
    * List information about available [network interfaces (NICs)](https://en.wikipedia.org/wiki/Network_card)
    * Can start/stop individual interfaces
    * Can [change an interface's MAC address](https://en.wikipedia.org/wiki/Ifconfig#Media_access_control_functions)
* Challenges
    * Run `ifconfig` and determine the name/id of your primary network interface
> primary network id: wlp2s0


    * What is your primary interface's IP address? Is it different from your public IP? Why or why not?
> ip: 172.16.36.55/23, it is not different from my public IP. I don't know why.

    * What is the MAC address of your primary interface?
> MAC address: can't write it here but it looks like HWaddr 00:0F:20:CF:8B:42

    * Identify and understand your [loopback interface](http://askubuntu.com/questions/247625/what-is-the-loopback-device-and-how-do-i-use-it)
> Done

> `ping`

* Facts
    * Determine the reachability of a specific destination
    * Determine the IP resolved from a specific hostname
    * Displays the latency of the network connection
* Challenges
    * What is the IP address of codepath.com?
> 198.58.125.217

    * What is the IP address of google.com?
> 172.217.8.14

    * Why would the IP address of google.com change between runs or from different locations?
> Because Google serves from multiple data centers.

nslookup
Facts
Command line interface for DNS server queries
Determine IP from hostname or hosts from IP
Often (but not always) helpful for determining an IP's provenance
Challenges
Using the IP for codepath.com from the previous, pass it to nslookup
Does the domain returned from nslookup match? If not, why not?
traceroute
Facts
Determine the path to a specific destination
Each step in the path is called a hop
Determine the latency at each hop
Challenges
Compare the traceroutes for codepath.com and google.com
How many of the hops are the same? What accounts for this?
Which has more hops? What accounts for the difference?
telnet
Facts
Both a protocol and a command line utility
Part of the early internet
Largely abandoned in favor of ssh
Still useful to determine if a port is open
Challenges
What's one thing that makes telnet insecure?
Can you telnet to codepath.com? What port is open and why?
Core Tools

The following are very widely used by engineers of all stripes and will be invaluable over the course of a career. If you don't know them, you need to. If you're going to spend extra time with any of the tools in this milestone, these are the ones to know.

curl and wget
Facts
Both can download contents from FTP, HTTP and HTTPS
Both can send HTTP POST requests and use HTTP cookies
Both are scriptable
Challenges
Identify some differences between the two
Which would you be more likely to use for interacting with a RESTful API from the command line?
Which support recursive downloading?
Which are you more likely to find pre-installed on a Linux OS?
What is the syntax for each for downloading a file to the current directory?
ssh and scp
Facts
Industry standard method for establishing a remote shell or copying files to/from remote machine
Authentication can be password based or via ~/.ssh/authorized_keys (see ssh-keygen)
A prime target for footprinting due to the access it provides. Scanners search for systems allowing password authentication and root access
Challenges
Why is key authentication preferred to passwords?
What is the syntax for copying a file from a local folder to a remote one?
Milestone 1: Security-Flavored Net Tools
The following tools are a bit more obscure or specialized, so we'll review some basic usage.

netcat (aka nc, ncat)
Supports network reads/writes using TCP/UDP
Can be used for most network diagnostic tasks (detect open ports)
Supports tunneling and can be used for backdoors
Open a terminal window and start netcat listening in server mode port 2389:

# Listen in server mode
$ nc -l 2389
Open a second terminal window and connect in client mode:

# Connect in client mode
$ nc localhost 2389
Now, type some text in window #2 and watch it appear in window #1. Magic. CTRL-C in window #2 will kill both processes.

Let's try a simple file transfer. In window #1, netcat listens in server mode and pipes the output to a file:

$ nc -l 2389 > received.txt
In window #2, create a simple file and send it via client mode:

$ echo "CodePath Rocks" > message.txt
$ cat message.txt | nc localhost 2389
Running cat received.txt in window #1 should echo CodePath Rocks

You can use netcat in a manner similar to telnet. For instance, the following will establish a connect to dict.org and let you lookup an English word's definition:

$ nc dict.org 2628
# 220 pan.alephnull.com dictd 1.12.1/rf on Linux 4.4.0-1-amd64 <auth.mime> <40503875.15857.1490380824@pan.alephnull.com>
$ DEFINE wn peripatetic
You can also use netcat for basic port scanning. Scan 192.168.0.1 for open ports in the range 80 - 90:

$ nc -z 192.168.0.1 80-90
# Connection to 192.168.0.1 port 80 [tcp/http] succeeded!
nmap
Maps a network by sending packets and analyzing responses
Can detect hosts, services, OS, and more
Widely used and extended for all sorts of network discovery and pentesting tasks
Install nmap via apt or brew
nmap is an old school hacking tool that is still actively used and developed. It's very scriptable and supports a whole community of users. The maintainers have even kindly setup a demo target at http://scanme.nmap.org/ for you to play around with. A few examples follow (note the second one requires superuser privs on certain systems):

# Scan a single server
$ nmap -v scanme.nmap.org

# -sS is stealth scan
# -O determine operating system
# /24 scans a class C size network (256 IPs)
$ sudo nmap -sS -O scanme.nmap.org/24

# host enumeration on IPs 0-255.1-127
# -V is version detection
$ nmap -sV -p 22,53,110,143,4564 198.116.0-255.1-127

# -R is random, picks 100,000 random hosts
# -Pn disables host enumeration
# -p 80 checks port 80 (web servers)
$ nmap -v -iR 100000 -Pn -p 80
Read the official nmap Reference Guide
Mac Users: installing nmap via homebrew also gives you ncat, an extended, more powerful version of netcat than the nc that comes with the OS
Challenge 1: Run nmap against your localhost IP to see all open ports

See how many of the ports you can match to services
Hint: try shutting down Docker or Virtualbox and re-running nmap
Optional Challenge 2: Map your home LAN using nmap

If you can, do this at home instead of while connected to a larger network (like school or work)
Start by reading about host discovery with nmap
Determine the target IP range from your local IP. For instance, 192.168.2.0/24 denotes the range of IPs between 192.168.2.0 through 192.168.2.255
Read Understanding IP Addresses, Subnets, and CIDR Notation for more background.
Use the stealth scan against your target IP range: sudo nmap -sP 192.168.2.1/24
In addition to the IPs, the manufacturer info should be displayed next to each device's MAC address. See if you can account for all the devices present: router, computers, mobile devices, printers, etc.
Milestone 2: Grabbing Packets with tcpdump
tcpdump
Command-line packet capture and analysis tool
Scriptable and widely used for capture
Typical usage: capturing packets for analysis in another app, such as Wireshark
Requires superuser privs on certain systems
Start by listing all interfaces available for listening (compare this output to ifconfig):

$ sudo tcpdump -D
You should have already identified your primary network interface (en0 in the example below), so start listening:

# Listen on a single interface
$ sudo tcpdump -i en0
Unless your machine is unusually quiet, you should see a lot of output. This is raw stream of packets you're seeing. If your primary interface is wireless, try disabling it for a moment (on macOS, click the wireless icon at the right side of the top bar and click Turn Wi-Fi Off). The stream should suddenly go quiet. Turn wifi on again to see it fire back up.

Does it seem like a lot of output? Try it in one of the verbose modes:

# -v: verbose, -vv: very verbose, -vvv: very, very verbose
$ sudo tcpdump -i en0 -vvv
...or trying dumping the hex and ASCII for each packet:

# -X: packet data in both hex and ASCII
$ sudo tcpdump -i en0 -v -X
You can also listen on any available interface:

# Listen on any available interface
$ sudo tcpdump -i any
tcpdump lets you filter by source and destination IP as well. You should have already identified your primary network interface's IP via ifconfig (the examples below use 192.168.1.1), so use tcpdump to filter packets with that destination and then with that source:

# target a destination host
$ sudo tcpdump -n dst host 192.168.1.1

# target a source host
$ sudo tcpdump -n src host 192.168.1.1
Challenge 1: Determine the IP address for codepath.com and use tcpdump to display packets with that IP as the destination. Then open http://www.codepath.com in the browser and check the output. Notice the output displays the HTTP requests in addition to the packets.

How many requests to load the main codepath.com page?
What type of resource accounts for most of the requests?
Now try the same exercise with https://security.codepath.com. What differences do you see in the output? What accounts for those differences?
Challenge 2: You can also monitor DNS queries on port 53 with tcpdump. Use this to determine the IP of your primary DNS:

Listen for DNS queries on port 53: sudo tcpdump -vvv -s 0 -l -n port 53
Think of a domain name that probably exists (common word or phrase + .com) but that you've never visited before (suggestion: zombo.com) and open it in a browser
Look at the tcpdump output for the UDP packets trying to resolve the domain. The destination IP should be the DNS
Read more about using tcpdump
Milestone 3: Hello, Wireshark
While tcpdump is important to understand and has its uses, you can probably already tell that packet analysis is a pretty data-intensive task for the command line. To do this properly, we're going to need something more powerful, and for packet sniffing, the weapon of choice is Wireshark.

Wireshark comes bundled with the Kali ISO, but since we're only running the base Kali image in a "headless" container and Wireshark features a rich GUI, you should just download and install it on your host machine. Mac users can install via the dmg or via homebrew: brew cask install wireshark

Open the app up and have a look:

wireshark-open

When you load the UI, you'll be confronted with a list of interfaces whose names should be at least a bit familiar right now. You should see your primary interface listed, the same one you used with tcpdump, and a little thumbnail graph to its right that displays current activity. Double-click that to open the capture screen.

wireshark-capture

The capture screen is divided into three sections: the top section is a scroll pane containing the tcpdump-style packet stream (scroll to the bottom and watch it keep moving). Packets are color-coded by type, and basic information like source and destination IP are displayed. The bottom two panes show the contents of an individual packet (the upper pane contains the packet summary data and the lower pane contains the raw hex/ASCII content) and will change when you click on a row in the top pane.

Wireshark features more functionality than could possibly be covered here. You can sort, filter, query, save, load, replay...if you can imagine it, Wireshark probably has a way to do it. There are lots of good resources available for learning more, but first, take a moment to appreciate what you're looking at.

If you're capturing your primary wireless interface, then you're seeing the raw feed, the firehose of your network activity. Unless you're using a test machine or VM, there's probably a lot of data streaming by...even if you're not "using" your network. A modern end-user system, such as macOS, is generally not very quiet. Even with the browser and all other network-capable apps closed, it surprisng just how much traffic there still is on the network.

Challenge: Get a capture of the baseline traffic on your wireless interface. Start by closing every running app on your system: every window, every tab, every shell...try to go completely dark (restart your system first if you need to). Then open Wireshark (which should be the only app running) and start capturing on your primary interface.

Look at the source and destination IPs; how much of the traffic is inbound vs. outboud?
Try nslookup on a couple of IPs that aren't in your network. See if you can figure out who those IPs belong to
Try to identify traffic associated with at least one process on your host that's either part of the OS itself or is auto-launched at startup
See if you can spot any ARP packets used to resolve IPs to MAC addresses
Badge Earned: Packet Sniffer
We could spend the rest of our lives becoming Wireshark power users and understanding the finer points of packet analysis and networking. Even just analyzing typical network traffic is a daunting amount of work. Hopefully, you can see that already just looking at the Wireshark output in front of you.

If you're new to this, practically every packet on the capture represents some strange and possibly arcane bit of computing knowledge. You may see printer traffic using the CUPS protocol, ICMPv6 router advertisements, TCP Keep-Alive packets, Multicast Listener Queries, Asynchronous IPv6 TPS Reporting packets...and you may not know what any of those are (or that the last one is just made up).

You probably also know that you could learn a lot of this, that with enough time and dedicated effort, you could eventually become conversant in this alien language of packets and ports and pings. For the technologist, time and attention are always precious commodities, and one must always choose how best to direct one's focus. So rather than going on a comprehensive survey of packet analysis from the ground up, we're going to zoom in and look at a few specialized applications of Wireshark.

danger-will-robinson

Warning: As in previous weeks, the milestones that follow introduce resources and tools that can get you into trouble if used recklessly. Follow instructions and practice good judgment.
Milestone 4: Traffic Analysis — Mike's Computer is Acting Weird
Malware is insidious by design, and the best malware is extremely difficult to detect. Professional hackers don't use off-the-shelf malware that can be easily caught by anti-virus scanners checking file signatures against a database; they typically roll their own. And the very best hackers have malware that doesn't even bother with filesystem persistence, but just lives in memory, hoarding secrets and phoning home.

But most malware needs to phone home at some point in order to operate, so one technique to detect malware is to analyze network traffic for malicious activity. For this challenge, we'll look at some examples of captured malware traffic courtesy of http://www.malware-traffic-analysis.net/

Before that, a word of caution: These examples are old and sufficiently sanitized such that they should be safe, but even though we're only looking at the captured network traffic and not the malware itself, still treat this as though it is possibly dangerous:

Don't go directly to a suspicious host
Don't blindly click on search results without considering the source (more on that below)
Don't attempt to replay or simulate any of the requests
First, setup Wireshark per the site's recommendations. This will walk you through a few common tasks such as adding/removing columns, changing the time format, filtering specific traffic, and displaying hostname as a column. A few notes that might help clarify the instructions:

Instead of actually removing the columns, you can just uncheck the "Displayed" checkbox
For the time format resolution, if Seconds: 0 isn't available just use Seconds
"Expand the breakout in the middle section, so you see the Host: line in the HTTP header." is referring to the middle pane in the capture window; you'll need to select an HTTP request in the upper pane first (look for a GET) so that the middle pane shows a Hypertext Transfer Protocol heading, which should have a Host entry underneath it. Right-click that and choose Apply as Column
Let's start with this example from the site. Read through the writeup first so you understand the premise.

Next, download the zip file linked from that page and unzip it (see the password here); inside is a pcap file which is the conventional extension used for packet capture files understood by Wireshark. Open it in Wireshark via File > Open so you can see the capture.

You should see six GET requests displayed from February 2015. Looking at the hostnames, it should be apparent which ones are potentially suspicious but don't go directly to those hosts. Searching for them in Google is all you need to do. Even then, don't trust just any result that comes back; look for results from known threat database sites:

sophos.com
malwr.com
virustotal.com
techhelplist.com
You should be able to quickly identify the probable malware involved (in this case it does have a known signature and isn't very advanced). See if you can understand what the malicious requests are doing, though:

What's the purpose of the request to checkip.dyndns.org?
What's up with the three jpg requests? Why two different domains?
How was the malware delivered? What isn't Mike telling us?
When you think you've pieced it all together, click the reveal page to compare your findings against the researcher's. Notice in particular the report with the writeup of what happened.

Milestone 5: Traffic Analysis — Mystery Meat Malware
Try another from the malware-traffic-analysis site: Identify the Activity

The pcap for this is a bit more realistic, but the malicious activity is still fairly easy to spot. The objective is to identify the probable malware from the traffic patterns. This one won't be quite so easy as googling the host names. Remember:

Don't go directly to a suspicious host
Don't blindly click on search results without considering the source
Don't try to replay or simulate any of the requests
Hint: one of the captured requests attempted to download an .exe file; try searching for the filename instead of the host.

Want to go further? Be careful. While the site has numerous training exercises, some examples include zipped samples of the actual malware, which should always be treated as dangerous. There are ways to setup safe lab environments to work with this material, but avoid experimenting directly with malware on your primary machine.
Milestone 6: Wi-Fi Hacking — Crack a Handshake
You'll need some tools for this one; go ahead and bash into your Kali container to start installing the tools for these milestones, aircrack-ng and wordlists. This will take a few minutes to complete, so you can read ahead while it finishes:

$ docker exec -it kali bash
root@kali:/# apt-get install -y aircrack-ng wordlists
Most of us take wireless internet connectivity for granted. Beyond some basic protective measures, such as using a VPN over public Wi-Fi, you probably don't think much about safety when it comes to Wi-Fi. Yet, like practically everything else, Wi-Fi can be and is routinely hacked for various purposes:

Anonymity: hackers avoid using access that can be tied to their identity
Infiltration: gaining access to a wireless AP can be a good starting point for compromising a network
Phishing: a rogue access point can be used to trick users into giving away credentials
Theft: Starbucks wanted you to buy a latte first
Like the rest of the topics we're covering this week, Wi-Fi hacking is its own rabbit hole of knowledge and skill, with an array of tools and techniques available. From a very high level, it looks like this:

Monitor: listen to the packets
Capture: grab some of the packets
Crack: decode the hashes in those packets
Zooming in a bit, it looks more like this:

Mapping the Airspace: using a wireless antenna to passively monitor wireless traffic to identify available access points (APs) and how they are configured and protected, signal strength, and any other information, such as make/model of the wireless router
Going After the Low-Hanging Fruit: most older Wi-Fi protection protocols like WEP and WPA are trivial to crack and are typically the first targets. Certain setups like WPS also can provide an easy way in.
Capturing a Handshake: once the easy options have been ruled out, in the most difficult case of a WPA2-protected AP, the attacker listens for the Wi-Fi handshake between the AP and a client device, which includes the hashed password
Cracking the Hash: the attacker attempts to crack the hash using the WPA-2 hashing algorithm, which is only practical if the password is weak or the SSID (the name the AP advertises, which is used as the salt) is very common
This is a broad summary, of course, and practically speaking, most modern APs are protected with WPA2, so the process often comes down to offline hash cracking. If the password is weak or the wireless router is still using factory settings, the search space can be reduced substantially. And with the advent of distributed, cloud-based, GPU-enabled hash cracking, the crackers are starting to catch up with the state of the art.

Challenge: Crack a WPA2 handshake

We'll begin at the end and crack a capture of a demonstration handshake. Assume this was captured via sniffing and stored as a pcap file, and now we want to test for a weak password, so we'll run through the rockyou.txt wordlist:

In your Kali container, prepare the setup:

# Download the demonstration cap file
root@kali:/# wget http://download.aircrack-ng.org/wiki-files/other/wpa.full.cap
# Unzip the wordlist
root@kali:/# gunzip /usr/share/wordlists/rockyou.txt.gz
# Show aircrack-ng usage
root@kali:/# aircrack-ng --help
Now you're ready for the fun part:

root@kali:/# aircrack-ng wpa.full.cap -w /usr/share/wordlists/rockyou.txt
Milestone 7: Wi-Fi Hacking — Grab a Handshake
You'll need access to a WPA2-protected Wi-Fi connection for this milestone. If you don't have one, you can just use the cap file from the last challenge to answer the challenge questions.

Cracking is actually the hard part, unless the password is weak. Monitoring and capturing the handshake containing the hash is still tricky, though, as it's less deterministic and more hardware-dependent. The easiest way to capture a handshake is to listen on your own interface.

Turn your Wi-Fi connection off
Open Wireshark and start capturing on your wireless interface (no data should appear)
Turn your Wi-Fi connection on
As soon as you are authenticated, in Wireshark, hit the stop capture button (the big red square)
The WPA-2 handshake, which is a four-step process, should appear near the top of the capture each labeled (Message # of 4).

Use Wireshark to answer the following challenge questions:

What is the protocol of the handshake packets?
Where is the SSID?
Where specifically is the hash?
What is a nonce?
Don't bother trying to crack the captured handshake using aircrack-ng, as it only accepts capture files with 802.11 packet headers, which require your wireless card to be in monitor mode (instead of actually connected to a network). Try the next (optional) milestone instead.

Bonus Milestone 8: Wi-Fi Hacking — Sniff Thy Neighbor's Packets
For this optional bonus milestone, in addition to a WPA2-protected Wi-Fi network, you'll need at least one other nearby device that can connect to that network (your smartphone or a lab partner's computer should work). You'll also need a network card capable of passive monitoring. Many network cards have this capability (these instructions were tested on a MacBook Pro running Sierra); however, we can't promise this will work for you. Additionally, passive monitoring typically isn't available from within VMs or containers, so this most likely will only work if you're running Wireshark on your host machine.

Open Wireshark and open Capture Options (fourth toolbar button from left)
On the Input tab, locate your Wi-Fi interface and check the Monitor checkbox
Set the value for Link-layer Header to 802.11 plus radiotap header
Click the Start button at the bottom right of the dialog
This will put your interface into monitor mode until you stop the capture. You may need to disconnect and reconnect Wi-Fi if it doesn't and you seem to lose connectivity. But if all goes well, you'll see wireless packets begin appear in Wireshark. That's not just your network: that's everything in the air that your card can hear.

In the filter box, type eapol and hit enter.
This will filter out everything except handshake packets, so there may be nothing displayed after you apply the filter. Now you need to use the other device to force a handshake by disconnecting and reconnecting it from the Wi-Fi network. This should work with a smartphone or a nearby laptop. The handshake packets should appear in Wireshark soon after.

Stop the capture and save the capture to a file (handshake.pcap is the name used in the instructions that follow) using the Wireshark/tcpdump format.
From your host machine, copy the file to the Kali container, then open a shell into Kali and verify the file is there:
$ docker cp ./handshake.pcap kali:/
$ docker exec -it kali bash
root@kali:/# ls -l handshake.pcap
-rw-r--r-- 1 root root 78965 Mar 25 00:42 handshake.pcap
Try to crack the hash in the capture with the wordlist:
root@kali:/# aircrack-ng handshake.pcap -w /usr/share/wordlists/rockyou.txt
Chances are there will be additional artifacts in your capture and aircrack-ng will list all of them with the corresponding SSIDs and ask you to pick one. Choose the SSID of your AP.

Will aircrack-ng crack your hash? Let's hope not, or someone needs to go back to password school. Of course, you could always use a custom wordlist containing the password to force a successful crack, but that's no fun.

Possibilities for Wi-Fi hacking open up when running Kali as a host (virtualization makes wireless monitoring tricky) and using specialized hardware (instead of a standard wireless NIC), but unless you're doing wireless pentesting, there are fewer legitimate applications to a specialized setup. Still, it's important to at least understand the threat landscape simply because the vast majority of us at this point connect to the internet via wireless APs.

Want to learn more? Check out SecurityTube's Wireless LAN Security and Penetration Testing Megaprimer, a jaw-droppingly comprehensive walkthrough of the topic in 52 video chapters. Same warnings as above apply, though: Be careful, use good judgment, and don't ever try to use these techniques against targets for which you have neither access nor express prior consent.
