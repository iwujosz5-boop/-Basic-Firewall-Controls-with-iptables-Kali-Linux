# -Basic-Firewall-Controls-with-iptables-Kali-Linux
This project demonstrates basic firewall configuration and testing using iptables on a Kali Linux virtual machine.
The lab focuses on controlling incoming traffic and demonstrates how a firewall rule can:

Allow normal traffic.
Block ICMP echo requests (ping).
Block traffic from a specific Windows host IP address.
Remove firewall rules and restore connectivity.

The testing environment consists of a Kali Linux VM and a Windows host machine connected through a virtual network.

⚠️ Lab Safety: Perform these tests only on systems and networks you own or have permission to administer. Firewall rules can interrupt network connectivity, so keep a local Kali terminal available while testing.

🎯 Objectives

By completing this lab, you will learn how to:

Check the current iptables configuration.
Test connectivity between Windows and Kali.
Block ICMP echo requests.
Verify that the ICMP rule blocks ping.
Remove an ICMP firewall rule.
Block an entire source IP address.
Test the effect of an IP-based firewall rule.
Remove the IP-based block.
Understand how INPUT, ACCEPT, and DROP work.
🧪 Lab Environment
System	Role
Kali Linux	Firewall / target
Windows Host	Testing client
Firewall	iptables
Protocol tested	ICMP
Network	Virtual lab network
Basic Network Diagram
                 Virtual Network
                       |
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
 ┌────────────────┐        ┌────────────────┐
 │ Windows Host   │        │   Kali Linux   │
 │                │        │                │
 │ Source         │───────►│   iptables     │
 │                │        │   Firewall     │
 └────────────────┘        └────────────────┘
1. Check if iptables Is Installed

Start by checking the current firewall configuration.

Run on Kali:

sudo iptables -L
Command Explanation
sudo

Runs the command with administrator privileges.

iptables

Linux firewall management utility.

-L

Lists the current firewall rules.

Example Output
Chain INPUT (policy ACCEPT)
target     prot opt source       destination

Chain FORWARD (policy ACCEPT)
target     prot opt source       destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source       destination
Understanding the Chains

iptables primarily works with three standard chains:

Chain	Purpose
INPUT	Traffic entering Kali
OUTPUT	Traffic leaving Kali
FORWARD	Traffic passing through Kali

For this lab, we mainly use the INPUT chain because Windows is sending traffic to Kali.

2. Test Basic Connectivity

Before creating firewall rules, confirm that Windows can communicate with Kali.

From the Windows host, run:

ping <Kali_IP>

For example:

ping 192.16*.**.1**

<img width="514" height="210" alt="Capture 12" src="https://github.com/user-attachments/assets/f8e91bed-7e85-4b19-83ad-36eec861af06" />


Replace the example address with the actual IPv4 address of your Kali VM.

Expected Result

If connectivity is working, Windows should receive replies similar to:

Reply from 192.16*.**.1**: bytes=32 time<1ms TTL=64
Reply from 192.16*.**.1**: bytes=32 time<1ms TTL=64

This establishes a baseline before modifying the firewall.

3. Block ICMP Traffic

Ping uses ICMP Echo Request and ICMP Echo Reply messages.

To block incoming ping requests from Windows, run on Kali:

sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
Command Breakdown
Option	Meaning
-A	Append a new rule
INPUT	Apply the rule to incoming traffic
-p icmp	Match ICMP traffic
--icmp-type echo-request	Match ping requests
-j DROP	Drop matching packets

<img width="436" height="36" alt="Capture 27" src="https://github.com/user-attachments/assets/ab53b434-7779-41b1-85ef-653ab921dfff" />



In simple terms:

Drop incoming ICMP Echo Requests.

4. Test the ICMP Block

Return to the Windows host and run:

ping <Kali_IP>
Expected Result

The ping should fail, commonly showing:

Request timed out.
Request timed out.
Request timed out.

<img width="471" height="239" alt="Capture 14" src="https://github.com/user-attachments/assets/64d6b232-dde3-4bdf-8a8d-6d3b1606eb70" />


This demonstrates that the Kali firewall is dropping the incoming ICMP Echo Requests.

5. Verify the Firewall Rule

On Kali, run:

sudo iptables -L INPUT -n -v --line-numbers

The output should contain a rule similar to:

num  pkts bytes target  prot source      destination
1    4    336   DROP    icmp  0.0.0.0/0  0.0.0.0/0
Why Are pkts and bytes Useful?


<img width="427" height="219" alt="Capture 15" src="https://github.com/user-attachments/assets/e85b8c62-23aa-4d68-9ce9-f3e68b13a0fa" />

They show how many packets and bytes have matched the rule.

For example:

pkts = 4
bytes = 336

means four packets matched the rule.

If you continue pinging from Windows, these counters should increase.

6. Remove the ICMP Block Rule

Once testing is complete, remove the rule:

sudo iptables -D INPUT -p icmp --icmp-type echo-request -j DROP
-D Means Delete

<img width="436" height="31" alt="Capture 21" src="https://github.com/user-attachments/assets/e9cc3d37-0b25-4a30-9ef2-01f499474912" />


The command removes the matching rule from the INPUT chain.

Verify:

sudo iptables -L INPUT -n --line-numbers

Then test again from Windows:

ping <Kali_IP>

If there are no other firewall or network issues, ping should work again.

<img width="470" height="86" alt="Capture 17" src="https://github.com/user-attachments/assets/52894119-b764-416b-aadc-7b54fd7b9989" />


7. Block the Entire Windows Host IP

The next test blocks all incoming traffic from the Windows host IP, rather than only ICMP.

First identify the Windows IPv4 address.

On Windows:

ipconfig

Look for:

IPv4 Address. . . . . . . . . . : <Windows_IP>

For example:

IPv4 Address. . . . . . . . . . : 192.1**.**.*

<img width="469" height="71" alt="Capture 19" src="https://github.com/user-attachments/assets/08c670fc-274d-433d-8f26-b0de061f8c70" />


Then on Kali:

sudo iptables -A INPUT -s <Windows_IP> -j DROP

Example:

sudo iptables -A INPUT -s 192.1**.**.* -j DROP
8. Understanding the IP Blocking Rule

The rule:

sudo iptables -A INPUT -s 192.1**.**.* -j DROP

can be broken down as:

-A INPUT

Add the rule to the incoming traffic chain.

-s 192.1**.**.*

Match packets whose source IP is 192.1**.**.1.

-j DROP

Discard the matching packets.

In Plain English

Drop all incoming traffic originating from 192.1**.**.1.

Unlike the previous rule, this rule is not limited to ICMP.

9. Test the IP Block

From Windows, try:

ping <Kali_IP>

The ping should fail if the Windows IP is the actual source address seen by Kali.

You can also test another service if one is intentionally running on Kali.

For example, if Kali has an SSH server running:

ssh <Kali_IP>

The connection should also be blocked by the source-IP rule.

10. Verify the IP Block

On Kali:

sudo iptables -L INPUT -n -v --line-numbers

You should see something similar to:

num  pkts  bytes  target  prot  source          destination
1    10    840    DROP    all   192.1**.**.1    0.0.0.0/0

Again, watch the packet counter.

If Windows sends traffic and the pkts value increases, the rule is matching that traffic.

11. Remove the IP Block

After completing the test, remove the rule:

sudo iptables -D INPUT -s <Windows_IP> -j DROP

For example:

sudo iptables -D INPUT -s 192.1**.**.1 -j DROP

Verify:

sudo iptables -L INPUT -n --line-numbers

Then test connectivity again:

ping <Kali_IP>
🔬 12. Test Results

Test 1 — Before Firewall Rule

Command:

ping <Kali_IP>

Result:


<img width="960" height="189" alt="Capture 20" src="https://github.com/user-attachments/assets/84981e13-ca99-47f3-8620-1e307aba6778" />


Observation:

Windows was able to communicate with the Kali VM before the ICMP firewall rule was applied.

Test 2 — ICMP Block

Kali command:

sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

<img width="436" height="36" alt="Capture 27" src="https://github.com/user-attachments/assets/fb3fa46d-5a20-4b3a-a6cf-3ddbb9807bb8" />


Windows command:

ping <Kali_IP>

Result:
<img width="471" height="239" alt="Capture 14" src="https://github.com/user-attachments/assets/3e529c6d-91a0-4069-be12-527e7e25f7ae" />



Request timed out.

Observation:

The ICMP Echo Requests were dropped by the Kali firewall, preventing Windows from receiving ping replies.

Test 3 — ICMP Rule Removed

Kali command:

sudo iptables -D INPUT -p icmp --icmp-type echo-request -j DROP

<img width="436" height="31" alt="Capture 21" src="https://github.com/user-attachments/assets/913a10ae-8862-496e-9048-5510d5137154" />

Windows command:

ping <Kali_IP>

Result:

<img width="508" height="206" alt="Capture 11" src="https://github.com/user-attachments/assets/20d13bb2-5170-4b0b-b6b3-a8babf60bf44" />



Observation:

After removing the ICMP blocking rule, connectivity was restored if no other network or firewall control was blocking the traffic.

Test 4 — Windows IP Block

Kali command:

sudo iptables -A INPUT -s <Windows_IP> -j DROP

<img width="480" height="51" alt="Capture 22" src="https://github.com/user-attachments/assets/0238daf3-ab11-4f3d-b605-ed055fa1b0f6" />

Windows test:

ping <Kali_IP>

Result:

<img width="467" height="76" alt="Capture 24" src="https://github.com/user-attachments/assets/52fe36d7-d28f-44c6-8fcd-31682e7d43e7" />

Request timed out.

Observation:

Traffic originating from the specified Windows IP address was dropped by the Kali firewall.

Test 5 — IP Block Removed

Kali command:

sudo iptables -D INPUT -s <Windows_IP> -j DROP

<img width="436" height="20" alt="Capture 25" src="https://github.com/user-attachments/assets/6e60787d-ae6c-4b7a-b696-f1deaac70f56" />

Windows command:

ping <Kali_IP>

Result:

<img width="508" height="206" alt="Capture 11" src="https://github.com/user-attachments/assets/1990ad3f-a8af-4f0b-b442-164f4c837755" />



Observation:

The IP-based firewall restriction was removed.

📸 Evidence / Screenshots

For a professional GitHub project, include screenshots showing each stage.

Recommended directory:

screenshots/
├── 01-iptables-installed.png
├── 02-ping-before-firewall.png
├── 03-icmp-drop-rule.png
├── 04-ping-blocked.png
├── 05-iptables-counters.png
├── 06-icmp-rule-removed.png
├── 07-ip-block-rule.png
├── 08-ip-block-test.png
└── 09-ip-block-removed.png

For example:

## ICMP Block Test

![ICMP Block Rule](screenshots/03-icmp-drop-rule.png)

![Blocked Ping](screenshots/04-ping-blocked.png)
🧠 What I Learned

This lab demonstrated that iptables can control traffic entering a Linux system based on different characteristics.

The first rule:

sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

blocks a specific type of traffic — ICMP Echo Requests.

The second rule:

sudo iptables -A INPUT -s <Windows_IP> -j DROP

blocks all incoming traffic from a particular source IP, regardless of protocol.

This demonstrates an important firewall concept:

                Incoming Traffic
                       |
                       ↓
                  iptables
                       |
            ┌──────────┴──────────┐
            ↓                     ↓
       Rule matches          Rule doesn't match
            |                     |
            ↓                     ↓
          DROP                  Continue
🔐 Security Considerations

Blocking an entire IP address can be useful in some situations, but it should be used carefully.

For example:

-A INPUT -s <Windows_IP> -j DROP

is much broader than:

-A INPUT -p icmp --icmp-type echo-request -j DROP

The first blocks all incoming traffic from the specified source, while the second only blocks ICMP Echo Requests.

In a production environment, firewall rules should follow the principle of least privilege and allow only the traffic that is actually required.

📚 Useful Commands
List rules
sudo iptables -L
List rules with IP addresses and counters
sudo iptables -L -n -v
Show rules as commands
sudo iptables -S
Show numbered rules
sudo iptables -L --line-numbers
Delete a specific rule
sudo iptables -D INPUT <rule-number>
Flush rules — lab use only
sudo iptables -F

⚠️ Be careful with iptables -F on a remote system. Removing firewall rules can change the system's security posture.

📁 Recommended GitHub Repository Structure
iptables-firewall-lab/
│
├── README.md
│
├── screenshots/
│   ├── 01-iptables-installed.png
│   ├── 02-ping-before-firewall.png
│   ├── 03-icmp-drop-rule.png
│   ├── 04-ping-blocked.png
│   ├── 05-iptables-counters.png
│   ├── 06-icmp-rule-removed.png
│   ├── 07-ip-block-rule.png
│   ├── 08-ip-block-test.png
│   └── 09-ip-block-removed.png
│
└── notes/
    └── firewall-concepts.md
✅ Conclusion

This lab demonstrated two fundamental firewall-control techniques using iptables:

ICMP-based filtering
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

This blocks incoming ping requests.

Source-IP filtering
sudo iptables -A INPUT -s <Windows_IP> -j DROP

This blocks incoming traffic from a specific IP address.

The tests demonstrate how a Linux firewall can control network traffic based on protocol, packet type, and source address.

A useful final workflow for the project is:

Check Firewall
      ↓
Test Connectivity
      ↓
Add ICMP Rule
      ↓
Test Ping
      ↓
Remove ICMP Rule
      ↓
Add Source-IP Rule
      ↓
Test Traffic
      ↓
Remove Source-IP Rule
      ↓
Verify Configuration
