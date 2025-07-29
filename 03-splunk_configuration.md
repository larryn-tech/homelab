# Parts
1. [Setting Up Virtual Machines](https://github.com/larryn-tech/homelab/blob/main/01-vm_setup.md)
2. [Configuring Active Directory and Domain Controller](https://github.com/larryn-tech/homelab/blob/main/02-active_directory.md)
3. Configuring Splunk (Current)
4. [Simulating Brute Force Attack and Atomic Red Team](https://github.com/larryn-tech/homelab/blob/main/04-attack.md)

# Splunk Configuration
In the previous part, we setup our virtual machines (VMs) and installed our operating systems.

Splunk is a SIEM (Security Information and Event Management) tool designed to collect, index, search, and analyze large volumes of machine-generated data (such as logs from computers, networks, and applications). In addition to being able to monitor and correlate data from a variety of sources, Splunk allows us to generate alerts, reports, and visualizations and gain insight into our network’s overall health and performance.

To get this data into Splunk, a lightweight agent called the Splunk Universal Forwarder is typically installed on the systems generating the logs. This forwarder efficiently streams the raw data to the main Splunk instance for processing.

Sysmon (System Monitor) is a tool that provides detailed event information on Windows machines. When used with Splunk, we can enhance our ability to detect and respond to suspicious or harmful activities.

In this part, we will:
- Install Splunk Server on the `splunk-server` VM
- Install Sysmon and Splunk Universal Forwarder on `win-workstation` and `dc-server`
- Configure `win-workstation` and `dc-server` to send application, security, system, and Sysmon logs to our Splunk server
- View logged events on Splunk Web

---

## splunk-server VM
### Splunk Server Installation
On your **host machine**:
1. Open [splunk.com](https://splunk.com) and sign up for an account if you don’t already have one
2. After signing on, navigate to **Platform** > **Free Trials & Downloads** and click on [Get My Free Trial](https://www.splunk.com/en_us/download/splunk-enterprise.html) under **Splunk Enterprise**
3. Download the `.deb` package for Linux

Back on the `splunk-server` VM, we will setup the VM to be able to access the Splunk Server installation package on our host machine:
1. Run the command `sudo apt-get install virtualbox-guest-additions-iso`
2. Press **Y** to continue
3. If presented with the **Daemons using outdated libraries** screen, press **Enter** to complete the installation
4. From the VM’s menu bar, navigate to **Devices** > **Shared Folders** > **Shared Folders Settings…**

![spl-server-04]

5. Click on the icon with the green plus sign to add a shared folder

![spl-server-05]

6. Select the folder path where you saved your Splunk Enterprise `.deb` package and select `Read-only`, `Auto-mount`, and `Make Permanent`
   - In my case, I created a `Splunk` folder within my `Downloads` folder and saved the package there

![spl-server-06]

7. Press `OK` to save and close the Settings window
8. Run the command `sudo reboot` to reboot the VM
9. Login again and run the command to add our user to the `vboxsf` group:
    - `sudo adduser goldrush vboxsf`
    - Substitute your username for `goldrush` if different
10. If you get the message `adducer: The group ‘vboxsf’ does not exist.`, run the command `sudo apt-get install virtualbox-guest-utils`
    - Reboot the machine again with `sudo reboot` and login
    - Run the `sudo adduser goldrush vboxsf` command again
    - You should get a message that confirms the user has been added to the group

![spl-server-10]

11. Create a `share` folder with the command `mkdir share`
12. We’ll mount our shared folder onto our `share` directory with the following command:
    - `sudo mount -t vboxsf -o uid=1000,gid=1000 Splunk share/`
    - Substitute the shared folder name from Step 6 for `Splunk` if different
    - When we run the command `ls -la`, we should see that the `share` folder is highlighted green

![spl-server-12]

13. Navigate to the shared folder with the command `cd share/`
14. Install the package by running the command `sudo dpkg -i splunk-[...]`

![spl-server-14a]
![spl-server-14b]

15. Once the installation is complete, switch to the `splunk` user by running `sudo -u splunk bash`
16. Navigate to the `bin` folder with `cd bin`
17. To start Splunk, run the command `./splunk start`
18. When presented with Splunk’s terms and agreement, press/hold **Space** to scroll to the bottom and press **Y** and then **Enter** to agree

![spl-server-18]

19. Enter an administrator name and password
    - The username and password will be used for logging on Splunk
    - I used `goldrush` and `splunk-serverPW1` once again
20. We can automatically start the Splunk server whenever the VM boots up with the following commands:
    - `exit` to switch back to the `goldrush` user
    - `sudo bin/splunk enable boot-start -user splunk`

![spl-server-20]

---

## win-workstation VM
### Splunk Universal Forwarder Installation
1. Open [splunk.com](https://splunk.com) and sign up for an account if you don’t already have one
2. After signing on, navigate to **Platform** > **Free Trials & Downloads** and click on [Get My Free Trial](https://www.splunk.com/en_us/download/universal-forwarder.html) under **Universal Forwarder**
3. Download the installation package for Windows
4. Open the `.msi` file to start installation
5. Check the box to accept the License Agreement, select `An on-premises Splunk Enterprise instance`, and press `Next`
6. We’ll use `admin` as the username and allow Splunk to generate a random password
7. We’ll leave the **Deployment Server** section empty and click `Next`
8. For the **Receiving Indexer**, we’ll enter the IP address of our Splunk server (`192.168.10.10`) and use the default port (`9997`)
9. Click `Install`

### Sysmon Installation
1. Download Sysmon from the [Sysinternals website](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) and save it to the Downloads folder
2. In a new tab or window, download this [Sysmon configuration file](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml) by Olaf Hartong by clicking on the `Download raw file` icon and save it to the Downloads folder

![win-sysmon-02]

3. Right-click on the downloaded `Sysmon` zip file and select `Extract All...`
4. Copy the default destination path before clicking `Extract`
5. Run PowerShell as administrator
6. Enter `cd` and paste the destination path
   - ex. `cd C:\Users\win-workstation\Downloads\Sysmon`
7. Run the command `.Sysmon64.exe -i ..\sysmonconfig.xml`

![win-sysmon-07]

8. Agree to the license terms

### Splunk Universal Forwarder Configuration
To instruct the Splunk Universal Forwarder on what information to send to the Splunk server, we’ll need to configure an `inputs.conf` file.
1. Open Notepad as administrator
2. Copy and paste the text below (courtesy of [MyDFIR](https://github.com/MyDFIR/Active-Directory-Project)) into Notepad
   ```
   [WinEventLog://Application]
   index = endpoint
   disabled = false

   [WinEventLog://Security]
   index = endpoint
   disabled = false

   [WinEventLog://System]
   index = endpoint
   disabled = false

   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   index = endpoint
   disabled = false
   renderXml = true
   source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
   ```
3. Save the file under **This PC** > **Local Disk (C:)** > **Program Files** > **SplunkUniversalForwarder** > **etc** > **system** > **local** with the file name `inputs.conf` and the save as type set to `All Files`

![win-uf-03]

4. Search for “Services” in the Windows search bar and run as administrator
5. Locate `SplunkForwarder` in the list of services and double-click
6. Switch to the **Log On** tab and select `Local System account`

![win-uf-06]

7. Click `Apply` and then `OK` to close the window
8. Back in the **Services** window, right-click on `SplunkForwarder` and click on `Restart` to apply the inputs configuration and log on as local system account settings
   - If you receive an error message saying that Windows could not stop the SplunkForwarder service, click `OK` to dismiss the message and right-click on `SplunkForwarder` and then click `Start`

![win-uf-08]

## Splunk Index Configuration
1. Open [192.168.10.10:8000](https://192.168.10.10:8000) in a web browser and login to Splunk with the username and password created in step 19 of the **Splunk Server Installation** section (`goldrush`/`splunk-serverPW1`)
2. Navigate to **Settings** > **Indexes** and click on `New Index`
3. Enter `endpoint` for the Index Name and click `Save`

![win-index-3]

4. Navigate to **Settings** > **Forwarding and receiving** and click on `Configure receiving` under the **Receive data** section

![win-index-04]

5. Click on the `New Receiving Port` button
6. Enter `9997` for the listening port and click `Save`
7. Navigate to **Apps** > **Search & Reporting**
8. In the search bar, enter `index=endpoint` and click on the search button
9. If everything was configured properly, we should have events returned in the search
10. When we click on `host` under **SELECTED FIELDS** in the left panel, we should see `WIN-WORKSTATION` as a value

![win-index-10]

11. When we click on `source` under **SELECTED FIELDS**, we should see the event sources match those specified in our `inputs.conf` file

---

## dc-server VM
Setting up Splunk on `dc-server` is nearly identical to the process for `win-workstation`.

### Splunk Universal Forwarder Installation
1. Sign on to Splunk and download the [Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)  for Windows Server 2025
2. Open the `.msi` file to start installation
3. Check the box to accept the License Agreement, select `An on-premises Splunk Enterprise instance`, and press `Next`
4. We’ll use `admin` as the username and allow Splunk to generate a random password
5. We’ll leave the **Deployment Server** section empty and click `Next`
6. For the **Receiving Indexer**, we’ll enter the IP address of our Splunk server (`192.168.10.10`) and use the default port (`9997`)
7. Click `Install`

### Sysmon Installation
1. Download Sysmon from the [Sysinternals website](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) and save it to the Downloads folder
2. In a new tab or window, download this [Sysmon configuration file](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml) by Olaf Hartong by clicking on the `Download raw file` icon and save it to the Downloads folder
3. Right-click on the downloaded `Sysmon` zip file and select `Extract All...`
4. Copy the default destination path before clicking `Extract`
5. Run PowerShell as administrator
6. Enter `cd` and paste the destination path
   - ex. `cd C:\Users\Administrator\Downloads\Sysmon`
7. Run the command `.Sysmon64.exe -i ..\sysmonconfig.xml`
8. Agree to the license terms

### Splunk Universal Forwarder Configuration
1. Open Notepad as administrator
2. Copy and paste the text below (courtesy of [MyDFIR](https://github.com/MyDFIR/Active-Directory-Project)) into Notepad
   ```
   [WinEventLog://Application]
   index = endpoint
   disabled = false

   [WinEventLog://Security]
   index = endpoint
   disabled = false

   [WinEventLog://System]
   index = endpoint
   disabled = false

   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   index = endpoint
   disabled = false
   renderXml = true
   source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
   ```
3. Save the file under **This PC** > **Local Disk (C:)** > **Program Files** > **SplunkUniversalForwarder** > **etc** > **system** > **local** with the file name `inputs.conf` and the save as type set to `All Files`
4. Search for “Services” in the Windows search bar and run as administrator
5. Locate `SplunkForwarder` in the list of services and double-click
6. Switch to the **Log On** tab and select `Local System account`
7. Click `Apply` and then `OK` to close the window
8. Back in the **Services** window, right-click on `SplunkForwarder` and click on `Restart` to apply the inputs configuration and log on as local system account settings
   - If you receive an error message saying that Windows could not stop the SplunkForwarder service, click `OK` to dismiss the message and right-click on `SplunkForwarder` and then click `Start`

### Verify on Splunk Web
1. Open [192.168.10.10:8000](https://192.168.10.10:8000) in a web browser and login to Splunk with the username and password created in step 19 of the **Splunk Server Installation** section (`goldrush`/`splunk-serverPW1`)
2. Navigate to **Apps** > **Search & Reporting**
3. In the search bar, enter `index=endpoint` and click on the search button
5. When we click on `host` under **SELECTED FIELDS** in the left panel, we should now see two values: `WIN-WORKSTATION` and `DC-SERVER`

![dcs-verify-05]

6. Click on the `DC-SERVER` value to filter events coming from `dc-server`
7. When we click on `source` under **SELECTED FIELDS**, we should see the event sources match those specified in our `inputs.conf` file

![dcs-verify-07]



[dcs-verify-05]: ./img/03/03-dcs-verify-05.png
[dcs-verify-07]: ./img/03/03-dcs-verify-07.png
[spl-server-04]: ./img/03/03-spl-server-04.png
[spl-server-05]: ./img/03/03-spl-server-05.png
[spl-server-06]: ./img/03/03-spl-server-06.png
[spl-server-10]: ./img/03/03-spl-server-10.png
[spl-server-12]: ./img/03/03-spl-server-12.png
[spl-server-14a]: ./img/03/03-spl-server-14a.png
[spl-server-14b]: ./img/03/03-spl-server-14b.png
[spl-server-18]: ./img/03/03-spl-server-18.png
[spl-server-20]: ./img/03/03-spl-server-20.png
[win-index-01]: ./img/03/03-win-index-01.png
[win-index-03]: ./img/03/03-win-index-03.png
[win-index-04]: ./img/03/03-win-index-04.png
[win-index-10]: ./img/03/03-win-index-10.png
[win-sysmon-02]: ./img/03/03-win-sysmon-02.png
[win-sysmon-07]: ./img/03/03-win-sysmon-07.png
[win-uf-03]: ./img/03/03-win-uf-03.png
[win-uf-06]: ./img/03/03-win-uf-06.png
[win-uf-08]: ./img/03/03-win-uf-08.png