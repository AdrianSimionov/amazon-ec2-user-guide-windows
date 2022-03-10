# Extend a Windows file system after resizing a volume<a name="recognize-expanded-volume-windows"></a>

After you increase the size of an EBS volume, use the Windows Disk Management utility or PowerShell to extend the disk size to the new size of the volume\. You can begin resizing the file system as soon as the volume enters the `optimizing` state\. For more information about this utility, see [Extend a basic volume](https://docs.microsoft.com/en-us/windows-server/storage/disk-management/extend-a-basic-volume) on the Microsoft Docs website\.

For more information about extending a file system on Linux, see [Extend a Linux file system after resizing a volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**Topics**
+ [Extend a Windows file system using the Disk Management utility](#recognize-expanded-volume-windows-disk-management)
+ [Extend a Windows file system using PowerShell](#recognize-expanded-volume-windows-powershell)

## Extend a Windows file system using the Disk Management utility<a name="recognize-expanded-volume-windows-disk-management"></a>

Use the following procedure to extend a Windows file system using Disk Management\.

**To extend a file system using Disk Management**

1. Before extending a file system that contains valuable data, it is a best practice to create a snapshot of the volume that contains it in case you need to roll back your changes\. For more information, see [Create Amazon EBS snapshots](ebs-creating-snapshot.md)\.

1. Log in to your Windows instance using Remote Desktop\.

1. In the **Run** dialog, enter **diskmgmt\.msc** and press Enter\. The Disk Management utility opens\.  
![\[Windows Server Disk Management Utility\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/Expand-Volume-Win2008-before.png)

1. On the **Disk Management** menu, choose **Action**, **Rescan Disks**\.

1. Open the context \(right\-click\) menu for the expanded drive and choose **Extend Volume**\.
**Note**  
**Extend Volume** might be disabled \(grayed out\) if:  
The unallocated space is not adjacent to the drive\. The unallocated space must be adjacent to the right side of the drive you want to extend\.
The volume uses the Master Boot Record \(MBR\) partition style and it is already 2TB in size\. Volumes that use MBR cannot exceed 2TB in size\.  
![\[Windows Server Disk Management Utility\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/Expand-Volume-Win2008-before-menu.png)

1. In the **Extend Volume** wizard, choose **Next**\. For **Select the amount of space in MB**, enter the number of megabytes by which to extend the volume\. Generally, you specify the maximum available space\. The highlighted text under **Selected** is the amount of space that is added, not the final size the volume will have\. Complete the wizard\.  
![\[Windows Server Extend Volume Wizard\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/Extend-Volume-Wizard-Win2008.png)

1. If you increase the size of an NVMe volume on an instance that does not have the AWS NVMe driver, you must reboot the instance to enable Windows to see the new volume size\. For more information about installing the AWS NVMe driver, see [AWS NVMe drivers for Windows instances](aws-nvme-drivers.md)\.

## Extend a Windows file system using PowerShell<a name="recognize-expanded-volume-windows-powershell"></a>

Use the following procedure to extend a Windows file system using PowerShell\.

**To extend a file system using PowerShell**

1. Before extending a file system that contains valuable data, it is a best practice to create a snapshot of the volume that contains it in case you need to roll back your changes\. For more information, see [Create Amazon EBS snapshots](ebs-creating-snapshot.md)\.

1. Log in to your Windows instance using Remote Desktop\.

1. Run PowerShell as an administrator\.

1. Run the `Get-Partition` command\. PowerShell returns the corresponding partition number for each partition, the drive letter, offset, size, and type\. Note the drive letter of the partition to extend\.

1. Run the following command to rescan the disk\.

   ```
   "rescan" | diskpart
   ```

1. Run the following command, using the drive letter you noted in step 4 in place of **<drive\-letter>**\. PowerShell returns the minimum and maximum size of the partition allowed, in bytes\.

   ```
   Get-PartitionSupportedSize -DriveLetter <drive-letter>
   ```

1. To extend the partition to a specified amount, run the following command, entering the new size of the volume in place of **<size>**\. You can enter the size in `KB`, `MB`, and `GB`; for example, `50GB`\.

   ```
   Resize-Partition -DriveLetter <drive-letter> -Size <size>
   ```

   To extend the partition to the maximum available size, run the following command\.

   ```
   Resize-Partition -DriveLetter <drive-letter> -Size $(Get-PartitionSupportedSize -DriveLetter <drive-letter>).SizeMax
   ```

   The following PowerShell commands show the complete command and response flow for extending a file system to a specific size\.  
![\[Extend a partition using PowerShell - specific\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ebs-extend-powershell-v3-specific.png)

   The following PowerShell commands show the complete command and response flow for extending a file system to the maximum available size\.  
![\[Extend a partition using PowerShell - max\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ebs-extend-powershell-v3-max.png)