# Parts
1. [Setting Up Virtual Machines](https://github.com/larryn-tech/homelab/blob/main/01-vm_setup.md)
2. Configuring Active Directory and Domain Controller (Current)
3. [Configuring Splunk](https://github.com/larryn-tech/homelab/blob/main/03-splunk_configuration.md)
4. [Simulating Brute Force Attack and Atomic Red Team](https://github.com/larryn-tech/homelab/blob/main/04-attack.md)

# Active Directory
Active Directory Domain Services (AD DS) is a component of Microsoft’s Active Directory that centrally manages and stores information about users, computers, and other network resources. It provides essential functions like user authentication and authorization, enabling single sign-on, enforcing security policies through Group Policy, and simplifying resource management across an organization's IT infrastructure. Servers running AD DS are called domain controllers, which provide authentication for the domain and ensure the availability and replication of the directory data.

In this part, we will:
- Install and configure Active Directory on `dc-server` and promote it to Domain Controller
- Create Organizational Units and add users
- Configure `win-workstation` to join the domain

---

## dc-server VM
### Configure Active Directory and Promote to Domain Controller
1. In the Server Manager Dashboard, click on `Manage` from the toolbar and then `Add Roles and Features` to open the **Add Roles and Features Wizard**

![dcs-configure-01]

2. Click `Next` to continue
3. Select `Role-based or feature-based installation` as the installation type and then click `Next`
4. Leave `Select a server from the server pool` as selected and click `Next`
5. From the list of server roles, select `Active Directory Domain Services` and then click `Add Features` in the pop-up window
6. Click `Next` until you reach the **Confirmation** step
7. Click `Install`
8. Once the installation is complete (as indicated by the `Configuration required. Installation succeeded on dc-server` message under the progress bar), click `Close`

![dcs-configure-08]

9. Click on the flag icon in the toolbar and then click on `Promote this server to a domain controller` to launch the **Active Directory Domain Services Configuration Wizard**

![dcs-configure-09]

10. Select `Add a new forest`, add `goldrush.sf` for the root domain name, and then click `Next`
11. Create a password (ex. `dc-serverPW1`) and then click `Next`
12. Continue clicking `Next` until you reach the **Prerequisite check** step and then click `Install`
13. Allow the computer to restart
14. When you return to the login screen, you’ll notice that the Administrator account is now prefixed by our domain name - indicating that the AD DS was successfully created and the server was promoted to the domain controller

![dcs-configure-14]

### Creating an Organizational Unit and User
1. Log on to the Administrator account and in the Server Manager Dashboard, navigate to **Tools** > **Active Directory Users and Computers**
2. Right-click on `goldrush.sf`, navigate to `New`, and click on `Organizational Unit`

![dcs-creating-02]

3. Name the OU as `Coach` and then click `OK`
4. Right-click on the `Coach` OU, navigate to `New`, and click on `User`

![dcs-creating-04]

5. For my first user, I entered:
   - First name: `Kyle`
   - Last name: `Shanahan`
   - Full name: `Kyle Shanahan`
   - User logon name: `kshanahan`

![dcs-creating-05]

6. Click `Next`
7. Enter a password (ex. `goldrush-PW1`) and uncheck `User must change password at next logon`
8. Click `Next` and then `Finish`

### Adding OUs and Users with PowerShell
We will use PowerShell to create three more OUs (Offense, Defense, and Special Teams) and create users from the `team.csv` file. The CSV contains a list of players with their full, first, and last name, number, OU (`team`) they belong to, and username.

| `full` | `first` | `last` | `number` | `team` | `user` |
|:----|:---|:---:|:---|:---|:---|
| George Kittle | George | Kittle | 85 | Offense | gkittle |

1. Download the `team.csv` [here](https://github.com/larryn-tech/homelab/blob/main/resources/team.csv) and save it to the Desktop
2. Search for "PowerShell ISE” in the Windows search bar and click `Open`
3. If you haven’t already, enable bidirectional clipboard to allow copying and pasting between the host and virtual machines
   - In the virtual machine’s menu bar, navigate to **Devices** > **Shared Clipboard** and select `Bidirectional`
4. Copy and paste the following script into PowerShell ISE:

```powershell
# Import team.csv file
$AD_Users = Import-csv C:\Users\Administrator\Desktop\team.csv

# Create Offense, Defense, and Special Teams OUs
New-ADOrganizationalUnit -Name "Offense" -ProtectedFromAccidentalDeletion $False
New-ADOrganizationalUnit -Name "Defense" -ProtectedFromAccidentalDeletion $False
New-ADOrganizationalUnit -Name "Special Teams" -ProtectedFromAccidentalDeletion $False

# Create users
foreach ($User in $AD_Users) {
  $User_name		= $User.user
  $User_Password	= "goldrush-PW1"
  $First_name		= $User.first
  $Last_name		= $User.last
  $User_Department	= $User.team
  $Employee_id		= $User.number

  # Check if the user already exists in Active Directory
  if (Get-ADUser -F {SamAccountName -eq $User_name}) {
      # Output a warning message if user exists
      Write-Warning "A user $User_name has already existed in Active Directory."
  }
  else {
      # Otherwise, create a new user
      New-ADUser `
      	-SamAccountName $User_name `
      	-Name "$First_name $Last_name" `
      	-GivenName $First_name `
      	-Surname $Last_name `
      	-Enabled $True `
      	-EmployeeID $Employee_id `
      	-DisplayName "$Last_name, $First_name" `
      	-Department $User_Department `
      	-UserPrincipalName ($User_name + "@goldrush.sf") `
      	-Path "ou=$User_Department,dc=goldrush,DC=sf" `
      	-AccountPassword (convertto-securestring $User_Password -AsPlainText -Force) `
      	-ChangePasswordAtLogon $False

      # Print user's last name to console
      Write-Host "Creating user: $($Last_name)" -BackgroundColor Black -ForegroundColor cyan
  }
}
```

5. Click on the green play icon (or press **F5**) to run the script
6. In the **Active Directory Users and Computers** window, right-click on `goldrush.sf` and click `Refresh`

![dcs-powershell-06]

7. We should now see our `Offense`, `Defense` and `Special Teams` OUs with users in each OU

![dcs-powershell-07]

---

## win-workstation VM

### Reconfigure DNS server
Before we can join the `win-workstation` machine to our domain, we must set `dc-server` as its DNS server. This way, `win-workstation` will be able to resolve the `goldrush.sf` domain.

1. Right-click on the network icon in the task bar and click on `Open Network & Internet settings`
2. Click on `Properties` under **Ethernet**
3. Scroll to the **IP settings** section and click on `Edit`
4. Update the Preferred DNS server to our `dc-server` (`192.168.10.7`) and click `Save`

### Joining Computer to goldrush.sf Domain

1. Search "This PC" in the Windows search bar and click on `Properties`
2. Scroll down to the **Related settings** section and click on `Rename this PC (advanced)`
3. In the **System Properties** window, click on `Change` to change the domain
4. Select `Domain` under the **Member of** section and enter the domain name `goldrush.sf`

![win-join-04]

5. Enter the credentials for the `dc-server`'s `Administrator` account
   - User name: `administrator`
   - Password: `dc-serverPW1`
6. If successful, you will get a message welcoming you to the domain
7. Click `OK` and then restart to apply changes
8. At the logon screen, select `Other user` and sign in using one of the created user accounts
   - Ex. `gkittle`(password: `goldrush-PW1`)

![win-join-08]



[dcs-configure-01]: ./img/02/ad-dcs-configure-01.png
[dcs-configure-08]: ./img/02/ad-dcs-configure-08.png
[dcs-configure-09]: ./img/02/ad-dcs-configure-09.png
[dcs-configure-14]: ./img/02/ad-dcs-configure-14.png
[dcs-creating-02]: ./img/02/ad-dcs-creating-02.png
[dcs-creating-04]: ./img/02/ad-dcs-creating-04.png
[dcs-creating-05]: ./img/02/ad-dcs-creating-05.png
[dcs-powershell-06]: ./img/02/ad-dcs-powershell-06.png
[dcs-powershell-07]: ./img/02/ad-dcs-powershell-07.png
[win-join-04]: ./img/02/ad-win-join-04.png
[win-join-08]: ./img/02/ad-win-join-08.png