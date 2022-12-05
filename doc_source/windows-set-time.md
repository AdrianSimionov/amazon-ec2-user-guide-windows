# Set the time for a Windows instance<a name="windows-set-time"></a>

A consistent and accurate time reference is crucial for many server tasks and processes\. Most system logs include a time stamp that you can use to determine when problems occur and in what order the events take place\. If you use the AWS CLI or an AWS SDK to make requests from your instance, these tools sign requests on your behalf\. If the date and time of your instance are not set correctly, the date in the signature may not match the date of the request, and AWS rejects the request\.

Amazon provides the Amazon Time Sync Service, which is accessible from all EC2 instances, and is also used by other AWS services\. This service uses a fleet of satellite\-connected and atomic reference clocks in each Region to deliver accurate current time readings of the Coordinated Universal Time \(UTC\) global standard through Network Time Protocol \(NTP\)\. The Amazon Time Sync Service automatically smooths any leap seconds that are added to UTC\. 

The Amazon Time Sync Service is available through NTP at the 169\.254\.169\.123 IPv4 address or the fd00:ec2::123 IPv6 address for any instance running in a VPC\. Your instance does not require access to the internet, and you do not have to configure your security group rules or your network ACL rules to allow access\. The latest versions of AWS Windows AMIs synchronize with the Amazon Time Sync Service by default\.

**Note**  
The examples in this section use the IPv4 address of the Amazon Time Sync Service: 169\.254\.169\.123\. If you are retrieving time for EC2 instances over the IPv6 address, ensure that you use the IPv6 address instead: fd00:ec2::123\. The IPv6 address is only accessible on [Instances built on the Nitro System](instance-types.md#ec2-nitro-instances)\.

**Should I use UTC for my instances?**  
We recommend that you use Coordinated Universal Time \(UTC\) for your instances to avoid human error and to facilitate synchronization across your CloudWatch Logs, Metrics, local logs, and other services\. You can, however, choose to use a different time zone to better suit your requirements\.

When you use local timezones rather than UTC, make sure that you account for aspects such as daylight savings time \(when applicable\) for automation, code, scheduled jobs, troubleshooting activities \(correlating logs\), and more\.

Use the following procedures to configure the Amazon Time Sync Service on your instance from the command prompt\. Alternatively, you can use external NTP sources\. For more information about NTP and public time sources, see [http://www\.ntp\.org/](http://www.ntp.org/)\. An instance must have access to the internet for the external NTP time sources to work\.

For Linux instances, see [Set the time for your Linux instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html)\.

**Topics**
+ [Change the time zone](#windows-changing-time-zone)
+ [Configure network time protocol \(NTP\)](#windows-configuring-ntp)
+ [Default network time protocol \(NTP\) settings for Amazon Windows AMIs](#default-ntp-settings)
+ [Amazon Time Sync Public NTP](#public-ntp-settings)
+ [Configure time settings for Windows Server 2008 and later](#windows-persisting-time-changes-w2k8)
+ [Related resources](#server-time-related-topics)

## Change the time zone<a name="windows-changing-time-zone"></a>

Windows instances are set to the UTC time zone by default\. You can change the time to correspond to your local time zone or a time zone for another part of your network\.

**To change the time zone on an instance**

1. From your instance, open a Command Prompt window\.

1. Identify the time zone to use on the instance\. To get a list of time zones, use the following command:

   ```
   tzutil /l
   ```

   This command returns a list of all available time zones, using the following format:

   ```
   display name
   time zone ID
   ```

1. Locate the time zone ID to assign to the instance\.

1. Assign to an alternate time zone by using the following command:

   ```
   tzutil /s "Pacific Standard Time"
   ```

   The new time zone should take effect immediately\.

**Note**  
You can assign the recommended UTC time zone by using the following command:  

```
tzutil /s "UTC"
```

**To configure Amazon Time Sync Public NTP for **Microsoft Windows****

For a backup to the Amazon Time Sync link\-local service, and to connect resources outside of Amazon EC2 to the Amazon Time Sync Service, you can use the Amazon Time Sync Public NTP pool located at `time.aws.com`\. Amazon Time Sync Public NTP, like the Amazon Time Sync service, automatically smooths any leap seconds that are added to UTC\. The Amazon Time Sync Service Public NTP is supported globally by our fleet of satellite\-connected and atomic reference clocks in each AWS Region See Configuring Clients to Amazon Time Sync Public NTP for configuration instructions\.

1. Open the **Control Panel**\. 

1. Choose the **Date and Time** icon\.

1. Choose the **Internet Time**tab\. This will not be available if your PC is part of a domain\. In that case, it will synchronize time with the domain controller\. You can configure the controller to use Amazon Time Sync Public NTP\. 

1. Choose **Change settings**\. 

1. Select the check box for **Synchronize with an Internet time server**\. 

1. Next to **Server**, enter **time\.aws\.com**\. 

**To configure Amazon Time Sync Public NTP for **Microsoft Windows Server****
+ Follow [Microsoft's instructions](https://support.microsoft.com/en-us/kb/816042) to update your registry\. 

## Configure network time protocol \(NTP\)<a name="windows-configuring-ntp"></a>

Amazon provides the Amazon Time Sync Service, which is accessible from all EC2 instances, and is also used by other AWS services\. We recommend that you configure your instance to use the Amazon Time Sync Service\. This service uses a fleet of satellite\-connected and atomic reference clocks in each AWS Region to deliver accurate current time readings of the Coordinated Universal Time \(UTC\) global standard\. The Amazon Time Sync Service automatically smooths any leap seconds that are added to UTC\. This service is available at the 169\.254\.169\.123 IPv4 address or the fd00:ec2::123 IPv6 address for any instance running in a VPC, and your instance does not require internet access to use it\. Starting with the August 2018 release, Windows AMIs use the Amazon Time Sync Service by default\.

**To verify the NTP configuration**

1. From your instance, open a Command Prompt window\.

1. Get the current NTP configuration by typing the following command:

   ```
   w32tm /query /configuration
   ```

   This command returns the current configuration settings for the Windows instance\.

1. \(Optional\) Get the status of the current configuration by typing the following command:

   ```
   w32tm /query /status
   ```

   This command returns information such as the last time the instance synced with the NTP server and the poll interval\.

**To change the NTP server to use the Amazon Time Sync Service**

1. From the Command Prompt window, run the following command:

   ```
   w32tm /config /manualpeerlist:169.254.169.123 /syncfromflags:manual /update
   ```

1. Verify your new settings by using the following command:

   ```
   w32tm /query /configuration
   ```

   In the output that's returned, verify that `NtpServer` displays the 169\.254\.169\.123 IP address\.

You can change the instance to use a different set of NTP servers if required\. For example, if you have Windows instances that do not have internet access, you can configure them to use an NTP server located within your private network\. If your instance is within a domain, you should change the settings to use the domain controllers as the time source to avoid time skew\. The security group of your instance must be configured to allow outbound UDP traffic on port 123 \(NTP\)\.

**To change the NTP servers**

1. From the Command Prompt window, run the following command:

   ```
   w32tm /config /manualpeerlist:"NTP servers" /syncfromflags:manual /update
   ```

   Where *NTP servers* is a space\-delimited list of NTP servers for the instance to use\.

1. Verify your new settings by using the following command:

   ```
   w32tm /query /configuration
   ```

## Default network time protocol \(NTP\) settings for Amazon Windows AMIs<a name="default-ntp-settings"></a>

Amazon Machine Images \(AMIs\) generally adhere to the out\-of\-the\-box defaults except in cases where changes are required to function on EC2 infrastructure\. The following settings have been determined to work well in a virtual environment, as well as to keep any clock drift to within one second of accuracy: 
+ **Update Interval** – governs how frequently the time service will adjust system time towards accuracy\. AWS configures the update interval to occur once every two minutes\.
+ **NTP Server** – starting with the August 2018 release, AMIs will now use the Amazon Time Sync Service by default\. This time service is accessible from any EC2 Region at the 169\.254\.169\.123 endpoint\. Additionally, the 0x9 flag indicates that the time service is acting as a client, and to use `SpecialPollInterval` to determine how frequently to check in with the configured time server\.
+ **Type** – "NTP" means that the service acts as a standalone NTP client instead of acting as part of a domain\.
+ **Enabled and InputProvider** – the time service is enabled and provides time to the operating system\.
+ **Special Poll Interval** – checks against the configured NTP Server every 900 seconds, or 15 minutes\. 


| Registry Path | Key Name | Data | 
| --- | --- | --- | 
|  HKLM:\\System\\CurrentControlSet\\services\\w32time\\Config  |  UpdateInterval  | 120 | 
| HKLM:\\System\\CurrentControlSet\\services\\w32time\\Parameters | NtpServer | 169\.254\.169\.123,0x9 | 
| HKLM:\\System\\CurrentControlSet\\services\\w32time\\Parameters | Type | NTP | 
| HKLM:\\System\\CurrentControlSet\\services\\w32time\\TimeProviders\\NtpClient | Enabled | 1 | 
| HKLM:\\System\\CurrentControlSet\\services\\w32time\\TimeProviders\\NtpClient | InputProvider | 1 | 
| HKLM:\\System\\CurrentControlSet\\services\\w32time\\TimeProviders\\NtpClient | SpecialPollInterval | 900 | 

## Amazon Time Sync Public NTP<a name="public-ntp-settings"></a>

**To configure Amazon Time Sync Public NTP for **Microsoft Windows****

For a backup to the Amazon Time Sync link\-local service, and to connect resources outside of Amazon EC2 to the Amazon Time Sync Service, you can use the Amazon Time Sync Public NTP pool located at `time.aws.com`\. Amazon Time Sync Public NTP, like the Amazon Time Sync service, automatically smooths any leap seconds that are added to UTC\. The Amazon Time Sync Service Public NTP is supported globally by our fleet of satellite\-connected and atomic reference clocks in each AWS Region See Configuring Clients to Amazon Time Sync Public NTP for configuration instructions\.

1. Open the **Control Panel**\. 

1. Choose the **Date and Time** icon\.

1. Choose the **Internet Time** tab\. This will not be available if your PC is part of a domain\. In that case, it will synchronize time with the domain controller\. You can configure the controller to use Amazon Time Sync Public NTP\.

1. Choose **Change settings**\. 

1. Select the check box for **Synchronize with an Internet time server**\. 

1. Next to **Server**, enter **time\.aws\.com**\. 

**To configure Amazon Time Sync Public NTP for **Microsoft Windows Server****
+ Follow [Microsoft's instructions](https://support.microsoft.com/en-us/kb/816042) to update your registry\. 

## Configure time settings for Windows Server 2008 and later<a name="windows-persisting-time-changes-w2k8"></a>

When you change the time on a Windows instance, you must ensure that the time persists through system restarts\. Otherwise, when the instance restarts, it reverts back to using UTC time\. For Windows Server 2008 and later, you can persist your time setting by adding a RealTimeIsUniversal registry key\. This key is set by default on all current generation instances\. To verify whether the RealTimeIsUniversal registry key is set, see Step 4 in the following procedure\. If the key is not set, follow the these steps from the beginning\.

**To set the RealTimeIsUniversal registry key**

1. From the instance, open a Command Prompt window\.

1. Use the following command to add the registry key:

   ```
   reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
   ```

1. If you are using a Windows Server 2008 AMI \(*not* Windows Server 2008 R2\) that was created before February 22, 2013, we recommend updating to the latest AWS Windows AMI\. If you are using an AMI running Windows Server 2008 R2 \(*not* Windows Server 2008\), you must verify that the Microsoft hotfix [KB2922223](https://support.microsoft.com/en-us/help/2922223/you-cannot-change-system-time-if-realtimeisuniversal-registry-entry-is) is installed\. If this hotfix is not installed, we recommend updating to the latest AWS Windows AMI\. 

1. \(Optional\) Verify that the instance saved the key successfully using the following command:

   ```
   reg query "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /s
   ```

   This command returns the subkeys for the TimeZoneInformation registry key\. You should see the RealTimeIsUniversal key at the bottom of the list, similar to the following:

   ```
   HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation
       Bias                            REG_DWORD     0x1e0
       DaylightBias                    REG_DWORD     0xffffffc4
       DaylightName                    REG_SZ        @tzres.dll,-211
       DaylightStart                   REG_BINARY    00000300020002000000000000000000
       StandardBias                    REG_DWORD     0x0
       StandardName                    REG_SZ        @tzres.dll,-212
       StandardStart                   REG_BINARY    00000B00010002000000000000000000
       TimeZoneKeyName                 REG_SZ        Pacific Standard Time
       DynamicDaylightTimeDisabled     REG_DWORD     0x0
       ActiveTimeBias                  REG_DWORD     0x1a4
       RealTimeIsUniversal             REG_DWORD     0x1
   ```

## Related resources<a name="server-time-related-topics"></a>

For more information about how the Windows operating system coordinates and manages time, including the addition of a leap second, see the following documentation:
+ [How the Windows Time Service Works](https://docs.microsoft.com/en-us/windows-server/networking/windows-time-service/how-the-windows-time-service-works) \(Microsoft\)
+ [W32tm](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-xp/bb491016(v=technet.10)) \(Microsoft\)
+ [How the Windows Time service treats a leap second](https://support.microsoft.com/en-us/help/909614/how-the-windows-time-service-treats-a-leap-second) \(Microsoft\)
+ [The story around Leap Seconds and Windows: It's likely not Y2K](https://blogs.msdn.microsoft.com/mthree/2015/01/08/the-story-around-leap-seconds-and-windows-its-likely-not-y2k/) \(Microsoft\)