# Network maximum transmission unit \(MTU\) for your EC2 instance<a name="network_mtu"></a>

The maximum transmission unit \(MTU\) of a network connection is the size, in bytes, of the largest permissible packet that can be passed over the connection\. The larger the MTU of a connection, the more data that can be passed in a single packet\. Ethernet frames consist of the packet, or the actual data you are sending, and the network overhead information that surrounds it\.

Ethernet frames can come in different formats, and the most common format is the standard Ethernet v2 frame format\. It supports 1500 MTU, which is the largest Ethernet packet size supported over most of the internet\. The maximum supported MTU for an instance depends on its instance type\.

The following rules apply to instances that are in Wavelength Zones:
+ Traffic that goes from one instance to another within a VPC in the same Wavelength Zone has an MTU of 1300\.
+ Traffic that goes from one instance to another that uses the carrier IP within a Wavelength Zone has an MTU of 1500\.
+ Traffic that goes from one instance to another between a Wavelength Zone and the Region that uses a public IP address has an MTU of 1500\.
+ Traffic that goes from one instance to another between a Wavelength Zone and the Region that uses a private IP address has an MTU of 1300\.

To see Network MTU information for Linux instances, switch to this page in the *Amazon EC2 User Guide for Linux Instances* guide: [Network maximum transmission unit \(MTU\) for your EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/network_mtu.html)\.

**Topics**
+ [Jumbo frames \(9001 MTU\)](#jumbo_frame_instances)
+ [Path MTU Discovery](#path_mtu_discovery)
+ [Check the path MTU between two hosts](#check_path_mtu)
+ [Check and set the MTU on your Windows instance](#set_mtu_windows)
+ [Troubleshoot](#mtu-troubleshooting)

## Jumbo frames \(9001 MTU\)<a name="jumbo_frame_instances"></a>

Jumbo frames allow more than 1500 bytes of data by increasing the payload size per packet, and thus increasing the percentage of the packet that is not packet overhead\. Fewer packets are needed to send the same amount of usable data\. However, traffic is limited to a maximum MTU of 1500 in the following cases:
+ Traffic over an internet gateway
+ Traffic over an inter\-region VPC peering connection
+ Traffic over VPN connections
+ Traffic outside of a given AWS Region for EC2\-Classic

If packets are over 1500 bytes, they are fragmented, or they are dropped if the `Don't Fragment` flag is set in the IP header\.

Jumbo frames should be used with caution for internet\-bound traffic or any traffic that leaves a VPC\. Packets are fragmented by intermediate systems, which slows down this traffic\. To use jumbo frames inside a VPC and not slow traffic that's bound for outside the VPC, you can configure the MTU size by route, or use multiple elastic network interfaces with different MTU sizes and different routes\.

For instances that are collocated inside a cluster placement group, jumbo frames help to achieve the maximum network throughput possible, and they are recommended in this case\. For more information, see [Placement groups](placement-groups.md)\.

You can use jumbo frames for traffic between your VPCs and your on\-premises networks over AWS Direct Connect\. For more information, and for how to verify Jumbo Frame capability, see [Setting Network MTU](https://docs.aws.amazon.com/directconnect/latest/UserGuide/set-jumbo-frames-vif.html) in the *AWS Direct Connect User Guide*\.

All Amazon EC2 instance types support 1500 MTU and all [current generation instance types](instance-types.md#instance-type-summary-table) support jumbo frames\. The following previous generation instance types support jumbo frames: A1, C3, G2, I2, M3, and R3\.

For more information about supported MTU sizes for transit gateways, see [MTU](https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-quotas.html#mtu-quota) in *Amazon VPC Transit Gateways*\.

## Path MTU Discovery<a name="path_mtu_discovery"></a>

Path MTU Discovery \(PMTUD\) is used to determine the path MTU between two devices\. The path MTU is the maximum packet size that's supported on the path between the originating host and the receiving host\. When there is a difference in the MTU size in the network between two hosts, PMTUD enables the receiving host to respond to the originating host with an ICMP message\. This ICMP message instructs the originating host to use the lowest MTU size along the network path and to resend the request\. Without this negotiation, packet drop can occur because the request is too large for the receiving host to accept\.

For IPv4, when a host sends a packet that's larger than the MTU of the receiving host or that's larger than the MTU of a device along the path, the receiving host or device drops the packet, and then returns the following ICMP message: `Destination Unreachable: Fragmentation Needed and Don't Fragment was Set` \(Type 3, Code 4\)\. This instructs the transmitting host to split the payload into multiple smaller packets, and then retransmit them\. 

The IPv6 protocol does not support fragmentation in the network\. When a host sends a packet that's larger than the MTU of the receiving host or that's larger than the MTU of a device along the path, the receiving host or device drops the packet, and then returns the following ICMP message: `ICMPv6 Packet Too Big (PTB)` \(Type 2\)\. This instructs the transmitting host to split the payload into multiple smaller packets, and then retransmit them\. 

By default, security groups do not allow any inbound ICMP traffic\. If you don't explicitly configure an ICMP inbound rule for your security group, PMTUD is blocked\. For more information about configuring ICMP rules in a network ACL, see [Path MTU Discovery](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#path_mtu_discovery) in the *Amazon VPC User Guide*\.

**Important**  
Path MTU Discovery does not guarantee that jumbo frames will not be dropped by some routers\. An internet gateway in your VPC will forward packets up to 1500 bytes only\. 1500 MTU packets are recommended for internet traffic\.

## Check the path MTU between two hosts<a name="check_path_mtu"></a>

You can check the path MTU between two hosts using the mturoute\.exe command,  which you can download and install from [http://www\.elifulkerson\.com/projects/mturoute\.php](http://www.elifulkerson.com/projects/mturoute.php)\. 

**To check path MTU using mturoute\.exe**

1. Download mturoute\.exe from [http://www\.elifulkerson\.com/projects/mturoute\.php](http://www.elifulkerson.com/projects/mturoute.php)\.

1. Open a Command Prompt window and change to the directory where you downloaded mturoute\.exe\.

1. Use the following command to check the path MTU between your EC2 instance and another host\. You can use a DNS name or an IP address as the destination\. If the destination is another EC2 instance, verify that the security group allows inbound UDP traffic\. This example checks the path MTU between an EC2 instance and `www.elifulkerson.com`\.

   ```
   .\mturoute.exe www.elifulkerson.com
   * ICMP Fragmentation is not permitted. *
   * Speed optimization is enabled. *
   * Maximum payload is 10000 bytes. *
   + ICMP payload of 1472 bytes succeeded.
   - ICMP payload of 1473 bytes is too big.
   Path MTU: 1500 bytes.
   ```

   In this example, the path MTU is 1500\.

## Check and set the MTU on your Windows instance<a name="set_mtu_windows"></a>

Some drivers are configured to use jumbo frames, and others are configured to use standard frame sizes\. You might want to use jumbo frames for network traffic within your VPC or standard frames for internet traffic\. Whatever your use case, we recommend that you verify that your instances behave as expected\.

If your instance runs in a Wavelength Zone, the maximum MTU value is 1300\.

### ENA Driver<a name="set-mtu-windows-ena-driver"></a>

**For Driver Versions 1\.5 and Earlier**

You can change the MTU setting using Device Manager or the Set\-NetAdapterAdvancedProperty command\.

To get the current MTU setting using the Get\-NetAdapterAdvancedProperty command, use the following command\. Check the entry for the interface name `MTU`\. A value of 9001 indicates that Jumbo frames are enabled\. Jumbo frames are disabled by default\.

```
Get-NetAdapterAdvancedProperty -Name "Ethernet"
```

Enable jumbo frames as follows:

```
Set-NetAdapterAdvancedProperty -Name "Ethernet" -RegistryKeyword "MTU" -RegistryValue 9001
```

Disable jumbo frames as follows:

```
Set-NetAdapterAdvancedProperty -Name "Ethernet" -RegistryKeyword "MTU" -RegistryValue 1500
```

**For Driver Versions 2\.1\.0 and Later**

You can change the MTU setting using Device Manager or the Set\-NetAdapterAdvancedProperty command\.

To get the current MTU setting using the Get\-NetAdapterAdvancedProperty command, use the following command\. Check the entry for the interface name `*JumboPacket`\. A value of 9015 indicates that Jumbo frames are enabled\. Jumbo frames are disabled by default\.

Run Get\-NetAdapterAdvancedProperty or use wildcard \(asterisk\) to detect all corresponding Ethernet names\. 

```
Get-NetAdapterAdvancedProperty -Name "Ethernet*"
```

Run the following commands and include the Ethernet name you want to query\.

```
Get-NetAdapterAdvancedProperty -Name "Ethernet"
```

Enable jumbo frames as follows\.

```
Set-NetAdapterAdvancedProperty -Name "Ethernet" -RegistryKeyword "*JumboPacket" -RegistryValue 9015
```

Disable jumbo frames as follows:

```
Set-NetAdapterAdvancedProperty -Name "Ethernet" -RegistryKeyword "*JumboPacket" -RegistryValue 1514
```

### Intel SRIOV 82599 driver<a name="set-mtu-windows-sriov-driver"></a>

You can change the MTU setting using Device Manager or the Set\-NetAdapterAdvancedProperty command\.

To get the current MTU setting using the Get\-NetAdapterAdvancedProperty command, use the following command\. Check the entry for the interface name `*JumboPacket`\. A value of 9014 indicates that Jumbo frames are enabled\. \(Note that the MTU size includes the header and the payload\.\) Jumbo frames are disabled by default\.

```
Get-NetAdapterAdvancedProperty -Name "Ethernet"
```

Enable jumbo frames as follows:

```
Set-NetAdapterAdvancedProperty -Name "Ethernet" -RegistryKeyword "*JumboPacket" -RegistryValue 9014
```

Disable jumbo frames as follows:

```
Set-NetAdapterAdvancedProperty -Name "Ethernet" -RegistryKeyword "*JumboPacket" -RegistryValue 1514
```

### AWS PV driver<a name="set-mtu-windows-pv-driver"></a>

You cannot change the MTU setting using Device Manager, but you can change it using the netsh command\.

Get the current MTU setting using the following command\. The name of the interface can vary\. In the output, look for an entry with the name "Ethernet," "Ethernet 2," or "Local Area Connection"\. You'll need the interface name to enable or disable jumbo frames\. A value of 9001 indicates that Jumbo frames are enabled\.

```
netsh interface ipv4 show subinterface
```

Enable jumbo frames as follows:

```
netsh interface ipv4 set subinterface "Ethernet" mtu=9001
```

Disable jumbo frames as follows:

```
netsh interface ipv4 set subinterface "Ethernet" mtu=1500
```

## Troubleshoot<a name="mtu-troubleshooting"></a>

If you experience connectivity issues between your EC2 instance and an Amazon Redshift cluster when using jumbo frames, see [Queries Appear to Hang](https://docs.aws.amazon.com/redshift/latest/mgmt/connecting-drop-issues.html) in the *Amazon Redshift Cluster Management Guide*