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

### OpenSUSE Tumbleweed

TODO

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


