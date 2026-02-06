# Active-Directory-Domain-Lab

## Table of Contents
- Project Overview
- Step 1: Adding AD DC
- Step 2: Adding Users
- Step 3: Adding Workstations
- Step 4: Group Policy
- Step 5: Security Groups
- Lessons Learned

## Project Overview
Now that we have a sandbox lab setup, I will install an Active Directory Domain Controller (AD DC) for two purposes. 
1.	Learn how Active Directory works by installing it and configuring it, and
2.	To intentionally break and secure it in a controlled environment. 

In a previous project, “Breaking Things Safely: A Virtual Cybersecurity Home Lab,” I installed a Windows Server 2022 virtual machine and assigned it the IP address **10.0.0.40.** The goal of this project was to build a functional Active Directory environment to understand core domain services, organizational structure, and access control, while intentionally encountering and resolving common misconfigurations.

## Step 1: Adding AD DC
I renamed the Windows Server 2022 VM to Cyber AD DC Server, then started it and logged in with the credentials created during installation. In Server Manager, I navigated to Local Server, selected Ethernet, and configured a static IP address of **10.0.0.40** per my network diagram. I also set the preferred DNS server to 10.0.0.40, because Active Directory is tightly integrated with DNS and relies on internal service records to locate domain services such as authentication, LDAP, and Kerberos.

<img src="Images/Screenshot 2026-02-05 153615.png" alt="AD Network Settings" width="800">

<br>

Next, I added the Active Directory role. From **Manage > Add Roles and Features**, I selected **Active Directory Domain Services** and proceeded through the wizard. After reviewing the selections, I installed the role. Once installation was completed, I promoted the server to a Domain Controller by selecting **Add a new forest** and naming the root domain **cyberlabstx.com**. I chose **Windows Server 2016** as the forest and domain functional level, enabled DNS, and configured the Directory Services Restore Mode (DSRM) password. After reviewing the final configuration, I completed the promotion process.
 
<img src="Images/Screenshot 2026-01-27 142115.png" alt="AD Network Settings" width="800">  

<br>

<img src="Images/Screenshot 2026-01-27 142344.png" alt="AD Network Settings" width="800">
 
<br>

<img src="Images/Screenshot 2026-01-27 142604.png" alt="AD Network Settings" width="800">

<br>

<img src="Images/Screenshot 2026-01-27 143148.png" alt="AD Network Settings" width="800">

<br>

<img src="Images/Screenshot 2026-01-27 143238.png" alt="AD Network Settings" width="800">

<br>

<img src="Images/Screenshot 2026-01-27 143539.png" alt="AD Network Settings" width="800">

<br>
 
To verify the installation, I opened **Tools > DNS** from Server Manager. In DNS Manager, I navigated to **Forward Lookup Zones > cyberlabstx.com** and confirmed the presence of the **_tcp** and **_udp** service records, indicating that Active Directory and DNS were successfully integrated.

<br>

<img src="Images/Screenshot 2026-01-27 150109.png" alt="AD Network Settings" width="800">

<br>
 
## Step 2: Adding Users

Once AD DC was up, it was time to start adding Organizational Units (OU). I began by going to **Tools** and then **Active Directory Users and Computers**.  

<br>

<img src="Images/Screenshot 2026-02-03 194859.png" alt="AD Network Settings" width="800">

<br>
 
I started by right-clicking on **cyberlabstx.com** and selecting **New > Organizational Unit** and creating the following OUs:
- Admins
- Domain Users
- Groups
- Servers
- Workstations

<br>

<img src="Images/Screenshot 2026-02-03 195005.png" alt="AD Network Settings" width="800">

<br>

<img src="Images/Screenshot 2026-02-03 195837.png" alt="AD Network Settings" width="800">

<br>

With the OU structure in place, I proceeded to create a domain user account to validate authentication and policy application. I right-clicked **Domain Users > New > User**, then created one for myself. I filled out the information and clicked Next, and then filled out the password information, unchecking the “User must change password at next logon and clicking “Password never expires”. (Just this Time) and next. For now, that’s it. I will be creating more later in this project.
 
 <br>

<img src="Images/Screenshot 2026-02-03 195958.png" alt="AD Network Settings" width="800">

<br>
 
<img src="Images/Screenshot 2026-02-03 200033.png" alt="AD Network Settings" width="600">

<br>

<img src="Images/Screenshot 2026-02-03 200120.png" alt="AD Network Settings" width="600">

<br>

## Step 3: Adding Workstations
At this point, the only Windows VM I had created was a Windows 11 Home. Which can’t be added to a Domain, so I had to create a new VM using Windows 11 Pro. After setting it up, I gave it a static IP address of **10.0.0.15** and a preferred DNS of **10.0.0.40**. I right-clicked the Windows > System and the Advanced System Settings. In this panel, I went to the Computer Name tab, clicked **Change** select domain, entered cyberlabstx.com, entered my administrator credentials, The configuration was successfully completed. I went back to my Cyber AD DC Server and verified that CYBERPC1 had been successfully added under AD Users and Computers > Computers. I then moved it to the workstation OU.

<br>

<img src="Images/Screenshot 2026-02-05 072634.png" alt="AD Network Settings" width="800">

<br>
 

## Step 4: Creating a Group Policy
With a domain user and workstation properly placed in their respective Organizational Units, I configured Group Policy Objects (GPOs) to enforce centralized controls.

### Workstations

1.	Open **Tools > Group Policy Management** here you will expand Forest > Domains > cyberlabstx.com. I right-clicked the **Workstation** OU and clicked **Create a GPO in this domain, and link it here…** I named it “GPO workstations Lockscreen”. I right-clicked the GPO, clicked EDIT, and navigated through the editor to Computer Configuration > Policies> Windows Settings > Security Settings > Local Policies > Security Options. I found “Interactive logon: Machine inactivity limit” and set the limit to 600 sec. I went back to my CYBERPC1, opened the terminal, executed gpupdate /force, waited 10 min, and the screen locked as expected.

<br>

<img src="Images/Screenshot 2026-02-05 075747.png" alt="AD Network Settings" width="800">

<br>
 
### Domain Users

2.	Open **Tools > Group Policy Management** here you will expand Forest > Domains > cyberlabstx.com. I right-clicked the Domain_User OU and clicked **Create a GPO in this domain, and linked it here…** I named it “GPO User Wallpaper”. I right-clicked the GPO, clicked EDIT, and navigated through the editor to User Configuration > Policies > Administrative Templates: Policy definitions > Desktop > Desktop. I opened “Desktop Wallpaper,” enabled it, and set the wallpaper to \\\CYBER-DC1\Wallpapers\cyberlabtx.jpg, a custom wallpaper I made for this project. I then verfied over at CYBERPC1, updated the GP, logged out and back in, and got a blank screen. 

<br>

<img src="Images/Screenshot 2026-02-05 093033.png" alt="AD Network Settings" width="800">

<br>

<img src="Images/Screenshot 2026-02-05 080129.png" alt="AD Network Settings" width="800">

<br>
 
I knew something was wrong, so after searching online and consulting CHATGPT, I realized that even though I had shared the folder, Domain Users needed security permissions. I clicked Security > Edit > Add, entered Domain Users, and set permissions for them. Switched back to CYBERPC1, updated the GP, logged out and back in, and got the correct background.

This issue reinforced the importance of understanding that NTFS permissions are enforced independently of Group Policy and must explicitly allow access for domain users when using network-based resources.

<br>

<img src="Images/Screenshot 2026-02-05 102350.png" alt="AD Network Settings" width="800">

<br>

<img src="Images/Screenshot 2026-02-05 120959.png" alt="AD Network Settings" width="800">

<br>

## Step 5: Security Groups
Now we connect people to roles. I created three department-based security groups (IT, HR, Finance) to represent organizational roles. I also created additional fictional users to cyberlabstx.com and added them to groups. Access was granted exclusively through department-based security groups rather than individual user accounts, in accordance with the role-based access control (RBAC) principle.
- **Finance:**
    - Daniel Wright
    - Kevin Brown
    - Rachel Chen
    - Susan Miller
- **HR:**
    - Linda Roberts
    - Mark Hill
    - Nina Garcia
- **IT:**
    - Alex Morgan
    - Jamie Lee
    - Sam Patel
    - Taylor Nguyen
 
<br>

<img src="Images/Screenshot 2026-02-05 134745.png" alt="AD Network Settings" width="800">

<br>

<img src="Images/Screenshot 2026-02-05 141949.png" alt="AD Network Settings" width="800">

<br>

I then created a folder in the C:\ drive and named it “Shares”. In that folder, I created 3 subfolders of Finance data, HR data, and IT data. I enabled sharing and added the correct department to the security tabs. I switched over to CYBERPC1, logged in as Daniel Wright from finance, went to the file explorer, and typed in \\CYBER-DC1\Finance_Data, and the folder opened. I logged out and back in with Linda Roberts from HR, and was refused access, as shown below.
 
<br>

<img src="Images/Screenshot 2026-02-05 144855.png" alt="AD Network Settings" width="800">

<br>

## Lessons Learned

1. Active Directory is not just a single tool, but a system of integrated services that must function together. A misconfiguration in one area, such as DNS or permissions, can cause issues in other areas that may not be immediately obvious.

2. Organizational Units (OUs) allow administrators to apply Group Policy in a targeted and controlled manner without impacting unrelated objects. Proper OU design simplifies management and reduces repetitive administrative tasks when adding or removing users and computers.

3. User policies and computer policies serve different purposes. Computer policies apply to a system regardless of who is logged in, while user policies follow the user account and apply across domain-joined systems.

4. Windows edition matters when joining a domain. Attempting to join a Home edition system demonstrated that not all Windows editions support Active Directory, reinforcing the importance of identifying operating system capabilities early in the deployment process.

5. Testing multiple scenarios is critical for validating access controls. Verifying permissions using different user accounts helped confirm that access was correctly assigned and denied based on group membership.

6. Troubleshooting is a core administrative skill. Resolving issues such as Group Policy not applying due to permission constraints emphasized the value of a methodical approach, using tools like gpupdate and rsop to identify and resolve configuration problems.

## Next Steps

Future iterations of this lab will focus on securing the Active Directory environment by implementing stronger password policies, account lockout rules, auditing, and simulated attack scenarios to further develop defensive and troubleshooting skills.
