Troubleshooting the Raspberry Pi setup

Most of the problems we have seen come from students trying to connect via Windows.  Macs, being built on Unix, have significantly fewer problems once a proper OS burn onto the micro-SD is achieved.  The below is geared towards Windows users, but also includes some tidbits if a student with a Mac is having troubles.

1. Problem 1 - No Ethernet Lights on the RPi board: this may result from many things, but has been most frequently seen when students are using a USB to Ethernet dongle.
  * Try a different USB port
  * Try a known-good cable.  There have been some issues where a particular cable will work with the Pi 3B but not the 3B+ (which introduced Ethernet subsystem hardware changes).
  * Have the student install drivers specifically for their dongle.
2. Problem 2 - Putty gives a hostname error
  * Check to ensure Bonjour is installed.  Windows 10 supposedly provides zeroconf support, but it is only partially implemented and highly suspect.
  * Check the power adapter.  Many students want to use an existing power adapter.  Pis are notorious power-hogs.  If the Pi isn't getting enough power, it will continually reboot.  We've seen this problem with a power adapter whose output was 750mA.  The official power adapters are 2.5A and work fine, but we've also had success with adapters which are > 2A.
  * Make sure the students are waiting for the green light to stop flashing before connecting.
  * Make sure Ethernet is plugged in and card is plugged in before turning on (plugging in) power to the Pi.  Make sure students wait until the green light stops flashing.  This takes longer than you think it should, particularly the first time they power the Pis on as the Pi has to reboot twice during the initial install.
3. Problem 3 - Student connected in the past but is now unable to.
  * Check power adapter and ethernet cable.
  * Check to see if the IP is still in the arp cache: `arp -a | grep "b8:27:eb"` (all Pis have the same MAC address).  Using this, you can find the IP address and try connecting directly to the Pi using `pi@IP.ADD.RESS` rather than `pi@raspberrypi.local`.  If you can connect directly using the IP than try reinstalling Bonjour.
  
  
  PEAP/WPA-Enterprise authentication is broken under Raspian Buster as of 9/2019.  (ARRRGGGHHH~!!!!!).  Two options:
  
  1. Edit both wpa_supplicant.conf and dhcpcd.conf:
 `/etc/wpa_supplicant/wpa_supplicant.conf` should be:
 ```
 ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
	ssid="FoxNet-v2"
	proto=RSN
	key_mgmt=WPA-EAP
	pairwise=CCMP
	auth_alg=OPEN
	eap=PEAP
	identity="yourUserNameHere"
	password="yourSuperSecretPasswordHere"
	phase1="peapver=0"
	phase2="MSCHAPV2"
}
```

Then edit `/etc/dhcpcd.conf` to include these three lines (at the bottom of the file is fine:
```
interface wlan0
env ifwireless=1
env wpa_supplicant_driver=wext,nl80211
``` 
The key thing here is to move the `wext` driver before the `n180211` driver.

reboot and then `ping 8.8.8.8` and all should be happy
  
2. Have students connect to a non WPA-Enterprise/PEAP network.  HOWEVER, they **MUST** rename their device's hostname so that you don't have 30+ devices with the same hostname on the same network.  That causes bad things to happen.  


Fire up `raspi-config` and do a few things:
a. Change the networking by entering just the network and passphrase (at my university, this is a network for printers and such which doesn't require the PEAP authentication).
b. Change the hostname to something like `raspberrypi-userID` so that it is unique for each student.  Make sure they now understand they will have to connect using this new hostname and not the old `pi@raspberrypi.local` hostname.
c. Have them change the default password.  This is good practice anyway and definitely should happen since we have so many Pi's on the same network and easily accessible to anyone.
