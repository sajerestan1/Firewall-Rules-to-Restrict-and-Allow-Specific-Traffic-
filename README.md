# Blocking Specific Gaming Sites with pfSense

## Project Overview

This project aims to demonstrate the use of pfSense firewall rules to control outbound traffic, restricting access to non-educational sites like gaming platforms (e.g., Roblox and Minecraft) while allowing only essential web browsing. The task includes configuring TCP and UDP rules to enable controlled traffic, then using Host Overrides in pfSense to block specific gaming sites. This project serves as an introduction to firewall rules, DNS configuration, and troubleshooting communication issues between virtual machines.

In this case, the school uses pfSense to create a firewall rule that blocks all outbound traffic by default and then creates a rule to explicitly allow HTTP and HTTPS traffic only. This ensures that pupils can access websites like Wikipedia for research but cannot misuse the internet connection. 

This project replicates that setup using a virtual lab environment where the Parrot OS VM represents a lab computer and pfSense acts as the firewall for the network. 

## Real-World Scenario: School Lab with Restricted Internet Access
Imagine a school computer lab where young pupils are allowed to access only educational websites and resources. The school wants to prevent the pupils from accessing gaming sites, downloading unauthorised applications, or using any other internet services beyond basic web browsing.  

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

From the main menu, click on services

![pfSense home arpa - Firewall_ Rules_ servies](https://github.com/user-attachments/assets/ce45af36-e4c4-476b-944e-28561f8e586e)

then click on DNS Resolver from the drop down menu


![- pfSense home arpa - Services_ DNS Resolver_ Host Override](https://github.com/user-attachments/assets/f8d8aaaf-4edb-433c-8352-bc093df533dc)


Minecraft: minecraft.net → 127.0.0.1

Roblox: roblox.com → 127.0.0.1


### Steps in pfSense

### Steps in pfSense

**Navigate to**: `Services > DNS Resolver > Host Overrides`

**Add Entries for the Following:**

| Host       | Domain         | IP Address   | Description         |
|------------|----------------|--------------|---------------------|
| roblox     | roblox.com     | 127.0.0.1    | Block Roblox        |
| www        | roblox.com     | 127.0.0.1    | Block www.Roblox    |
| mindcraft  | mindcraft.net  | 127.0.0.1    | Block Mindcraft     |
| www        | mindcraft.net  | 127.0.0.1    | Block www.Mindcraft |


Save and Apply Changes

![override home](https://github.com/user-attachments/assets/43a228d3-80b2-4967-b3dc-a1f5db513559)

























### Issue Encountered: Parrot DNS Not Using pfSense

Although Host Overrides were configured correctly, Parrot OS still accessed blocked sites. Diagnosis revealed that it was using **Google DNS (8.8.8.8)** instead of pfSense’s **192.168.1.1**.

---

### **Fix: Configuring Parrot OS to Use pfSense for DNS**

#### **1. Release and Renew DHCP**
```bash
sudo dhclient -r ens33
sudo dhclient ens33
```

#### **2. Verify DNS Server**
```bash
grep '^nameserver' /etc/resolv.conf
```

#### **3. Manually Set pfSense as DNS Server**
```bash
sudo nano /etc/resolv.conf

```
Now add this:

```
nameserver 192.168.1.1
```

#### **4. Make `/etc/resolv.conf` Persistent**

I noticed that the overwritten by Parrot NetworkManager, so I did this:

- Edit NetworkManager config:
  
```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

- Ensure the following:
```ini
[main]
plugins=ifupdown,keyfile
dns=none

[ifupdown]
managed=false
```

- Restart NetworkManager:
```bash
sudo systemctl restart NetworkManager
```



### **Testing**

After setting pfSense as the DNS server:

```bash
nslookup roblox.com

```

![nslookup for roblox](https://github.com/user-attachments/assets/d27bf96c-51c9-4340-acb4-44a14a344955)



### 🧪 DNS Resolution Observation from Terminal

From the terminal outputs above, I observe the following:

1. **`nslookup https://www.roblox.com`**  
   - Server: `192.168.1.1`  
   - Result: `** server can't find roblox.com: NXDOMAIN`  
   ✅ Indicates the override for `www.roblox.com` is working correctly.

2. **`nslookup www.roblox.com`**  
   - Server: `192.168.1.1`  
   - Result: `** server can't find roblox.com: NXDOMAIN`  
   ✅ Confirms `www.roblox.com` is being successfully blocked by pfSense.

3. **`nslookup roblox.com`**  
   - Server: `192.168.1.1`  
   - Result:
     ```
     Name: roblox.com
     Address: 128.116.13.3
     ```
   ❌ This shows that `roblox.com` is **not being blocked**, as it still resolves to its real IP address. This suggests that a host override for the base domain `roblox.com` is missing or misconfigured.

---

### ✅ Solution: Add Host Override for the Base Domain (`roblox.com`)

To block the root domain (i.e., `roblox.com` without any prefix like `www`), I did these steps in pfSense:

1. Navigate to: **Services > DNS Resolver > Host Overrides**
   
3. Click on **`+ Add`**
   
5. Fill in the fields as follows:
   
   - **Host**: *(leave this field blank)*
   - **Domain**: `roblox.com`
   - **IP Address**: `127.0.0.1`
   - **Description**: `Block base roblox.com`
7. Click **Save**
8. Click **Apply Changes**

![roblox1](https://github.com/user-attachments/assets/49197053-4ba3-44e2-8a8d-7a70f07a754d)


![roblox2](https://github.com/user-attachments/assets/a0bf87c7-750c-4a87-80fe-f9fc19ae9dbc)


After applying this fix, run the following command again on Parrot OS to verify:

```bash
nslookup roblox.com
```

![1](https://github.com/user-attachments/assets/8e1aa04e-0223-4747-bd9d-2af1c5c5e77a)


This confirms that the base domain is correctly blocked through pfSense's DNS Resolver.


**Firefox** failed to load Roblox or Mindcraft sites, confirming DNS redirection worked.

![2](https://github.com/user-attachments/assets/1bf96eba-e1f5-44fb-a3a7-80331570e02b)



### **Conclusion**

This project successfully:

- Configured **firewall rules** for HTTP/HTTPS
- Blocked gaming websites using **Host Overrides**
- Fixed **DNS redirection** by manually configuring Parrot OS


### **Project Highlights**

- **Hands-on pfSense DNS configuration**
- **Firewall rule prioritisation**
- **VM-to-VM troubleshooting**
- **Manual DNS client config on Linux**
- **Blocking Specific Gaming Sites with pfSense**

















