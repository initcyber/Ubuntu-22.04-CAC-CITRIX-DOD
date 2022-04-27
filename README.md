# Ubuntu-22.04-CAC-AVHE-DOD
Notes on DOD CAC Certifcate Installation and Walkthrough + AVHE for Ubuntu's latest 22.04 LTS Release (April 2022).

Important: CoolKey/CACkey (pkcs11 management software) as most guides out there will advise you to use, isn't feasible with right out the box with Ubuntu 22.04. CoolKey doesn't play well with PIV certificates, and CACKey is only available after being authenticated with a ... CAC... Chicken and Egg problem... OpenSC is our only choice here. Chances are you have tried everything up to this point and are just now finding this GitHub Repo, so either start a fresh install or uninstall CACKey and CoolKey following the steps below:


## Dependencies
After a fresh install of Ubuntu 22.04:
(or if you are coming to this github page after trying everthing else)

```
sudo apt update -y
```
Then we will remove anything you may have already tried to do before coming here:
```
sudo apt remove cackey coolkey libckyapplet1 libckyapplet1-dev -y && sudo apt purge -y 
```
Then install the software dependencies:

```
sudo apt update -y && sudo apt upgrade -y && sudo apt install pcsc-tools libccid libpcsc-perl libpcsclite1 pcscd opensc opensc-pkcs11 vsmartcard-vpcd libnss3-tools -y
```

## Middleware Installation/Check
Now that all your dependencies are installed it’s time to start checking if things are operating how they should. The first step is to ensure your middleware actually sees your CAC; this is done via pcscd and it’s associated tools.

In your terminal type:
```
pcsc_scan
```
Running pcsc_scan should give the following output:
```
Scanning present readers...
0: Virtual PCD 00 00
1: Virtual PCD 00 01
2: SCM Microsystems Inc. SCR 3310 [CCID Interface] (53312028233379) 00 00
 
Wed Apr 27 06:43:26 2022
 Reader 0: Virtual PCD 00 00
  Event number: 0
  Card state: Card removed, 
 Reader 1: Virtual PCD 00 01
  Event number: 0
  Card state: Card removed, 
 Reader 2: SCM Microsystems Inc. SCR 3310 [CCID Interface] (53312028233379) 00 00
  Event number: 0
  Card state: Card inserted, Shared Mode, 
  ATR: 3B F9 18 00 00 00 53 43 45 37 20 03 00 20 46

ATR: 3B F9 18 00 00 00 53 43 45 37 20 03 00 20 46
+ TS = 3B --> Direct Convention
+ T0 = F9, Y(1): 1111, K: 9 (historical bytes)
  TA(1) = 18 --> Fi=372, Di=12, 31 cycles/ETU
    129032 bits/s at 4 MHz, fMax for Fi = 5 MHz => 161290 bits/s
  TB(1) = 00 --> VPP is not electrically connected
  TC(1) = 00 --> Extra guard time: 0
  TD(1) = 00 --> Y(i+1) = 0000, Protocol T = 0 
-----
+ Historical bytes: 53 43 45 37 20 03 00 20 46
  Category indicator byte: 53 (proprietary format)

Possibly identified card (using /usr/share/pcsc/smartcard_list.txt):
3B F9 18 00 00 00 53 43 45 37 20 03 00 20 46
	G+D FIPS 201 SCE 7.0 (PKI)
```


If you get 
```
SCardEstablishContext: Service not available.
```
Check to see if pcsc is running by typing:
```
sudo systemctl status pcscd.s*       
```
If it isn't then restart the daemon:
```
sudo systemctl restart pcscd.socket && sudo systemctl restart pcscd.service
```

Optionally, set it to start at time of system boot:
```
sudo systemctl enable pcscd.socket
```

Rare Instances: If you get the following message from the pcsc_scan: 

```
"scanning present readers waiting for the first reader..."
```
Run this code to unload the kernel modules and allow your cac reader to reclaim the usb slot.
```
modprobe -r pn533 nfc
```

## CAC Reader

Now that the middleware is installed, type the following command
```
opensc-tools -l
```
in order to query information about your CAC and to see if OpenSC can communicate with the middleware we just installed. You should see something of the following (where the CAC reader is listed out)
```
$ opensc-tool -l
# Detected readers (pcsc)
Nr.  Card  Features  Name
0    No              Virtual PCD 00 00
1    No              Virtual PCD 00 01
2    Yes             SCM Microsystems Inc. SCR 3310 [CCID Interface] (53312028233379) 00 00
```

### Troubleshooting CAC Reader
1) If your CAC Reader is not listed, try uninstalling OpenSC and reinstalling OpenSC. Refer to OpenSC documentation for further troubleshooting.

2) If the CAC drivers are not under "Available card drivers", then we need to modify OpenSC configuration to use them.


Edit */etc/opensc/opensc.conf* using your editor of choice (nano, vim, vi, gedit, etc).
```
card_drivers = cac
force_card_driver = cac
```

*opensc.conf* should look like this:
```
    cat opensc.conf
    app default {
        # debug = 3;
        # debug_file = opensc-debug.txt;
        card_drivers = cac
        force_card_driver = cac
        framework pkcs15 {	
        # use_file_caching = true;
        }
    }
```    
    
Now in terminal type
```
opensc-tools -l
```
again and see if it detects the drivers.

Onto Certificates!

# DOD Certificates and Browser Configuration

Obtain the DOD Certificates here:

If you have gotten this far and do not need Citrix Workspace installed, congrats. Otherwises, continue on.

# DOD Certificates and Citrix Workspace Configuration
For Citrix Workspace, I have found having a separate
