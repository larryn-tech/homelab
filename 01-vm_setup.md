# Parts
1. Setting Up Virtual Machines (Current)
2. [Configuring Active Directory and Domain Controller](https://github.com/larryn-tech/homelab/blob/main/02-active_directory.md)
3. [Configuring Splunk](https://github.com/larryn-tech/homelab/blob/main/03-splunk_configuration.md)
4. [Simulating Brute Force Attack and Atomic Red Team](https://github.com/larryn-tech/homelab/blob/main/04-attack.md)
5. [Login Hardening and Splunk Alerts](https://github.com/larryn-tech/homelab/blob/main/05-login_hardening.md)
6. [Adding a Metasploitable 2 to the Network](https://github.com/larryn-tech/homelab/blob/main/06-metasploitable.md)


# Virtual Machines
Virtual machines (VMs) are software-based computers that run on top of your existing physical computer, allowing you to operate multiple independent operating systems simultaneously. A home lab built with VMs lets you safely practice setting up networks, servers (like Active Directory Domain Services), and various software without the risk of damaging your main computer or needing expensive dedicated hardware. Oracle VirtualBox is an open-source virtualization software that allows users to create and run VMs on their computers.

## VirtualBox Installation
VirtualBox can be downloaded on their [website](https://www.oracle.com/virtualization/technologies/vm/downloads/virtualbox-downloads.html?source=:ow:o:p:nav:mmddyyVirtualBoxHero&intcmp=:ow:o:p:nav:mmddyyVirtualBoxHero).

During setup, you may be prompted to install Microsoft Visual C++ 2019 Redistributable Package first. The download link for the dependency can be found [here](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170).

## Network
We will create a NAT Network that will connect each of the virtual machines (VMs). Recall that the IP settings for the network will be as follows:
|	| IP Address	|
|:----|:----|
| `Network` | `192.168.10.0/24` |
| `Gateway` | `192.168.10.1` |
| `dc-server` | `192.168.10.7` |
| `splunk-server` | `192.168.10.10` |
| `win-workstation` | `192.168.10.100` |
| `vuln-machine` | `192.168.10.101` |
| `attacker` | `192.168.10.250` |

1. In Oracle VirtualBox Manager, navigate to **File** > **Tools** > **Network Manager**
2. Select the **NAT Networks** tab and then click on the **Create** button
3. Under **General Options**, add a name and the network’s IPv4 prefix:
	- Name: `goldrush-network`
	- IPv4 Prefix: `192.168.10.0/24`
4. Click `Apply`

## Virtual Machine Settings

| Machine | Operating System | Memory / Processors | Storage |
|:----|:----|:----|:----|
| `dc-server` | [Windows Server 2025](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025) | 4096 MB / 2 CPU | 50 GB |
| `splunk-server` | [Ubuntu Server](https://ubuntu.com/download/server) | 8192 MB / 2 CPU | 100 GB |
| `win-workstation` | [Windows 10 Pro](https://www.microsoft.com/en-us/software-download/windows10ISO) | 4096 MB / 2 CPU | 80 GB |
| `attacker` | [Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines) | 2048 MB / 1 CPU | 55 GB |

> Installation and setup for `vuln-workstation` will be covered in [Part 6](https://github.com/larryn-tech/homelab/blob/main/06-metasploitable.md) of this series.

### Creating a Virtual Machine
For each VM, we’ll need to download the corresponding ISO file linked under **Operating System** in the table above.
1. To create a VM in VirtualBox, go to **Machine** > **New**

![vbx-create-01]

2. For the **Name and Operating System** section, enter the name of the VM, select where to save it, and select the matching `.iso` file
	- The correct type and version should be automatically selected based on the `.iso` file, although I did need to change the version for `dc-server` from Windows 2022 to Windows 2025

	Check the `Skip Unattended Installation` box

![vbx-create-02]

3. For the **Hardware** section, select the amount of memory and processors to allocate to the VM based on the table above

![vbx-create-03]

4. In the **Hard Disk** section, select `Create a Virtual Hard Disk Now` and select the virtual hard disk storage storage size according to the table

![vbx-create-04]

5. Click `Finish` to create the new VM

6. For each machine, make sure it is connected to the created NAT network by clicking on it and going to **Settings** > **Network**. Select **Expert** and and configure the following settings:
	- Attached to: `NAT Network`
	- Name: `goldrush-network`

![settings-01]

---

## `dc-server` VM
### Windows Server Installation
Similar to the Windows workstation setup, you will be guided through Windows Setup upon powering on the `dc-server` VM.
1. Select your language, time and currency format, then click `Next`
2. Select your keyboard or input method, then click `Next`
3. Select `Install Windows Server`, check the `I agree everything will be deleted including files, apps, and settings` box, and then click `Next`
4. At the **Select Image** screen, select `Windows Server 2025 Standard Evaluation (Desktop Experience)` and then click `Next`
5. Agree to the notices and license terms by clicking `Accept`
6. Click `Next` to install Windows Server on `Drive 0`
7. Click `Install` at the `Ready to install` screen
8. Once the installation is complete, you will be prompted to create a password for the built-in administrator account at the **Customize settings** screen.
	- For this guide, we’ll use the password `dc-serverPW1`
9. At the Lock Screen, navigate to the menu bar of the VM and go to **Input** > **Keyboard** > **Insert Ctrl-Alt-Del** to unlock and open the login screen

![dcs-windows-10]

10. Enter the password for `Administrator` (`dc-serverPW1`)

**Optional**: We can disable the need to enter **CTRL + ALT + DEL** to unlock in **Edit Group Policy**.
1. Search “Edit Group Policy” in the Windows search bar and click `Open`

![dcs-windows_opt-01]

2. In the left pane, navigate to **Local Computer Policy** > **Computer Configuration** > **Windows Settings** > **Security Settings** > **Local Policies** > **Security Options**
3. Double-click on `Interactive logon: Do not require CTRL+ALT+DEL`, select `Enable`, and click `OK` to save

### Rename PC
We’ll rename the PC to `dc-server`.
1. Enter “This PC” in the search bar and click on `Open`
2. Click on `Rename this PC`
3. Enter `dc-server`, click `Next`, and then `Restart Now` to apply the change

### IP Address Configuration
1. Right-click the network icon in the taskbar and click on `Network and Internet settings`
2. Click on `Ethernet` and then the `Edit` button in the **IP assignment** row

![dcs-ip-02]

3. Select `Manual` for **Edit IP settings**
4. Toggle IPv4 to `On`
5. Enter the following settings:
	- IP address: `192.168.10.7`
  	- Subnet mask: `255.255.255.0`
	- Gateway: `192.168.10.1`
	- Preferred DNS: `8.8.8.8`

![dcs-ip-05]

6. Click on `Save`
7. In Command Prompt, we should see our updated IP settings when running `ipconfig` and be able to ping Google

![dcs-ip-07]

---

## `splunk-server` VM
### Ubuntu Server Installation
After powering up the `splunk-server` VM, we are presented with the GNU GRUB boot loader. For most of the options, we will simply press `Done` to select the default options.
1. Select `Try or Install Ubuntu Server`
2. Select your language
3. Select `Continue without updating`
4. Select your keyboard layout
5. Select `Ubuntu Server` for the type of install
6. Select `enp0s3` interface
7. Leave `Proxy address` blank
8. Keep the default mirror for Ubuntu (you can continue while the check is still running)
9. Check `Use an entire disk` and `Set up this disk as an LVM group`
10. Press `Done` at the **Storage configuration** screen and then `Continue`
11. For **Profile setup**, I used the following:
	- Name: `Sam`
	- Server’s name: `splunk`
	- Username: `goldrush`
	- Password: `splunk-serverPW1`

![spl-ubuntu-11]

12. Select `Skip for now` for **Upgrade to Ubuntu Pro**
13. Skip SSH setup
14. Skip featured server snaps
15. Once the install is complete, select `Reboot Now`
16. You can ignore the `Failed unmounting /cdrom` message and press **Enter**

![spl-ubuntu-16]

### Package Update
1. After the installation is complete, login with the username and password you configured earlier (`goldrush`/`splunk-serverPW1`).

2. We’ll run the command `sudo apt-get update && sudo apt-get upgrade -y` in the command line to make sure all of our packages are up to date.

![spl-package-02]

3. When presented with the **Daemons using outdated libraries** screen, we can leave the default services selected and press `OK`.

![spl-package-03]

### IP Address Configuration
1. To set a static IP address for **`splunk-server`**, we’ll run the command `sudo nano /etc/netplan/00-installer-config.yaml` and enter the following into the text editor:

```yaml
network:
    ethernets:
        enp0s3:
            dhcp4: no
            addresses: [192.168.10.10/24]
            nameservers:
                addresses: [0.0.0.0]
        routes:
            - to: default
             via: 192.168.10.1
    version: 2
```

---

## `win-workstation` VM
### Windows Installation
After starting the `win-workstation` VM, we’ll be guided through the Windows 10 installation process.
1. Select your language, time and currency format, and keyboard layout
2. Click `Install now`
3. Click on `I don’t have a product key` to activate Windows later
4. Select `Windows 10 Pro` for the operating system
5. Accept the license terms
6. Choose `Custom: Install Windows only (advanced)`
7. Install Windows on Drive 0 (should be the only option listed)
8. Allow the installation to complete and the VM to restart
9. Select your region to complete the setup

### IP Address Configuration
1. Right-click on the network icon in the toolbar and select `Open Network & Internet settings`
2. Under **Ethernet**, click on `Change adapter options`
3. Right-click on `Ethernet` and select `Properties`

![win-ip-03]

4. In the **Ethernet Properties** window, double-click on `Internet Protocol Version 4 (TCP/IPv4)`
5. Enter the following settings:
   - **IP address**: `192.168.10.100`
   - **Subnet mask**: `255.255.255.0`
   - **Default gateway**: `192.168.10.1`
   - **Preferred DNS server**: `192.168.10.7`
6. Press `OK`, then `OK` for the **Ethernet Properties** window
7. We can verify the IP settings by running `ipconfig` in Command Prompt

![win-ip-07]

---

## `attacker` VM
### Kali Linux Installation
1. Start the `attacker` VM
2. At the Kali Linux Boot screen, select the `Graphical Install` option
3. Select your preferred language, location, and keyboard layout
4. Change the hostname from `kali` to `attacker`
5. Leave the domain name empty
6. Enter `attacker` for both the full name and username
7. Enter `attackerPW1` for the password
8. Set your time zone
9. Select `Guided - use entire disk` for the disk partitioning method
10. Continue with the selected disk to partition
11. Select `All files in one partition` for the partitioning scheme
12. Select `Finish partitioning and write changes to disk`
13. Select `Yes` to write the changes to disks
14. Keep the default software to install selected and continue
15. Select `Yes` to install the GRUB boot loader
16. Select `/dev/sda` device for boot loader installation
17. Click `continue` to finish the installation and reboot the system
18. Login with `attacker` and `attackerPW1`
19. Open the terminal and run the following command to update and install packages:

```bash
sudo apt-get update && sudo apt-get install -y
```

> If you run into a GPG error where signatures couldn’t be verified because the public key is not available, run the following command to manually download and install the [new key](https://archive.kali.org/archive-keyring.gpg):

```bash
sudo wget https://archive.kali.org/archive-keyring.gpg -O /usr/share/keyrings/kali-archive-keyring.gpg
```

> And then rerun the update and install command above

![atk-kali-19]


### IP Address Configuration
1. Right-click on the network icon in the menu bar and select `Edit Connections`

![atk-ip-01]

2. Select `Wired connection 1` and click on the gears icon to edit

![atk-ip-02]

3. In the **IPv4 Settings** tab, click on `Add` and configure the following settings:
	- **Address**: `192.168.10.250`
    - **Netmask**: `24`
    - **Gateway**: `192.168.10.1`
    - **DNS servers**: `8.8.8.8`

![atk-ip-03]

4. Click `Save`
5. Verify the IP settings by opening the terminal and running the `ifconfig` command
6. We can also check that our network has been configured correctly by  pinging Google

![atk-ip-06]



[atk-ip-01]: ./img/01/01-atk-ip-01.png
[atk-ip-02]: ./img/01/01-atk-ip-02.png
[atk-ip-03]: ./img/01/01-atk-ip-03.png
[atk-ip-06]: ./img/01/01-atk-ip-06.png
[atk-kali-19]: ./img/01/01-atk-kali-19.png
[dcs-ip-02]: ./img/01/01-dcs-ip-02.png
[dcs-ip-05]: ./img/01/01-dcs-ip-05.png
[dcs-ip-07]: ./img/01/01-dcs-ip-07.png
[dcs-windows_opt-01]: ./img/01/01-dcs-windows_opt-01.png
[dcs-windows-10]: ./img/01/01-dcs-windows-10.png
[settings-01]: ./img/01/01-settings-01.png
[spl-ip-01]: ./img/01/01-spl-ip-01.png
[spl-ip-04]: ./img/01/01-spl-ip-04.png
[spl-package-02]: ./img/01/01-spl-package-02.png
[spl-package-03]: ./img/01/01-spl-package-03.png
[spl-ubuntu-11]: ./img/01/01-spl-ubuntu-11.png
[spl-ubuntu-16]: ./img/01/01-spl-ubuntu-16.png
[win-ip-03]: ./img/01/01-win-ip-03.png
[win-ip-07]: ./img/01/01-win-ip-07.png
[vbx-create-01]: ./img/01/01-vbx-create-01.png
[vbx-create-02]: ./img/01/01-vbx-create-02.png
[vbx-create-03]: ./img/01/01-vbx-create-03.png
[vbx-create-04]: ./img/01/01-vbx-create-04.png