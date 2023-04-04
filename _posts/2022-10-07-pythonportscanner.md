---
layout: post
title:  "Python Port Scanner"
tags: [python,networking,portscanner]
categories: [Projects,Python]
---

Hi guys!,

Mj0ln1r is here, back again with a new post this is meta data about my new python script/tool Simple Python Port Scanner.

As you know i am learning and building some simple python automated tools. Port Scanner is my second tool which is programmed by my own knowledge.

I wrote this script called `Port_Scanner` in python. I learned `basics of socket programming` , little bit about `platform` module, `multi-threading in python` and a basic tool`ping` which is useful in computer networking.

So, here's a brief information about my simple python port scanner.

# Port Scanner

This is a basic computer networking based project. The script is developed using python which can perform following tasks.
1. Scanning open ports on a single target
2. Scanning open ports in every system in a network
Let's start with the main function.

```python 
def main():

	print("""
	1 > Scan one device
	2 > Scan Entire network
		""")
	choice = int(input("Enter your choice(1/2) : "))

	if choice == 1:

		print("Scanning only one target.. \n")
		global target_ip
		target_ip = input("Enter Target IP or URL : ")

		start_port = int(input("Enter start port : "))
		end_port = int(input("Enter end port : "))
		
		input_check(start_port,end_port)
		single_host_scan(start_port,end_port)
		sys.exit()
	elif choice == 2:

		print("Scanning Entire Network..\n")
		target_ip = input("Enter your IP address : ")
		start_host = int(input("Enter start host : "))
		end_host = int(input("Enter end host : "))
		start_port = int(input("Enter start port : "))
		end_port = int(input("Enter end port : "))
		
		input_check(start_port,end_port)
		host_check(start_host,end_host)
		multi_host_scan(start_host,end_host,start_port,end_port)
		sys.exit()
	else :
		print("Invalid choice.")
		sys.exit()
main()
```
In the beginning, this tool will asks the users for their choice to perform a specific task.
i.e, to scan a single target or an entire network. If the user choose 1, then the tool will take start port and end port as an input from the user. Then the program is going to validate the inputs given, this is done by the `input_check` function. Next the `single_host_scan` will be called. Take look at these two functions.

`input_check()`
	
```python
def input_check(start_port,end_port):
	global target_ip
	ip = re.search(r'(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})',target_ip)
	if not ip:
		try :
			target_ip = socket.gethostbyname(target_ip)
		except socket.gaierror :
			print("Unable resolve host address")
			sys.exit()
	if ((start_port < 1) or (end_port > 65535)):
		print("Port numbers are invalid.. \n")
		sys.exit()
```
 In the try block of the input_check function, if a user gave a website as target that url will be changed to IP address here.

`single_host_scan()`

```python
def single_host_scan(start_port,end_port):

	if islive():
		for port in range(start_port,end_port+1):
			thread = threading.Thread(target = single_port,args = (port,))
			thread.start()
			thread.join()
	else:
		print("\t Not reachable.\n")
```
In the above function the islive function will checks that the given host is live or not. If it is live then it is going to scan every port in a given range of that target host.

`islive()`

```python
def islive():
	try:
		if "Linux" in system():
			p = subprocess.run(["ping","-c","1",target_ip],capture_output=True,text=True)
			if p.returncode == 0:
				return True

		if "Windows" in system():
			p = subprocess.run(["ping","-n","1",target_ip],capture_output=True,text=True)
			if p.returncode == 0:
				return True

	except subprocess.CalledProcessError:
		return False
```
islive() function uses a ping scan to check the target is live or not.

If the target is live, a thread is created for each port to increase the speed of the scan. In every thread single_port() function is being called.

`single_port`

```python
def single_port(port):
		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		socket.setdefaulttimeout(1)
		status = s.connect_ex((target_ip,port))
		if(not status):
			print("\tOpen port ",port)
			s.close()
		s.close()
```
By using socket programming the connection request for a port will be sent to the remote host. If the connection was established with a port the status of the port will be printed as open.


Now if the user choose to scan entire network, the same concept is used to check the open ports.The functions involved in entire network scan are `host_check` and `multi_host_scan`.

`host_check()`

```python
def host_check(start_host,end_host):
	if (start_host < 1 or end_host > 255):
		print("Host range in not valid..\n")
		sys.exit()
```

`multi_host_scan()`

```python
def multi_host_scan(start_host,end_host,start_port,end_port):
	global target_ip
	tmp = target_ip
	ip = tmp.split(".")
	ip = ".".join(ip[0:3])+"."
	for host in range(start_host,end_host+1):
		target_ip = ip + str(host)
		print("Scannig Host : "+target_ip+"\n")
		single_host_scan(start_port,end_port)
	sys.exit()
```

I devoloped this script with less additional modules this can be executed in any system without any additional requirements.This scanning process will take less time. 

### Requirements

You may require some python modules

1. subprocess
2. socket
3. platform
4. threading
5. sys

These modules will be pre installed on every system, if not then install them with `pip3 or pip`
### Installation

```text 
git clone https://github.com/TheMj0ln1r/Port_Scanner.git
cd Port_Scanner
python3 Port_Scanner.py
```

### Preview 
![Preview](/assets/img/project_img/portscanner.png){: w="600",h="600"}

> If anything should be modified in the script please let me know.
{: .prompt-info }

You can find complete source code of Port Scanner here : [https://github.com/TheMj0ln1r/Port_Scanner](https://github.com/TheMj0ln1r/Port_Scanner)
