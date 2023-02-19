This was useful for setting up NFS: https://linuxize.com/post/how-to-install-and-configure-an-nfs-server-on-ubuntu-20-04/


## Setup LVM

sudo apt install lvm2
lsblk
sudo pvcreate /dev/sba1
sudo pvscan
sudo vgcreate "longhorn" /dev/sda1
sudo vgscan
sudo lvcreate -l 100%FREE -next4 "longhorn"
sudo lvdisplay
sudo mkfs.ext4 /dev/longhorn/ext4
sudo mkdir -p /data/longhorn
sudo mount /dev/longhorn/ext4 /data/longhorn
sudo cat /proc/mounts | grep longhorn
sudo nano /etc/fstab

add:
/dev/mapper/longhorn-ext4 /data/longhorn ext4 rw,relatime 0 0