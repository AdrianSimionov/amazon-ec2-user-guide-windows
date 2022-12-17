# Troubleshoot an unreachable instance<a name="screenshot-service"></a>

If you are unable to reach your Windows instance through SSH or RDP, you can capture a screenshot of your instance and view it as an image\. This provides visibility into the status of the instance, and allows for quicker troubleshooting\. You can also use [EC2 Rescue](Windows-Server-EC2Rescue.md) on instances running Windows Server 2008 or later to gather and analyze data from offline instances\.

**Topics**
+ [Get a screenshot of an unreachable instance](#how-to-ics)
+ [Common screenshots](#ics-common)

For information about troubleshooting an unreachable Linux instance, see [Troubleshoot an unreachable instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-console.html)\.

## Get a screenshot of an unreachable instance<a name="how-to-ics"></a>

You can get screenshots of an instance while it is running or after it has crashed\. There is no data transfer cost for the screenshot\. The image is generated in JPG format and is no larger than 100 kb\.

This feature is supported on all instances, except in:
+ Bare metal instances \(instance types that end in \.metal\)
+ Instance is using an NVIDIA GRID driver
+ Instances powered by Arm\-based Graviton processors

This feature is available in the following Regions: 
+ Asia Pacific \(Hong Kong\) Region
+ Asia Pacific \(Tokyo\) Region
+ Asia Pacific \(Seoul\) Region
+ Asia Pacific \(Singapore\) Region
+ Asia Pacific \(Sydney\) Region
+ Asia Pacific \(Mumbai\) Region
+ US East \(N\. Virginia\) Region
+ US East \(Ohio\) Region
+ US West \(Oregon\) Region
+ US West \(N\. California\) Region
+ Europe \(Ireland\) Region
+ Europe \(Frankfurt\) Region
+ Europe \(Milan\) Region
+ Europe \(London\) Region
+ Europe \(Paris\) Region
+ Europe \(Stockholm\) Region
+ Europe \(Paris\) Region
+ South America \(São Paulo\) Region
+ Canada \(Central\) Region
+ Middle East \(Bahrain\) Region
+ Africa \(Cape Town\) Region
+ China \(Beijing\) Region
+ China \(Ningxia\) Region

**To get a screenshot of a running instance using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the left navigation pane, choose **Instances**\.

1. Select the instance to capture\.

1. Choose **Actions**, **Monitor and troubleshoot**, **Get instance screenshot**\.

1. Choose **Download**, or right\-click the image to download and save it\.

**To get a screenshot of a running instance using the command line**

You can use one of the following commands\. For more information about these command line interfaces, see [Access Amazon EC2](concepts.md#access-ec2)\.
+ [get\-console\-screenshot](https://docs.aws.amazon.com/cli/latest/reference/ec2/get-console-screenshot.html) \(AWS CLI\)
+ [GetConsoleScreenshot](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-GetConsoleScreenshot.html) \(Amazon EC2 Query API\)

For API calls, the returned output is base64\-encoded\. For command line tools, the decoding is performed for you\.

## Common screenshots<a name="ics-common"></a>

You can use the following information to help you troubleshoot an unreachable instance based on screenshots returned by the service\.
+ [Log on screen \(Ctrl\+Alt\+Delete\)](#logon-screen) 
+ [Recovery console screen](#recovery-console-screen) 
+ [Windows boot manager screen](#boot-manager-screen) 
+ [Sysprep screen](#sysprep-screen) 
+ [Getting ready screen](#getting-ready-screen) 
+ [Windows Update screen](#windows-update-screen) 
+ [Chkdsk](#Chkdsk) 

### Log on screen \(Ctrl\+Alt\+Delete\)<a name="logon-screen"></a>

Console Screenshot Service returned the following\.

![\[Log on screen\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ts-cs-1.png)

If an instance becomes unreachable during logon, there could be a problem with your network configuration or Windows Remote Desktop Services\. An instance can also be unresponsive if a process is using large amounts of CPU\. 

#### Network configuration<a name="network-config"></a>

Use the following information to verify that your AWS, Microsoft Windows, and local \(or on\-premises\) network configurations aren't blocking access to the instance\.


**AWS network configuration**  

| Configuration | Verify | 
| --- | --- | 
| Security group configuration | Verify that port 3389 is open for your security group\. Verify you are connecting to the right public IP address\. If the instance was not associated with an Elastic IP, the public IP changes after the instance stops/starts\. For more information, see [Remote Desktop can't connect to the remote computer](troubleshoot-connect-windows-instance.md#rdp-issues)\. | 
| VPC configuration \(Network ACLs\) | Verify that the access control list \(ACL\) for your Amazon VPC is not blocking access\. For information, see [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html) in the Amazon VPC User Guide\. | 
| VPN configuration | If you are connecting to your VPC using a virtual private network \(VPN\), verify VPN tunnel connectivity\. For more information, see [How do I troubleshoot VPN tunnel connectivity to an Amazon VPC?](https://aws.amazon.com/premiumsupport/knowledge-center/vpn-tunnel-troubleshooting/) | 


**Windows network configuration**  

| Configuration | Verify | 
| --- | --- | 
| Windows Firewall | Verify that Windows Firewall isn't blocking connections to your instance\. Disable Windows Firewall as described in bullet 7 of the remote desktop troubleshooting section, [Remote Desktop can't connect to the remote computer](troubleshoot-connect-windows-instance.md#rdp-issues)\.  | 
| Advanced TCP/IP configuration \(Use of static IP\) | The instance may be unresponsive because you configured a static IP address\. For a VPC, [create a network interface](using-eni.md#create_eni) and [attach it to the instance](using-eni.md#attach_eni)\. For EC2 Classic, enable DHCP\. | 

**Local or on\-premises network configuration**

Verify that a local network configuration isn't blocking access\. Try to connect to another instance in the same VPC as your unreachable instance\. If you can't access another instance, work with your local network administrator to determine whether a local policy is restricting access\.

#### Remote Desktop Services issues<a name="rds-issue"></a>

If the instance can't be reached during logon, there could a problem with Remote Desktop Services \(RDS\) on the instance\.


**Remote Desktop Services configuration**  

| Configuration | Verify | 
| --- | --- | 
| RDS is running | Verify that RDS is running on the instance\. Connect to the instance using the Microsoft Management Console \(MMC\) Services snap\-in \(services\.msc\)\. In the list of services, verify that Remote Desktop Services is Running\. If it isn't, start it and then set the startup type to Automatic\. If you can't connect to the instance by using the Services snap\-in, detach the root volume from the instance, take a snapshot of the volume or create an AMI from it, attach the original volume to another instance in the same Availability Zone as a secondary volume, and modify the [Start](https://technet.microsoft.com/en-us/library/cc959920.aspx) registry key\. When you are finished, reattach the root volume to the original instance\. For more information about detaching volumes, see [Detach an Amazon EBS volume from a Windows instance](ebs-detaching-volume.md)\. | 
| RDS is enabled |  Even if the service is started, it might be disabled\. Detach the root volume from the instance, take a snapshot of the volume or create an AMI from it, attach the original volume to another instance in the same Availability Zone as a secondary volume, and enable the service by modifying the **Terminal Server** registry key as described in [Enable Remote Desktop on an EC2 Instance With Remote Registry](troubleshoot-connect-windows-instance.md#troubleshooting-windows-rdp-remote-registry)\. When you are finished, reattach the root volume to the original instance\. For more information, see [Detach an Amazon EBS volume from a Windows instance](ebs-detaching-volume.md)\.  | 

#### High CPU usage<a name="high-cpu"></a>

Check the **CPUUtilization \(Maximum\)** metric on your instance by using Amazon CloudWatch\. If **CPUUtilization \(Maximum\)** is a high number, wait for the CPU to go down and try connecting again\. High CPU usage can be caused by:
+ Windows Update
+ Security Software Scan
+ Custom Startup Script
+ Task Scheduler

For more information, see [Get Statistics for a Specific Resource](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/US_SingleMetricPerInstance.html) in the *Amazon CloudWatch User Guide*\. For additional troubleshooting tips, see [High CPU usage shortly after Windows starts](troubleshooting-launch.md#high-cpu-issue)\.

### Recovery console screen<a name="recovery-console-screen"></a>

Console Screenshot Service returned the following\.

![\[Recovery console screenshot\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ts-cs-2.png)

The operating system might boot into the Recovery console and get stuck in this state if the `bootstatuspolicy` is not set to `ignoreallfailures`\. Use the following procedure to change the `bootstatuspolicy` configuration to `ignoreallfailures`\.

By default, the policy configuration for public Windows AMIs provided by AWS is set to `ignoreallfailures`\.

1. Stop the unreachable instance\.

1. Create a snapshot of the root volume\. The root volume is attached to the instance as `/dev/sda1`\. 

   Detach the root volume from the unreachable instance, take a snapshot of the volume or create an AMI from it, and attach it to another instance in the same Availability Zone as a secondary volume\. For more information, see [Detach an Amazon EBS volume from a Windows instance](ebs-detaching-volume.md)\.
**Warning**  
If your temporary instance and the original instance were launched using the same AMI, you must complete additional steps or you won't be able to boot the original instance after you restore its root volume because of a disk signature collision\. If you must create a temporary instance using the same AMI, to avoid a disk signature collision, complete the steps in [Disk signature collision](common-issues.md#disk-signature-collision)\.  
Alternatively, select a different AMI for the temporary instance\. For example, if the original instance uses an AMI for Windows Server 2008 R2, launch the temporary instance using an AMI for Windows Server 2012\.

1. Log in to the instance and run the following command from a command prompt to change the `bootstatuspolicy` configuration to `ignoreallfailures`\.

   ```
   bcdedit /store Drive Letter:\boot\bcd /set {default} bootstatuspolicy ignoreallfailures
   ```

1. Reattach the volume to the unreachable instance and start the instance again\.

### Windows boot manager screen<a name="boot-manager-screen"></a>

Console Screenshot Service returned the following\.

![\[Windows Boot Manager Screen\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ts-cs-3.png)

The operating system experienced a fatal corruption in the system file and/or the registry\. When the instance is stuck in this state, you should recover the instance from a recent backup AMI or launch a replacement instance\. If you need to access data on the instance, detach any root volumes from the unreachable instance, take a snapshot of those volume or create an AMI from them, and attach them to another instance in the same Availability Zone as a secondary volume\. For more information, see [Detach an Amazon EBS volume from a Windows instance](ebs-detaching-volume.md)\.

### Sysprep screen<a name="sysprep-screen"></a>

Console Screenshot Service returned the following\.

![\[Sysprep Screen\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ts-cs-4.png)

You may see this screen if you did not use the EC2Config Service to call Sysprep or if the operating system failed while running Sysprep\. You can reset the password using [EC2Rescue](Windows-Server-EC2Rescue.md)\. Otherwise, [Create a standardized Amazon Machine Image \(AMI\) using Sysprep](Creating_EBSbacked_WinAMI.md#ami-create-standard)\.

### Getting ready screen<a name="getting-ready-screen"></a>

Console Screenshot Service returned the following\.

![\[Getting Ready Screen\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ts-cs-5.png)

Refresh the Instance Console Screenshot Service repeatedly to verify that the progress ring is spinning\. If the ring is spinning, wait for the operating system to start up\. You can also check the **CPUUtilization \(Maximum\)** metric on your instance by using Amazon CloudWatch to see if the operating system is active\. If the progress ring is not spinning, the instance may be stuck at the boot process\. Reboot the instance\. If rebooting does not solve the problem, recover the instance from a recent backup AMI or launch a replacement instance\. If you need to access data on the instance, detach the root volume from the unreachable instance, take a snapshot of the volume or create an AMI from it\. Then attach it to another instance in the same Availability Zone as a secondary volume\.

### Windows Update screen<a name="windows-update-screen"></a>

Console Screenshot Service returned the following\.

![\[Windows Update Screen\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ts-cs-6.png)

The Windows Update process is updating the registry\. Wait for the update to finish\. Do not reboot or stop the instance as this may cause data corruption during the update\.

**Note**  
The Windows Update process can consume resources on the server during the update\. If you experience this problem often, consider using faster instance types and faster EBS volumes\. 

### Chkdsk<a name="Chkdsk"></a>

Console Screenshot Service returned the following\.

![\[Chkdsk Screen\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/ts-cs-7.png)

Windows is running the chkdsk system tool on the drive to verify file system integrity and fix logical file system errors\. Wait for process to complete\.