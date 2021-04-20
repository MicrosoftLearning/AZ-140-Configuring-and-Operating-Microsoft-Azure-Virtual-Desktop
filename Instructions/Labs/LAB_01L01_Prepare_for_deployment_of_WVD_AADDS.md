---
lab:
    title: 'Lab: Prepare for deployment of Azure Windows Virtual Desktop (Azure AD DS)'
    module: 'Module 1: Plan a WVD Architecture'
---

# Lab - Prepare for deployment of Azure Windows Virtual Desktop (Azure AD DS)
# Student lab manual

## Lab dependencies

- An Azure subscription
- A Microsoft account or an Azure AD account with the Global Administrator role in the Azure AD tenant associated with the Azure subscription and with the Owner or Contributor role in the Azure subscription

> **Note**: At the time of authoring this course, the MSIX app attach functionality for Windows Virtual Desktop is in public preview. If you intend to run the lab that involves the use of MSIX app attach included in this course, you need to submit a request via on [online form](https://aka.ms/enablemsixappattach) to enable MSIX app attach in your subscription. The approval and processing of requests can take up to 24 hours during business days. You'll receive an email confirmation once your request has been accepted and completed.

## Estimated Time

150 minutes

>**Note**: Provisioning of an Azure AD DS takes involves about 90-minute wait time.

## Lab scenario

You need to prepare for deployment of Azure Windows Virtual Desktop in an Azure Active Directory Domain Services (Azure AD DS) environment

## Objectives
  
After completing this lab, you will be able to:

- Implement an Azure AD DS domain
- Configure the Azure AD DS domain environment

## Lab files

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## Instructions

### Exercise 0: Increase the number of vCPU quotas

The main tasks for this exercise are as follows:

1. Identify current vCPU usage
1. Request vCPU quota increase

#### Task 1: Identify current vCPU usage

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. In the Azure portal, open **Cloud Shell** pane by selecting the toolbar icon directly to the right of the search textbox.
1. If prompted to select either **Bash** or **PowerShell**, select **PowerShell**. 

   >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1. In the Azure portal, in the PowerShell session of the **Cloud Shell**, run the following to identify the current usage of vCPUs and the corresponding limits for the **StandardDSv3Family** and **StandardBSFamily** Azure VMs (replace the `<Azure_region>` placeholder with the name of the Azure region that you intend to use for this lab, such as, for example, `eastus`):

   ```powershell
   $location = '<Azure_region>'
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

   > **Note**: To identify the names of Azure regions, in the **Cloud Shell**, at the PowerShell prompt, run `(Get-AzLocation).Location`.
   
1. Review the output of the command executed in the previous step and ensure that you have at least **20** available vCPUs in both the **Standard DSv3 Family** and **StandardBSFamily** of Azure VMs in the target Azure region. If that's already the case, proceed directly to the next exercise. Otherwise, proceed to the next task of this exercise. 

#### Task 2: Request vCPU quota increase

1. In the Azure portal, search for and select **Subscriptions** and, from the **Subscriptions** blade, select the entry representing the Azure subscription you intend to use for this lab.
1. In the Azure portal, on the subscription blade, in the vertical menu on the left side, in the **Settings** section, select **Usage + quotas**. 
1. On the subscription's **Usage + quotas** blade, select **Request Increase**.
1. On the **Basics** tab of the **New support request** blade, specify the following and select **Next: Solutions >**:

   |Setting|Value|
   |---|---|
   |Issue type|**Service and subscription limits (quotas)**|
   |Subscription|the name of the Azure subscription you will be using in this lab|
   |Quota type|**Compute-VM (cores-vCPUs) subscription limit increases**|
   |Support plan|the name of the support plan associated with the target subscription|

1. On the **Details** tab of the **New support request** blade, select the **Provide details** link.
1. On the **Quota details** tab of the **New support request** blade, specify the following and select **Save and continue**:

   |Setting|Value|
   |---|---|
   |Deployment model|**Resource Manager**|
   |Location|the name of the Azure region you intend to use in this lab|
   |Types|**Standard**|
   |Standard|**BS Series**|
   |New vCPU Limit|the new limit|
   |Standard|**DSv3 Series**|
   |New vCPU Limit|the new limit|

   >**Note**: The use of **BS Series** Azure VMs is in this case intended to minimize the cost of running the lab environment. It is not meant to represent the intended usage of the **BS Series** Azure VMs in the Windows Virtual Desktop scenarios.

1. Back on the **Details** tab of the **New support request** blade, specify the following and select **Next: Review + create >**:

   |Setting|Value|
   |---|---|
   |Severity|**C - Minimal impact**|
   |Preferred contact method|choose your preferred option and provide your contact details|
    
1. On the **Review + create** tab of the **New support request** blade, select **Create**.

   > **Note**: Quota increase requests within this range of vCPUs are typically completed within a few hours.


### Exercise 1: Implement an Azure Active Directory Domain Services (AD DS) domain

The main tasks for this exercise are as follows:

1. Create and configure an Azure AD user account for administration of Azure AD DS domain
1. Deploy an Azure AD DS instance by using the Azure portal
1. Configure the network and identity settings of the Azure AD DS deployment

#### Task 1: Create and configure an Azure AD user account for administration of Azure AD DS domain

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab and the Global Administrator role in the Azure AD tenant associated with the Azure subscription.
1. In the Azure portal, open **Cloud Shell** pane by selecting on the toolbar icon directly to the right of the search textbox.
1. If prompted to select either **Bash** or **PowerShell**, select **PowerShell**. 

   >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1. From the Cloud Shell pane, run the following to sign in to your Azure AD tenant:

   ```powershell
   Connect-AzureAD
   ```

1. From the Cloud Shell pane, run the following to retrieve the primary DNS domain name of the Azure AD tenant associated with your Azure subscription:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. From the Cloud Shell pane, run the following to create a new Azure AD user:

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = 'Pa55w.rd1234'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   ```

1. From the Cloud Shell pane, run the following to assign the Global Administrator role to the newly created Azure AD user:

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **Note**: Azure AD PowerShell module refers to the Global Administrator role as Company Administrator.

1. From the Cloud Shell pane, run the following to identify the user principal name of the newly created Azure AD user:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **Note**: Record the user principal name. You will need it later in this exercise. 

1. Close the Cloud Shell pane.
1. Within the Azure portal, search for and select **Subscriptions** and, from the **Subscriptions** blade, select the Azure subscription you are using in this lab. 
1. On the blade displaying properties of your Azure subscription, select **Access control (IAM)**, select **+ Add**, and, in the drop-down list, select **Add role assignment**. 
1. On the **Add role assignment** blade, specify the following settings and select **Save**:

   |Setting|Value|
   |---|---|
   |Role|**Owner**|
   |Assign access to|**User, group, or service principal**|
   |Select|**aadadmin1**|

   > **Note**: You will use the **aadadmin1** account to manage your Azure subscription and the corresponding Azure AD tenant from an Azure AD DS joined Windows 10 Azure VM later in the lab. 


#### Task 2: Deploy an Azure AD DS instance by using the Azure portal

1. From your lab computer, in the Azure portal, search for and select **Azure AD Domain Services** and, from the **Azure AD Domain Services** blade, select **+ Add**. This will open the **Create Azure AD Domain Services** blade.
1. On the **Basics** tab of the **Create Azure AD Domain Services** blade, specify the following settings and select **Next** (leave others with their existing values):

   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you are using in this lab|
   |Resource group|the name of a new resource group **az140-11a-RG**|
   |Domain name|**adatum.com**|
   |Region|the name of the region where you want to host your WVD deployment|
   |SKU|**Standard**|
   |Forest type|**User**|

   > **Note**: While this is technically not required, in general, you should assign an Azure AD DS domain name different from any existing Azure or on-premises DNS name space.

1. On the **Networking** tab of the **Create Azure AD Domain Services** blade, next to the **Virtual network** drop-down list, select **Create new**.
1. On the **Create virtual network** blade, specify the following settings and select **OK** (leave others with their existing values):

   |Setting|Value|
   |---|---|
   |Name|**az140-aadds-vnet11**|
   |Address range|**10.10.0.0/16**|
   |Subnet name|**aadds-Subnet**|
   |Subnet name|**10.10.0.0/24**|

1. Back on the **Networking** tab of the **Create virtual network** blade, select **Next** (leave others with their existing values).
1. On the **Administration** tab of the **Create Azure AD Domain Services** blade, accept the default settings and select **Next**.
1. On the **Synchronization** tab of the **Create Azure AD Domain Services** blade, ensure that **All** is selected and then select **Next**.
1. On the **Review + create** tab of the **Create Azure AD Domain Services** blade, select **Create**. 
1. Review the notification regarding settings that you will not be able to change following creation of the Azure AD DS domain and select **OK**.

   >**Note**: The settings that you will not be able to change following provisioning of an Azure AD DS domain include its DNS name, its Azure subscription, its resource group, the virtual network and subnet hosting its domain controllers, and the forest type.

   > **Note**: Wait for the deployment to complete before you proceed to the next exercise. This might take about 90 minutes. 

#### Task 3: Configure the network and identity settings of the Azure AD DS deployment

1. From your lab computer, in the Azure portal, search for and select **Azure AD Domain Services** and, from the **Azure AD Domain Services** blade, select the **adatum.com** entry to navigate to the newly provisioned Azure AD DS instance. 
1. On the **adatum.com** blade of the Azure AD DS instance, click the warning stating **Network configuration issues detected. See detailed diagnosis**. 
1. On the **adatum.com | Configuration diagnostics (preview)** blade, click **Run**.
1. In the **Validation** section, expand the **DNS records** pane and click **Fix**.
1. On the **DNS records** blade, click **Fix** again.
1. Navigate back to the **adatum.com** blade of the Azure AD DS instance and, in the **Required configuration steps** section, review the information regarding the Azure AD DS password hash synchronization. 

   > **Note**: Any existing cloud-only users that need to be able to access Azure AD DS domain computers and their resources must either change their passwords or have them reset. This applies to the **aadadmin1** account you created earlier in this lab.

1. From your lab computer, in the Azure portal, open a **PowerShell** session in the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to identify the objectID attribute of the Azure AD **aadmin1** user account:

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to reset the password of the **aadmin1** user account, which objectId you identified in the previous step:

   ```powershell
   $password = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **Note**: In real-world scenarios you would typically set the value of the **-ForceChangePasswordNextLogin** to $true. We chose **$false** in this case to simplify the lab steps. 


### Exercise 2: Configure the Azure AD DS domain environment
  
The main tasks for this exercise are as follows:

1. Deploy an Azure VM running Windows 10 by using an Azure Resource Manager QuickStart template
1. Review the default configuration of the Azure AD DS domain
1. Create AD DS users and groups that will be synchronized to Azure AD DS

#### Task 1: Deploy an Azure VM running Windows 10 by using an Azure Resource Manager QuickStart template

1. From your lab computer, in the Azure portal, from the PowerShell session in the Cloud Shell pane, run the following to add a subnet named **cl-Subnet** to the virtual network named **az140-aadds-vnet11** you created in the previous task:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. In the Azure portal, in the toolbar of the Cloud Shell pane, select the **Upload/Download files** icon, in the drop-down menu select **Upload**, and upload the files **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** and **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** into the Cloud Shell home directory.
1. From the PowerShell session in the Cloud Shell pane, run the following to deploy an Azure VM running Windows 10 that will serve as a Windows Virtual Desktop client and join it to the Azure AD DS domain:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **Note**: The deployment might take about 10 minutes. Wait for the deployment to complete before you proceed to the next task. 


#### Task 2: Review the default configuration of the Azure AD DS domain

> **Note**: Before you can sign in to the newly Azure AD DS joined computer, you need to add the user account you intend to sign in with to the **AAD DC Administrators** Azure AD group. This Azure AD group is created automatically in the Azure AD tenant associated with the Azure subscription where you provisioned the Azure AD DS instance.

> **Note**: You have the option of populating this group with existing Azure AD user accounts when you provision an Azure AD DS instance.

1. From your lab computer, in the Azure portal, from the Cloud Shell pane, run the following to add the **aadadmin1** Azure AD user account to the **AAD DC Administrators** Azure AD group:

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1  Close the Cloud Shell pane.
1. From your lab computer, in the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select the **az140-cl-vm11a** entry. This will open the **az140-cl-vm11a** blade.
1. On the **az140-cl-vm11a** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az140-cl-vm11a \| Connect** blade, in the **IP address** drop-down list, select the **Public IP address** entry, and then select **Download RDP File**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\\aadadmin1**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop to the **az140-cl-vm11a** Azure VM, start **Windows PowerShell ISE** as Administrator and, from the **Administrator: Windows PowerShell ISE** script pane, run the following to install the Active Directory and DNS-related Remote Server Administration Tools:

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **Note**: Wait for the installation to complete before you proceed to the next step. This might take about 2 minutes.

1. Within the Remote Desktop to the **az140-cl-vm11a** Azure VM, in the **Start** menu, navigate to the **Windows Administrative Tools** folder, expand it, and, from the list of tools, start **Active Directory Users and Computers**. 
1. In the **Active Directory Users and Computers** console, review the default hierarchy, including the **AADDC Computers** and **AADDC Users** organizational units. Note that the former includes the **az140-cl-vm11a** computer account and the latter includes the user accounts synchronized from the Azure AD tenant associated with the Azure subscription hosting the deployment of Azure AD DS instance. The **AADDC Users** organizational unit also includes the **AAD DC Administrators** group synchronized from the same Azure AD tenant, along with its group membership. This membership cannot be modified directly within the Azure AD DS domain, but instead, you have to manage it within the Azure AD DS tenant. Any changes are automatically synchronized with the replica of the group hosted in the Azure AD DS domain. 

   > **Note**: Currently, the group includes only the **aadadmin1** user account.

1. In the **Active Directory Users and Computers** console, in the **AADDC Users** OU, select the **aadadmin1** user account, display its **Properties** dialog box, switch to the **Accounts** tab, and note that the user principal name suffix matches the primary Azure AD DNS domain name and is not modifiable. 
1. In the **Active Directory Users and Computers** console, review the content of the **Domain Controllers** organizational unit and note that it includes computer accounts of two domain controllers with randomly generated names. 

#### Task 3: Create AD DS users and groups that will be synchronized to Azure AD DS

1. Within the Remote Desktop to the **az140-cl-vm11a** Azure VM, start Microsoft Edge, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing user principal name of the **aadmin1** user account with **Pa55w.rd1234** as its password.
1. In the Azure portal, open a PowerShell session in the **Cloud Shell**.
1. When prompted to select either **Bash** or **PowerShell**, select **PowerShell**. 

   >**Note**: Since this is the first time you are starting **Cloud Shell** by using the **aadadmin1** user account, you will need to configure its Cloud Shell home directory. When presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1. From the PowerShell session in the Cloud Shell pane, run the following to sign in to authenticate to your Azure AD tenant:

   ```powershell
   Connect-AzureAD
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to retrieve the primary DNS domain name of the Azure AD tenant associated with your Azure subscription:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to create the Azure AD user accounts you will use in the upcoming labs:

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = 'Pa55w.rd1234'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to create an Azure AD group named **az140-wvd-aadmins** and add to it the **aadadmin1** user account:

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. From the Cloud Shell pane, repeat the previous step to create Azure AD groups for Windows Virtual Desktop users that you will use in the upcoming labs and add to them previously created Azure AD user accounts:

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. Close the Cloud Shell pane.
1. Within the Remote Desktop to the **az140-cl-vm11a** Azure VM, in the Microsoft Edge window displaying the Azure portal, search for and select **Azure Active Directory** blade, on your Azure AD tenant blade, in the vertical menu bar on the left side, in the **Manage* section, select **Users** and, on the **Users \| All users** blade, verify that new user accounts have been created.
1. Navigate back to the Azure AD tenant blade, in the vertical menu bar on the left side, in the **Manage** section, select **Groups** and, on the **Groups \| All groups** blade, verify that new group accounts have been created.
1. Within the Remote Desktop to the **az140-cl-vm11a** Azure VM, switch to the **Active Directory Users and Computers** console, in the **Active Directory Users and Computers** console, navigate to the **AADDC Users** OU, and verify that it contains the same user and group accounts.

   >**Note**: You might have to refresh the view of the console.
