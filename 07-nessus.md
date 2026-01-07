# Parts
1. [Setting Up Virtual Machines](https://github.com/larryn-tech/homelab/blob/main/01-vm_setup.md)
2. [Configuring Active Directory and Domain Controller](https://github.com/larryn-tech/homelab/blob/main/02-active_directory.md)
3. [Configuring Splunk](https://github.com/larryn-tech/homelab/blob/main/03-splunk_configuration.md)
4. [Simulating Brute Force Attack and Atomic Red Team](https://github.com/larryn-tech/homelab/blob/main/04-attack.md)
5. [Login Hardening and Splunk Alerts](https://github.com/larryn-tech/homelab/blob/main/05-login_hardening.md)
6. [Adding a Metasploitable 2 to the Network](https://github.com/larryn-tech/homelab/blob/main/06-metasploitable.md)
7. Vulnerability Scanning with Nessus (Current)

# Vulnerability Scanning

In this part, we will:
- Install Nessus Essentials on `dc-server`
- Run a non-credentialed and credentialed vulnerable scan on `vuln-machine`
- Review scan results and generate a report

---
## Registering for a Nessus Essentials Activation Key
1. Use this [link](https://www.tenable.com/products/nessus/nessus-essentials) to register for an activation code using your name and email address
2. We’ll receive the code via email from no-reply@tenable.com

---

## `dc-server`

### Nessus Installation
1. Use this [link](https://www.tenable.com/downloads/nessus) to download Nessus
2. Open the file and follow the installation steps
3. Once the installation completes, a welcome page will automatically open in the web browser

![dcs-install-03]

4. Click on `Connect via SSL`
5. We’ll get a warning indicating that the connection isn’t private
   - Click on `Advanced` and `Continue to localhost (unsafe)` to bypass the warning

![dcs-install-05]

6. At the welcome screen, check the `Register Offline` box and continue
7. Keep the `Nessus Expert` option selected and continue
8. At the **Register Nessus** screen, copy the challenge code

![dcs-install-08]

9. Click on the `Offline Registration` link and paste the challenge code into the first text box

![dcs-install-09]

10. Retrieve the activation code that was emailed to you, enter it into the second text box, and click `Submit`
11. At the **Thank you** screen, copy the entire license code block
12. Return to the `localhost` tab in step 8 and paste the license key and continue
13. Create a username and password for the Nessus administrator account and submit

![dcs-install-13]

14. At the Nessus Essentials homepage, go to **Settings** > **Software Update**
15. Select the `Update all components` option for Automatic Updates, keep the frequency to `Daily`

![dcs-install-15]


16. Once we click `Save`, this will trigger Nessus to update its plugins and components

> Note: The update can take a long time to complete (mine took ~3 hours) and the VM may need to be restarted a few times. We can monitor the progress by viewing the **Events** tab or hovering over the progress circle icon.

![dcs-install-16]

17. We’ll know when the update is complete when the `New Scan` and `Create a new scan` buttons are no longer disabled

### Running a non-credentialed scan on `vuln-machine`
> For the vulnerability scans, make sure that the `vuln-machine` VM is running

1. Click on `New Scan`
2. Click on `Basic Network Scan`

![dcs-noncred-02]

3. We’ll add a name, description (optional), and specify `vuln-machine`’s IP address (`192.168.10.101`) for the target

![dcs-noncred-03]

4. We’ll keep the default settings for everything else and click `Save`
5. Click on the play button to launch run the scan

![dcs-noncred-05]

6. We can monitor the progress of the scan by clicking on the scan

![dcs-noncred-06]

### Running a credentialed scan on `vuln-machine`
1. Navigate back to **My Scans** using the side menu, and click on `New Scan`
2. Click on `Basic Network Scan`
3. Again, we’ll add a name, description, and specify `vuln-machine`’s IP address for the target

![dcs-cred-03]

4. Click on the **Credentials** tab, and click on `SSH` from the list
5. Change the authentication method to `password` and enter `msfadmin` for both the username and password

![dcs-cred-05]

6. Click `Save` and run the scan

### Reviewing scan results
The non-credentialed scan found 72 vulnerabilities, 18 of which are critical or high severity.

![dcs-review-01]

When we click on the **Vulnerabilities** tab, we’ll see a list of all the vulnerabilities detected in the scan.

![dcs-review-03]

In addition to the severity and name, we can see the family, or category, the vulnerability belongs to and their various vulnerability scores.

> The Common Vulnerability Scoring System (CVSS) score is used to quantify the severity of a vulnerability and help prioritize remediation efforts. The score ranges from 0-10, with 10 being most severe, and is based on factors such as impact, exploitability, and availability of patches.
>
> ![CVSS Ratings]
> *Image source: [NIST National Vulnerability Database](https://nvd.nist.gov/vuln-metrics/cvss)*

> The Vulnerability Priority Rating (VPR) is a dynamic metric developed by Tenable that measures a vulnerability's severity. Unlike the CVSS score, a vulnerability's VPR changes over time to reflect the current threat landscape. It considers factors such as how long a vulnerability has been in the National Vulnerability Database and how recently a threat event for it has occurred. The VPR score ranges from 0.1-10, with 10 indicating the highest exploitability.

> The Exploit Prediction Scoring System (EPSS) score measures the likelihood that an exploitation attempt against a vulnerability occurs in the next 30 days. Per the developers of the scoring system, the Forum of Incident Response and Security Teams (FIRST), "EPSS is best used when there is no other evidence of active exploitation." The score ranges between 0-1 (0-100%) and represents the probability that a vulnerability will be exploited.
>
> ![EPSS Scores]

The credentialed scan, on the other hand, found 91 vulnerabilities.

### Generating a vulnerability report
1. Filter the critical and high severity vulnerabilities
2. Click `Report`
3. Select `Complete List of Vulnerabilities by Host` and click `Generate Report`