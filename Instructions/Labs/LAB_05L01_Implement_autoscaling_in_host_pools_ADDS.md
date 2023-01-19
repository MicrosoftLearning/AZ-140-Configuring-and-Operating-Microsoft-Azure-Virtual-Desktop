---
lab:
    title: 'Lab: Implement autoscaling in host pools (AD DS)'
    module: 'Module 5: Monitor and Maintain a WVD Infrastructure'
---

# Lab - Implement autoscaling in host pools (AD DS)
# Student lab manual

## Lab dependencies

- An Azure subscription you will be using in this lab.
- A Microsoft account or an Azure AD account with the Owner or Contributor role in the Azure subscription you will be using in this lab and with the Global Administrator role in the Azure AD tenant associated with that Azure subscription.
- The completed lab **Prepare for deployment of Azure Virtual Desktop (AD DS)**
- The completed lab **Deploy host pools and session hosts by using the Azure portal (AD DS)**

## Estimated Time

60 minutes

## Lab scenario

You need to configure autoscaling of Azure Virtual Desktop session hosts in an Active Directory Domain Services (AD DS) environment.

## Objectives
  
After completing this lab, you will be able to:

- Configure autoscaling of Azure Virtual Desktop session hosts
- Verify autoscaling of Azure Virtual Desktop session hosts

## Lab files

- None

## Instructions

### Exercise 1: Configure autoscaling of Azure Virtual Desktop session hosts

The main tasks for this exercise are as follows:

1. Prepare for autoscaling of Azure Virtual Desktop session hosts
1. Create and configure an Azure Automation account
1. Create an Azure Logic app

#### Task 1: Prepare for autoscaling of Azure Virtual Desktop session hosts

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. On the lab computer and, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to start the Azure Virtual Desktop session host Azure VMs you will be using in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**Note**: The command executes asynchronously (as determined by the -NoWait parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the Azure VMs are actually started. 

#### Task 2: Create and configure an Azure Automation account

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. In the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-dc-vm11**.
1. On the **az140-dc-vm11** blade, select **Connect**, in the drop-down menu, select **Bastion**, on the **Bastion** tab of the **az140-dc-vm11 \| Connect** blade, select **Use Bastion**.
1. When prompted, provde the following credentials and select **Connect**:

   |Setting|Value|
   |---|---|
   |User Name|**Student**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop session to **az140-dc-vm11**, start **Windows PowerShell ISE** as administrator.
1. From the **Administrator: Windows PowerShell ISE** console, run the following to sign in to your Azure subscription:

   ```powershell
   Connect-AzAccount
   ```

1. When prompted, sign in with the Azure AD credentials of the user account with the Owner role in the subscription you are using in this lab.
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to download the PowerShell script you will use to create the Azure Automation account that is part of the autoscaling solution:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   New-Item -ItemType Directory -Path $labFilesfolder -Force
   Set-Location -Path $labFilesfolder
   $uri = 'https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1'
   Invoke-WebRequest -Uri $Uri -OutFile '.\CreateOrUpdateAzAutoAccount.ps1'
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to set the values of variables that you will assign to script parameters:

   ```powershell
   $aadTenantId = (Get-AzContext).Tenant.Id
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   $suffix = Get-Random
   $automationAccountName = "az140-automation-51$suffix"
   $workspaceName = "az140-workspace-51$suffix"
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create the resource group you will use in this lab:

   ```powershell
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create an Azure Log Analytics workspace you will use in this lab:

   ```powershell
   New-AzOperationalInsightsWorkspace -Location $location -Name $workspaceName -ResourceGroupName $resourceGroupName
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE**, select File from the top menu, open the **C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzAutoAccount.ps1** script, and add a single-line comment character in the lines **97**, **98**, and **99**, such that they look as follows:

   ```powershell
   #	'Az.Compute'
   #	'Az.Resources'
   #	'Az.Automation'
   ```

1. In the **C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzAutoAccount.ps1** script, enclose the code between lines **82** an **86** into the multiline comment, such that they look as follows, and save the changes to the file:

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
	throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```
   
1. Within the Remote Desktop session to **az140-dc-vm11**, open a new tab in the **Administrator: Windows PowerShell ISE** script pane, paste the following script, and run it to create the Azure Automation account that is part of the autoscaling solution:

   ```powershell
   $Params = @{
     "AADTenantId" = $aadTenantId
     "SubscriptionId" = $subscriptionId 
     "UseARMAPI" = $true
     "ResourceGroupName" = $resourceGroupName
     "AutomationAccountName" = $automationAccountName
     "Location" = $location
     "WorkspaceName" = $workspaceName
   }

   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   .\CreateOrUpdateAzAutoAccount.ps1 @Params
   ```

   >**Note**: Wait for the script to complete. This might take about 10 minutes.

1. Within the Remote Desktop session to **az140-dc-vm11**, in the **Administrator: Windows PowerShell ISE** script pane, review the output of the script. 

   >**Note**: The output includes a webhook URI, the Log Analytics Workspace ID and the corresponding primary key values that you need to provide when provisioning the Azure Logic App that is part of the autoscaling solution. 
   
   >**Note**: Record the value of the webhook URI. You will need it later in this lab.

1. To verify the configuration of the Azure Automation account, within the Remote Desktop session to **az140-dc-vm11**, start Microsoft Edge and navigate to the [Azure portal](https://portal.azure.com). If prompted, sign in by using the Azure AD credentials of the user account with the Owner role in the subscription you are using in this lab.
1. Within the Remote Desktop session to **az140-dc-vm11**, in the Microsoft Edge window displaying the Azure portal, search for and select **Automation accounts** and, on the **Automation accounts** blade, select the entry representing the newly provisioned Azure Automation account (with the name starting with the **az140-automation-51** prefix).
1. On the Automation Account blade, in the vertical menu on the left side, in the **Process Automation** section, select **Runbooks** and, in the list of runbooks, verify the presence of the **WVDAutoScaleRunbookARMBased** runbook.
1. On the Automation Account blade, in the vertical menu on the left side, in the **Account Settings** section, select **Identity**.
1. On the **System assigned** tab of the Identity blade of the automation account, set the **Status** to **On**, select **Save**, and, when prompted to  confirm, select **Yes**.
<!-- 1. On the Automation Account blade, in the vertical menu on the left side, in the **Account Settings** section, select **Run as accounts** and, in the list of accounts on the right side, next to the **+ Azure Run As Account**, click **Create**.
1. On the **Add Azure Run As Account** blade, click **Create** and verify that the new account was successfully created. 
-->

#### Task 3: Create an Azure Logic app

1. Within the Remote Desktop session to **az140-dc-vm11**, switch to the **Administrator: Windows PowerShell ISE** window and, from the **Administrator: Windows PowerShell ISE** script pane, run the following to run the following to download the PowerShell script you will use to create the Azure Logic app that is part of the autoscaling solution:

   ```powershell
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   Set-Location -Path $labFilesfolder
   $uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
   Invoke-WebRequest -Uri $uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE**, Select the **File** from the top menu and open the **C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzLogicApp.ps1** script, enclose the code between lines **134** an **138** into the multiline comment and save, such that they look as follows:

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
	throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to set the values of variables that you will assign to script parameters (replace the `<webhook_URI>` placeholder with the value of the webhook URI you recorded earlier in this lab):

   ```powershell
   $AADTenantId = (Get-AzContext).Tenant.Id
   $AzSubscription = (Get-AzContext).Subscription.Id
   $ResourceGroup = Get-AzResourceGroup -Name 'az140-51-RG'
   $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" -Name 'az140-21-hp1'
   $LogAnalyticsWorkspace = (Get-AzOperationalInsightsWorkspace -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $LogAnalyticsWorkspaceId = $LogAnalyticsWorkspace.CustomerId
   $LogAnalyticsWorkspaceKeys = (Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName $ResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspace.Name)
   $LogAnalyticsPrimaryKey = $LogAnalyticsWorkspaceKeys.PrimarySharedKey
   $RecurrenceInterval = 2
   $BeginPeakTime = '1:00'
   $EndPeakTime = '1:01'
   $TimeDifference = '0:00'
   $SessionThresholdPerCPU = 1
   $MinimumNumberOfRDSH = 1
   $MaintenanceTagName = 'CustomMaintenance'
   $LimitSecondsToForceLogOffUser = 5
   $LogOffMessageTitle = 'Autoscaling'
   $LogOffMessageBody = 'Forcing logoff due to autoscaling'

   $AutoAccount = (Get-AzAutomationAccount -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

   $WebhookURIAutoVar = '<webhook_URI>'
   ```

   >**Note**: The values of parameters are geared towards accelerating the autoscaling behavior. In your production environment, you should adjust them to match your own specific requirements.

1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to create the Azure Logic app that is part of the autoscaling solution:

   ```powershell
   $Params = @{
     "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
     "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
     "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
     "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
     "UseARMAPI"                     = $true
     "HostPoolName"                  = $WVDHostPool.Name
     "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
     "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
     "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
     "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
     "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
     "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
     "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
     "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
     "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
     "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
     "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
     "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
     "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shut down."
     "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
     "WebhookURI"                    = $WebhookURIAutoVar
   }

   .\CreateOrUpdateAzLogicApp.ps1 @Params
   ```

   >**Note**: Wait for the script to complete. This might take about 2 minutes.

1. To verify the configuration of the Azure Logic app, within the Remote Desktop session to **az140-dc-vm11**, switch to the Microsoft Edge window displaying the Azure portal, search for and select **Logic Apps** and, on the **Logic apps** blade, select the entry representing the newly provisioned Azure Logic app named **az140-21-hp1_Autoscale_Scheduler**.
1. On the **az140-21-hp1_Autoscale_Scheduler** blade, in the vertical menu on the left side, in the **Development Tools** section, select **Logic app designer**. 
1. On the designer pane, click the rectangle labeled **Recurrence** and note that you can use it to control frequency in which the need for autoscaling is evaluated. 

### Exercise 2: Verify and review autoscaling of Azure Virtual Desktop session hosts

The main tasks for this exercise are as follows:

1. Verify autoscaling of Azure Virtual Desktop session hosts
1. Use Azure Log Analytics to track Azure Virtual Desktop events

#### Task 1: Verify autoscaling of Azure Virtual Desktop session hosts

1. To verify the autoscaling of the Azure Virtual Desktop session hosts, within the Remote Desktop session to **az140-dc-vm11**, in the Microsoft Edge window displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, review the status of the three Azure VMs in the **az140-21-RG** resource group.
1. Verify that two of the three Azure VMs are either in the process of being deallocated or are already **Stopped (deallocated)**.

   >**Note**: As soon as you verify that autoscaling is working, you should disable the Azure Logic app to minimize the corresponding charges.

1. To disable the Azure Logic app, within the Remote Desktop session to **az140-dc-vm11**, in the Microsoft Edge window displaying the Azure portal, search for and select **Logic Apps** and, on the **Logic apps** blade, select the entry representing the newly provisioned Azure Logic app named **az140-21-hp1_Autoscale_Scheduler**.
1. On the **az140-21-hp1_Autoscale_Scheduler** blade, in the toolbar, click **Disable**. 
1. On the **az140-21-hp1_Autoscale_Scheduler** blade, in the **Essentials** section, review the information including the number of successful runs in the last 24 hours and the **Summary** section providing the frequency of recurrence. 
1. Within the Remote Desktop session to **az140-dc-vm11**, in the Microsoft Edge window displaying the Azure portal, search for and select **Automation accounts** and, on the **Automation accounts** blade, select the entry representing the newly provisioned Azure Automation account (with the name starting with the **az140-automation-51** prefix).
1. On the **Automation Account** blade, in the vertical menu on the left side, in the **Process Automation** section, select **Jobs** and review the list of jobs corresponding to individual invocations of the **WVDAutoScaleRunbookARMBased** runbook.
1. Select the most recent job and, on its blade, click **All Logs** tab header. This will display detailed listing of job execution steps.

#### Task 2: Use Azure Log Analytics to track Azure Virtual Desktop events

>**Note**: To analyze autoscaling and any other Azure Virtual Desktop events, you can use Log Analytics.

1. Within the Remote Desktop session to **az140-dc-vm11**, in the Microsoft Edge window displaying the Azure portal, search for and select **Log Analytics workspaces** and, on the **Log Analytics workspaces** blade, select the entry representing the Azure Log Analytics workspace used in this lab (which name starts with the **az140-workspace-51** prefix.
1. On the Log Analytics workspace blade, in the vertical menu on the left side, in the **General** section, click **Logs**, if needed, close the **Welcome to Log Analytics** window, and proceed to the **Query** pane.
1. On the **Queries** pane, in the **All Queries** vertical menu on the left side, select **Azure Virtual Desktop** and review the predefined queries.
1. Close the **Queries** pane. This will automatically display the **New Query 1** tab.
1. In the query window, paste the following query, click **Run** to display all events for the host pool used in this lab:

   ```kql
   WVDTenantScale_CL
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

   >**Note**: If there was an extra pipe character (|) in second line when using the cut an paste contrsuct, remove it to avoid a failure. This could apply to each query.
   >**Note**: If you don't see any results, wait a few minutes and try again.

1. In the query window, paste the following query, click **Run** to display the total number of currently running session hosts and active user sessions in the target host pool:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Number of running session hosts:"
     or logmessage_s contains "Number of user sessions:"
     or logmessage_s contains "Number of user sessions per Core:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. In the query window, paste the following query, click **Run** to display the status of all session host VMs in a host pool:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Session host:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. In the query window, paste the following query, click **Run** to display any scaling related errors and warnings:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

>**Note**: Ignore the error message regarding `TenantId`

### Exercise 3: Stop and deallocate Azure VMs provisioned in the lab

The main tasks for this exercise are as follows:

1. Stop and deallocate Azure VMs provisioned in the lab

>**Note**: In this exercise, you will deallocate the Azure VMs provisioned in this lab to minimize the corresponding compute charges

#### Task 1: Deallocate Azure VMs provisioned in the lab

1. Switch to the lab computer and, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to list all Azure VMs created in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to stop and deallocate all Azure VMs you created in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Note**: The command executes asynchronously (as determined by the -NoWait parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the Azure VMs are actually stopped and deallocated.
