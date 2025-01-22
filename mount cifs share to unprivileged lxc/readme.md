Since unprivileged LXCs are not allowed to mount CIFS shares and priviliged LXCs are considered unsafe (for a reason) I was scraping my head around how to still have my NAS shares available in my LXCs, f.e. (Jellyfin, Plex, ...).

The solution provided by the Proxmox Wiki would require many changes to the PVE host config, which I was not willing to do.
https://pve.proxmox.com/wiki/Unprivileged_LXC_containers#Using_local_directory_bind_mount_points

Thanks to the following source I was able to assemble a solution that should work for everyone in under 10 minutes.
https://bayton.org/docs/linux/lxd/mount-cifssmb-shares-rw-in-lxd-containers/


How does it work?
By default CIFS shares are mounted as user root(uid=0) and group root(gid=0) on the PVE host which makes them inaccessible to other users,groups and LXCs.
This is because UIDs/GIDs on the PVE host and LXC guests are both starting at 0. But a UID/GID=0 in an unprivileged LXC is actually a UID/GID=100000 on the PVE host. See the above Proxmox Wiki link for more information on this.
@Jason Bayton's solution was to mount the share on the PVE host with the UID/GID of the LXC-User that is going to access the share. While this is working great for a single user it would not work for different LXCs with different users having different UIDs and GIDs. I mean it would work, but then you would have to create a single mount entry for your CIFS share for each UID/GID.

My solution is doing this slightly different and more effective I think.
You simply mount the CIFS share to the UID that belongs to the unprivileged LXC root user, which by default is always uid=100000.
But instead of also mounting it to the GID of the LXC root user, your are going to create a group in your LXC called lxc_cifs_shares with a gid=10000 which refers to gid=110000 on the PVE host.
PVE host (UID=100000/GID=110000) <--> unprivileged LXC (UID=0/GID=10000)


How to configure it

1. In the LXC (run commands as root user)

    Create the group "lxc_shares" with GID=10000 in the LXC which will match the GID=110000 on the PVE host.
    ``groupadd -g 10000 lxc_shares``
    Add the user(s) that need access to the CIFS share to the group "lxc_shares".
    f.e.: jellyfin, plex, ... (the username depends on the application)
    ``usermod -aG lxc_shares USERNAME``
    Shutdown the LXC.

2. On the PVE host (run commands as root user)

    Create the mount point on the PVE host.
    ``mkdir -p /mnt/lxc_shares/nas_rwx``
    Add NAS CIFS share to /etc/fstab.

!!! Adjust //NAS/nas/ in the middle of the command to match your CIFS hostname (or IP) //NAS/ and the share name /nas/. !!!
!!! Adjust user=smb_username,pass=smb_password at the end of the command. !!!
Code:

{ echo '' ; echo '# Mount CIFS share on demand with rwx permissions for use in LXCs (manually added)' ; echo '//NAS/nas/ /mnt/lxc_shares/nas_rwx cifs _netdev,x-systemd.automount,noatime,uid=100000,gid=110000,dir_mode=0770,file_mode=0770,user=smb_username,pass=smb_password 0 0' ; } | tee -a /etc/fstab

Mount the share on the PVE host.
``mount /mnt/lxc_shares/nas_rwx``
Add a bind mount of the share to the LXC config.
!!! Adjust the LXC_ID at the end of the command. !!!
Code:

You can mount it in the LXC with read+write+execute (rwx) permissions.
{ echo 'mp0: /mnt/lxc_shares/nas_rwx/,mp=/mnt/nas' ; } | tee -a /etc/pve/lxc/LXC_ID.conf

You can also mount it in the LXC with read-only (ro) permissions.
{ echo 'mp0: /mnt/lxc_shares/nas_rwx/,mp=/mnt/nas,ro=1' ; } | tee -a /etc/pve/lxc/LXC_ID.conf

Start the LXC.