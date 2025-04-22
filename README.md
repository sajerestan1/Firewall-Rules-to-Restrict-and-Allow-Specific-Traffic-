# Firewall Rules to Restrict and Allow Specific Traffic

## Project Overview

This project aims to demonstrate the use of pfSense firewall rules to control outbound traffic, restricting access to non-educational sites like gaming platforms (e.g., Roblox and Minecraft) while allowing only essential web browsing. The task includes configuring TCP and UDP rules to enable controlled traffic, then using Host Overrides in pfSense to block specific gaming sites. This project serves as an introduction to firewall rules, DNS configuration, and troubleshooting communication issues between virtual machines.

## Real-World Scenario: School Lab with Restricted Internet Access
In a school setting, administrators often need to ensure that pupils can access only educational content and websites. The goal is to prevent students from accessing specific gaming sites like Roblox.com and Mindcraft.net. 

<b>Please see <a href= "https://github.com/sajerestan1/How-I-Installed-pfSense-on-VMware-linked-to-parot-os"> how I resolved my struggles with setting up pfSense and connecting it with my Parrot OS</a> </b>


## Step-by-Step Rule Configuration

Start the pfSense VM and the other VM (Mine is Parrot OS)


### Login to pfSense Web GUI

URL: http://192.168.1.1

Username: admin, Password: pfsense

![pfSense - Login -](https://github.com/user-attachments/assets/6005714e-a1a8-4b1c-b9cd-bde15cbc838c)



## ALLOW TCP RULE

Navigate to Firewall Rules

Go to: Firewall > Rules > LAN

Allow Only HTTP and HTTPS Traffic

Action: Pass

Protocol: TCP

Destination port range: HTTP (80) to HTTPS (443)

Description: Allow Web Access

Place this rule above the block rule

Click Save, then Apply Changes

![pfSense home arpa - Firewall_ Rules_ TCP](https://github.com/user-attachments/assets/898ea1f1-fe70-4700-943c-aaaf6915a86e)



## ALLOW UDP RULE

Navigate to Firewall Rules

Go to: Firewall > Rules > LAN
Allow DNS (UDP) Traffic

Action: Pass

Protocol: UDP

Destination Port: 53 (DNS)

Description: Allow DNS Lookup

Click Save, then Apply Changes

![pfSense home arpa - Firewall_ Rules_Allow 1 1](https://github.com/user-attachments/assets/75bbd9f6-38d8-452f-8ea4-990d661ec8ba)


## BLOCK ALL OUTBOUND RULE
Navigate to Firewall Rules

Go to: Firewall > Rules > LAN

Create a Block All Rule

Action: Block

Interface: LAN

Protocol: Any

Destination: Any

Description: Block all outbound traffic

Click Save, then Apply Changes

![pfSense home arpa - Firewall_ Rules_ Block](https://github.com/user-attachments/assets/0e63f4fc-5532-4c01-953c-b021c35396fd)



Ensure that the Block Rule is below the UDP and TCP Rules
Test the Rules in Parrot OS

After applying the rules, I tested the setup in Parrot OS:

1. ping 8.8.8.8 → Fails (because ICMP is blocked)

![Ping Google](https://github.com/user-attachments/assets/fc663988-db87-45f1-9d76-35e680ab2f2e)

2. firefox wikipedia.org → Loads successfully (since HTTP/HTTPS is allowed)

![open duckgo](https://github.com/user-attachments/assets/64755b4c-d242-473e-8145-cb0dca623c2e)

![open Duckduckgo](https://github.com/user-attachments/assets/1cf217db-1c3d-4cb2-9814-fcf194a05d86)


ping roblox.com → Fails (because DNS is blocked without UDP rule)

![ping roblox](https://github.com/user-attachments/assets/fe215bbc-db31-47c2-ba5f-6985269aab4a)


After adding the UDP rule, I was able to access websites normally.

![open roblox](https://github.com/user-attachments/assets/74de2f25-dc8d-40b3-934a-1b44f4dd345a)


## Initial Firewall Configuration: 

After getting both systems connected, I set up a simple block all traffic rule and allowed only HTTP and HTTPS traffic (TCP).

### Struggle: 

After configuring the rules, I couldn’t access any websites from Parrot OS using Firefox, even though I expected HTTP/HTTPS to work.

### Solution:

I discovered that the UDP protocol needed to be allowed for DNS to function correctly, so I added a rule allowing UDP traffic on port 53.



## Explanation: Difference Between TCP and UDP

### TCP (Transmission Control Protocol):

TCP is a connection-oriented protocol, ensuring reliable communication with packet ordering and error checking.

Real-world example: Web browsing (HTTP/HTTPS) relies on TCP because it requires reliable, in-order delivery of data. This ensures that websites load correctly without missing or out-of-order content.

UDP (User Datagram Protocol):

UDP is a connectionless, lightweight protocol used for applications where speed is more important than reliability. It doesn’t guarantee delivery or order of packets.

Real-world example: DNS queries use UDP because DNS responses need to be delivered quickly, and small losses of data packets are acceptable.

In my case, the absence of a UDP rule for DNS (port 53) prevented web browsing from working after setting up the HTTP/HTTPS rules. Once the UDP rule was added, the DNS requests were properly resolved, allowing access to websites. So even though web pages use TCP, DNS lookups must succeed first, and they use UDP. This is why adding a UDP rule for DNS above the TCP rule made the system function properly. 

## Host Overrides


Since the primary objective was to block known gaming sites like Roblox and Minecraft, I turned to Host Overrides in pfSense to block these sites by resolving their domains to 127.0.0.1.

### Host Overrides Configuration:

![pfSense home arpa - Firewall_ Rules_ servies](https://github.com/user-attachments/assets/ce45af36-e4c4-476b-944e-28561f8e586e)

![- pfSense home arpa - Services_ DNS Resolver_ Host Override](https://github.com/user-attachments/assets/f8d8aaaf-4edb-433c-8352-bc093df533dc)


Minecraft: minecraft.net → 127.0.0.1

Roblox: roblox.com → 127.0.0.1

Struggle: Despite setting up Host Overrides in pfSense, I was still able to access Roblox and Minecraft from Parrot OS using Firefox.

Solution: After some troubleshooting, I found that Parrot OS DNS settings weren’t properly pointing to the pfSense DNS server (192.168.1.1). Instead, it was using Google’s DNS (8.8.8.8), which bypassed the pfSense rules.

Fix: I manually updated the /etc/resolv.conf in Parrot OS to ensure that the primary DNS server was set to 192.168.1.1 (the pfSense server).

Testing DNS Settings:

After fixing the DNS settings in Parrot OS, I confirmed that the system was using the pfSense DNS by running nslookup roblox.com:

Output confirmed that the DNS request was routed through pfSense and that roblox.com was resolved to 127.0.0.1, effectively blocking the site.

