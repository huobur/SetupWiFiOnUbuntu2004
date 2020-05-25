# How to Setup WiFi on Raspberry Pi 4 with Ubuntu 20.04 LTS 64-bit ARM Server

After reading the article "Ubuntu 20.04 LTS is certified for the Raspberry Pi" [1], with an empty micro SD card lying around, I decided to give it shot.[2]

Everything went smoothly and I even installed the latest .Net Core 3.1 SDK to play with.

As usual, after playing for a while I wanted to setup WiFi such that I could get rid of my Ethernet wire.

Boy, did I have trouble on getting it up and running!

## Where to Find Info?

I basically made couple of mistakes:

1. Somehow, I thought that similar to Raspian which had a lot of customizations, for instance, raspi-config utility. I searched and there were a lot of suggestions that I needed to install it and setup locale to the country I would operate wireless machine. I did install it and it didn't help much.
2. That got me started on the wrong path. There are just so many leads and info on the net with various tips and suggestions, though couple of them were very close, but I only got it partially working, for instance, I was advised to make an interfaces file in /etc/network/. That didn't fly. 
3. I installed quite a few network tools to play with with hald-baked or dead-end suggestions. At one point, I was trying to figure out "Set Mode (8B06) Error" with so many leads and it's actually security protocol compatibility error, which suggested that I need to make change to my router to "wep", which of cause completely stopped me there to say "NO"...

## On the Right Path
And finally, I read the correct info [3] and setup my WiFi successfully. It's actually pretty simple and here are the steps:
### 1. You need 3 pieces of info in order to setup your WiFi:
    * Raspberry Pi wireless card name on your system;
    * WiFi router name you are trying to login (SSID), and
    * WiFi login password

    You should already know your WiFi login name and password, the only info we need is WiFi name.
    To clearly illustrate how to setup, let's assume that your WiFi login name is "MyWiFi" and your password is "MyPass". 
### 2. Finding WiFi card name:
```
   $ ls /sys/class/net
   eth0  lo  wlan0
```
   on my machine, it's "wlan0". To simplify our demo, assume that yours is "wlan0" as well.

### 3. Edit network configuration file to add WiFi info:
```
   sudo nano /etc/netplan/50-cloud-init.yaml
```
   you will see the following in the original file:
```
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
```
   add your WiFi info such that it should look somthing like the following:

```
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    wifis:
        wlan0:
            optional: true
            access-points:
                "MyWiFi":
                    password: "MyPass"
            dhcp4: true
```
### 4. Make sure that the 3 pieces of info above is replaced with yours. 
* wlan0
* "MyWiFi"
* "MyPass"

And also please make sure that you are a very careful person to input the above info with exact format as original file started, and all indents should be typed with spaces (4 spaces each level), NOT tabs, which is very important!
### 5. Save the file and reboot, you should have your WiFi setup when the machine is up and running again.

## What the Hell?
When I got up this morning, the whole thing didn't work any more:
* I carefully examed everthing, nothing had been changed since it was working last night. 
* However, when I looked my router, the IP address assigned to Raspberry WiFi was changed and I couldn't ssh or ping Raspberry any more.
* I then plugged Ethernet wire and rebooted Raspberry, everything was working again. Finally, I noticed something from router side: (BTW, what's the point if I need Ethernet wire to get my WiFi to work?)
    
    1. If the Ethernet wire was plugged in, I had WiFi IP: 10.0.0.11 and Ethernet IP: 10.0.0.12
    2. If I unplugged Ethernet, the WiFi IP was <b>CHANGED</b> to 10.0.0.6 and there was no access to Raspberry Pi through either 10.0.0.6 or original assigned IP 10.0.0.11. How strange!
* I'll save the space to directly give the solution here without telling much the process (there is a clue in this article...):

I created another file:
```
$ sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
and put the following line in it:
network: {config: disabled}
```

After rebooting, I got my access back to Raspberry Pi (at 10.0.0.11) without Ethernet wire!

## Notes:
1. Default name and password (ubuntu, ubuntu) didn't work well for me, after failing multiple times, it would spell out messages and only then I could log in and change my password;
2. Warning -- I consider this scenario dangerous: if you mess up that "/etc/netplan/50-cloud-init.yaml" file, you could get stuck. Not even the Ethernet wire can save you. At one time, I had to reimage it!.


## References
[1] https://ubuntu.com/blog/ubuntu-20-04-lts-is-certified-for-the-raspberry-pi

[2] https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04&architecture=arm64+raspi

[3] https://linuxconfig.org/ubuntu-20-04-connect-to-wifi-from-command-line
