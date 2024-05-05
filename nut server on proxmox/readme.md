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

```
cp /etc/nut/nut.conf /etc/nut/nut.example.conf
```

Edit /etc/nut/nut.conf :

```
nano /etc/nut/nut.conf
```

Delete all and add :

`
MODE=netserver
`

Ref : nut.conf(5) https://networkupstools.org/docs/man/nut.conf.html

b) Backup /etc/nut/ups.conf :

```
cp /etc/nut/ups.conf /etc/nut/ups.example.conf
```

Edit /etc/nut/ups.conf :

```
nano /etc/nut/ups.conf
```

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

Use the values returned by the nut-scanner command above (add serial number of your ups if applicable).

After saving the file, you can test it by entering the command :
`upsdrvctl start`

You should get a return like :

Network UPS Tools - UPS driver controller 2.7.4

Network UPS Tools - Generic HID driver 0.41 (2.7.4)

USB communication driver 0.33

Using subdriver: ____________ HID 0.96


Ref : ups.conf(5) https://networkupstools.org/docs/man/ups.conf.html and usbhid-ups(8)  https://networkupstools.org/docs/man/usbhid-ups.html

c) /etc/nut/upsd.conf

Backup /etc/nut/upsd.conf : 

```
cp /etc/nut/upsd.conf /etc/nut/upsd.example.conf
```

Edit /etc/nut/upsd.conf :

```
nano /etc/nut/upsd.conf
```

Delete all and add :

```
LISTEN 0.0.0.0 3493
LISTEN :: 3493

```
By using 0.0.0.0 instead of 127.0.0.1 we instruct the server to respond to requests from all networks.

Ref : upsd.conf(5) https://networkupstools.org/docs/man/upsd.conf.html

d) /etc/nut/upsd.users

Backup /etc/nut/upsd.users : 

```
cp /etc/nut/upsd.users /etc/nut/upsd.example.users
```

Edit /etc/nut/upsd.users :

```
nano /etc/nut/upsd.users
```

Delete all and add :
```
[upsadmin]
# Administrative user
password = ********
# Allow changing values of certain variables in the UPS.
actions = SET
# Allow setting the "Forced Shutdown" flag in the UPS.
actions = FSD
# Allow all instant commands
instcmds = ALL
upsmon master

[upsuser]
# Normal user
password = ********
upsmon slave
Replace ******** with your own password(s).
```

Ref : upsd.users(5)  https://networkupstools.org/docs/man/upsd.users.html


e) /etc/nut/upsmon.conf

Backup /etc/nut/upsmon.conf :

```
cp /etc/nut/upsmon.conf /etc/nut/upsmon.example.conf
```
Edit /etc/nut/upsmon.conf :

```
nano /etc/nut/upsmon.conf
```
Delete all and add : 
```
RUN_AS_USER root
MONITOR apc@localhost 1 upsadmin ******* master

MINSUPPLIES 1
SHUTDOWNCMD "/sbin/shutdown -h"
NOTIFYCMD /usr/sbin/upssched
POLLFREQ 4
POLLFREQALERT 2
HOSTSYNC 15
DEADTIME 24
MAXAGE 24
POWERDOWNFLAG /etc/killpower

NOTIFYMSG ONLINE "UPS %s on line power"
NOTIFYMSG ONBATT "UPS %s on battery"
NOTIFYMSG LOWBATT "UPS %s battary is low"
NOTIFYMSG FSD "UPS %s: forced shutdown in progress"
NOTIFYMSG COMMOK "Communications with UPS %s established"
NOTIFYMSG COMMBAD "Communications with UPS %s lost"
NOTIFYMSG SHUTDOWN "Auto logout and shutdown proceeding"
NOTIFYMSG REPLBATT "UPS %s battery needs to be replaced"
NOTIFYMSG NOCOMM "UPS %s is unavailable"
NOTIFYMSG NOPARENT "upsmon parent process died - shutdown impossible"

NOTIFYFLAG ONLINE   SYSLOG+WALL+EXEC
NOTIFYFLAG ONBATT   SYSLOG+WALL+EXEC
NOTIFYFLAG LOWBATT  SYSLOG+WALL+EXEC
NOTIFYFLAG FSD      SYSLOG+WALL+EXEC
NOTIFYFLAG COMMOK   SYSLOG+WALL+EXEC
NOTIFYFLAG COMMBAD  SYSLOG+WALL+EXEC
NOTIFYFLAG SHUTDOWN SYSLOG+WALL+EXEC
NOTIFYFLAG REPLBATT SYSLOG+WALL
NOTIFYFLAG NOCOMM   SYSLOG+WALL+EXEC
NOTIFYFLAG NOPARENT SYSLOG+WALL

RBWARNTIME 43200
NOCOMMWARNTIME 600

FINALDELAY 5
```

Replace ******** with your own password(s).


f) /etc/nut/upssched.conf

Backup /etc/nut/upssched.conf : 

```
cp /etc/nut/upssched.conf /etc/nut/upssched.example.conf
```

Edit /etc/nut/upssched.conf :

```
nano /etc/nut/upssched.conf
```

Delete all and add :
```
CMDSCRIPT /etc/nut/upssched-cmd
PIPEFN /etc/nut/upssched.pipe
LOCKFN /etc/nut/upssched.lock

AT ONBATT * START-TIMER onbatt 30
AT ONLINE * CANCEL-TIMER onbatt online
AT LOWBATT * EXECUTE onbatt
AT COMMBAD * START-TIMER commbad 30
AT COMMOK * CANCEL-TIMER commbad commok
AT NOCOMM * EXECUTE commbad
AT SHUTDOWN * EXECUTE powerdown
AT SHUTDOWN * EXECUTE powerdown
```
Be sure PIPEFN and LOCKFN point to a folder that esists, I’ve seen it point to /etc/nut/upssched/ instead of /etc/nut/.
If it does, create the folder or update these variables. mkdir /etc/nut/upssched/

Ref : upssched.conf(5) https://networkupstools.org/docs/man/upssched.conf.html and TechnoTim https://techno-tim.github.io/posts/NUT-server-guide/

g) /etc/nut/upssched-cmd

(If exists) Backup /etc/nut/upssched-cmd :

```
cp /etc/nut/upssched-cmd /etc/nut/upssched-cmd.example
```

Edit /etc/nut/upssched-cmd :

```
nano /etc/nut/upssched-cmd
```

(Delete all and) Add :
```
#!/bin/sh
 case $1 in
       onbatt)
          logger -t upssched-cmd "UPS running on battery"
          ;;
       shutdowncritical)
          logger -t upssched-cmd "UPS on battery critical, forced shutdown"
          /usr/sbin/upsmon -c fsd
          ;;
       upsgone)
          logger -t upssched-cmd "UPS has been gone too long, can't reach"
          ;;
       *)
          logger -t upssched-cmd "Unrecognized command: $1"
          ;;
 esac

Then :

chmod +x /etc/nut/upssched-cmd
```

Ref : upssched.conf(5) https://networkupstools.org/docs/man/upssched.conf.html


Now reboot your Proxmox server. Or enter the following command sequence to restart the services : 
```
service nut-server restart
service nut-client restart
systemctl restart nut-monitor
upsdrvctl stop
upsdrvctl start
```

Test your configuration by running the command :

```
upsc myups@localhost
```

You should see the UPS replying its data.

Tip : If the above does not work, replace the USB cable and reboot the system once again.


DIY HOME SERVER - PROXMOX - NUT Monitoring

Create LXC container. I preffer install Ubuntu OS

a) Install an Apache webserver and nut-cgi :
```
apt install apache2 nut-cgi -y
```

Install NUT client :
```
apt install nut-client -y
```

b) /etc/nut/nut.conf

Backup /etc/nut/nut.conf :
```
cp /etc/nut/nut.conf /etc/nut/nut.example.conf
```
Edit /etc/nut/nut.conf :
```
nano /etc/nut/nut.conf
```
Delete all and add :
`MODE=netclient`

Ref : nut.conf(5)

c) /etc/nut/hosts.conf

Backup /etc/nut/hosts.conf :

```
cp /etc/nut/hosts.conf /etc/nut/hosts.example.conf
```

Edit /etc/nut/hosts.conf :
```
nano /etc/nut/hosts.conf
```

Delete all and add :
`
MONITOR myups@xxx.xxx.xxx.xxx "name of your ups"
`
Replace xxx.xxx.xxx.xxx with the IP address of your Proxmox (NUT) server.

Ref : hosts.conf(5)

d) /etc/nut/upsset.conf

Backup /etc/nut/upsset.conf :
```
cp /etc/nut/upsset.conf /etc/nut/upsset.example.conf
```

Edit /etc/nut/upsset.conf :
```
nano /etc/nut/upsset.conf
```

Delete all and add :
```
I_HAVE_SECURED_MY_CGI_DIRECTORY
```
Ref : upsset.conf(5)

e) /etc/nut/upsmon.conf

Backup /etc/nut/upsmon.conf : 
```
cp /etc/nut/upsmon.conf /etc/nut/upsmon.example.conf
```
Edit /etc/nut/upsmon.conf :
```
nano /etc/nut/upsmon.conf
```
Delete all and add :
```
RUN_AS_USER root
MONITOR myups@xxx.xxx.xxx.xxx 1 upsuser ******* slave
```
Replace xxx.xxx.xxx.xxx with the IP address of your Proxmox (NUT) server.
Replace ******** with your own password(s).

Ref : upsmon.conf(5)

Enter the command :
`sudo a2enmod cgi`

Then restart the webserver :
`sudo systemctl restart apache2`

Now test the monitoring. Open a new browser tab and go to the UPS status page :

http://xxx.xxx.xxx.xxx/cgi-bin/nut/upsstats.cgi


Testing

a) System shutdown

Now you can test if your system shuts down gracefully.
In Proxmox, select your PVE node → Shell.

Issue the following command :

`upsmon -c fsd`

Your system should shut down all its subsystems and then shutdown itself.

b) Power outage

Now you are ready to pull the AC power plug going from your AC wall outlet to the UPS.
At the moment the UPS reaches the values set in battery.runtime.low or battery.charge.low, the shutdown should be initiated.

