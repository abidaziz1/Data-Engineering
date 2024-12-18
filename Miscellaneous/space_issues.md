**Resolving Disk Space Issue on Ubuntu Using LVM and Parted**

**Objective**:  
The goal of this report is to describe the steps and commands used to resolve the disk space issue on an Ubuntu machine running inside VirtualBox, particularly to increase the available disk space by resizing the partition and logical volume.

---

### **1. Increase Virtual Machine Disk Size in VirtualBox**

The first step was to increase the size of the virtual machine’s hard disk. This can be done via the VirtualBox settings.

**Steps**:
1. Open VirtualBox and select the virtual machine (VM).
2. Go to **Settings** > **Storage**.
3. Select the **virtual hard disk** and click the **"Edit"** button (disk icon).
4. Increase the size of the disk to the desired value (in this case, 100 GB).
5. Save and start the virtual machine.

---

### **2. Verify New Disk Size**

After resizing the disk in VirtualBox, the operating system may not automatically recognize the new size. The first step is to check if the new disk space is detected.

**Command**:
```bash
df -h
```
Output:
```bash
Filesystem                         Size  Used Avail Use% Mounted on
/dev/sda1                          60G   30G   30G  50% /
```

---

### **3. Fix the GPT Partition Table Using Parted**

After resizing the disk in VirtualBox, the partition table needs to be updated to make use of the new space. The `parted` utility is used for partition resizing.

**Steps**:
1. Run the following command to check the current partition table:
   ```bash
   sudo parted -l
   ```
   The system will show the available disk and partition information.
   
2. If the partition table doesn’t use all available space, `parted` will ask if you want to fix it. Choose **Fix** to allow it to reallocate the space:
   ```bash
   sudo parted /dev/sda
   ```

3. Resize the partition to use all available space:
   ```bash
   resizepart 3 100%
   ```
4. Exit `parted`:
   ```bash
   quit
   ```

---

### **4. Extend the Logical Volume (LVM)**

After resizing the partition, the next step is to extend the logical volume (LV) to use the new space. Logical Volume Management (LVM) is used to manage disk space more flexibly.

**Steps**:
1. Use the `lvdisplay` command to verify the logical volume information:
   ```bash
   sudo lvdisplay
   ```
2. Resize the logical volume to use all available space:
   ```bash
   sudo lvresize -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
   ```
   This command increases the logical volume to use 100% of the free space available.

---

### **5. Resize the Filesystem**

After resizing the logical volume, the filesystem needs to be resized to make use of the new logical volume size. This can be done using the `resize2fs` command.

**Steps**:
1. Run the following command to resize the filesystem:
   ```bash
   sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
   ```
2. After completion, verify that the filesystem is resized:
   ```bash
   df -h
   ```

**Output**:
```bash
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   63G   30G   31G  50% /
```

---

### **6. Check the Disk Space**

Finally, verify that the system is now using the new disk space effectively. Use the `df -h` command to confirm:

**Command**:
```bash
df -h
```
This will show the disk usage, and you should see the increased size for the root partition.

---

### **Conclusion**

By following the above steps, the disk space issue on the Ubuntu VM was resolved. The steps involved resizing the virtual machine’s disk in VirtualBox, adjusting the partition table using `parted`, expanding the logical volume with LVM, and finally resizing the filesystem with `resize2fs`. These changes allowed the Ubuntu system to use the newly available disk space efficiently, ensuring optimal performance for running services like Hadoop, Hive, Spark, and Docker.

This approach provided a flexible way to expand disk space while maintaining data integrity and minimizing system downtime.
