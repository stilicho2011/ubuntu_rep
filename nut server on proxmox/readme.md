In Proxmox, select your PVE node → Shell.
Or use your favorite remote connection tool to connect to your proxmox server.
List all USB ports :

`lsusb`

Note the Bus and Device of your UPS device.

`lsusb -v -s [bus]:[device]`

If the above does not work, replace the USB cable and reboot the system.

INSTALLATION OF NUT SERVER IN PROXMOX

Install NUT :

`apt install nut -y`

Probe the NUT devices :

`nut-scanner -U`

Take note of values.


NUT SERVER SET UP IN PROXMOX


a) Backup /etc/nut/nut.conf :

```cp /etc/nut/nut.conf /etc/nut/nut.example.conf```

Edit /etc/nut/nut.conf :

```nano /etc/nut/nut.conf```

Delete all and add :

`MODE=netserver`

Ref : nut.conf(5) https://networkupstools.org/docs/man/nut.conf.html

b) Backup /etc/nut/ups.conf :

```cp /etc/nut/ups.conf /etc/nut/ups.example.conf```

Edit /etc/nut/ups.conf :

```nano /etc/nut/ups.conf```

Delete all and add :

```
pollinterval = 15
maxretry = 3

offdelay = 120
ondelay = 240

[myups]
        driver = "usbhid-ups"
        port = "auto"
        desc = "Cyber Power System, Inc. CP1500 AVR UPS"
        vendorid = "0764"
        productid = "0501"
        product = "BR1000ELCD"
        vendor = "CPS"
        bus = "001"

```

Use the values returned by the nut-scanner command above.

After saving the file, you can test it by entering the command :
`upsdrvctl start`

You should get a return like :

`Network UPS Tools - UPS driver controller 2.7.4
Network UPS Tools - Generic HID driver 0.41 (2.7.4)
USB communication driver 0.33
Using subdriver: APC HID 0.96`


Ref : ups.conf(5) and usbhid-ups(8) 

