
Install the required drivers on the Proxmox host.

```
apt-get -y install {va-driver-all,ocl-icd-libopencl1,intel-opencl-icd,vainfo,intel-gpu-tools}
```

```
chgrp video /dev/dri
```
```
chmod 755 /dev/dri
```
``` 
chmod 660 /dev/dri/*
```

```
adduser $(id -u -n) video
```
```
adduser $(id -u -n) render
```
