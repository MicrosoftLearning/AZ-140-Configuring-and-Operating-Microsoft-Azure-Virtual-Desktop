---
lab:
    title: 'Lab: Deploy host pools and hosts by using Azure Resource Manager templates'
    module: 'Module 2: Implement a WVD Infrastructure'
---

# Lab - Deploy host pools and hosts by using Azure Resource Manager templates
# Student lab manual

## Lab dependencies

- An Azure subscription you will be using in this lab.
- A Microsoft account or an Azure AD account with the Owner or Contributor role in the Azure subscription you will be using in this lab and with the Global Administrator role in the Azure AD tenant associated with that Azure subscription.
- The completed lab **Prepare for deployment of Azure Virtual Desktop (AD DS)** or **Prepare for deployment of Azure Virtual Desktop (Azure AD DS)**
- The completed lab **Deploy host pools and session hosts by using the Azure portal (AD DS)**

## Estimated Time

45 minutes

## Lab scenario

You need to automate deployment of Azure Virtual Desktop host pools and hosts by using Azure Resource Manager templates.

## Objectives
  
After completing this lab, you will be able to:

- Deploy Azure Virtual Desktop host pools and hosts by using Azure Resource Manager templates

## Lab files

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## Instructions

### Exercise 1: Deploy Azure Virtual Desktop host pools and hosts by using Azure Resource Manager templates
  
The main tasks for this exercise are as follows:

1. Prepare for deployment of an Azure Virtual Desktop host pool by using an Azure Resource Manager template
1. Deploy an Azure Virtual Desktop host pool and hosts by using an Azure Resource Manager template
1. Verify deployment of the Azure Virtual Desktop host pool and hosts
1. Prepare for adding of hosts to the existing Azure Virtual Desktop host pool by using an Azure Resource Manager template
1. Add hosts to the existing Azure Virtual Desktop host pool by using an Azure Resource Manager template
1. Verify changes to the Azure Virtual Desktop host pool
1. Manage personal desktop assignments in the Azure Virtual Desktop host pool

#### Task 1: Prepare for deployment of an Azure Virtual Desktop host pool by using an Azure Resource Manager template

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. In the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-dc-vm11**.
1. On the **az140-dc-vm11** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az140-dc-vm11 \| Connect** blade, in the **IP address** drop-down list, select the **Load balancer DNS name** entry, and then select **Download RDP File**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\\Student**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop session to **az140-dc-vm11**, start **Windows PowerShell ISE** as administrator.
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to identify the distinguished name of the organizational unit named **WVDInfra** that will host the computer objects of the Azure Virtual Desktop pool hosts:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the user principal name attribute of the **ADATUM\\Student** account that you will use to join the Azure Virtual Desktop hosts to the AD DS domain (**student@adatum.com**):

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to identify the user principal name of the **ADATUM\\aduser7** and **ADATUM\\aduser8** accounts that you will use to test personal desktop assignments later in this lab:

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **Note**: Record all user principal name values you identified. You will need them later in this lab.

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to calculate the token expiration time necessary to perform a template-based deployment:

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **Note**: The value should resemble the format `2020-12-27T00:51:28.3008055Z`. Record it since you will need it in the next task.

   > **Note**: A registration token is required to authorize a host to join the pool. The value of token's expiration date must be between one hour and one month from the current date and time.

1. Within the Remote Desktop session to **az140-dc-vm11**, start Microsoft Edge and navigate to the [Azure portal](https://portal.azure.com). If prompted, sign in by using the Azure AD credentials of the user account with the Owner role in the subscription you are using in this lab.
1. Within the Remote Desktop session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Virtual networks** and, on the **Virtual networks** blade, select **az140-adds-vnet11**. 
1. On the **az140-adds-vnet11** blade, select **Subnets**, on the **Subnets** blade, select **+ Subnet**, on the **Add subnet** blade, specify the following settings (leave all other settings with their default values) and click **Save**:

   |Setting|Value|
   |---|---|
   |Name|**hp2-Subnet**|
   |Subnet address range|**10.0.2.0/24**|

1. Within the Remote Desktop session to **az140-dc-vm11**, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the Azure portal page to search for and navigate to **Network security groups** and, on the **Network security groups** blade, select the network security group in the **az140-11-RG** resource group.
1. On the network security group blade, in the vertical menu on the left, in the **Settings** section, click **Properties**.
1. On the **Properties** blade, click the **Copy to clipboard** icon on the right side of the **Resource ID** textbox. 

   > **Note**: The value should resemble the format `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, although the subscription ID will differ. Record it since you will need it in the next task.

#### Task 2: Deploy an Azure Virtual Desktop host pool and hosts by using an Azure Resource Manager template

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. From your lab computer, in the same web browser window, open another web browser tab and navigate to the GitHub Azure RDS templates repository page [ARM Template to Create and provision new Azure Virtual Desktop hostpool](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool). 
1. On the **ARM Template to Create and provision new Azure Virtual Desktop hostpool** page, select **Deploy to Azure**. This will automatically redirect the browser to the **Custom deployment** blade in the Azure portal.
1. On the **Custom deployment** blade, select **Edit parameters**.
1. On the **Edit parameters** blade, select **Load file**, in the **Open** dialog box, select **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json**, select **Open**, and then select **Save**. 
1. Back on the **Custom deployment** blade, specify the following settings (leave others with their existing values):

   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you are using in this lab|
   |Resource Group|the name of a new resource group **az140-23-RG**|
   |Location|the name of the Azure region into which you deployed Azure VMs hosting AD DS domain controllers in the lab **Prepare for deployment of Azure Virtual Desktop (AD DS)**|
   |Workspace location|the name of the same Azure region as the one set as the value of the **Location** parameters|
   |Workspace Resource Group|none, since, if null, its value will be automatically set to match the deployment target resource group|
   |All Application Group Reference|none, since there are no existing application groups in the target workspace (there is no workspace)|
   |Vm location|the name of the same Azure region as the one set as the value of the **Location** parameters|
   |Create Network Security Group|**false**|
   |Network Security Group Id|the value of the resourceID parameter of the existing network security group you identified in the previous task|
   |Token Expiration Time| the value of the token expiration time you calculated in the previous task|

   > **Note**: The deployment provisions a pool with personal desktop assignment type.

1. On the **Custom deployment** blade, select **Review + create** and select **Create**.

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This might take about 15 minutes. 

#### Task 3: Verify deployment of the Azure Virtual Desktop host pool and hosts

1. From your lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** blade, select **Host pools** and, on the **Azure Virtual Desktop \| Host pools** blade, select the entry **az140-23-hp2** representing the newly deployed pool.
1. On the **az140-23-hp2** blade, in the vertical menu on the left side, in the **Manage** section, click **Session hosts**. 
1. On the **az140-23-hp2 \| Session hosts** blade, verify that the deployment consists of two hosts.
1. On the **az140-23-hp2 \| Session hosts** blade, in the vertical menu on the left side, in the **Manage** section, click **Application groups**.
1. On the **az140-23-hp2 \| Application groups** blade, verify that the deployment includes the **Default Desktop** application group named **az140-23-hp2-DAG**.

#### Task 4: Prepare for adding of hosts to the existing Azure Virtual Desktop host pool by using an Azure Resource Manager template

1. From your lab computer, switch to the Remote Desktop session to **az140-dc-vm11**. 
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to generate the token necessary to join new hosts to the pool you provisioned earlier in this exercise:

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to retrieve the value of the token and paste it into Clipboard:

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **Note**: Record the value copied into Clipboard since you will need it in the next task. Make sure to that the value you are using includes a single line of text, without any line breaks. 

   > **Note**: A registration token is required to authorize a host to join the pool. The value of token's expiration date must be between one hour and one month from the current date and time.

#### Task 5: Add hosts to the existing Azure Virtual Desktop host pool by using an Azure Resource Manager template

1. From your lab computer, in the same web browser window, open another web browser tab and navigate to the GitHub Azure RDS templates repository page [ARM Template to Add sessionhosts to an existing Azure Virtual Desktop hostpool](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool). 
1. On the **ARM Template to Add sessionhosts to an existing Azure Virtual Desktop hostpool** page, select **Deploy to Azure**. This will automatically redirect the browser to the **Custom deployment** blade in the Azure portal.
1. On the **Custom deployment** blade, select **Edit parameters**.
1. On the **Edit parameters** blade, select **Load file**, in the **Open** dialog box, select **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json**, select **Open**, and then select **Save**. 
1. Back on the **Custom deployment** blade, specify the following settings (leave others with their existing values):

   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you are using in this lab|
   |Resource Group|**az140-23-RG**|
   |Hostpool Token|the value of the token you generated in the previous task|
   |Hostpool Location|the name of the Azure region into which you deployed the hostpool earlier in this lab|
   |Vm Administrator Account Username|**student**|
   |Vm Administrator Account Password|**Pa55w.rd1234**|
   |Vm location|the name of the same Azure region as the one set as the value of the **Hostpool Location** parameters|
   |Create Network Security Group|**false**|
   |Network Security Group Id|the value of the resourceID parameter of the existing network security group you identified in the previous task|

1. On the **Custom deployment** blade, select **Review + create** and select **Create**.

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This might take about 5 minutes.

#### Task 6: Verify changes to the Azure Virtual Desktop host pool

1. From your lab computer, in the web browser displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, note that the list includes an additional virtual machine named **az140-23-p2-2**.
1. From your lab computer, switch to the Remote Desktop session to **az140-dc-vm11**. 
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to verify that the third  host was successfully joined to the **adatum.com** AD DS domain:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. Switch back to your lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** blade, select **Host pools** and, on the **Azure Virtual Desktop \| Host pools** blade, select the entry **az140-23-hp2** representing the newly modified pool.
1. On the **az140-23-hp2** blade, review the **Essentials** section and verify that the **Host pool type** is set to **Personal** with the **Assignment type** set to **Automatic**.
1. On the **az140-23-hp2** blade, in the vertical menu on the left side, in the **Manage** section, click **Session hosts**. 
1. On the **az140-23-hp2 \| Session hosts** blade, verify that the deployment consists of three hosts. 

#### Task 7: Manage personal desktop assignments in the Azure Virtual Desktop host pool

1. On your lab computer, in the web browser displaying the Azure portal, on the **az140-23-hp2 \| Session hosts** blade, in the vertical menu on the left side, in the **Manage** section, select **Application groups**. 
1. On the **az140-23-hp2 \| Application groups** blade, in the list of application groups select **az140-23-hp2-DAG**.
1. On the **az140-23-hp2-DAG** blade, in the vertical menu on the left, select **Assignments**. 
1. On the **az140-23-hp2-DAG \| Assignments** blade, select **+ Add**.
1. On the **Select Azure AD users or user groups** blade, select **az140-wvd-personal** and click **Select**.

   > **Note**: Now let's review the experience of a user connecting to the Azure Virtual Desktop host pool.

1. From your lab computer, in the browser window displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, select the **az140-cl-vm11** entry.
1. On the **az140-cl-vm11** blade, select **Connect**, in the drop-down menu, select **RDP**, select the Public IP address, and then select **Download RDP File**.
1. When prompted, sign in as the **ADATUM\\aduser7** user with **Pa55w.rd1234** as its password.
1. In the **Stay signed in to all your apps** window, clear the checkbox **Allow my organization to manage my device** checkbox and select **No, sign in to this app only**. 
1. Within the Remote Desktop session to **az140-cl-vm11**, click **Start** and, in the **Start** menu, select the **Remote Desktop** client app.
1. In the **Remote Desktop** client window, select **Subscribe** and, when prompted, sign in with the **aduser7** credentials, by providing its userPrincipalName and **Pa55w.rd1234** as its password.

   > **Note**: Alternatively, in the **Remote Desktop** client window, select **Subscribe with URL**, in the **Subscribe to a Workspace** pane, in the **Email or Workspace URL**, type **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, select **Next**, and, once prompted, sign in with the **aduser7** credentials (using its userPrincipalName attribute as the user name and **Pa55w.rd1234** as its password). 

1. On the **Remote Desktop** page, double-click the **SessionDesktop** icon, when prompted for credentials, type **Pa55w.rd1234**, select the **Remember me** checkbox, and click **OK**.
1. Verify that **aduser7** successfully signed in via Remote Desktop to a host.
1. Within the Remote Desktop session to one of the hosts as **aduser7**, right-click **Start**, in the right-click menu, select **Shut down or sign out** and, in the cascading menu, click **Sign out**.

   > **Note**: Now let's switch the personal desktop assignment from the direct mode to automatic. 

1. Switch to your lab computer, to the web browser displaying the Azure portal, on the **az140-23-hp2-DAG \| Assignments** blade, in the informational bar directly above the list of assignments, click the **Assign VM** link. This will redirect you to the **az140-23-hp2 \| Session hosts** blade. 
1. On the **az140-23-hp2 \| Session hosts** blade, verify that one of the hosts has **aduser7** listed in the **Assigned User** column.

   > **Note**: This is expected since the host pool is configured for automatic assignment.

1. On your lab computer, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to switch to the direct assignment mode:

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. On your lab computer, in the web browser window displaying the Azure portal, navigate to the **az140-23-hp2** host pool blade, review the **Essentials** section and verify that the **Host pool type** is set to **Personal** with the **Assignment type** set to **Direct**.
1. Switch back to the Remote Desktop session to **az140-cl-vm11**, in the **Remote Desktop** window, click the ellipsis icon in the upper right corner next to the **Azure Virtual Desktop** label, in the dropdown menu, click **Unsubscribe**, and, when prompted for confirmation, click **Continue**.
1. Within the Remote Desktop session to **az140-cl-vm11**, in the **Remote Desktop** window, on the **Let's get started** page, click **Subscribe**.
1. When prompted to sign in, on the **Pick an account** pane, click **Use another account**, and, when prompted, sign in by using the user principal name of the **aduser8** user account with **Pa55w.rd1234** as the password.
1. In the **Stay signed in to all your apps** window, clear the checkbox **Allow my organization to manage my device** checkbox and select **No, sign in to this app only**. 
1. On the **Remote Desktop** page, double-click the **SessionDesktop** icon, verify that you receive an error message stating **We couldn't connect because there are currently no available resources. Try again later or contact tech support for help if this keeps happening**, and click **OK**.

   > **Note**: This is expected since the host pool is configured for direct assignment and **aduser8** has not been assigned a host.

1. Switch to your lab computer, to the web browser displaying the Azure portal and, on the **az140-23-hp2 \| Session hosts** blade, select the **(Assign)** link in the **Assigned User** column next to one of the two remaining unassigned hosts.
1. On the **Assign a user**, select **aduser8**, click **Select** and, when prompted for confirmation, click **OK**.
1. Switch back to the Remote Desktop session to **az140-cl-vm11**, in the **Remote Desktop** window, double-click the **SessionDesktop** icon, when prompted for the password, type **Pa55w.rd1234**, click **OK**, and verify that you can successfully sign in to the assigned host.

### Exercise 2: Stop and deallocate Azure VMs provisioned in the lab

The main tasks for this exercise are as follows:

1. Stop and deallocate Azure VMs provisioned in the lab

>**Note**: In this exercise, you will deallocate the Azure VMs provisioned in this lab to minimize the corresponding compute charges

#### Task 1: Deallocate Azure VMs provisioned in the lab

1. Switch to the lab computer and, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to list all Azure VMs created in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to stop and deallocate all Azure VMs you created in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Note**: The command executes asynchronously (as determined by the -NoWait parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the Azure VMs are actually stopped and deallocated.
