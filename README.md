# RAID
A comprehensive guide and lab for managing Linux Software RAID 1 (Mirroring) using mdadm. Includes installation, configuration, and disk failure/recovery simulation.

🛠 Installation
Ensure your system is up to date and install the mdadm utility, which is the standard tool for managing software RAID on Linux.

```bash
sudo apt update
sudo apt install mdadm -y
```

🏗 Setup & Configuration
1. Create the RAID 1 Array
We combine two physical disks (/dev/sdc and /dev/sdd) into one virtual device (/dev/md0).
```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdc /dev/sdd
```
2. Check Status
Verify that the array was created successfully and is currently syncing.

```bash
cat /proc/mdstat
# OR for more detail:
sudo mdadm --detail /dev/md0
```
3. Format and Mount
To use the RAID, we must treat it like a new hard drive.
```bash
# Format with ext4 filesystem
sudo mkfs.ext4 /dev/md0 

# Create mount point and mount it
sudo mkdir -p /mnt/raid1
sudo mount /dev/md0 /mnt/raid1

# Verify space and mount status
df -h
```

🧪 Failure & Recovery Test
This section demonstrates how RAID 1 protects your data even when a drive fails.

Step 1: Write Test Data
Create a "witness" file to prove data survives the crash.
```bash
echo "Raid 1 Working" | sudo tee /mnt/raid1/test.txt
cat /mnt/raid1/test.txt
```
Step 2: Simulate Drive Failure
Manually flag /dev/sdc as faulty.
```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdc
```
Step 3: Verify Integrity
Check the status and the file. The system will now be in a degraded state but the file remains accessible.
```bash
cat /proc/mdstat
cat /mnt/raid1/test.txt # Data is still here!
```
Step 4: Recover & Rebuild
Remove the "failed" disk and add it back as a "new" drive to trigger a resync.
```bash
# Remove the faulty disk
sudo mdadm --manage /dev/md0 --remove /dev/sdc

# Wipe old metadata (critical for VM testing)
sudo mdadm --zero-superblock /dev/sdc

# Re-add to the array
sudo mdadm --manage /dev/md0 --add /dev/sdc
```
Step 5: Monitor Progress
Wait for the resync to reach 100% to return to a healthy, redundant state.
```bash
watch cat /proc/mdstat
```

Optional: 
```bash
sudo mdadm --detail /dev/md0
```
