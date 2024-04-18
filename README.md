<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

- Create VMs (Windows 10 and Ubuntu) and Install Wireshark
- Observe ICMP Traffic (No Port #)
- Observe SSH Traffic (TCP Port 22)
- Observe DHCP Traffic (UDP Port 67 & 68)
- Observe DNS Traffic (UDP Port 53)
- Observe RDP Traffic (TCP Port 3389)

<h2>Actions and Observations</h2>

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
In this tutorial we will observe several network traffic protocols between two Azure Virtual Machines using Wireshark. To begin, create a Resource Group and set up two VMs: one running Windows 10 and the other Ubuntu. Both VMs will need to be inside the same Network (vnet), the same region, and have at least 2 vcpus. When creating the VMs, also make sure they are in the same region as the Resource Group. Don't forget to establish a username and password for remote access to your VMs. For this tutorial, I will name the Windows virtual machine "VM-1" and the Ubuntu virtual machine "VM-2".
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Following the creation of the VMs, connect to VM-1 through Remote Desktop using VM-1’s public IP address. Open a web browser and download Wireshark from https://www.wireshark.org/download.html. After installation is complete, launch Wireshark, choose “Ethernet”, and click on the blue fin icon in the top left corner to start observing traffic.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next, let's focus on ICMP traffic. In Wireshark's search bar, type “icmp” to filter for ICMP traffic. ICMP (Internet Control Message Protocol) is a network layer protocol that sends error messages and operational info. It's often used for network diagnostics and troubleshooting. 
</p>
<br />

<p>
To observe ICMP traffic in Wireshark, we will now ping VM2's private IP address from VM-1. Ping uses ICMP messages to check if a device is reachable over a network and measures the time it takes for a message to travel to and from the device. Go to Azure and copy VM-2’s private IP address. From VM-1, open PowerShell (run as admin) and ping [VM-2’s private ip address]. In Wireshark you will be able to see the ping request and reply packets being sent to and from the Ubuntu VM. In Wireshark, we can also examine each packet individually and view the actual data transmitted during each ping. We can use the PowerShell command “ping www.google.com -4" to observe additional ICMP traffic, where the “-4” flag specifies the use of the IPv4 protocol.
</p>
<br />

<p>
Next, we'll block inbound ICMP traffic on our Ubuntu VM to see its effects in Wireshark. Start by continuously pinging the Ubuntu VM with the command “ping [VM-2’s private IP address] -t” in PowerShell. Then, return to the Azure Portal and go to the Network Security Group page for VM-2, labeled as VM-2-nsg. In the Inbound Security Rules section, click “Add” to set up a new security rule. Create a rule that denies ICMP traffic from any source. For this tutorial, keep the source and destination port ranges as (*), meaning “Any”. Note that ICMP doesn't use ports, so the destination port doesn't affect this rule. Choose “ICMP” as the protocol and set the Action to “Deny”. Assign a priority level of 200 to ensure this rule takes precedence over others (lower numbers have higher priority). You can name this rule as you like, but I’ve named it “Deny_ICMP_Ping_From_Anywhere” for clarity. Click “Add” to apply the new rule. 
</p>
<br />

<p>
Returning to VM-1, PowerShell will indicate that the ping requests have timed out, and Wireshark will display only “request” packets without any corresponding “reply” packets. To allow pinging to VM-2 again, navigate back to Azure. To restore the ability to ping VM-2, you can either delete the previously created Inbound Security rule or modify its Action from "Deny" to “Allow". After making this change, PowerShell will stop showing timed-out requests, and Wireshark will once again capture “reply” packets. To stop the continuous pinging in PowerShell, press Control+C. 
</p>
<br />

<p>
Back on VM-1, let's look at SSH traffic in Wireshark. You can filter SSH traffic by typing “ssh” or “tcp.port==22” into the Wireshark search bar. You will need the username you used when creating VM-2 in Azure; for example, my username was “labuser”. In PowerShell, enter the command “ssh labuser@[VM2's Private IP]”, replacing “labuser” with your VM-2 username, then press Enter. You'll encounter a warning message asking if you want to continue connecting; type “yes” and press Enter. Proceed to enter your password (which will not be displayed). Here are some commands you can enter to observe traffic in Wireshark: “id”, “uname -a”, “ls -lasth”, and “touch hi.txt” (to create a .txt file). Re-enter “ls -lasth” to view the .txt file you created. When finished, type “exit” to end the session. 
</p>
<br />

<p>
Next we will observe DNS (Domain Name System) traffic. To filter DNS traffic in Wireshark, you can type “dns” or “udp.port==53” in the search bar. For this tutorial we will use the “nslookup” command in PowerShell to examine DNS traffic. The “nslookup” command helps retrieve information about domain names or IP addresses by querying a DNS server. To give it a try, open PowerShell and enter the command “nslookup www.google.com". Then, observe the network traffic in Wireshark. 
</p>
<br />

<p>
For the final part of this tutorial, let's examine DHCP (Dynamic Host Configuration Protocol) traffic. To view DHCP traffic in Wireshark, type “dhcp” into the search bar. You can also combine filters using the "or" logical operator. Since DHCP uses UDP ports 67 and 68, you can filter for these ports by typing “udp.port==67 or udp.port==68” in Wireshark's search bar. In PowerShell, run the command “ipconfig /renew” to request a lease renewal for the current IP address from the DHCP server. While the renewal process is ongoing, keep an eye on the network traffic in Wireshark. 
</p>
<br />
