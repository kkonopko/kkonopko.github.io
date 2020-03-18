---
layout: my-post
title: Taking advantage of full disk encryption (FDE) on laptop
date: 2020-03-18 09:15
---

Encrypting disks, especially on laptops, seems like a no-brainer these days.
There are plenty of stories of laptops being lost or stolen with important
information stored on them (medical records, financial information).  To me this
is about the peace of mind: I don't want to care too much about losing equipment
if I only know the data stored on it cannot be accessed (easily).  Of course
with enough effort, expertise and the right equipment, it's likely possible to
recover encrypted data, but I assume I'm not the target.

## Self-encrypting drives

I do like the idea of self- (always) encrypting drives.  It's very convenient
and transparent, also with regards to disk I/O performance.  There's nothing
required to encrypt data, as it is encrypted all the time by default.  The key
is to only lock the disk when powered off, which is enabled by setting a
password.  This is also convenient for having full-system encryption, which
typically is not trivial when using software solutions like LUKS (boot and
possibly system partitions are left not encrypted, mostly for simplicity).

I do have a bad experience with self-encrypting drive which was used as an
experiment in a self-built FreeNAS box.  FreeNAS offered storing the disk
password and unlocking the disk on start-up which seemed to work great.  One day
though, possibly after a power outage, it started to misbehave.  After one or
two reboots, the disk no longer accepted the password and remained locked.

Anyway, maybe that was not the right use of the self-encrypted drive.  For
laptops though, it seems like a good choice, at least for me.  What matters next
is the (in)convenience of entering the disk password during boot.  A "generic"
laptop may not have support in its boot firmware (aka BIOS/UEFI) for
self-encrypted drives, so in that case what I ended up with is this (Tuxedo
laptop):

* power-on (BIOS/UEFI) password
* disk password
* (re)boot (BIOS/UEFI) password

The disk itself is unlocked with a UEFI PBA.  I followed the instructions on the
[Drive Trust
Alliance](https://github.com/Drive-Trust-Alliance/sedutil/wiki/Encrypting-your-drive)
website.

What I found on my ThinkPad P52 though turned out to be much more convenient:
not only it supports unlocking self-encrypted drives in its BIOS/UEFI, but also
has a fingerprint reader.  Personally I consider bio-metric authentication
inferior to passwords, but most of the time it's fine and yet very convenient.
The advantage of fingerprint authentication in my opinion is apparent in the
presence of pervasive CCTV, and possibly other people peeking over my shoulder
when typing in the password.  So I end up typing in no passwords at all for
power-on, disk and (re)boot authentication.

## Disk(s) without self-encryption feature

As much as my ThinkPad is convenient, I found that its second disk is not
self-encrypting.  This turns out not to be a big problem: I decided to use it
for my `/home` partition and encrypt with LUKS with a random 4096B key stored on
the self-encrypted disk used for all other (boot, system etc.) partitions.

## Fast boot

I tried to remove as many steps from the boot sequence as possible, so I also
decided not to bother with GRUB.  On my "production" laptop I do not tend to
experiment with my own-built Linux kernels nor tweak boot parameters.  The last
time I needed this, I used `kexec` which I also found way much convenient.

I took even a step further, and gave up on any bootloader and instead boot Linux
directly from UEFI, thanks to the UEFI stub.  These days I'm using Arch Linux
which by default provides Linux kernels with UEFI stub enabled, so that's even
easier.  With the EFI partition mounted as `/boot`, it also becomes transparent
for the system updates when it comes to upgrading the Linux kernel.

## Installing Arch Linux

Given the above, here's a brief list of steps I took to create my best (so far)
laptop installation.

Since this was on my ThinkPad P52, I configured power-on, (re)boot and disk
passwords in its BIOS set-up/configuration (Enter to interrupt normal start-up,
Enter to keep the menu, F1).  For the fingerprint reader, I might have set it up
(trained) in Windows when I still had it originally installed.  Not sure how
much of a problem this might be without Windows whatsoever.  For the other
laptop (Tuxedo), I could only set power-on and (re)boot passwords while for the
disk password I followed the instructions on the [Drive Trust
Alliance](https://github.com/Drive-Trust-Alliance/sedutil/wiki/Encrypting-your-drive)
website.

More details on installing Arch Linux are available in its [installation
guide](https://wiki.archlinux.org/index.php/Installation_guide).  Here is the
gist of it, relevant to my installation.  I skipped all the usual steps.

### Optional vconsole locking

Since I don't like to leave my laptop unlocked, and it might take a while to
download and install packages, I had to the following on the Arch Liux installer
system:

- set the root password
{% highlight terminal %}
passwd
{% endhighlight %}
- create `/etc/pam.d/vlock`
{% highlight terminal %}
#%PAM-1.0
auth required pam_unix.so
account required pam_unix.so
password required pam_unix.so
session required pam_unix.so
{% endhighlight %}

Then I was able to use `vlock -a` from the other virtual terminal.

### Partitioning

First, I listed the disks/partitions:
{% highlight terminal %}
fdisk -l
{% endhighlight %}

From this point, I assumed these disks:
- `/dev/nvme0n1` - non-OPAL (no FDE)
- `/dev/nvme1n1` - OPAL (FDE)

First disk (non-OPAL) dedicated as `/home`:
{% highlight terminal %}
gdisk /dev/nvme1n0
{% endhighlight %}

In `gdisk`:
{% highlight terminal %}
o
n
# (defaults, partition type: 8300) [home]
w
{% endhighlight %}

Second disk (OPAL):
{% highlight terminal %}
gdisk /dev/nvme1n1
{% endhighlight %}

In `gdisk`:
{% highlight terminal %}
o
n
# (defaults, type: ef00) [p1: efi]
n
# (defaults, size: +80G, type: 8304) [p2: root]
n
# (defaults, type: 8300) [p3: data]
w
{% endhighlight %}

Next step was formatting the partitions:
{% highlight terminal %}
mkfs.fat -F32 /dev/nvme1n1p1
mkfs.ext4 /dev/nvme1n1p2
mkfs.ext4 /dev/nvme1n1p3
{% endhighlight %}

Mounted the "FDE" partitions:
{% highlight terminal %}
mount /dev/nvme1n1p2 /mnt
mkdir /mnt/{boot,data,etc,home}

mount /dev/nvme1n1p1 /mnt/boot
mount /dev/nvme1n1p3 /mnt/data
{% endhighlight %}

The last step was to encrypt the non-OPAL disk:
{% highlight terminal %}
dd if=/dev/urandom of=/mnt/etc/home.key bs=4096 count=1
cryptsetup luksFormat --key-file=/mnt/etc/home.key /dev/nvme0n1p1
crytpsetup open --key-file=/mnt/etc/home.key /dev/nvme0n1p1 home
mkfs.ext4 /dev/mapper/home
mount /dev/mapper/home /mnt/home
{% endhighlight %}

What followed was a typical installation and configuration steps, not so much
interesting here.  What's important here is to include `efibootmgr` in the list
of installed packages so it's available in `chroot` environment.

### Boot manager (EFISTUB)

After entering `chroot`-environment, here's how I configured UEFI to boot my new
Arch Linux installation.

First, I dumped the list of partitions into the file:
{% highlight terminal %}
ls -l /dev/disk/by-partuuid/ > cmd.sh
{% endhighlight %}

Next, I edited `cmd.sh` so it ended up looking something similar to below, with
"XXX..." bits replaced with the boot partition UUID:

{% highlight shell %}
efibootmgr --disk /dev/nvme1n1 --part 1 --create \
           --label "Arch Linux" --loader /vmlinuz-linux \
		   --unicode 'root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw initrd=\initramfs-linux.img' \
		   --verbose
{% endhighlight %}

Then I executed the script and removed it, as it's no longer needed:
{% highlight terminal %}
chmod +x cmd.sh
./cmd.sh
rm ./cmd.sh
{% endhighlight %}

And that's it, voilà!  Now the system should reboot straight from UEFI into the
new Arch Linux kernel.

## Summary

This is just yet another option for setting up disk encryption and
partitioning.  This seems to work very well for me, and so far is the best I've
come up with so far.  Far on the horizon might be trying the same with Alpine
Linux, but for now I stick with Arch Linux.
