Troubleshooting the Raspberry Pi setup

1. Problem 1 - No Ethernet Lights on the RPi board: this may result from many things, but has been most frequently seen when students are using a USB to Ethernet dongle.
  * Try a different USB port
  * Try a known-good cable.  There have been some issues where a particular cable will work with the Pi 3B but not the 3B+ (which introduced Ethernet subsystem hardware changes).
  * Have the student install drivers specifically for their dongle.
2. Problem 2 - Putty gives a hostname error
  * Check to ensure Bonjour is installed.  Windows 10 supposedly provides zeroconf support, but it is only partially implemented and highly suspect.
  * Check the power adapter.  Many students want to use an existing power adapter.  Pis are notorious power-hogs.  If the Pi isn't getting enough power, it will continually reboot.  We've seen this problem with a power adapter whose output was 750mA.  The official power adapters are 2.5A and work fine, but we've also had success with adapters which are > 2A.
  
