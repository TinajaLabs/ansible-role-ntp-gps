Ansible NTP-GPS
===============

A simple Ansible role to configure a GPS based NTP Server which can deliver a time signal to a network that is not connected to the internet.

This build uses a Raspberry Pi and time signals derived from a connected GPS device.


Requirements
------------

Designed and tested with: 

* Raspberry Pi 3 Model B V1.2 running minimal version of Raspbian Stretch.
* [Adafruit Ultimate GPS Breakout V3 board](https://www.adafruit.com/product/746)


![Alt text](meta/RasPiB-GPIO.png?raw=true "Raspi pin out")

![Alt text](meta/adafruit-gps-breakout.jpg?raw=true "NTP GPS Breakout from Adafruit")



Wiring connections:

```
from Raspi pin:  4 (+5V)            to GPS Breakout pin: VIN +5V        
from Raspi pin:  6 (Ground)         to GPS Breakout pin: GND Gnd        
from Raspi pin:  8 (TXD0) (GPIO14)  to GPS Breakout pin: RX TX
from Raspi pin: 10 (RXD0) (GPIO15)  to GPS Breakout pin: TX RX
from Raspi pin: 12 (GPIO18)         to GPS Breakout pin: PPS (Pulses Per Second)
```

Role Variables
-------------

None.


Dependencies
------------

None.


Example Playbook
----------------

Create playbook, `tinaja-ntp-gps.yml`:


```
    - hosts: ntpserver
      roles:
         - { role: tinaja.ntp-gps }
```

Basic Steps
-------------

* Install Ansible - will assume this is already done...
* Download the latest version of the Raspbian image from: https://downloads.raspberrypi.org/raspbian_lite_latest
* Burn the image on a MicroSD card (8G or more) using [etcher](https://www.balena.io/etcher/)
* Add a magic empty file named SSH onto the boot partition (configures default SSH service)
* Plug in the MicroSD image into the Raspi and boot up.
* After bootup, copy your public key to the raspberrypi using user:pi, password:raspberry:

 `$ ssh-copy-id -f -i ~/.ssh/id_rsa.pub pi@raspberrypi.local`
* Setup a host inventory file with a reference like this:

`tinaja-ntp ansible_host=<raspi ipaddress> ansible_user=pi`
* Run your playbook:

`$ ansible-playbook tinaja-ntp-gps.yml -i hosts/hosts.ini -u pi -b -c ssh`



**Run these tests after rebooting:**

Log into your raspi:

`$ ssh pi@raspberrypi.local`

Escalate to user, root:

`# sudo su -`

See the serial data streaming in from the GPS device

`# cat /dev/ttyAMA0`

See the /dev/pps0 device streaming in

`# ppstest /dev/pps0`

See the gps device statistics including the time, Latitude, Longitude from the GPS receiver

`# gpsmon`

See the list of ntp servers.  The PPS reference should have an asterisk indicating the primary source.

`# ntpq -p`

`*SHM(2) .PPS. 0 l 1 64 377 0.000 -51.298 4.627`



**On your local server**

Set up the NTP server to point to the new NTP server.  Edit `/etc/ntp.cfg`:

`# nano /etc/ntp.conf`

Change this:

```
# You do need to talk to an NTP server or two (or three).  
#server ntp.your-provider.example  
```
to this:

```
# You do need to talk to an NTP server or two (or three).  
#server ntp.your-provider.example  
server <your ntp server ip or fqdn>
```

Save the file and restart the NTP service:

`# systemctl restart ntp.service`

See the magic, by running this command:

`# date`


References
-----------
* How Ansible Works - https://www.ansible.com/overview/how-ansible-works
* Get Started with Ansible - https://docs.ansible.com/ansible/latest/network/getting_started/first_playbook.html
* NTP - https://en.wikipedia.org/wiki/Network_Time_Protocol
* Raspberry Pi GPS Time - http://unixwiz.net/techtips/raspberry-pi3-gps-time.html
* Raspberry Pi NTP - https://www.satsignal.eu/ntp/Raspberry-Pi-NTP.html
* Stratum-1 NTP Server using Raspberry Pi - https://developers.redhat.com/blog/2017/02/22/how-to-build-a-stratum-1-ntp-server-using-a-raspberry-pi/
* Adafruit Ultimate GPS - https://learn.adafruit.com/adafruit-ultimate-gps


License
-------
MIT

Author Information
------------------
Chris Jefferies - `chris@tinajalabs.com`

