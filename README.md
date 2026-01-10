# 🔬Building AD Lab, LLMNR/NetBIOS Poisioning, and NTLMv2 Cracking w / Hashcat + JohnTheRipper
![towfiqu-barbhuiya-FnA5pAzqhMM-unsplash](https://github.com/user-attachments/assets/b31dd01b-7190-4d20-85e7-dbd24ea13388)

# Description
I am conducting a demonstration of LLMNR/NetBIOS poisoning against a vulnerable Active Directory environment in a virtualized setting using VMware. For this project, I provisioned and configured virtual machines running Kali Linux, Windows 11, and Windows Server 2022.
The setup process began with the installation and configuration of Active Directory Domain Services (AD DS) on the Windows Server 2022 instance. This included promoting the server to a domain controller and creating a test user account.
Following the successful configuration of the AD environment, I executed the LLMNR/NBT-NS poisoning attack from the Kali Linux system. The attack successfully induced the target Windows 11 client to authenticate using NTLMv2, allowing me to capture the corresponding NTLMv2 hash for the previously created domain user.
The primary objective of this project—demonstrating the feasibility of LLMNR/NBT-NS poisoning and successfully capturing an NTLMv2 hash in a controlled Active Directory environment—was achieved.

# Prerequisites + Tools Used
- VMware(Virtualization Software)
- Kali Linux
- Windows 11 iso
- 2022 Windows Server iso
- Install Updated version of Impacket

# Objectives
- Set up and Configure VMWare, Kali Linux, Windows 11 iso, Windows 2022 Server iso
- Set up and configure Windows Active Directory and Domain Controller
- Create a share on the Windows 2022 Server and access the share using a created user for the Windows 11 Virtual Machine
- Use Responder to recieve the hash from 2022 Windows Server that includes user's Username, Domain Name, and the users password encrypted with NTLMv2 hashes
  
# Responder.py(Upadated Impacket)
1. Although Kali Linux already comes with impacket we install a new version of impacket for kali
    - get rid of all impacket associated files
    - apt purge **impacket**
    - go to: https://github.com/fortra/impacket
    - move to opt folder
        - cd /opt/
2. git clone https://github.com/fortra/impacket
        - clones file from github
3. Run service apache2 start to start apache server. Apache Default page shows up. Then run:
    - service ssh start
    - service postgresql start(for metasploit)
    - service apache2 stop - Stops the apach server
   
All of these services are started but stops once the machine is cut off and needs to be restarted again, this is where systemctl comes in
- use systemctl to enable these services upon start up
- systemctl enable ssh(done)
- systemctl enable postgresql(done)

# Brief Tutorial on Building AD Lab
- Download Windows 11 iso and Windows 2022 Server iso files and upload them to VMWare as a new Virtual Machine
- Rename the Server(May require restart)
  
### Setting up Domain
-  In the Server Manager go to Add Roles and Features, make sure Role/Feature based installation is selected. Then press next and select the Server that you named
- Check the Active Directory and Domain Services box and click add features. The click next until you get the button to install and click install
- Promote your serer to Domain Controller by clicking the caution sign by the flag which pops up after the role was created
- Select add a new forest and give it a name. I used jawun.local. Then press next. Set up a password for the Domain Controller, then press next until it generates your NetBIOS name. Mine was JAWUN
- After the name is created, press next. Leave the paths as they are and press next until you get to the prerequisite check. After the check validates, click install and the computer wil restart
<img width="1280" height="686" alt="image" src="https://github.com/user-attachments/assets/0fd58b17-c68e-4aec-b86e-a8f816a50b0c" />

### Adding a User
- Go to the tools tab and click on Active Directory Users and Computers. Click on the newly created forest. Go to the Users folder, right click, hover over New and click User
- Add User info click next then and password and check password never expires(for testing purposes ofcourse) and click next then finish
<img width="941" height="663" alt="image" src="https://github.com/user-attachments/assets/3f094019-01fe-4e51-9800-06866900eecc" />

  
### Joining the Server with Windows 11
- Change Computer name(May require restart)
- Get IP Address from Server 2022
- Go to Network and internet settings and click Ethernet0
- Change adapter options
- Go to properties and click internet version 4 (TCP/IPv4)
- Add the Server's IP address in the Perferred DNS Server placeholder and 8.8.8.8 for Alternate DNS Server
<img width="686" height="738" alt="image" src="https://github.com/user-attachments/assets/25f2a666-c60c-4c93-851e-41eff4a18bd3" />

- Go to Access work or school in the search bar and click it. Click Join this device to a local Active Directory Domain
- Enter the domain name and click next
- Enter the new user's username and password and click ok. The choose standard user for Account Type and click next, then your system will restart

# LLMNR Poisioning
> What is LLMNR Poisining? Link-Local Multicast Name Resolution(LLMNR) is a Microsoft Protocol that allows computers on the same local network to resolve hostnames. Also when standard DNS fails due to circumstances such as a typo LLMNR uses muticast queries over UDP port 5355. NetBIOS Name Service(NBT-NS) is an older protocol that serves the same fallback purpose(UDP port 137). These protocols lack authentication, for example, queries are broadcast to the local network and the first response is often trusted.

# How the Attack works
- The user attempts to access a resource with an unresolved hostname(e.g., mistyped server name or non existent host
- DNS Fails > the system relies on LLMNR/NBT-NS and broadcasts a query asking "Who is this hostname?"
- The attacker(using tools like Responder) listens for these queries and claims them with their own IP Address (poisioning the resolution)
- The victim connects to the attacker's machine instead of the intended one
- If the resource requires Authentication(common for file shares via SMB) The victim automatically sends NTLM creds(username + hashed password, often NTLMv2)

# LLMNR/NetBIOS Poisining Project
- Start off on Windows Server 2022 to File and Storage Services, then click shares (the picture below shows a share I've already created)

<img width="965" height="373" alt="image" src="https://github.com/user-attachments/assets/459270bd-7af2-473e-b5a7-0400333a4329" />

- Open your files and go to the local C: drive on your computer and create a folder called hackme

![Screenshot 2025-12-16 102257.png](images/Screenshot_2025-12-16_102257.png)

- Back on the Server Manager screen, click tasks in the top right corner, then click new share in the dropdown menu

![Screenshot 2025-12-16 102710.png](images/Screenshot_2025-12-16_102710.png)

- Select SMB Share quick, then Next >

![Screenshot 2025-12-16 103037.png](images/Screenshot_2025-12-16_103037.png)

- Select the type a custom path for your share location

![Screenshot 2025-12-16 103231.png](images/Screenshot_2025-12-16_103231.png)

- Select the hackme folder that was made on the C: drive, then click Next>

![Screenshot 2025-12-16 103411.png](images/Screenshot_2025-12-16_103411.png)

- In the Share Name section, we are able to see all of or share information including the share’s local and remote path locations

![Screenshot 2025-12-16 103625.png](images/Screenshot_2025-12-16_103625.png)

- After clicking Next> click the box to enable access based enumeration

![Screenshot 2025-12-16 104116.png](images/Screenshot_2025-12-16_104116.png)

- In the Permissions section you can see that Users and Administrators have both read and write permissions

![Screenshot 2025-12-16 104339.png](images/Screenshot_2025-12-16_104339.png)

- Now click create

![Screenshot 2025-12-16 104515.png](images/Screenshot_2025-12-16_104515.png)

- Share was successfully created

![Screenshot 2025-12-16 104640.png](images/Screenshot_2025-12-16_104640.png)

- Now we head back to Windows 11 VM to add that share we just created. Log in as created user
- Head to this PC in the file explorer and select to map a network drive (The drive was already mapped from the previous time I ran the project)
    
    ![image.png](images/image.png)
    
- Enter a drive and enter the name of the server and the folder for the share and click finish

![image.png](images/image%201.png)

- Now we have access to the JAWUNSERVER hackme share

![image.png](images/image%202.png)

- Go to Kali and open the root terminal to locate [Responder.py](http://Responder.py) then cd into it. The responder picks up network traffic

![image.png](images/image%203.png)

- While still in the responder folder run the command python [Responder.py](http://Responder.py) -I eth0 -wd to start the listener

![image.png](images/image%204.png)

- Notice how the LLMNR and NBT-NS poisiners are on and also the servers that [Responder.py](http://Responder.py) is listening on
- Now head back to the windows VM logged in as a created user and head to the hackme share

![image.png](images/image%205.png)

- The folder is empty but the responder captures this instance

![image.png](images/image%206.png)

- The response by the Responder on the Linux VM

![image.png](images/image%207.png)

- Open a new tab in the Linux VM terminal and get your IP address

![image.png](images/image%208.png)

- Place the IP address in the path name box

![image.png](images/image%209.png)

- You are not going to be able to get into it but

![image.png](images/image%2010.png)

- There is the hash sent to the the Linux Terminal for user bfranks
- Includes the Domain name(JAWUN), the username(bfranks), and the user’s NTLMv2 hash then skips the hashes that have already been captured

![image.png](images/image%2011.png)

- Responder stores the hashes in /usr/share/responder/logs

![image.png](images/image%2012.png)

- Now copy the entire hash including the user and domain

![image.png](images/image%2013.png)

- Navigate to the root folder and create a file to post the hash in, Nano is used here

![image.png](images/image%2014.png)

- Now to cracking the hashes
- Using Hashcat on Kali Linux to find the password in the hashes.txt file created with the hashes from the LLMNR poisioning
- After typing hashcat —help see, in the Network Protocol section, that NetNTLMv2 is the is the protocol that is going to be used to crack this hash becasuse this is a NTLMv2 hash

![image.png](images/image%2015.png)

- Type in the terminal
    - hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt
    - -m 5600 specifies the protocol to use when cracking the hash
    - Also use a good password list, in this example rockyou.txt.gz is used. A built in wordlist on Kali Linux
    
    ![image.png](images/image%2016.png)

# Results
It turns out that I was not able to crack the hash using rockyou.txt or milw0rm-dictionary.txt wordlists. I also used both JohnTheRipper and Hashcat on the Windows 11 VM and on my personal Windows 11 system and still could not crack the NTLMv2 hash

# Mitigation
- Disable LLMNR, select "TURN OFF MULTICAST NAME RESOLUTION" under Local Computer Policy > Computer Configuration >Administrative Templates > Network > DNS Client in the Group Policy Editor
- Diable NBT-NS, navigate to Network Connections > Network > Network Adapter Properties > TCP/IPv4 Properties > Advanced Tab > Wins tab and select "Disable NetBIOS over TCP/IPv4"

If a company must use or connot disable LLMNR/NBT-NS the best thing to do is:
- Require network access control
- Require stong passwords. The more complex the password the harder it is for the attacker to crack the hash
  
