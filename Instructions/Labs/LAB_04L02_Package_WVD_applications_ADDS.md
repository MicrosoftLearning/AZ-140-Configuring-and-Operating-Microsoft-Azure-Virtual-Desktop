---
lab:
    title: 'Lab: Package Windows Virtual Desktop applications (AD DS)'
    module: 'Module 4: Manage User Environments and Apps'
---

# Lab - Package Windows Virtual Desktop applications (AD DS)
# Student lab manual

## Lab dependencies

- An Azure subscription
- A Microsoft account or an Azure AD account with the Global Administrator role in the Azure AD tenant associated with the Azure subscription and with the Owner or Contributor role in the Azure subscription
- The completed lab **Prepare for deployment of Azure Windows Virtual Desktop (AD DS)** or **Prepare for deployment of Azure Windows Virtual Desktop (Azure AD DS)**
- The completed lab **Windows Virtual Desktop profile management (AD DS)** or **Windows Virtual Desktop profile management (Azure AD DS)**

> **Note**: At the time of authoring this lab, the MSIX app attach functionality for Windows Virtual Desktop is in public preview. In order to try it, you need to submit a request via on [online form](https://aka.ms/enablemsixappattach) to enable MSIX app attach in your subscription. The approval and processing of requests can take up to 24 hours during business days. You'll receive an email confirmation once your request has been accepted and completed.

## Estimated Time

90 minutes

## Lab scenario

You need to package and deploy Windows Virtual Desktop applications in an Active Directory Domain Services (AD DS) environment.

## Objectives
  
After completing this lab, you will be able to:

- Prepare for and create MSIX app packages
- Implement MSIX app attach container for Windows Virtual Desktop in AD DS environment
- Implement the MSIX app attach on Windows Virtual Desktop in AD DS environment

## Lab files

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## Instructions

### Exercise 1: Prepare for and create MSIX app packages

The main tasks for this exercise are as follows:

1. Prepare for configuration of Windows Virtual Desktop session hosts
1. Deploy an Azure VM running Windows 10 by using an Azure Resource Manager QuickStart template
1. Prepare the Azure VM running Windows 10 for MSIX packaging
1. Generate a signing certificate
1. Download software to package
1. Install the MSIX Packaging Tool
1. Create an MSIX package

#### Task 1: Prepare for configuration of Windows Virtual Desktop session hosts

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. On the lab computer and, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to start the Windows Virtual Desktop session host Azure VMs you will be using in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**Note**: The command executes asynchronously (as determined by the -NoWait parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the Azure VMs are actually started. 

   >**Note**: Proceed directly to the next task without waiting for the Azure VMs to start.

#### Task 2: Deploy an Azure VM running Windows 10 by using an Azure Resource Manager QuickStart template

1. From your lab computer, in the web browser window displaying the Azure portal, in the toolbar of the Cloud Shell pane, select the **Upload/Download files** icon, in the drop-down menu select **Upload**, and upload the files **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** and **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json** into the Cloud Shell home directory.
1. From the PowerShell session in the Cloud Shell pane, run the following to deploy an Azure VM running Windows 10 that you will use for creating MSIX packages to and to join it to the Azure AD DS domain:

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This might take about than 10 minutes. 

#### Task 3: Prepare the Azure VM running Windows 10 for MSIX packaging

1. From your lab computer, in the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, in the list of virtual machines, select the **az140-cl-vm42** entry. This will open the **az140-cl-vm42** blade.
1. On the **az140-cl-vm42** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az140-cl-vm42 \| Connect** blade, and then select **Download RDP File**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\\wvdadmin1**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop session to **az140-cl-vm42**, start **Windows PowerShell ISE** as administrator, from the **Administrator: Windows PowerShell ISE** console, run the following to prepare the operating system for MSIX packaging:

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   ```

   > **Note**: The last of these registry changes disables User Access Control. This is technically not required but simplifies the process illustrated in this lab.

#### Task 4: Generate a signing certificate

> **Note**: In this lab, you will use a self-signed certificate. In a production environment, you should be using a certificate issued by either a public Certification Authority or an internal one, depending on the intended use.

1. Within the Remote Desktop session to **az140-cl-vm42**, start **Windows PowerShell ISE** as administrator, from the **Administrator: Windows PowerShell ISE** console, run the following to generate a self-signed certificate with the Common Name attribute set to **Adatum**, and store the certificate in the **Personal** folder of the **Local Machine** certificate store:

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to start the **Certificates** console targeting the Local Machine certificate store:

   ```powershell
   certlm.msc
   ```

1. In the **Certificates** console pane, expand the **Personal** folder, select the **Certificates** subfolder, right-click the **Adatum** certificate, in the right-click menu, select **All Tasks** followed by **Export**. This will launch the **Certificate Export Wizard**. 
1. On the **Welcome to the Certificate Export Wizard** page of the **Certificate Export Wizard**, select **Next**.
1. On the **Export Private Key** page of the **Certificate Export Wizard**, select the option **Yes, export the private key** option and select **Next**.
1. On the **Export File Format** page of the **Certificate Export Wizard**, select the checkbox **Export all extended properties**, clear the checkbox **Enable certificate privacy**, and select **Next**.
1. On the **Security** page of the **Certificate Export Wizard**, select the **Password** checkbox, in the textboxes below, type **Pa55w.rd1234**, and select **Next**.
1. On the **File to Export** page of the **Certificate Export Wizard**, in the **File name** textbox, select **Browse**, in the **Save As** dialog box, navigate to the **C:\\Allfiles\\Labs\\04** folder (create the folder if needed), in the **File name** textbox, type **adatum.pfx**, and select **Save**.
1. Back on the **File to Export** page of the **Certificate Export Wizard**, ensure that the textbox contains the entry **C:\\Allfiles\\Labs\\04\\adatum.pfx**, and select **Next**.
1. On the **Completing Certificate Export Wizard** page of the **Certificate Export Wizard**, select **Finish**, and select **OK** to acknowledge successful export. 

   > **Note**: Since you are using a self-signed certificate, you need to install it in the **Trusted People** certificate store on the target session hosts.

1. from the **Administrator: Windows PowerShell ISE** console, run the following to install the newly generated certificate in the **Trusted People** certificate store on the target session hosts.

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   ForEach ($wvdhost in $wvdhosts){
      $localPath = 'C:\Allfiles\Labs\04\'
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### Task 5: Download software to package

1. Within the Remote Desktop session to **az140-cl-vm42**, start **Microsoft Edge** and browse to **https://github.com/microsoft/XmlNotepad**.
1. On the **microsoft/XmlNotepad** **readme.md** page, select the download link for [Standalone downloadable installer](http://www.lovettsoftware.com/downloads/xmlnotepad/xmlnotepadsetup.zip) and download the compressed installation files.
1. Within the Remote Desktop session to **az140-cl-vm42**, start File Explorer, navigate to the **Downloads** folder, open the compressed file, copy its content, create a folder **C:\\AllFiles\\Labs\\04\\**, and paste the copied content into the newly created folder. 

#### Task 6: Install the MSIX Packaging Tool

1. Within the Remote Desktop session to **az140-cl-vm42**, start the **Microsoft Store** app.
1. In the **Microsoft Store** app, search for and select **MSIX Packaging Tool**, on the **MSIX Packaging Tool** page, select **Get**.
1. When prompted, skip signing in, wait for the installation to complete, select **Launch** and, in the **Send diagnostic data** dialog box, select **Decline**, 

#### Task 7: Create an MSIX package

1. Within the Remote Desktop session to **az140-cl-vm42**, switch to the **Administrator: Windows PowerShell ISE** window and, from the **Administrator: Windows PowerShell ISE** script pane, run the following to disable the Windows Search service:

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to create the folder that will host the MSIX package:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to remove the Zone.Identifier alternate data stream from the extracted installer files, which has a value of "3" to indicate that they were downloaded from the Internet:

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, switch to the **MSIX Packaging Tool** interface, on the **Select task** page, select **Application package - Create your app package** entry. This will start the **Create new package** wizard.
1. On the **Select environment** page of the **Create new package** wizard, ensure that the **Create package on this computer** option is selected, select **Next**, and wait for the installation of the **MSIX Packaging Tool Driver**.
1. On the **Prepare computer** page of the **Create new package** wizard, review the recommendations. If there is a pending reboot, restart the operating system, sign in back by using the **ADATUM\wvdadmin1** account, and restart the **MSIX Packaging Tool** before you proceed. 

   >**Note**: MSIX Packaging Tool disables temporarily Windows Update and Windows Search. In this case, the Windows Search service is already disabled. 

1. On the **Prepare computer** page of the **Create new package** wizard, click **Next**.
1. On the **Select installer** page of the **Create new package** wizard, next to the **Choose the installer you want to package** text box, select **Browse**, in the **Open** dialog box, browse to the **C:\\AllFiles\\Labs\\04** folder, select **XmlNotepadSetup.msi**, and click **Open**, 
1. On the **Select installer** page of the **Create new package** wizard, in the **Signing preference** drop-down list, select the **Sign with a certificate (.pfx)** entry, next to the **Browse for certificate** textbox, select **Browse**, in the **Open** dialog box, navigate to the **C:\\AllFiles\\Labs\\04** folder, select the **adatum.pfx** file, click **Open**, in the **Password** text box, type **Pa55w.rd1234**, and select **Next**.
1. On the **Package information** page of the **Create new package** wizard, review the package information, validate that the publisher name is set to **CN=Adatum**, and select **Next**. This will trigger installation of the downloaded software.
1. In the **XMLNotepad Setup** window, accept the terms in the License Agreement and select **Install** and, once the installation completes, select the **Launch XML Notepad** checkbox and select **Finish**.
1. Verify that XML Notepad is running, close it, switch back to the **Create new package** wizard in the **MSIX Packaging Tool** window, and select **Next**.

   > **Note**: In this case, restart is not required to complete the installation.

1. On the **First launch tasks** page of the **Create new package** wizard, review the provided information and select **Next**.
1. When prompted **Are you done?**, select **Yes, move on**.
1. On the **Services report** page of the **Create new package** wizard, verify that no services are listed and select **Next**.
1. On the **Create package** page of the **Create new package** wizard, in the **Save location** textbox, type **C:\\Allfiles\\Labs\\04\\XmlNotepad** and click **Create**.
1. In the **Package successfully created** dialog box, note the location of the saved package and select **Close**.
1. Switch to the File Explorer window, navigate to the **C:\\Allfiles\\Labs\\04\\XmlNotepad** folder and verify that it contains the *.msix and *.xml files. 


### Exercise 2: Implement MSIX app attach container for Windows Virtual Desktop in Azure AD DS environment

The main tasks for this exercise are as follows:

1. Enable Hyper-V on the Azure VMs running Window 10 Enterprise Edition
1. Create an app attach container

#### Task 1: Enable Hyper-V on the Azure VMs running Window 10 Enterprise Edition

1. Within the Remote Desktop session to **az140-cl-vm42**, start **Windows PowerShell ISE** as administrator, from the **Administrator: Windows PowerShell ISE** console, run the following to prepare the target Windows Virtual Desktop hosts for MSIX app attach: 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to install Hyper-V and its management tools, including the Hyper-V PowerShell module on the Windows Virtual Desktop hosts:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. Following the installation of the Hyper-V components on each host, type **Y** and press the **Enter** key to restart the target operating system.
1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to install Hyper-V and its management tools, including the Hyper-V PowerShell module on the local computer:

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Once the installation of the Hyper-V components completes, type **Y** and press the **Enter** key to restart the operating system. Following the restart, sign in back by using the **ADATUM\wvdadmin1** account with the **Pa55w.rd1234** password.

#### Task 2: Create an app attach container

1. Within the Remote Desktop session to **az140-cl-vm42**, start **Microsoft Edge**, browse to **https://aka.ms/msixmgr** and, when prompted whether to open or save **msixmgr.zip** file, click **Save**. This will download the MSIX mgr tool archive into the **Downloads** folder.
1. In File Explorer, navigate to the **Downloads** folder, open the compressed file and copy the **x64** folder to the **C:\\AllFiles\\Labs\\04** folder. 
1. Within the Remote Desktop session to **az140-cl-vm42**, start **Windows PowerShell ISE** as administrator and, from the **Administrator: Windows PowerShell ISE**  script pane, run the following to create the VHD file that will serve as the app attach container:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to mount the newly created VHD file:

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to initialize the disk, create a new partition, format it, and assign to it the first available drive letter:

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create a folder structure that will host the MSIX files and unpack into it the MSIX package you created in the previous task:

   ```powershell
   $appName = 'XmlNotepad'
   $msixPackage = Get-ChildItem -Path "C:\AllFiles\Labs\04\$appName" -Filter *.msix -File 
   C:\AllFiles\Labs\04\x64\msixmgr.exe -Unpack -packagePath $msixPackage.FullName -destination "$($partition.DriveLetter):\Apps\$appName" -applyacls
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, in File Explorer, navigate to the **F:\\Apps\\XmlNoteppad** folder and review its content.
1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the GUID of volume hosting the unpacked MSIX package

   ```powershell
   $uniqueId = (Get-Volume -DriveLetter "$($partition.DriveLetter)").UniqueId
   $regex = [regex]"\{(.*)\}"
   [regex]::match($uniqueId, $regex).Groups[1].value
   ```

   > **Note**: Record the GUID value you identified. You will need it in the next exercise of this lab. 

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to unmount the VHD file that will serve as the app attach container:

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### Exercise 3: Implement MSIX app attach on Windows Virtual Desktop session hosts

The main tasks for this exercise are as follows:

1. Configure an Azure File share for MSIX app attach
1. Configure Active Directory groups containing Windows Virtual Desktop hosts
1. Set up the Azure Files share
1. Mount and register the MSIX App attach container on Windows Virtual Desktop session hosts
1. Publish MSIX apps to an application group
1. Validate the functionality of MSIX App attach

#### Task 1: Configure Azure File share for MSIX app attach

1. Within the Remote Desktop session to **az140-cl-vm42**, start Microsoft Edge in the InPrivate mode, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.

   > **Note**: Ensure to use the Microsoft Edge InPrivate mode.

1. Within the Remote Desktop session to **az140-cl-vm42**, in the Microsoft Edge window displaying the Azure portal, search for and select **Storage accounts** and, on the **Storage accounts** blade, select the storage account you configure to host user profiles.

   > **Note**: This part of the lab is contingent on completing the lab **Windows Virtual Desktop profile management (AD DS)** or **Windows Virtual Desktop profile management (Azure AD DS)**

   > **Note**: In production scenarios, you should consider using a separate storage account. This would require configuring that storage account for Azure AD DS authentication, which you already implemented for the storage account hosting user profiles. You are using the same storage account to minimize duplicate steps across individual labs.

1. On the storage account blade, in the vertical menu on the left side, in the **File services** section, select **File shares** and then select **+ File share**.
1. On the **New file share** blade, specify the following settings and select **Create** (leave other settings with their default values):

   |Setting|Value|
   |---|---|
   |Name|**az140-42-msixvhds**|


#### Task 2: Configure Active Directory groups containing Windows Virtual Desktop hosts

1. Switch to the lab computer, in the web browser displaying the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-dc-vm11**.
1. On the **az140-dc-vm11** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az140-dc-vm11 \| Connect** blade, in the **IP address** drop-down list, select the **Load balancer DNS name** entry, and then select **Download RDP File**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\\Student**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop session to **az140-dc-vm11**, start **Windows PowerShell ISE** as administrator.
1. From the **Administrator: Windows PowerShell ISE** script pane, run the following to create an AD DS group object that will be synchronized to the Azure AD tenant used in this lab:

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **Note**: You will use this group to grant Windows Virtual Desktop hosts permissions to the **az140-42-msixvhds** file share.

1. From the **Administrator: Windows PowerShell ISE** console, run the following to add members to the groups you created in the previous step:

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. From the **Administrator: Windows PowerShell ISE** script pane, run the following to restart the servers which are members of the 'az140-hosts-42-p1' group:

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | Restart-Computer
   ```

   > **Note**: This step ensures that the group membership change takes effect. 

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to identify the user principal name of the **aadsyncuser** Azure AD user account:

   ```powershell
   (Get-AzADUser -DisplayName 'aadsyncuser').UserPrincipalName
   ```
   > **Note**: Record the user principal name you identified in this step. You will need it later in this task.

1. Within the Remote Desktop session to **az140-dc-vm11**, in the **Start** menu, expand the **Azure AD Connect** folder and select **Azure AD Connect**.
1. On the **Welcome to Azure AD Connect** page of the **Microsoft Azure Active Directory Connect** window, select **Configure**.
1. On the **Additional tasks** page in the **Microsoft Azure Active Directory Connect** window, select **Customize synchronization options** and select **Next**.
1. On the **Connect to Azure AD** page in the **Microsoft Azure Active Directory Connect** window, authenticate by using the user principal name of the **aadsyncuser** user account you identified earlier in this task and the **Pa55w.rd1234** password.
1. On the **Connect your directories** page in the **Microsoft Azure Active Directory Connect** window, select **Next**.
1. On the **Domain and OU filtering** page in the **Microsoft Azure Active Directory Connect** window, ensure that the option **Sync selected domains and OUs** is selected, expand the **adatum.com node, select the checkbox next to the **WVDInfra** OU (leave any other selected checkboxes unchanged), and select **Next**.
1. On the **Optional features** page in the **Microsoft Azure Active Directory Connect** window, accept the default settings, and select **Next**.
1. On the **Ready to configure** page in the **Microsoft Azure Active Directory Connect** window, ensure that the checkbox **Start the synchronization process when configuration completes** is selected and select **Configure**.
1. Review the information on the **Configuration complete** page and select **Exit** to close the **Microsoft Azure Active Directory Connect** window.
1. Within the Remote Desktop session to **az140-dc-vm11**, start Internet Explorer and navigate to the [Azure portal](https://portal.azure.com). When prompted, sign in by using the Azure AD credentials of the user account with the Global Administrator role in the Azure AD tenant associated with the Azure subscription you are using in this lab.
1. Within the Remote Desktop session to **az140-dc-vm11**, in the Internet Explorer window displaying the Azure portal, search for and select **Azure Active Directory** to navigate to the Azure AD tenant associated with the Azure subscription you are using for this lab.
1. On the Azure Active Directory blade, in the vertical menu bar on the left side, in the **Manage** section, click **Groups**. 
1. On the **Groups | All groups** blade, in the list of groups, select the **az140-hosts-42-p1** entry.
1. On the **az140-hosts-42-p1** blade, in the vertical menu bar on the left side, in the **Manage** section, click **Members**.
1. On the **az140-hosts-42-p1 | Members** blade, verify that the list of **Direct members** include the three hosts of the Windows Virtual Desktop pool you added to the group earlier in this task.

#### Task 3: Set up the Azure Files share

1. On the lab computer, switch back to the Remote Desktop session to **az140-cl-vm42**.
1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** window, run the following to copy the VHD file you created in the previous exercise to the newly created Azure Files share:

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, switch to the Microsoft Edge window displaying the Azure portal, navigate back to the blade of the storage account containing the **az140-42-msixvhds** file share you created earlier in this exercise.
1. On the storage account blade, in the vertical menu on the left side, in the **File Service** section, click **File shares** and, in the list of file shares, click the **az140-42-msixvhds** entry.
1. On the **az140-42-msixvhds** blade, in the vertical menu on the left side, select **Access Control (IAM)**.
1. On the **az140-42-msixvhds \| Access Control (IAM)** blade of the storage account, select **+ Add** and, in the drop-down menu, select **Add role assignment**, 
1. On the **Add role assignment** blade, specify the following settings and select **Save**:

   |Setting|Value|
   |---|---|
   |Role|**Storage File Data SMB Share Elevated Contributor**|
   |Assign access to|**User, group, or service principal**|
   |Select|**az140-wvd-admins**|

   > **Note**: The **az140-wvd-admins** group contains the **wvdadmin1** user account, which you'll use to configure share permissions. 

1. Repeat the previous two steps to configure the following role assignments:

   |Setting|Value|
   |---|---|
   |Role|**Storage File Data SMB Share Elevated Contributor**|
   |Assign access to|**User, group, or service principal**|
   |Select|**az140-hosts-42-p1**|

   |Setting|Value|
   |---|---|
   |Role|**Storage File Data SMB Share Reader**|
   |Assign access to|**User, group, or service principal**|
   |Select|**az140-wvd-users**|

   > **Note**: Windows Virtual Desktop users and hosts need at least read access to the file share.

1. Back on the **az140-42-msixvhds \| Access Control (IAM)** blade, in the vertical menu on the left side, in the **Settings** section, click **Properties**.
1. On the **az140-42-msixvhds \| Properties** blade, record the value of the **URL** textbox, excluding the **https://** prefix and, within the recorded URL, replace each instance of a forward slash (**/**) with a backslash (**\**). You will need it later in this task.
1. On the **az140-42-msixvhds \| Properties** blade, in the vertical menu on the left side, click **Overview** and, in the list of files on the right side, click the **XmlNotepad.vhd** entry. 
1. On the **File properties** blade, record the value of the **URL** textbox, excluding the **https://** prefix. You will need it in the next task.
1. Within the Remote Desktop session to **az140-cl-vm42**, start **Command Prompt** and, from the **Command Prompt** window, run the following to map a drive to the **az140-42-msixvhds** share (replace the `<storage-account-name>` placeholder with the name of the storage account) and verify that the command completes successfully:

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Command Prompt** window, run the following to grant the required NTFS permissions to the computer accounts of session hosts:

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**Note**: You could set these permissions by using File Explorer while signed in as **ADATUM\\wvdadmin1**. 

   >**Note**: Next you will validate the functionality of MSIX App attach

#### Task 4: Run the MSIX app attach staging script on Windows Virtual Desktop hosts

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to authenticate to the Azure subscription using in this lab:

   ```powershell
   Connect-AzAccount
   ```

1. When prompted, sign in with the **wvdadmin1** user principal name and **Pa55w.rd1234** as its password.
1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to identify the name of the Azure storage account hosting the file share containing the MSIX package:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName  
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to enable CredSSP authentication for PowerShell Remoting to the target Windows Remote Desktop hosts:

   ```powershell
   winrm qc
   Enable-WSManCredSSP -Role client -DelegateComputer * -Force

   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
        winrm qc
        Enable-PSRemoting -Force
        Enable-WSManCredSSP -Role server -Force
      } 
   }
   ```

   >**Note**: This is done to allow running the staging script remotely. Alternatively, you could run the script locally on each target host.

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to perform the MSIX app attach staging (replace the `<volume_guid>` placeholder with the volume GUID you identified in the previous exercise):

   ```powershell
   $username = 'ADATUM\wvdadmin1'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString -AsPlainText $cleartextPassword -Force
   $creds = New-Object System.Management.Automation.PSCredential -ArgumentList $username,$securePassword

   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -Authentication Credssp -Credential $creds -ScriptBlock {

         $vhdSrc = "\\$using:storageAccountName.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd"
         Mount-Diskimage -ImagePath $vhdSrc -NoDriveLetter -Access ReadOnly
         $volumeGuid = '<volume_guid>'
         $msixDest = "\\?\Volume{" + $volumeGuid + "}\"

         $parentFolder = '\Apps\XmlNotepad\'
         $msixJunction = 'C:\Allfiles\Labs\04\AppAttach\'
         $packageName = "XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8"

         If (!(Test-Path -Path $msixJunction)) {New-Item -ItemType Directory -Path $msixJunction}
         $msixJunction = $msixJunction + $packageName
         cmd.exe /c mklink /j $msixJunction $msixDest

         [Windows.Management.Deployment.PackageManager,Windows.Management.Deployment,ContentType=WindowsRuntime] | Out-Null
         Add-Type -AssemblyName System.Runtime.WindowsRuntime
         $asTask = ([System.WindowsRuntimeSystemExtensions].GetMethods() | Where { $_.ToString() -eq 'System.Threading.Tasks.Task`1[TResult] AsTask[TResult,TProgress](Windows.Foundation.IAsyncOperationWithProgress`2[TResult,TProgress])'})[0]
         $asTaskAsyncOperation = $asTask.MakeGenericMethod([Windows.Management.Deployment.DeploymentResult], [Windows.Management.Deployment.DeploymentProgress])
         $packageManager = [Windows.Management.Deployment.PackageManager]::new()
         $path = $msixJunction + $parentFolder + $packageName
         $path = ([System.Uri]$path).AbsoluteUri
         $asyncOperation = $packageManager.StagePackageAsync($path, $null, "StageInPlace")
         $task = $asTaskAsyncOperation.Invoke($null, @($asyncOperation))
         $task 
      }
   }
    ```

#### Task 5: Initiate a Remote Desktop session to one of the pool hosts

1. Within the Remote Desktop session to **az140-cl-vm42**, start Microsoft Edge and navigate to [Windows Desktop client download page](https://go.microsoft.com/fwlink/?linkid=2068602) and, when prompted, select **Run** to start its installation. On the **Installation Scope** page of the **Remote Desktop Setup** wizard, select the option **Install for all users of this machine** and click **Install**. 
1. Once the installation completes, ensure that the **Launch Remote Desktop when setup exits** checkbox is selected and click **Finish** to start the Remote Desktop client.
1. In the **Remote Desktop** client window, select **Subscribe** and, when prompted, sign in with the **aduser1** user principal name and **Pa55w.rd1234** as its password.
1. In the **Remote Desktop** client window, within the **az140-21-ws1** section, double-click **SessionDesktop** icon to open a Remote Desktop session to the host pool that is part of the **az140-21-ws1** workspace.

#### Task 6: Run the MSIX app attach registration script on Windows Virtual Desktop hosts

1. Within the Remote Desktop session from **az140-cl-vm42** to a host pool in the **az140-21-ws1** workspace, while signed in as **ADATUM\aduser1**, start **Windows PowerShell ISE** and, from the **Windows PowerShell ISE** console, run the following to perform the MSIX app attach registration:

   ```powershell
   $packageName = 'XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8'
   $path = 'C:\Program Files\WindowsApps\' + $packageName + '\AppxManifest.xml'
   Add-AppxPackage -Path $path -DisableDevelopmentMode -Register
   ```

1. Within the Remote Desktop session from **az140-cl-vm42** to a host pool in the **az140-21-ws1** workspace, while signed in as **ADATUM\aduser1**, click **Start**, in the **Start** menu, click **XML Notepad** and verify that it successfully launches the XML Notepad app.
1. Close XML Notepad and the **Windows PowerShell ISE** window, right-click **Start**, in the right-click menu, select **Shut down or sign out** and, in the cascading menu, select **Sign out**.

#### Task 7: Run the MSIX app attach deregistration script

1. Back within the Remote Desktop session to **az140-cl-vm42**, switch to the **Administrator: Windows PowerShell ISE** window and, from the **Administrator: Windows PowerShell ISE** script pane, run the following to perform the MSIX app deregistration on all hosts in the **az140-21-hp1** host pool:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -Authentication Credssp -Credential $creds -ScriptBlock {
         $packageName = "XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8"
         Remove-AppxPackage -AllUsers -Package $packageName
      }
   }
    ```

1. Verify that the script completed successfully.

#### Task 8: Run the MSIX app attach destaging script

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** console, run the following to identify the name of the Azure storage account hosting the file share containing the MSIX package:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName  
   ```

1. Within the Remote Desktop session to **az140-cl-vm42**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to perform the MSIX app destaging on all hosts in the **az140-21-hp1** host pool:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -Authentication Credssp -Credential $creds -ScriptBlock {
         $vhdSrc = "\\$using:storageAccountName.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd"
         $packageName = "XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8"
         $msixJunction = 'C:\Allfiles\Labs\04\AppAttach'
         Remove-Item "$msixJunction\$packageName" -Recurse -Force -Verbose
         Dismount-DiskImage -ImagePath $vhdSrc -Confirm:$false
      }
   }
    ```

1. Verify that the script completed successfully.


### Exercise 4: Stop and deallocate Azure VMs provisioned and used in the lab

The main tasks for this exercise are as follows:

1. Stop and deallocate Azure VMs provisioned and used in the lab

>**Note**: In this exercise, you will deallocate the Azure VMs provisioned and used in this lab to minimize the corresponding compute charges

#### Task 1: Deallocate Azure VMs provisioned and used in the lab

1. Switch to the lab computer and, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to list all Azure VMs created and used in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to stop and deallocate all Azure VMs you created and used in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Note**: The command executes asynchronously (as determined by the -NoWait parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the Azure VMs are actually stopped and deallocated.
