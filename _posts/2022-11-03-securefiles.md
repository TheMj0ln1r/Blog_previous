---
layout: post
title:  "SecureFiles"
tags: [python,securefiles,fileencryptor,filedecryptor]
categories: [Projects,Python]
---

Hello buddy,

Today Mj0ln1r is posting this file encryptor and decryptor named "SecureFiles".
SecureFiles is a encryption and decryption tool to share important files over any media.
I wrote this script called `SecureFiles` in python. I learned about `XOR` encryption, basic encoding and decoding using python, file management, `magic numbers` and random seed values in python.

So, here's a brief information about my simple SecureFiles tool.

# SecureFiles

With this tool we can able to encrypt text files, images, video's, audio's and any other files. And the decryption option is also available to decrypt the files.
This tool should be installed in the both reciever's and sender's system. 

Okay, Let me show some code..

Look at the `main()`

```python
def main():
	print("""
File Encrypter and Decrypter Menu
---------------------------------------------
Copy and paste the file in the "files" directory...
		1. Encrypt Image files      
		2. Encrypt Text files		
		3. Encrypt Audio files		
		4. Encrypt Video files		
		5. Encrypt other files		
		6. Decrypt Image files		
		7. Decrypt Text files		
		8. Decrypt Audio files		
		9. Decrypt Video files		
		10. Decrypt other files		
		0. Exit						
---------------------------------------------
		""")
	choice = int(input("Choose your option[0/1-10] : "))
	
	if choice == 1:
		file = input("Enter image name : ")
		check_file(file)
		encrypt(file,1)
	elif choice == 2:
		file = input("Enter Text file name : ")
		check_file(file)
		encrypt(file,2)
	elif choice == 3:
		file = input("Enter Audio file name : ")
		check_file(file)
		encrypt(file,3)
	elif choice == 4:
		file = input("Enter Video file name : ")
		check_file(file)
		encrypt(file,4)
	elif choice == 5:
		file = input("Enter File name : ")
		check_file(file)
		encrypt(file,5)
	elif choice == 6:
		file = input("Enter Image name : ")
		check_file(file)
		decrypt(file,6)
	elif choice == 7:
		file = input("Enter Text file name : ")
		check_file(file)
		decrypt(file,7)
	elif choice == 8:
		file = input("Enter Audio file name : ")
		check_file(file)
		decrypt(file,8)
	elif choice == 9:
		file = input("Enter Video file name : ")
		check_file(file)
		decrypt(file,9)
	elif choice == 10:
		file = input("Enter File name : ")
		check_file(file)
		decrypt(file,10)
	elif choice == 0:
		print("Terminating..")
		exit(1)
	else:
		print("Invalid choice..")
		exit(1)
```
The main() is displaying the menu of the tool and calling the functions based on the users choice.The check_file() function is checking wheather the file
is existed in the files directory or not.

Lets look at the image encryption case. The encrypt funtion is doing the encryption of the image.

```python
def encrypt(source_file,choice):
	file = open("./files/"+source_file,"rb")
	data = file.read()
	b_array = bytearray(data)
	key = input("Enter Key to encrypt {}: ".format(menu[choice]))
	random.seed("CH4R4NU"+key)
	for i in range(len(b_array)):
		k = random.getrandbits(8)
		b_array[i] = b_array[i] ^ k
	file.close()
	b64_array = base64.b64encode(b_array)
	if "." in source_file:
		ext = "."+source_file.split(".")[-1]
		file = open("./files/"+source_file.strip(ext)+"_encrypted"+ext,"wb")
	else:
		file = open("./files/"+source_file+"_encrypted","wb")
	file.write(b64_array)
	print("\tThe {} is encrypted and saved in files directory..".format(menu[choice]))
```
The encryption is done with the xor operation. The random.seed and base64 encoding is used to make the encryption stronger. The user can input his/her password called key as while encrypting the file. The key is the only way to decrypt the encrypted file.

If we encrypted our image successfully, the encrypted image will be saved at files folder. We can share this encrypted file to anyone but no one can view the actual image. To get the actual image we have to decrypt the image by specifiying the key which is used for encryption. The decryption process is done by the 
`decrypt` function.

```python
def decrypt(source_file,choice):
	file = open("./files/"+source_file,"rb")
	data = file.read()
	b_array = bytearray(base64.b64decode(data))
	key = input("Enter Key to decrypt image : ")
	random.seed("CH4R4NU"+key)
	for i in range(len(b_array)):
		k = random.getrandbits(8)
		b_array[i] = b_array[i] ^ k 
	file.close()
	if "." in source_file:
		ext = "."+source_file.split(".")[-1]
		source_file = "./files/"+source_file.strip(ext)+"_decrypted"+ext
		file = open(source_file,"wb")
	else:
		source_file = "./files/"+source_file+"_decrypted"
		file = open(source_file,"wb")
	file.write(b_array)
	isdecrypted(source_file,choice)
```
The bytes of the encrypted image file is decoded first from the base64, then the function asking the key and setting up the same random seed which is used in the encryption. Then the xor operation is performed, the bytes are written into a file and saved it in files folder. At the end of the decrypt function the
 `isdecrypted()` is called.
 ```python
def isdecrypted(file,choice):
	print("-"*50)
	typ = magic.from_file(file,mime=True)
	if ("text" in typ) and choice == 7:
		print("Text File is decrypted and saved in files directory..:-)")
	elif ("image" in typ) and choice == 6:
		print("Image is decrypted and saved in files directory..:-)")
	elif ("audio" in typ) and choice == 8:
		print("Audio file is decrypted and saved in files directory..:-)")
	elif ("video" in typ) and choice == 9:
		print("Video is decrypted and saved in files directory..:-)")
	elif ("text/plain" in typ ) and choice == 10:
		print("File is decrypted and saved in files directory..:-)")
	else:
		print("The key is wrong...")
		exit(1)
	print("-"*50)

```
The isdecrypted function is checking wheather the decrypted image is valid or not. The magical numbers of the image is being tested by the magic module.
If the magical numbers are matched the function is going to tell that the decryption is successfull, if not the function will tell that the key is wrong to decrypt. If the key is wrong the program is going to be exited.


### Requirements

You require these additional modules 

1. base64
2. magic
3. os
4. random

These modules will be pre installed on every system, if not then install them with `pip3 or pip`

>You need to copy your file and paste it in files folder of present in the SecureFiles folder to encrypt or decrypt the file
{: .prompt-info}

### Installation

```text 
git clone https://github.com/TheMj0ln1r/SecureFiles.git
cd SecureFiles
python3 SecureFiles.py
```
### Preview

![Preview](/assets/img/project_img/securefiles.png){: w="600",h="600"}

> If anything should be modified in the script please let me know.
{: .prompt-info }

You can find complete source code of SecureFiles here : [https://github.com/TheMj0ln1r/SecureFiles](https://github.com/TheMj0ln1r/SecureFiles)
