# A guide on Proxmox LXC resizing by @adrian_vg

> This has been copied from the Proxmox forum and formatted for easier reading.
>
> The original post can be found [here](https://forum.proxmox.com/threads/resize-lxc-disk-on-proxmox.68901/post-382737).

On your proxmox node, do this.

List the containers:
```shell
pct list
```

Stop the particular container you want to resize:
```shell
pct stop 999
```

Find out it's path on the node:
```shell
lvdisplay | grep "LV Path\|LV Size"
```

For good measure one can run a file system check:
```shell
e2fsck -fy /dev/pve/vm-999-disk-0
```

Resize the file system:
```shell
resize2fs /dev/pve/vm-999-disk-0 10G
```

Resize the local volume:
```shell
lvreduce -L 10G /dev/pve/vm-999-disk-0
```

Edit the container's conf, look for the rootfs line and change accordingly:
```shell
nano /etc/pve/lxc/999.conf
```

Change:
```shell
rootfs: local-lvm:vm-999-disk-0,size=32G >> rootfs: local-lvm:vm-999-disk-0,size=10G
```

Start it:
```shell
pct start 999
```

Enter it and check the new size:
```shell
pct enter 999
df -h
```
