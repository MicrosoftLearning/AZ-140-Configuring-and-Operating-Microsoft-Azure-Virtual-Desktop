---
lab:
    title: 'Lab: Deploy and manage host pools and hosts by using PowerShell'
    module: 'Module 2: Implement a WVD Infrastructure'
---

# Lab - Deploy and manage host pools and hosts by using PowerShell
# Student lab manual

## Lab dependencies

- An Azure subscription you will be using in this lab.
- A Microsoft account or an Azure AD account with the Owner or Contributor role in the Azure subscription you will be using in this lab and with the Global Administrator role in the Azure AD tenant associated with that Azure subscription.
- The completed lab **Prepare for deployment of Azure Windows Virtual Desktop (AD DS)** or **Prepare for deployment of Azure Windows Virtual Desktop (Azure AD DS)**

## Estimated Time

60 minutes

## Lab scenario

You need to automate deployment of Windows Virtual Desktop host pools and hosts by using PowerShell in an Active Directory Domain Services (AD DS) environment.

## Objectives
  
After completing this lab, you will be able to:

- Deploy Azure Windows Virtual Desktop host pools and hosts by using PowerShell
- Add hosts to the Windows Virtual Desktop host pool by using PowerShell

## Lab files

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## Instructions

### Exercise 1: Implement Azure Windows Virtual Desktop host pools and session hosts by using PowerShell
  
The main tasks for this exercise are as follows:

1. Prepare for deployment of Windows Virtual Desktop host pool by using PowerShell
1. Create a Windows Virtual Desktop host pool by using PowerShell
1. Perform a template-based deployment of an Azure VM running Windows 10 Enterprise by using PowerShell
1. Add an Azure VM running Windows 10 Enterprise as a session host to the Windows Virtual Desktop host pool by using PowerShell
1. Verify the deployment of the Azure Windows Virtual Desktop session host

#### Task 1: Prepare for deployment of Windows Virtual Desktop host pool by using PowerShell

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. In the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-dc-vm11**.
1. On the **az140-dc-vm11** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az140-dc-vm11 \| Connect** blade, in the **IP address** drop-down list, select the **Load balancer DNS name** entry, and then select **Download RDP File**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\\Student**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop session to **az140-dc-vm11**, start **Windows PowerShell ISE** as administrator.
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to identify the distinguished name of the organizational unit named **WVDInfra** that will host the computer objects of the Windows Virtual Desktop pool session hosts:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the UPN suffix of the **ADATUM\\Student** account that you will use to join the Windows Virtual Desktop hosts to the AD DS domain (**student@adatum.com**):

   ```powershell
   (Get-ADUser -Filter {sAMAccountName -eq 'student'} -Properties userPrincipalName).userPrincipalName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to install the DesktopVirtualization PowerShell module (when prompted, click **Yes to All**):

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **Note**: Ignore any warnings regarding existing PowerShell modules in use.

1. Within the Remote Desktop session to **az140-dc-vm11**, start Microsoft Edge and navigate to the [Azure portal](https://portal.azure.com). If prompted, sign in by using the Azure AD credentials of the user account with the Owner role in the subscription you are using in this lab.
1. Within the Remote Desktop session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Virtual networks** and, on the **Virtual networks** blade, select **az140-adds-vnet11**. 
1. On the **az140-adds-vnet11** blade, select **Subnets**, on the **Subnets** blade, select **+ Subnet**, on the **Add subnet** blade, specify the following settings (leave all other settings with their default values) and click **Save**:

   |Setting|Value|
   |---|---|
   |Name|**hp3-Subnet**|
   |Subnet address range|**10.0.3.0/24**|

1. Within the Remote Desktop session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Network security groups** and, on the **Network security groups** blade, select the security group in the **az140-11-RG** resource group.
1. On the network security group blade, in the vertical menu on the left, in the **Settings** section, click **Properties**.
1. On the **Properties** blade, click the **Copy to clipboard** icon on the right side of the **Resource ID** textbox. 

   > **Note**: The value should resemble the format `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, although the subscription ID will differ. Record it since you will need it in the next task.

#### Task 2: Create a Windows Virtual Desktop host pool by using PowerShell

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to sign in to your Azure subscription:

   ```powershell
   Connect-AzAccount
   ```

1. When prompted, provide the credentials of the user account with the Owner role in the subscription you are using in this lab.
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the Azure region hosting the Azure virtual network **az140-adds-vnet11**:

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create a resource group that will host the host pool and its resources:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create an empty host pool:

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **Note**: The **New-AzWvdHostPool** cmdlet allows you to create a host pool, workspace, and the desktop app group, as well as to register the desktop app group with the workspace. You have the option of creating a new workspace or using an existing one.

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to retrieve the objectID attribute of the Azure AD group named **az140-wvd-pooled**:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to assign the Azure AD group named **az140-wvd-pooled** to the default desktop app group of the newly created host pool:

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### Task 3: Perform a template-based deployment of an Azure VM running Windows 10 Enterprise by using PowerShell

1. From your lab computer, use the Remote Desktop session to the **az140-dc-vm11** Azure VM to copy the lab files **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json** and **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json** to the **C:\\AllFiles\\Labs\\02** folder (create it if needed).
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to deploy an Azure VM running Windows 10 Enterprise (multi-session) that will serve as a Windows Virtual Desktop session host in the host pool you created in the previous task:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This might take about 5 minutes. 

   > **Note**: The deployment uses an Azure Resource Manager template to provision an Azure VM and applies a VM extension that automatically joins the operating system to the **adatum.com** AD DS domain.

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to verify that the third session host was successfully joined to the **adatum.com** AD DS domain:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### Task 4: Add an Azure VM running Windows 10 Enterprise as a host to the Windows Virtual Desktop host pool by using PowerShell

1. Within the Remote Desktop session to **az140-dc-vm11**, in the browser window displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, in the list of virtual machines, select **az140-24-p3-0**.
1. On the **az140-24-p3-0** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az140-24-p3-0 \| Connect** blade, in the **IP address** drop-down list, select the **Private IP address (10.0.3.4)** entry, and then select **Download RDP File**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\\Student**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop session to **az140-24-p3-0**, start **Windows PowerShell ISE** as administrator.
1. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create a folder that will host files required to add the newly deployed Azure VM as a session host to the host pool you provisioned earlier in this lab:

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

1. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to download the Windows Virtual Desktop Agent and Boot Loader installers, required to add the session host to the host pool:

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to install the latest version of the PowerShellGet module (select **Yes** when prompted for confirmation):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. From the **Administrator: Windows PowerShell ISE** console, run the following to install the latest version of the Az.DesktopVirtualization PowerShell module:

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -AllowClobber -Force
   Install-Module -Name Az -AllowClobber -Force
   ```

1. From the **Administrator: Windows PowerShell ISE** console, run the following to modify the PowerShell execution policy and sign in to your Azure subscription:

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser -Force
   Connect-AzAccount
   ```

1. When prompted, provide the credentials of the user account with the Owner role in the subscription you are using in this lab.
1. Within the Remote Desktopliveid session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** console, run the following to generate the token necessary to join new session hosts to the pool you provisioned earlier in this exercise:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **Note**: A registration token is required to authorize a session host to join the host pool. The value of token's expiration date must be between one hour and one month from the current date and time.

1. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** console, run the following to install the Windows Virtual Desktop Agent:

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. Within the Remote Desktop session to **az140-24-p3-0**, from the **Administrator: Windows PowerShell ISE** console, run the following to install the Windows Virtual Desktop Boot Loader:

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

#### Task 5: Verify the deployment of the Azure Windows Virtual Desktop host

1. Switch to the lab computer, in the web browser displaying the Azure portal, search for and select **Windows Virtual Desktop**, on the **Windows Virtual Desktop** blade, select **Host pools** and, on the **Windows Virtual Desktop \| Host pools** blade, select the entry **az140-24-hp3** representing the newly modified pool.
1. On the **az140-24-hp3** blade, in the vertical menu on the left side, in the **Manage** section, click **Session hosts**. 
1. On the **az140-24-hp3 \| Session hosts** blade, verify that the deployment includes a single host.

#### Task 6: Manage app groups using PowerShell

1. From the lab computer, switch to the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to create a Remote App group:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to list the **Start** menu apps on the pool's hosts and review the output:

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **Note**: For any application you want to publish, you should record the information included in the output, including such parameters as **FilePath**, **IconPath**, and **IconIndex**.

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to publish Microsoft Word:

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to publish Microsoft Word:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. Switch to the lab computer, in the web browser displaying the Azure portal, on the **az140-24-hp3 \| Session hosts** blade, in the vertical menu on the left side, in the **Manage** section, select **Application groups**.
1. On the **az140-24-hp3 \| Application groups** blade, in the list of application groups, select the **az140-24-hp3-Office365-RAG** entry.
1. On the **az140-24-hp3-Office365-RAG** blade, verify the configuration of the application group, including the applications and assignments.

### Exercise 2: Stop and deallocate Azure VMs provisioned in the lab

The main tasks for this exercise are as follows:

1. Stop and deallocate Azure VMs provisioned in the lab

>**Note**: In this exercise, you will deallocate the Azure VMs provisioned in this lab to minimize the corresponding compute charges

#### Task 1: Deallocate Azure VMs provisioned in the lab

1. Switch to the lab computer and, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to list all Azure VMs created in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to stop and deallocate all Azure VMs you created in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Note**: The command executes asynchronously (as determined by the -NoWait parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the Azure VMs are actually stopped and deallocated.
