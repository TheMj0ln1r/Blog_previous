---
layout: post
title:  "Zip Cracker"
---

Hi mates!,

Mj0ln1r is here, this time with another simple project named "PythonZipCracker" developed with python.

As you know i am learning and building some simple python automated tools. PythonZipCracker is my third tool which is programmed by my own knowledge.

I wrote this script called `PythonZipCracker` in python. I learned `pyzipper and zipfile` modules in python.

So, here's a brief information about my simple PythonZipCracker.


# PythonZipCracker

This is a multiway password protected zip file cracker. The script is developed using python which can perform following tasks.

1. It can crack the password of a zip file with the rockyou.txt
2. It can crack the password of a zip file using your own password list
3. It can crack the password of a zip file with the bruteforcing of digits

Let's explore the main()


```python

def main():
	zfile = input("Enter absolute path of the zip file : ")
	if not os.path.isfile(zfile):
		print(zfile," is not found..")
		exit()
	dir = zfile[0:len(zfile)-4]
	if os.path.isdir(dir):
		print(dir, "is already present.. extracting may leads to overwrite of old files.")
		exit()
	print("""
Select method 
	1. Crack using built-in wordlist
	2. Crack with custom wordlist
	3. Crack using bruteforce method
	0. Exit
		""")
	choice = int(input("Enter your choice[1/2/3/0] : "))
	if choice == 1 :
		rocku(zfile)
	elif choice == 2 :
		wfile = input("Enter the absolute path of your password list: ")
		if not os.path.isfile(wfile):
			print(wfile," is not found..")
			exit()
		custom_passwd(zfile,wfile)
	elif choice == 3 :
		print("""
----------------------------------------------------------------------------
Bruteforcing method takes too much of time and system resources to perform..
Use this method to crack the passwords of length upto 4 only.
And the passwords are combination of digits only.
----------------------------------------------------------------------------
			""")
		passlen = int(input("Enter the length of password[Note: <5] : "))
		bruteforce(zfile,passlen)
	else :
		exit()
```
The main() function is printing the menu of the script and performing the input files are validations.
If the user chooses built-in wordlist method to crack the zip file then the `rocku()` function will be invoked.
```python
def rocku(zfile):
	wordlist = open("rockyou.txt","r")
	with pyzipper.AESZipFile(zfile) as zf:
		for i in wordlist:
			pwd = bytes(i.strip(),"utf-8")
			print("[-] Checking password : ",i)
			try:
				zf.extractall(pwd = pwd)
				print(50*"*")
				print("[+] Password is found : ",i)
				print("Files in ",zfile)
				for f in zf.namelist():
					print("\t",f)
				print(50*"*")
				break
		
			except:
				continue
```
The rocku() function is opening the rockyou.txt file in read mode and the zip file is opened using the pyzipper module. Then it is iterating over the each password in the rockyou.txt file and converting it into bytes. The password bytes is given as arguement to the extractall() function. If the password is found then the files will be extracted and the list of files and the password will be printed on screen. 

If the user chooses custom wordlist method to crack zip file the `custom_password()` function will be invoked.

```python
def custom_passwd(zfile,wfile):
	wordlist = open(wfile,"r")
	with pyzipper.AESZipFile(zfile) as zf:
		for i in wordlist:
			pwd = bytes(i.strip(),"utf-8")
			print("[-] Checking password : ",i)
			try:
				zf.extractall(pwd = pwd)
				print(50*"*")
				print("[+] Password is found : ",i)
				print("Files in ",zfile)
				for f in zf.namelist():
					print("\t",f)
				print(50*"*")
				break
		
			except:
				continue

```
The custom_password() function is doing the same thing as the rocku() function. Along with the zipfile the custom wordlist is needed to proceed.

If the user chooses bruteforce method the `bruteforce()` method will be invoked.
```python
def bruteforce(zfile,passlen):

	with pyzipper.AESZipFile(zfile) as zf:
		total_passwords = int(passlen* "9")
		for i in range(total_passwords+1):
			pwd = bytes(str(i),"utf-8")
			print("[-] Checking password : ",i)
			try:
				zf.extractall(pwd = pwd)
				print(50*"*")
				print("[+] Password is found : ",i)
				print("Files in ",zfile)
				for f in zf.namelist():
					print("\t",f)
				print(50*"*")
				break
		
			except:
				continue

```
A brute force attack is a hacking method that uses trial and error to crack passwords, login credentials, and encryption keys. It is a simple yet reliable tactic for gaining unauthorized access to individual accounts and organizations’ systems and networks. The hacker tries multiple usernames and passwords, often using a computer to test a wide range of combinations, until they find the correct login information.
The brute force approach is inefficient. For real-time problems, algorithm analysis often goes above the O(N!) order of growth.

As it is very time consuming for the long passwords i recommend to use this method for the passwords upto the length of 6. And this passwords are the combination of digits only.

### Requirements

You require these additional modules 

1. pyzipper

These modules will be pre installed on every system, if not then install them with `pip3 or pip`

**You need to download rockyou.txt and paste it in PythonZipCracker folder**
>Rockyou.txt : [https://www.kaggle.com/datasets/wjburns/common-password-list-rockyoutxt](https://www.kaggle.com/datasets/wjburns/common-password-list-rockyoutxt)

### Installation

```text 
git clone https://github.com/TheMj0ln1r/PythonZipCracker.git
cd PythonZipCracker
python3 crack.py
```
### Preview

![Preview](/assets/img/project_img/pythonzipcrack.png){: w="600",h="600"}

> If anything should be modified in the script please let me know.
{: .prompt-info }

You can find complete source code of PythonZipCracker here : [https://github.com/TheMj0ln1r/PythonZipCracker](https://github.com/TheMj0ln1r/PythonZipCracker)
