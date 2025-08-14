# Parts
1. [Setting Up Virtual Machines](https://github.com/larryn-tech/homelab/blob/main/01-vm_setup.md)
2. [Configuring Active Directory and Domain Controller](https://github.com/larryn-tech/homelab/blob/main/02-active_directory.md)
3. [Configuring Splunk](https://github.com/larryn-tech/homelab/blob/main/03-splunk_configuration.md)
4. [Simulating Brute Force Attack and Atomic Red Team](https://github.com/larryn-tech/homelab/blob/main/04-attack.md)
5. [Login Hardening and Splunk Alerts](https://github.com/larryn-tech/homelab/blob/main/05-login_hardening.md)
6. Adding a Metasploitable 2 to the Network (Current)

# Metasploitable 2
Created by the Rapid7 Metasploit team, Metasploitable 2 is an intentionally vulnerable version of Ubuntu Linux that contains several [security flaws](https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/) that can be exploited to gain unauthorized access, create backdoors, perform enumeration, and more. The virtual machine also includes DVWA, a vulnerable web application that is useful for learning the [OWASP Top 10](https://owasp.org/www-project-top-ten/). By adding Metasploitable 2 to our home lab, we’ll have a machine to practice vulnerability scanning, perform penetration testing, and remediation on.

In this part, we will:
- Create our `vuln-machine` using Metasploitable 2 in VirtualBox
- Configure a static IP address for `vuln-machine`
- Verify that Metasploitable 2 has been installed and visit its webpage.

---

## Installing Metasploitable 2 in VirtualBox
The process for installing Metasploitable 2 is slightly different compared to our other VMs. Rather than using an ISO file, we’ll be using a `.vmdk` (virtual hard disk) file.
1. Download Metasploitable 2 from [vulnhub.com](https://www.vulnhub.com/entry/metasploitable-2,29/)
2. Extract the downloaded zip file
3. In VirtualBox, we’ll create a virtual machine with the following settings:

	**Name and Operating System**
   - **Name**: `vuln-machine`
   - **Type**: `Linux`
   - **Subtype**: `Oracle Linux`
   - **Version**: `Oracle Linux (64-bit)`

	![vbx-install-03a]


   **Hardware** (Default settings)
   - **Base Memory**: `2048 MB`
   - **Processors**: `1 CPU`

	![vbx-install-03b]

   **Hard Disk**
   - Select "Use an Existing Virtual Hard Disk File” and click on the folder icon to choose a .`vmdk` file
   - Click on the `Add` button and select the `Metasploitable.vmdk` file extracted from the downloaded  `.zip` file

	![vbx-install-03c]
	![vbx-install-03d]

   - Click on `Choose`, then `Finish` to create the VM

	![vbx-install-03e]

3. Change the network settings for `vuln-machine` to connect to the `goldrush-network` NAT network

---

## `vuln-machine` VM

### Logging In
1. Start the `vuln-machine` in VirtualBox
2. Once the VM has loaded, you will be prompted to login with the default credentials:
   - **metasploitable login**: `msfadmin`
   - **password**: `msfadmin`

### IP Address Configuration
1. Run the command `sudo nano /etc/network/interfaces`
2. Change the last line (`face eth0 net dhcp`) to `face eth0 inet static`
3. Enter the following IP address, net mask, and default gateway:
   - **address**: `192.168.10.101`
   - **netmask**: `255.255.255.0`
   - **gateway**: `192.168.10.1`

![vln-ip-03]

4. Press **CTRL+X**, **Y**, and then **Enter** to save and exit
5. Restart networking with `sudo /etc/init.d/networking restart` to apply the changes
6. Verify the configuration with `ifconfig eth0`

![vln-ip-06]

7. Return to the root directory with `cd ~`

---

## `attacker` VM
We can verify that our `vuln-machine` has been setup correctly by opening [192.168.10.101](https://192.168.10.101) in Firefox.

![atk-verify-01]

This brings up the Metasploitable 2 webpage, where we can access the vulnerable web applications that comes with the VM.



[atk-verify-01]: ./img/06/06-atk-verify-01.png
[vbx-install-03a]: ./img/06/06-vbx-install-03a.png
[vbx-install-03b]: ./img/06/06-vbx-install-03b.png
[vbx-install-03c]: ./img/06/06-vbx-install-03c.png
[vbx-install-03d]: ./img/06/06-vbx-install-03d.png
[vbx-install-03e]: ./img/06/06-vbx-install-03e.png
[vln-ip-01]: ./img/06/06-vln-ip-01.png
[vln-ip-03]: ./img/06/06-vln-ip-03.png
[vln-ip-06]: ./img/06/06-vln-ip-06.png