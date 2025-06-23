Create LXCu in Proxmox. I will use Debian

Template: Debian 12
Disk: 32G 
CPU: 2
Memory: 2048
Swap: 0
Assign static IP

Visit the Traefik-Proxy GitHub repository to grab the latest software image.

Copy the Link Address of the software image applicable to your environment.

In the Traefik-Proxy LXC type the following commands:
```
wget https://github.com/traefik/traefik/releases/download/v3.4.1/traefik_v3.4.1_linux_amd64.tar.gz
# just replace the https: string with the earlier copied Link Address.
```

```
tar -zxvf traefik_v3.0.2_linux_amd64.tar.gz
```

Below is a suggested directory structure. Use your own if you wish.

where the single binary Traefik-Proxy executable is saved
```
mv ./traefik /usr/local/bin
```
we will save a single static config file and all certificates in this directory

```
mkdir /etc/traefik
```
all dynamic config files will be saved in this directory
```
mkdir /etc/traefik/dynamic 
```
create a cert-store file to save all LE certificates
```
touch /etc/traefik/acme.json  
```
secure the LE cert-store file; Traefik-Proxy will not start unless this is done
```
chmod 600 /etc/traefik/acme.json 
```
create test static config using sample_traefik.yml. We will use it just for testing purposes, to ensure that traefik is working.
```
nano /etc/traefik/traefik.yaml
```
```
traefik
```
visit traefik dashboard http://your_ip:8080
If everything went according to your plan you can build your reverse-proxy operations with specific sstatic and dynamic configuration

To exit paste ctrl-c in LXCu
