Access your nodes shell
``Proxmox > Your Node > Shell``
    
Create a mounting point for the share

```
mkdir /mnt/media
```

Edit fstab so that the share mounts automatically on reboot
Open: 
        
```
nano /etc/fstab
```
Add: 
```
your_ip:/mnt/user/downloads/ /mnt/media nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```
        
Save
    
Mount the share
Mount shares: 
        
```
mount -a
```
Reload systemd: 
```
systemctl daemon-reload
```
        
Add the pointing point to your LXC
Open:
```
nano /etc/pve/lxc/101.conf
```

Add: 

```
mp0: /mnt/media/,mp=/mnt/media
```

Save
    
Start the LXC
Update the LXC user's permissions
``groupadd -g 10000 lxc_shares``
Note: I think you can use whatever group name you want as long as you use again in the next step.
``usermod -aG lxc_shares root``
Note: Your username is probably root, but substitute for whatever user you want to configure permissions for.
Reboot the LXC
Verify permissions
Create a file in your mountpoint: ``touch foobar``
Attempt to delete foobar from another machine.
If successful, you should be done.

https://forum.proxmox.com/threads/tutorial-mounting-nfs-share-to-an-unprivileged-lxc.138506/