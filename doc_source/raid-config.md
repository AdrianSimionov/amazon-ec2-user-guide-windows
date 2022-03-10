# RAID configuration on Windows<a name="raid-config"></a>

With Amazon EBS, you can use any of the standard RAID configurations that you can use with a traditional bare metal server, as long as that particular RAID configuration is supported by the operating system for your instance\. This is because all RAID is accomplished at the software level\. 

Amazon EBS volume data is replicated across multiple servers in an Availability Zone to prevent the loss of data from the failure of any single component\. This replication makes Amazon EBS volumes ten times more reliable than typical commodity disk drives\. For more information, see [Amazon EBS Availability and Durability](https://aws.amazon.com/ebs/details/#Amazon_EBS_Availability_and_Durability) in the Amazon EBS product detail pages\.

**Note**  
You should avoid booting from a RAID volume\. If one of the devices fails, you may be unable to boot the operating system\.

If you need to create a RAID array on a Linux instance, see [RAID configuration on Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**Topics**
+ [RAID configuration options](#raid-config-options)
+ [Create a RAID 0 array on Windows](#windows-raid)
+ [Create snapshots of volumes in a RAID array](#ebs-snapshots-raid-array)

## RAID configuration options<a name="raid-config-options"></a>

Creating a RAID 0 array allows you to achieve a higher level of performance for a file system than you can provision on a single Amazon EBS volume\. Use RAID 0 when I/O performance is of the utmost importance\. With RAID 0, I/O is distributed across the volumes in a stripe\. If you add a volume, you get the straight addition of throughput and IOPS\. However, keep in mind that performance of the stripe is limited to the worst performing volume in the set, and that the loss of a single volume in the set results in a complete data loss for the array\.

The resulting size of a RAID 0 array is the sum of the sizes of the volumes within it, and the bandwidth is the sum of the available bandwidth of the volumes within it\. For example, two 500 GiB `io1` volumes with 4,000 provisioned IOPS each create a 1000 GiB RAID 0 array with an available bandwidth of 8,000 IOPS and 1,000 MiB/s of throughput\.

**Important**  
RAID 5 and RAID 6 are not recommended for Amazon EBS because the parity write operations of these RAID modes consume some of the IOPS available to your volumes\. Depending on the configuration of your RAID array, these RAID modes provide 20\-30% fewer usable IOPS than a RAID 0 configuration\. Increased cost is a factor with these RAID modes as well; when using identical volume sizes and speeds, a 2\-volume RAID 0 array can outperform a 4\-volume RAID 6 array that costs twice as much\.  
RAID 1 is also not recommended for use with Amazon EBS\. RAID 1 requires more Amazon EC2 to Amazon EBS bandwidth than non\-RAID configurations because the data is written to multiple volumes simultaneously\. In addition, RAID 1 does not provide any write performance improvement\. 

## Create a RAID 0 array on Windows<a name="windows-raid"></a>

This documentation provides a basic RAID 0 setup example\.

Before you perform this procedure, you need to decide how large your RAID 0 array should be and how many IOPS you want to provision\.

Use the following procedure to create the RAID 0 array\. Note that you can get directions for Linux instances from [Create a RAID 0 array on Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html#linux-raid) in the *Amazon EC2 User Guide for Linux Instances*\.

**To create a RAID 0 array on Windows**

1. Create the Amazon EBS volumes for your array\. For more information, see [Create an Amazon EBS volume](ebs-creating-volume.md)\.
**Important**  
Create volumes with identical size and IOPS performance values for your array\. Make sure you do not create an array that exceeds the available bandwidth of your EC2 instance\.

1. Attach the Amazon EBS volumes to the instance that you want to host the array\. For more information, see [Attach an Amazon EBS volume to an instance](ebs-attaching-volume.md)\.

1. Connect to your Windows instance\. For more information, see [Connect to your Windows instance](connecting_to_windows_instance.md)\.

1. Open a command prompt and type the diskpart command\.

   ```
   diskpart
   
   Microsoft DiskPart version 6.1.7601
   Copyright (C) 1999-2008 Microsoft Corporation.
   On computer: WIN-BM6QPPL51CO
   ```

1. At the DISKPART prompt, list the available disks with the following command\.

   ```
   DISKPART> list disk
   
     Disk ###  Status         Size     Free     Dyn  Gpt
     --------  -------------  -------  -------  ---  ---
     Disk 0    Online           30 GB      0 B
     Disk 1    Online            8 GB      0 B
     Disk 2    Online            8 GB      0 B
   ```

   Identify the disks you want to use in your array and take note of their disk numbers\.

1. <a name="windows_raid_disk_step"></a>Each disk you want to use in your array must be an online dynamic disk that does not contain any existing volumes\. Use the following steps to convert basic disks to dynamic disks and to delete any existing volumes\.

   1. Select a disk you want to use in your array with the following command, substituting *n* with your disk number\.

      ```
      DISKPART> select disk n
      
      Disk n is now the selected disk.
      ```

   1. If the selected disk is listed as `Offline`, bring it online by running the online disk command\.

   1. If the selected disk does not have an asterisk in the `Dyn` column in the previous list disk command output, you need to convert it to a dynamic disk\.

      ```
      DISKPART> convert dynamic
      ```
**Note**  
If you receive an error that the disk is write protected, you can clear the read\-only flag with the ATTRIBUTE DISK CLEAR READONLY command and then try the dynamic disk conversion again\.

   1. Use the detail disk command to check for existing volumes on the selected disk\.

      ```
      DISKPART> detail disk
      
      XENSRC PVDISK SCSI Disk Device
      Disk ID: 2D8BF659
      Type   : SCSI
      Status : Online
      Path   : 0
      Target : 1
      LUN ID : 0
      Location Path : PCIROOT(0)#PCI(0300)#SCSI(P00T01L00)
      Current Read-only State : No
      Read-only  : No
      Boot Disk  : No
      Pagefile Disk  : No
      Hibernation File Disk  : No
      Crashdump Disk  : No
      Clustered Disk  : No
      
        Volume ###  Ltr  Label        Fs     Type        Size     Status     Info
        ----------  ---  -----------  -----  ----------  -------  ---------  --------
        Volume 2     D   NEW VOLUME   FAT32  Simple      8189 MB  Healthy
      ```

      Note any volume numbers on the disk\. In this example, the volume number is 2\. If there are no volumes, you can skip the next step\.

   1. \(Only required if volumes were identified in the previous step\) Select and delete any existing volumes on the disk that you identified in the previous step\.
**Warning**  
This destroys any existing data on the volume\. 

      1. Select the volume, substituting *n* with your volume number\.

         ```
         DISKPART> select volume n
         Volume n is the selected volume.
         ```

      1. Delete the volume\.

         ```
         DISKPART> delete volume
         
         DiskPart successfully deleted the volume.
         ```

      1. Repeat these substeps for each volume you need to delete on the selected disk\.

   1. Repeat [Step 6](#windows_raid_disk_step) for each disk you want to use in your array\.

1. Verify that the disks you want to use are now dynamic\. In this case, we're using disks 1 and 2 for the RAID volume\.

   ```
   DISKPART> list disk
   
     Disk ###  Status         Size     Free     Dyn  Gpt
     --------  -------------  -------  -------  ---  ---
     Disk 0    Online           30 GB      0 B
     Disk 1    Online            8 GB      0 B   *
     Disk 2    Online            8 GB      0 B   *
   ```

1. Create your raid array\. On Windows, a RAID 0 volume is referred to as a striped volume\.

   To create a striped volume array on disks 1 and 2, use the following command \(note the `stripe` option to stripe the array\):

   ```
   DISKPART> create volume stripe disk=1,2
   DiskPart successfully created the volume.
   ```

1. Verify your new volume\.

   ```
   DISKPART> list volume
   
     DISKPART> list volume
   
     Volume ###  Ltr  Label        Fs     Type        Size     Status     Info
     ----------  ---  -----------  -----  ----------  -------  ---------  --------
     Volume 0     C                NTFS   Partition     29 GB  Healthy    System
     Volume 1                      RAW    Stripe        15 GB  Healthy
   ```

   Note that the `Type` column now indicates that Volume 1 is a `stripe` volume\.

1. Select and format your volume so that you can begin using it\.

   1. Select the volume you want to format, substituting *n* with your volume number\.

      ```
      DISKPART> select volume n
      
      Volume n is the selected volume.
      ```

   1. Format the volume\.
**Note**  
To perform a full format, omit the `quick` option\.

      ```
      DISKPART> format quick recommended label="My new volume"
      
        100 percent completed
      
      DiskPart successfully formatted the volume.
      ```

   1. Assign an available drive letter to your volume\.

      ```
      DISKPART> assign letter f
      
      DiskPart successfully assigned the drive letter or mount point.
      ```

   Your new volume is now ready to use\.

## Create snapshots of volumes in a RAID array<a name="ebs-snapshots-raid-array"></a>

If you want to back up the data on the EBS volumes in a RAID array using snapshots, you must ensure that the snapshots are consistent\. This is because the snapshots of these volumes are created independently\. To restore EBS volumes in a RAID array from snapshots that are out of sync would degrade the integrity of the array\.

To create a consistent set of snapshots for your RAID array, use [EBS multi\-volume snapshots](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateSnapshots.html)\. Multi\-volume snapshots allow you to take point\-in\-time, data coordinated, and crash\-consistent snapshots across multiple EBS volumes attached to an EC2 instance\. You do not have to stop your instance to coordinate between volumes to ensure consistency because snapshots are automatically taken across multiple EBS volumes\. For more information, see the steps for creating multi\-volume snapshots under [Creating Amazon EBS snapshots](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ebs-creating-snapshot.html)\. 