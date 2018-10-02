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
