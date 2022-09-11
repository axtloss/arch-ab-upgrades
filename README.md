# arch-ab-upgrades
use a/b upgrade styles on arch linux, similiar to the steam deck

# How this would work
instead of utilizing multiple partitions, this would use btrfs subvolumes:
- @root1 
- @root2
- @var
- @var/@home
- @var/@etc

#### @root1 & @root2

@root1 and @root2 are the seperate roots, only one is mounted at /, but both contain a bootable arch system by default.
When upgrading, the other root, which is not booted into, gets mounted at /mnt and upgraded by using arch-chroot, when this is successful
the grub config and fstab get modified to boot into the root that got upgraded.
However if this upgrade doesn't work, so if for example pacman returns an error, the whole upgrade will cancel and grub/fstab will not get modified.
If the update still makes the system unbootable, even if there was no direct error, the user can still boot into the other root that boots and fix the upgraded root.

so the update process would look like this:
1) mount unbooted root at /mnt
2) arch-chroot to /mnt
3) run command to upgrade (pacman -Syu)

if the update fails:

  4) give an error
  5) exit
  
if the upgrade succeeds:

  4) change fstab to mount different root at /
  5) update grub to use different root
  6) reboot
  

#### @var

@var itself is always mounted at /var
any extra subvolume in @var, by default @etc and @home, get mounted at /var/<subvol name, without the @> and bind mounted in /
so @var/@home would be mounted at /var/home and then bind mounted to /home

##### @var/@etc

@var/@etc is a bit special:
linux reads the fstab *before* mounting all subvolumes, meaning it'll read the /etc on the root subvolume
due to this, on any change made to /etc, the bind mount will have to be unmounted, then copy the contents of /var/etc to /etc and do the bind mount again.
