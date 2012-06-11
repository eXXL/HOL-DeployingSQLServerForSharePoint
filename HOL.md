<a name="Title" />
# Configuring SQL Server 2012 Server for SharePoint in Windows Azure #

---
<a name="Overview" />
## Overview ##

In this hands-on lab you will learn how to create and configure a virtual machine running SQL Server 2012 as part of the deploying a SharePoint farm hands-on lab.

<a name="Objectives" />
### Objectives ###

In this hands-on lab, you will learn how to:

1. Create a virtual machine using the Windows Azure portal that is hosted within the same cloud service as another virtual machine.
1. Configure a SQL Server 2012

### Setup ###

The Windows Azure PowerShell Cmdlets are required for this lab. If you have not configured them yet please see the **Automating VM Management** hands-on lab in the **Automating Windows Azure with PowerShell** module. 

>**Note:** In order to run through the complete hands-on lab, you must have network connectivity. 

<a name="Prerequisites" />
### Prerequisites ###

The following is required to complete this hands-on lab:

- [Windows Azure PowerShell CmdLets](http://msdn.microsoft.com/en-us/library/windowsazure/jj156055)
- A Windows Azure subscription with the Virtual Machines Preview enabled - you can sign up for free trial [here](http://bit.ly/WindowsAzureFreeTrial)

>**Note:** This lab was designed to use Windows 7 Operating System.

<a name="Exercises" />
## Exercises ##

This hands-on lab includes the following exercises:

1. [Creating and configuring Windows Server VM with SQL Server 2012 using the Windows Azure portal](#Exercise1)
 
Estimated time to complete this lab: **30 minutes**.

<a name="Exercise1" />
### Exercise 1: Creating and configuring Windows Server VM with SQL Server 2012 ###

You will now create the Windows Server VM and configure SQL Server. You will automatically provision a new virtual machine that is joined to the Active Directory domain at boot.

1. If you do not have the IP address of the Domain Controller virtual mMachine, Navigate to the **Windows Azure Portal** using a Web browser and sign in using the **Microsoft Account** associated with your Windows Azure account.

1. Go to **Virtual Machines**, select the VM where you deployed the AD and select the **Connect** button at the bottom panel.

1. In the VM, go to **Start**, type **cmd** and press **ENTER**.

1. Type **ipconfig** and press **ENTER**. Take note of the **IPv4 address**, you will use it later on this exercise. Close the **Remote Desktop** connection.

	![IP Address](images/ip-address.png?raw=true "IP Address")

	_IP Address_

1. Open **Windows Azure PowerShell**. Select **Start** | **All Programs** | **Windows Azure** | **Windows Azure PowerShell**, right-click **Windows Azure Powershell** and choose **Run as Administrator**.

1. Execute the following command to obtain the names of the available OS Disk images. Take note of the **SQL Server 2012** image disk name. This image is a Windows Server 2008 R2 that has a SQL Server 2012 already installed.

	<!-- mark:1 -->
	````PowerShell
	Get-AzureVMImage | Select ImageName
	````

1. Execute the following command to define the OS disk image name for the new Virtual Machine.
	
	<!-- mark:1 -->
	````PowerShell
	$imgName = '[OS-IMAGE-NAME]'
	````

1. Set up the VM's DNS settings. To do this, you will use the virtual machine you created in "Deploying Active Directory in Windows Azure" hands-on lab, were you configured the Active Directory. Replece the placeholders before executing the following command. Use the IP address you copied at the beginning of the exercise.
	
	<!-- mark:1-4 -->
	````PowerShell
	$advmIP = '[AD-IP-ADDRESS]'
	$advmName = '[AD-VM-NAME]'
	# Point to IP Address of Domain Controller Created Earlier
	$dns1 = New-AzureDns -Name $advmName -IPAddress $advmIP
	````


1. Set up the new VM's configuration settings to automatically join the domain in the provisioning process. Before executing the command, replace the placeholders with the administrator and domain passwords.

	<!-- mark:1-12 -->
	````PowerShell
	$vmName = 'SqlServer2012VM'
	$adminPassword = '[YOUR-PASSWORD]'
	$domainPassword = '[YOUR-PASSWORD]'
	$domainUser = 'administrator'
	$FQDomainName = 'contoso.com'
	$subNet = 'AppSubnet'
	# Configuring VM to Automatically Join Domain
	$advm1 = New-AzureVMConfig -Name $vmName -InstanceSize Small -ImageName $imgName | 
				Add-AzureProvisioningConfig -WindowsDomain -Password $adminPassword `
				-Domain 'contoso' -DomainPassword $domainPassword `
				-DomainUserName $domainUser -JoinDomain $FQDomainName |
		 Set-AzureSubnet -SubnetNames $subNet
	````

	>**Note:** The previous command asumes that you used the proposed names for the Domain Name and the Subnets that are shown in the **Deploying Active Directory** hands on lab. You may need to update the values if you used different names.

1. Create a new Virtual Machine using the Domain and DNS settings you defined in the previous steps. Replace the placeholder with a unique Service Name.

	````PowerShell
	$serviceName = [YOUR-SERVICE-NAME]
	$affinityGroup = 'adag'
	$adVNET = 'ADVNET'
	# New Cloud Service with VNET and DNS settings
	New-AzureVM –ServiceName $serviceName -AffinityGroup $affinityGroup `
									-VMs $advm1 -DnsSettings $dns1 -VNetName $adVNET
	````

1. Once the provisioning proces finish, connect to the VM using Remote Desktop and verify if it was automatically joined to your existing domain.

<a name="Ex1Task2" />
#### Task 2 - Configuring Disks for SQL Server ####

1. In the **Windows Azure Portal**, select the _SQLServer2012VM_ virtual machine you created in Task 1, and click **Attach**.

     ![attachemptydisk](images/attachemptydisk.png?raw=true)

	_Attaching an empty disk_

1. Select ATTACH EMPTY DISK and select 50 GB 

1. Wait until the disk has been provisioned and repeat.

1. You should now have 2 50 GB data disks attached to this virtual machine. 

1. Click connect and login to the virtual machine using RDP. 

1. Once logged in start **Computer Management** from | **Start** | **Administrative Tools** and under **Storage** click **Disk Management**.

     ![rawdisks](images/rawdisks.png?raw=true)

1. Right-click on each disk (on the left side) and mark it online. 

1. Once the disks are online you will need to right-click on one and click **Initialize** (on the left side).

    ![initializedisk](images/initializedisk.png?raw=true)

1. Once the disks are initialized you will then need to right click on the right side and select Create Simple Volume (software RAID is also support so those are options are available as well). The create simple volume wizard will allow you to format the disks and mount them for use.

    ![initializeddisks](images/initializeddisks.png?raw=true)

1. You will now configure database default location. To do so, launch SQL Enterprise Manager, right click on the server name, click **properties** and click on **Database Settings**.

1. Specify the new data disks for the default data, logs and backup folders and click OK to close.

    ![dbsettings](images/dbsettings.png?raw=true)

<a name="Ex1Task3" />
#### Task 3 - Updating SQL Server Network Configuration ####

1. In the **Windows Azure Portal**, select the _SQLServerVM1_ virtual machine you created in Task 1, and click **Connect**.

1. Click **Open** and log on using the Administrator credentials you defined when creating the VM.

1. Open **SQL Server Configuration Manager** from **Start | All Programs | Microsoft SQL Server 2012 | Configuration Tools**.
1. Expand the **SQL Server Network Configuration** node and select **Protocols for MSSQLServer** (this option might change if you used a different instance name when installing SQL Server). Make sure **Shared Memory**, **Named Pipes** and **TCP/IP** protocols are enabled. To enable a protocol, right-click the Protocol Name and select **Enable**.

	![Enabling SQL Server Protocols](images/enabling-sql-server-protocols.png?raw=true "Enabling SQL Server Protocols")

1. Close the **SQL Server Configuration Manager**.

1. Open Windows Firewall with Advanced Security from **Start | Administrative Tools**

1. Right click on Inbound Rules and Select New Rule.

1. Select Port and Click Next

1. Specify 1433 for the port and accept defaults until the last screen. Name the rule _SQL_.


<a name="Ex1Task4" />
#### Task 4 - Joining the Active directory Domain ####


1. From **Start**, right-click **Computer** and select **Properties**. 

1. Click **Advanced System Settings** and switch to **Computer Name** tab. Then click **Change**.

1. Select Domain, and enter contoso.com. When prompted use contoso\administrator and allow the reboot.

1. Once rebooted login via remote desktop again 

> **Note:** You will receive the following error which you can safely ignore.

![domjoinerror](images/domjoinerror.png?raw=true)

1. Start SQL Enterprise Manager and Under Security | Logins add the contoso\administrator login and specify the sysadmin server role.

<a name="summary" />
## Summary ##

In this lab you learned how to create and configure a SQL Server 2012 Database by provisioning a Virtual Machine in the Windows Azure portal and then applying the configuration in SQL Server. You also learned how to join a virtual machine to an existing cloud service and domain join a virtual machine.
