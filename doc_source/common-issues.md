# Common issues with Windows instances<a name="common-issues"></a>

The following are troubleshooting tips to help you solve common issues with EC2 instance running Windows Server\.

**Topics**
+ [EBS volumes don't initialize on Windows Server 2016 and later](#init-disks-win2k16)
+ [Boot an EC2 Windows instance into Directory Services Restore Mode \(DSRM\)](#boot-dsrm)
+ [Instance loses network connectivity or scheduled tasks don't run when expected](#instance-loses-network-connectivity)
+ [Unable to get console output](#no-console-output)
+ [Windows Server 2012 R2 not available on the network](#server-2012-network-loss)
+ [Disk signature collision](#disk-signature-collision)

## EBS volumes don't initialize on Windows Server 2016 and later<a name="init-disks-win2k16"></a>

Instances created from Amazon Machine Images \(AMIs\) for Windows Server 2016 and later use the EC2Launch service for a variety of startup tasks, including initializing EBS volumes\. By default, EC2Launch does not initialize secondary volumes\. You can configure EC2Launch to initialize these disks automatically\.

**To map drive letters to volumes**

1. Connect to the instance to configure and open the `C:\ProgramData\Amazon\EC2-Windows\Launch\Config\DriveLetterMappingConfig.json` file in a text editor\.

1. Specify the volume settings using the following format:

   ```
   {
   "driveLetterMapping": [
   {
     "volumeName": "sample volume",
     "driveLetter": "H"
   }
   ]
   }
   ```

1. Save your changes and close the file\.

1. Open Windows PowerShell and use the following command to run the EC2Launch script that initializes the disks:

   ```
   PS C:\>  C:\ProgramData\Amazon\EC2-Windows\Launch\Scripts\InitializeDisks.ps1
   ```

   To initialize the disks each time the instance boots, add the `-Schedule` flag as follows:

   ```
   PS C:\>  C:\ProgramData\Amazon\EC2-Windows\Launch\Scripts\InitializeDisks.ps1 -Schedule
   ```
**Note**  
If the EC2Launch script does not initialize the volumes, ensure that the volumes are online\. If the volumes are offline, run the following command to bring all disks online\.  

   ```
   PS C:\> Get-Disk | Where-Object IsOffline -Eq $True | Set-Disk -IsOffline $False
   ```

## Boot an EC2 Windows instance into Directory Services Restore Mode \(DSRM\)<a name="boot-dsrm"></a>

If an instance running Microsoft Active Directory experiences a system failure or other critical issues you can troubleshoot the instance by booting into a special version of Safe Mode called *Directory Services Restore Mode* \(DSRM\)\. In DSRM you can repair or recover Active Directory\.

### Driver support for DSRM<a name="boot-dsrm-driver"></a>

How you enable DSRM and boot into the instance depends on the drivers the instance is running\. In the EC2 console you can view driver version details for an instance from the System Log\. The following table shows which drivers are supported for DSRM\.


| Driver Versions | DSRM Supported? | Next Steps | 
| --- | --- | --- | 
| Citrix PV 5\.9 | No | Restore the instance from a backup\. You cannot enable DSRM\. | 
| AWS PV 7\.2\.0 | No | Though DSRM is not supported for this driver, you can still detach the root volume from the instance, take a snapshot of the volume or create an AMI from it, and attach it to another instance in the same Availability Zone as a secondary volume\. You can then enable DSRM \(as described in this section\)\. | 
| AWS PV 7\.2\.2 and later | Yes | Detach the root volume, attach it to another instance, and enable DSRM \(as described in this section\)\. | 
| Enhanced Networking | Yes | Detach the root volume, attach it to another instance, and enable DSRM \(as described in this section\)\. | 

For information about how to enable Enhanced Networking, see [Enabling Enhanced Networking on Windows Instances in a VPC](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/enhanced-networking.html)\. For more information about upgrading AWS PV drivers, see [Upgrade PV drivers on Windows instances](Upgrading_PV_drivers.md)\.

### Configure an instance to boot into DSRM<a name="configure-boot-dsrm"></a>

EC2 Windows instances do not have network connectivity before the operating system is running\. For this reason, you cannot press the F8 button on your keyboard to select a boot option\. You must use one of the following procedures to boot an EC2 Windows Server instance into DSRM\.

If you suspect that Active Directory has been corrupted and the instance is still running, you can configure the instance to boot into DSRM using either the System Configuration dialog box or the command prompt\.

**To boot an online instance into DSRM using the System Configuration dialog box**

1. In the **Run** dialog box, type `msconfig` and press Enter\.

1. Choose the **Boot** tab\.

1. Under **Boot options** choose **Safe boot**\.

1. Choose **Active Directory repair** and then choose **OK**\. The system prompts you to reboot the server\.

**To boot an online instance into DSRM using the command line**  
From a Command Prompt window, run the following command:

```
bcdedit /set safeboot dsrepair
```

If an instance is offline and unreachable, you must detach the root volume and attach it to another instance to enable DSRM mode\.

**To boot an offline instance into DSRM**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Locate and select the affected instance\. Choose **Instance state**, **Stop instance**\.

1. Choose **Launch instances** and create a temporary instance in the same Availability Zone as the affected instance\. Choose an instance type that uses a different version of Windows\. For example, if your instance is Windows Server 2008, then choose a Windows Server 2008 R2 instance\.
**Important**  
If you do not create the instance in the same Availability Zone as the affected instance you will not be able to attach the root volume of the affected instance to the new instance\.

1. In the navigation pane, choose **Volumes**\.

1. Locate the root volume of the affected instance\. [Detach](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ebs-detaching-volume.html) the volume and [attach](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ebs-attaching-volume.html) it to the temporary instance you created earlier\. Attach it with the default device name \(xvdf\)\.

1. Use Remote Desktop to connect to the temporary instance, and then use the Disk Management utility to [make the volume available for use](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ebs-using-volumes.html)\.

1. Open a command prompt and run the following command\. Replace *D* with the actual drive letter of the secondary volume you just attached:

   ```
   bcdedit /store D:\Boot\BCD /set {default} safeboot dsrepair
   ```

1. In the Disk Management Utility, choose the drive you attached earlier, open the context \(right\-click\) menu, and choose **Offline**\.

1. In the EC2 console, detach the affected volume from the temporary instance and reattach it to your original instance with the device name `/dev/sda1`\. You must specify this device name to designate the volume as a root volume\.

1. [Start](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/Stop_Start.html) the instance\.

1. After the instance passes the health checks in the EC2 console, connect to the instance using Remote Desktop and verify that it boots into DSRM mode\.

1. \(Optional\) Delete or stop the temporary instance you created in this procedure\.

## Instance loses network connectivity or scheduled tasks don't run when expected<a name="instance-loses-network-connectivity"></a>

If you restart your instance and it loses network connectivity, it's possible that the instance has the wrong time\.

By default, Windows instances use Coordinated Universal Time \(UTC\)\. If you set the time for your instance to a different time zone and then restart it, the time becomes offset and the instance temporarily loses its IP address\. The instance regains network connectivity eventually, but this can take several hours\. The amount of time that it takes for the instance to regain network connectivity depends on the difference between UTC and the other time zone\.

This same time issue can also result in scheduled tasks not running when you expect them to\. In this case, the scheduled tasks do not run when expected because the instance has the incorrect time\.

To use a time zone other than UTC persistently, you must set the **RealTimeIsUniversal** registry key\. Without this key, an instance uses UTC after you restart it\.

**To resolve time issues that cause a loss of network connectivity**

1. Ensure that you are running the recommended PV drivers\. For more information, see [Upgrade PV drivers on Windows instances](Upgrading_PV_drivers.md)\.

1. Verify that the following registry key exists and is set to `1`: **HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\TimeZoneInformation\\RealTimeIsUniversal**

## Unable to get console output<a name="no-console-output"></a>

For Windows instances, the instance console displays the output from tasks performed during the Windows boot process\. If Windows boots successfully, the last message logged is `Windows is Ready to use`\. Note that you can also display event log messages in the console, but this feature is not enabled by default\. For more information, see [EC2 service properties](ec2config-service.md#UsingConfigInterface_WinAMI)\.

To get the console output for your instance using the Amazon EC2 console, select the instance, and then choose **Actions**, **Monitor and troubleshoot**, **Get system log**\. To get the console output using the command line, use one of the following commands: [get\-console\-output](https://docs.aws.amazon.com/cli/latest/reference/ec2/get-console-output.html) \(AWS CLI\) or [Get\-EC2ConsoleOutput](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2ConsoleOutput.html) \(AWS Tools for Windows PowerShell\)\.

For instances running Windows Server 2012 R2 and earlier, if the console output is empty, it could indicate an issue with the EC2Config service, such as a misconfigured configuration file, or that Windows failed to boot properly\. To fix the issue, download and install the latest version of EC2Config\. For more information, see [Install the latest version of EC2Config](UsingConfig_Install.md)\.

## Windows Server 2012 R2 not available on the network<a name="server-2012-network-loss"></a>

For information about troubleshooting a Windows Server 2012 R2 instance that is not available on the network, see [Windows Server 2012 R2 loses network and storage connectivity after an instance reboot](pvdrivers-troubleshooting.md#server2012R2-instance-unavailable)\.

## Disk signature collision<a name="disk-signature-collision"></a>

You can check for and resolve disk signature collisions using [EC2Rescue for Windows Server](Windows-Server-EC2Rescue.md)\. Or, you can manually resolve disk signature issues by performing the following steps\.
**Warning**  
The following procedure describes how to edit the Windows Registry using Registry Editor\. If you are not familiar with the Windows Registry or how to safely make changes using Registry Editor, see [Configure the Registry](https://technet.microsoft.com/en-us/library/cc725612.aspx)\.

1. Open a command prompt, type regedit\.exe, and press Enter\.

1. In the **Registry Editor**, choose **HKEY\_LOCAL\_MACHINE** from the context menu \(right\-click\), and then choose **Find**\.

1. Type Windows Boot Manager and then choose **Find Next**\.

1. Choose the key named `11000001`\. This key is a sibling of the key you found in the previous step\.

1. In the right pane, choose `Element` and then choose **Modify** from the context menu \(right\-click\)\.

1. Locate the four\-byte disk signature at offset 0x38 in the data\. This is the Boot Configuration Database signature \(BCD\)\. Reverse the bytes to create the disk signature, and write it down\. For example, the disk signature represented by the following data is `E9EB3AA5`:

   ```
   ...
   0030  00 00 00 00 01 00 00 00
   0038  A5 3A EB E9 00 00 00 00
   0040  00 00 00 00 00 00 00 00
   ...
   ```

1. In a Command Prompt window, run the following command to start Microsoft DiskPart\.

   ```
   diskpart
   ```

1. Run the following DiskPart command to select the volume\. \(You can verify that the disk number is 1 using the **Disk Management** utility\.\)

   ```
   DISKPART> select disk 1
   
   Disk 1 is now the selected disk.
   ```

1. Run the following DiskPart command to get the disk signature\.

   ```
   DISKPART>  uniqueid disk
   
   Disk ID: 0C764FA8
   ```

1. If the disk signature shown in the previous step doesn't match the disk signature from BCD that you wrote down earlier, use the following DiskPart command to change the disk signature so that it matches:

   ```
   DISKPART> uniqueid disk id=E9EB3AA5
   ```