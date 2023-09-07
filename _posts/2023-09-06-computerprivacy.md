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