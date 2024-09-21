# Linux hibernation

Step-by-step tutorial to enable hibernation on Linux.

## Verified configurations

- Linux Mint 22 on Lenovo Thinkbook 16p g2

Please, update this list.

## 1. Verify Your Swap Partition

Ensure your system recognizes the swap partition and it’s being used.

Open a terminal and run:

```bash
sudo swapon --show
```

> [!IMPORTANT]
> The swap partition size must be at least the size of RAM plus square root of RAM size.

This command will show if your swap partition is active. If it doesn’t show any output, you need to enable the swap partition.

To check if your swap partition is listed, run:

```bash
sudo fdisk -l
```

Locate the partition labeled as swap, usually something like /dev/sdaX. If your swap partition is not active, you can activate it with:

```bash
sudo swapon /dev/sdaX
```

## 2. Find the UUID of the Swap Partition

You need to identify the UUID of your swap partition to configure the hibernation settings.

In the terminal, run:

```bash
blkid | grep swap
```

This will return the UUID of your swap partition. Copy this UUID, as you will need it later.

## 3. Modify the GRUB Configuration

Now, you need to tell the system where to find the swap partition for hibernation.

Open the GRUB configuration file:

```bash
sudo nano /etc/default/grub
```

Look for the line that starts with:

```bash
GRUB_CMDLINE_LINUX_DEFAULT
```

It should look something like this:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

Modify it to include the UUID of your swap partition, like this:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=your-swap-uuid"
```

Replace `your-swap-uuid` (without quotes) with the UUID you found earlier.

Save the file by pressing Ctrl+O, then press Enter, and then exit by pressing Ctrl+X.

Update GRUB by running:

```bash
sudo update-grub
```

## 4. Edit the Initramfs Configuration

### Debian and derivates

Next, we need to ensure the system uses the swap partition for hibernation.

Open the `initramfs-tools` configuration file:

```bash
sudo nano /etc/initramfs-tools/conf.d/resume
```

Add or modify the line to specify your swap partition UUID:

```bash
RESUME=UUID=your-swap-uuid
```

Again, replace your-swap-uuid with the UUID you obtained earlier.

Save the file (Ctrl+O, Enter, then Ctrl+X).

Regenerate the initramfs:

```bash
sudo update-initramfs -u
```

### OpenSUSE (Tumbleweed)

On OpenSUSE (Tumbleweed) there is not `initramfs`, but there is `dracut`.

First of all, you must create the file `/etc/dracut.conf.d/resume.conf` if it doesn't already exist and insert in it:

```
add_device+=" /dev/disk/by-uuid/<SWAP-UUID> "
```

In addiction, if your system doesn't have the `resume` module, you must include it in the same `resume.conf` file:

```
add_dracutmodules+=" resume "
install_items+=" /usr/lib/systemd/system/systemd-hibernate-resume.service "

add_device+=" /dev/disk/by-uuid/2e44238b-0e3e-44aa-b260-b1a1c6721d76 "
```

You can check if your system has already `resume` module running:

```
sudo lsinitrd | grep resume
```

If this command display nothing, then your system doesn't have the `resume` module, instead you should see something like that:

```
-rw-r--r--   1 root     root           50 Sep 13 15:27 etc/cmdline.d/95resume.conf
-rwxr-xr-x   1 root     root        26936 Aug 19 17:00 usr/lib/systemd/systemd-hibernate-resume
-rwxr-xr-x   1 root     root        26984 Aug 19 17:00 usr/lib/systemd/system-generators/systemd-hibernate-resume-generator
-rw-r--r--   1 root     root          666 Aug 19 17:00 usr/lib/systemd/system/systemd-hibernate-resume.service
```

Close and save `resume.conf` file and then run:

```
sudo dracut --force
```

## 5. Test Hibernation

Reboot your system to apply the changes:

```bash
sudo reboot
```

After rebooting, you can test hibernation by running:

```bash
sudo systemctl hibernate
```

If everything is set up correctly, your system should hibernate and resume successfully.

In particular, it should restart as a normal boot (you should see the GRUB interfaces), but loading the contents.


