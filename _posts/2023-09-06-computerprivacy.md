---
layout: post
title:  "Extreme Privacy Setup for Computers"
command: cat computer-privacy
---

Hello mate,

This is a blog post aiming to achieve extreme privacy setup/settings in computer systems. I am going to walk you through the methodologies and settings mentioned by Michael Bazzell in his Extreme Privacy Edition. Lets try to maximize the privacy and security of our machines..!

# Introduction

In an age where data breaches and online surveillance have become all too common, taking steps to protect your digital privacy is paramount. While most people are aware of basic privacy measures like using strong passwords and enabling two-factor authentication, setting up extreme privacy on your computer takes things to a whole new level. In this comprehensive guide, we'll walk you through the steps to create an ironclad fortress around your digital life.

# Linux is your BF

Yes, when it matters to privacy linux is the only recommended operating system to work on. Many people argue that Apple is more private than linux, but its not. Apple requires an online account to download and use many applications on the system. Apple tracks many details related to the associated Apple ID that a user has. Apple collects information such as Approximate location, IP address history, Search history, Programs Downloaded and opened, etc. So, with Apple we cant acheive extreme privacy. Then WINDOWS...? Absolutely not. I know, You know and every other single person who is a tech savy knows that Microsoft tracks huge information of a windows machine. Dont worry I will walk you through the **Hardening of Windows & MacOs** too. 

> So, Finally. Linux is our BF.

# New Linux Computer Configuration

Some cool qualities of one of the popular linux system ubuntu are :

1. Ubuntu allows easy access to software packages 1n a graphical interface.
2. Ubuntu has some of the highest compatibility with existing computers.
3. Ubuntu provides easy software update options.
4. Ubuntu reflects a large portion of Linux users, and online support is abundant.
5. Ubuntu has fewer driver issues than other systems when adding new hardware.
6. Ubuntu has removed the controversial Amazon affiliate links in previous builds.
7. Ubuntu’s large user base makes us all a smaller needle in the Linux haystack.


### Installing Ubuntu

Go to the official Ubuntu website at <a href="https://www.ubuntu.com/download/desktop" target=_blank>Ubuntu</a>. Download the latest Long-Term Support (LTS) Desktop version. At the time of writing, it was 22.04.3 LTS, but check for the most recent version. The download will be a large ISO file. After downloading ISO image navigate to <a href="https://etcher.balena.io/" target=_blank>Etcher</a> and downlad the Balena Etcher software to boot the ISO image into USB. 

<!-- ![LCG](/assets/img/post_img/googlectf23_lcg.png) -->

<img src ="/assets/img/blog_img/blog1_etcher.png" alt="balena etcher" class="autoimg">
<center> <i> Balena Etcher </i></center>

Insert the Ubuntu USB device into your computer. Power on the computer and, if the Ubuntu install screen doesn't appear, research the appropriate key (usually F1, F2, F10, delete, or escape) to select the boot device or access the BIOS settings. Choose USB device as the intaller. 

<img src="/assets/img/blog_img/blog1_boot.png" alt="Boot menu" class="autoimg"/>
<center> <i>Sample boot menu of HP Computer </i></center>

- On the Welcome screen, choose "Install Ubuntu" and select your preferred language.

<img src="/assets/img/blog_img/blog1_ubuntu1.png" alt="Ubuntu install 1" class="autoimg"/>
<center> <i> Welcome Screen</i></center>

- Choose "Normal Installation" and check both download options under "Other."

<img src="/assets/img/blog_img/blog1_ubuntu2.png" alt="Ubuntu install 2" class="autoimg"/>
<center> <i>Select Normal Install </i></center>

- If you no longer need any data on your computer's drive, select "Erase disk and install Ubuntu." Be cautious; this action will erase all data on the drive.
- Click "Advanced features," select "Use LVM with the...," and choose the "Encrypt the new..." option.
Click OK, then click "Install Now."

<img src="/assets/img/blog_img/blog1_ubuntu3.png" alt="Ubuntu install 3" class="autoimg"/>
<center> <i>Select erase disk and install</i></center>

Now follow these basic steps to complete the installation. 

- Create a secure password that you can remember and is not used elsewhere.
- If overwriting a used computer, consider the "Overwrite empty disk space" option, which will delete all data on the drive (this might take a while).
- Click "Install Now," "Continue," choose a location, and click "Continue."
- Provide a generic name for your computer, like "Laptop," and enter a secure password. You can use the same password as the encryption password for convenience or choose a unique one for extra security. These passwords will be required each time you boot the computer.
- Confirm your selections, let the installation complete, and then reboot your computer.
- Provide your passwords, and click "Skip" on the welcome screen.
- Select "No, don't send system info," then click "Next," "Next," and "Done."
- If prompted, click "Install Now" to install updates and allow your computer to reboot.

### Post Installation

The LVM full disk encryption prevents someone from accessing your data even if they remove your hard drive. And there few things to do after a fresh install of ubuntu to stop sending system status information to ubuntu managers. 

- Disable Ubuntu’s crash reporting and usage statistics with following commands. 

```sh
mj0ln1r@Ubuntu:~$ sudo apt purge -y apport
mj0ln1r@Ubuntu:~$ sudo apt remove -y popularity-contest
mj0ln1r@Ubuntu:~$ sudo apt autoremove -y
```

- Disable Notifications : Open "Settings",goto "Notifications" disable both options. 
- Disable File History : Open "Settings",goto "Privacy" then "File history & Trash" disable any options.
- Disable Diagnostics : Open "Settings",goto "Privacy" then "Diagnostics" Change to "Never".

### Installing Additional Applications

#### Antivirus

Viruses are rare in linux systems, but they do exists. To perform an antivirus scan we can use `ClamAV` applications.

Installing and performing a scan using ClamAV.

```bash
mj0ln1r@Ubuntu:~$ sudo apt update
mj0ln1r@Ubuntu:~$ sudo apt install -y clamav clamav-daemon
mj0ln1r@Ubuntu:~$ sudo systemctl stop clamav-freshclam
mj0ln1r@Ubuntu:~$ sudo freshclam
mj0ln1r@Ubuntu:~$ sudo systemctl start clamav-freshclam
mj0ln1r@Ubuntu:~$ clamscan -r -i --remove=yes /
```

#### System Cleaner

`Bleachbit` is a system cleaner software which is free and easy to use and effective to clean the system. I use `bleachbit` whenever I am going to shutdown my laptop. 

```bash
mj0ln1r@Ubuntu:~$ sudo apt update
mj0ln1r@Ubuntu:~$ sudo apt install bleachbit -y
mj0ln1r@Ubuntu:~$ bleachbit
```

#### Backups

Eventhough linux is stable and robust enough, but bad things may happen. So, keeping a backup of our important files is important to avoid any loss after a disaster. In ubuntu we can backup our data using backups tool which presents with multiple location options to store backups. We could choose local file server, local folder, or any cloud storage like google drive. Note that these backups are not encrypted.


# New Apple Computer Configuration

Lets, try to maximize the privacy settings on new Apple laptop or computer. Apple macOS systems are more secure than microsoft systems. We are going to discuss the things that we could mind to maximize the privacy of macOS. 

#### Apple ID

When first booting a new or reformatted macOS device, you will be prompted to provide an Apple ID, or create a new Apple ID account providing your name, physical address, and email address. You have the option to bypass this requirement, but you will be prohibited from using the App Store.

> I suggest you to eliminate the Apple ID

We can still install softwares without using apple store.

#### FileVault

Applying full disk encryption makes the file access without the user password impossible. To enable this goto "System Preferences" application and selecting "Security & Privacy". Choose the "FileVault" option to see the current state of encryption on your device. Enabling FileVault displays a 24 letters recovery key, choose to not to store on iCloud store this in a password manager. Once you have enabled FileVault's full-disk encryption, your system possesses an extremely important level of security. The entire contents of your computer’s storage can only be read once your password has been entered upon initial login or after standby login.

#### Brew

Brew is a package manager which is used to install applications on system without using Apple Store. To intsall brew run the following command in terminal.

```bash
mj0ln1r@Ubuntu:~$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/ HEAD /install.sh)"
```

#### Antivirus

We can consider using `ClamAV` for virus scanning on macOS. Follow these steps to install and run the ClamAV. 

```bash
mj0ln1r@Ubuntu:~$ brew analytics off
mj0ln1r@Ubuntu:~$ brew install clamav
mj0ln1r@Ubuntu:~$ sudo mkdir /usr/local/sbin
mj0ln1r@Ubuntu:~$ sudo chown -R “‘whoami*:admin /usr/local/sbin
mj0ln1r@Ubuntu:~$ brew link clamav
mj0ln1r@Ubuntu:~$ cd /usr/local/etc/clamav/
mj0ln1r@Ubuntu:~$ cp freshclam.conf.sample freshclam.conf
mj0ln1r@Ubuntu:~$ sed -ie 's/“Example/#Example/g' freshclam.conf

mj0ln1r@Ubuntu:~$ clamscan -i -r --remove=yes /
```

#### OverSight

Oversight application detects applications which uses the external devices of the computer such as microphones. Run the following command to install oversight.

```bash
mj0ln1r@Ubuntu:~$ brew install oversight
```

#### Onyx

Onyx corrects the undesirable behaviours and glitches. To install onyx use the following command. 

```bash
mj0ln1r@Ubuntu:~$ brew install onyx
```

#### VirtualBox

If you want to run any windows applications in an isolated environment you can consider using virtualbox.

```bash
mj0ln1r@Ubuntu:~$ brew install virtualbox
```

#### Updates

Use the following commands to update and upgrade the brew package manager. The cleanup and autoremove options will be used to remove the cached and unnecessary files. 

```bash
mj0ln1r@Ubuntu:~$ brew update
mj0ln1r@Ubuntu:~$ brew upgrade
mj0ln1r@Ubuntu:~$ brew cleanup -s
mj0ln1r@Ubuntu:~$ brew autoremove
```

#### System Cleaner

Unfortunately bleachbit not available on macOS. But we can use the customized cleaner script provided by the `Privacy.sexy` to clean the system according to our choice.

#### Little Snitch

Little Snitch is an application similiar to a firewall. Little Snitch controlls the in and out traffic of the system. 


# New Microsoft Windows Computer Configuration

Okay, as most of you are using windows lets try to setup basic privacy settings on windows. 

Windows requires microsoft account to enable few services like appstore, but we can bypass this. While installing windows turn off wife in offline mode we will be prompted to create the microsoft account later. Eventhough we bypass the microsoft account, they collect these many information with telemetry services. 

1. Typed text on keyboard
2. Microphone transmissions
3. Index of all media files on your computer
4. Webcam data
5. Browsing history
6. Search history
7. Location activity
8. Heath activity collected by HealthVault, Microsoft Band, other trackers
9. Privacy settings across Microsoft application ecosystem

#### Full disk encryption

If you have windows 10/11 pro version you can enable `Bitlocker` to enable fulldisk encryption. If not we can use `VeraCrypt` to enable fulldisk encryption.

Download veracrypt from www.veracrypt.fr and run the installer. Use "Normal" system encryption and choose "Encrypt whole drive" and select "Single-Boot". Keep the rescue disk created by veracrypt to decrypt your files if the bootloader was failed anytime. Select "none" for the Wipe Mode and enter encryption key and when you attempt to login again you will be propted with encrypt button to encrypt the disk. 

#### System Cleaner

`Bleeeaaaachhhhhbbbbiiiittt agaiiiinnnn`

#### Glasswire/Portmaster

Glasswire and Portmaster replicates the work of Little Snitch, these applications used to control the system traffic and monitors the ports being used on system.

#### Package Management

Run the following command on powershell to display the list of applications installed on the build of windows. 

```sh
Powershell> Get-AppxProvisionedPackage -Online | Format-Table DisplayName, PackageName
```

You can run the following powershell script to remove the all the applications installed by the microsoft on the build. Feel free to add more applications if anything is missed. 


```sh
$ProvisionedAppPackageNames = @(
“Microsoft.3DBuilder”
‘“Microsoft.BingFinance”’
‘“Microsoft.BingNews”
“Microsoft.BingSports”’
“Microsoft.BingWeather”’
“Microsoft.ConnectivityStore”
““Microsoft.Getstarted”
““Microsoft.Messaging”’
““Microsoft.Microsoft3D
Viewer”’
‘“Microsoft.MicrosoftOfficeHub”
“Microsoft.MicrosoftSolitaireCollection’
“Microsoft.MicrosoftStickyNotes”
‘“Microsoft.MSPaint”
‘“Microsoft.Office. OneNote”
“Microsoft.People”
“Microsoft.Print3D”
‘“Microsoft.skypeApp”’
“Microsoft.StorePurchaseApp”’
“microsoft.windowscommunicationsapps” # Mail,Calendar
“Microsoft. WindowsFeedbackHub”
“Microsoft.WindowsPhone”’
“Microsoft.WindowsStore”’
“Microsoft.
Xbox. TCUI”
“Microsoft.XboxApp”
“Microsoft.XboxGameOverlay”
“Microsoft. XboxIdentityProvider”’
“Microsoft.XboxSpeechToTextOverlay”
“Microsoft.ZuneMusic”’
“Microsoft.ZuneVideo”’
“Microsoft. YourPhone’’)
?
foreach ($ProvisionedAppName in $ProvisionedAppPackageNames)
{
Get-AppxPackage -Name $ProvisionedAppName -AllUsers | Remove-AppxPackage
Get-AppXProvisionedPackage -Online | Where-Object DisplayName -EQ $ProvisionedAppName |
Remove-AppxProvisionedPackage -Online}
exit
```


# Conclusion

My bottom line is that Linux is more private and secure than Windows or macOS, and macOS is more private and secure than Windows. Some may say that Mac computers are more secure than Linux due to their "walled garden" which prevents many malicious apps from executing within the operating system. And Do you know I havent used windows for more than 10 to 20 days in my life and I tried nearly 30+ Linux distributions and settled with Ubuntu as the Home OS. 