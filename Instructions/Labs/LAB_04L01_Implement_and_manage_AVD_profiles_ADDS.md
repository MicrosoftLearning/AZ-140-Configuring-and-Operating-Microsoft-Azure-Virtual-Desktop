---
lab:
    title: 'Lab: Implement and manage Azure Virtual Desktop profiles (AD DS)'
    module: 'Module 4: Manage User Environments and Apps'
---

# Lab - Implement and manage Azure Virtual Desktop profiles (AD DS)
# Student lab manual

## Lab dependencies

- An Azure subscription you will be using in this lab.
- A Microsoft account or a Microsoft Entra account with the Owner or Contributor role in the Azure subscription you will be using in this lab and with the Global Administrator role in the Microsoft Entra tenant associated with that Azure subscription.
- The completed lab **Prepare for deployment of Azure Virtual Desktop (AD DS)**
- The completed lab **Implement and manage storage for AVD (AD DS)**

## Estimated Time

30 minutes

## Lab scenario

You need to implement Azure Virtual Desktop profile management in an Active Directory Domain Services (AD DS) environment.

## Objectives
  
After completing this lab, you will be able to:

- Implement FSLogix based profiles for Azure Virtual Desktop

## Lab files

- None

## Instructions

### Exercise 1: Implement FSLogix based profiles for Azure Virtual Desktop

The main tasks for this exercise are as follows:

1. Configure FSLogix-based profiles on Azure Virtual Desktop session host VMs
1. Test FSLogix-based profiles with Azure Virtual Desktop

#### Task 1: Configure FSLogix-based profiles on Azure Virtual Desktop session host VMs

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.
1. In the Azure portal, open **Cloud Shell** pane by selecting the toolbar icon directly to the right of the search textbox.
1. In the **Cloud Shell** pane, run the following to start the Azure Virtual Desktop session host Azure VMs you will be using in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Note**: Wait until the Azure VMs are running before you proceed to the next step.

1. In the **Cloud Shell** pane, run the following to enable PowerShell Remoting on the Session Hosts.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```

1. Close the Cloud Shell
1. In the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-21-p1-0**.
1. On the **az140-21-p1-0** blade, select **Connect**, in the drop-down menu, select **Connect via Bastion**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**student@adatum.com**|
   |Password|**Pa55w.rd1234**|

1. Within the Bastion session to **az140-21-p1-0**, start Microsoft Edge, browse to [FSLogix download page](https://aka.ms/fslogix_download), download FSLogix compressed installation binaries, extract them into the **C:\\Allfiles\\Labs\\04** folder (create the folder if needed), navigate to the **x64\\Release** subfolder, double-click the **FSLogixAppsSetup.exe** file to launch the **Microsoft FSLogix Apps Setup** wizard, and step through the installation of Microsoft FSLogix Apps with the default settings.

   > **Note**: Installation of FSLogix is not necessary if the image already includes it.

1. Within the Bastion session to **az140-21-p1-0**, start **Windows PowerShell ISE** as administrator.
1. From the **Administrator: Windows PowerShell ISE** console, run the following to install the latest version of the Az PowerShell module (enter **Y** when prompted for installing and importing NuGet):

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck -Force
   ```

   > **Note**: You may need to wait 3-5 minutes before any output from the installation of the Az module appears. You may also need to wait a further 5 minutes **after** output has stopped. This is expected behavior.

1. From the **Administrator: Windows PowerShell ISE** console, run the following to disable Windows Account Manager:

   ```powershell
   Update-AzConfig -EnableLoginByWam $false
   ```

1. From the **Administrator: Windows PowerShell ISE** console, run the following to sign in to your Azure subscription:

   ```powershell
   Connect-AzAccount
   ```

1. When prompted, sign in with the Microsoft Entra credentials of the user account with the Owner role in the subscription you are using in this lab.
1. Within the Bastion session to **az140-21-p1-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to retrieve the name of the Azure Storage account you configured earlier in this lab:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. Within the Bastion session to **az140-21-p1-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to configure profile registry settings:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. Within the Bastion session to **az140-21-p1-0**, right-click **Start**, in the right-click menu, select **Run**, in the **Run** dialog box, in the **Open** text box, type the following and select **OK** to launch the **Local Users and Groups** console:

   ```cmd
   lusrmgr.msc
   ```

1. In the **Local Users and Groups** console, note the four groups which names start with the **FSLogix** string:

   - FSLogix ODFC Exclude List
   - FSLogix ODFC Include List
   - FSLogix Profile Exclude List
   - FSLogix Profile Include List

1. In the **Local Users and Groups** console, in the list of groups, double-click the **FSLogix Profile Include List** group, note that it includes the **\\Everyone** group, and select **OK** to close the group **Properties** window. 
1. In the **Local Users and Groups** console, in the list of groups, double-click the **FSLogix Profile Exclude List** group, note that it does not include any group members by default, and select **OK** to close the group **Properties** window. 

   > **Note**: To provide consistent user experience, you need to install and configure FSLogix components on all Azure Virtual Desktop session hosts. You will perform this task in the unattended manner on the other session hosts in our lab environment. 

   > **Note**: The following step is not required if FSLogix is already installed on the session hosts.

1. Within the Bastion session to **az140-21-p1-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to install FSLogix components on the **az140-21-p1-1** and **az140-21-p1-2** session hosts:

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **Note**: Wait for the script execution to complete. This might take about 2 minutes.

1. Within the Bastion session to **az140-21-p1-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to configure profile registry settings on the **az140-21-p1-1** and **az140-21-p1-1** session hosts:

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **Note**: Before you test the FSLogix-based profile functionality, you need to remove the locally cached profile of the **ADATUM\\aduser1** account you will be using for testing from the Azure Virtual Desktop session hosts you used in the previous lab.

1. Within the Bastion session to **az140-21-p1-0**, from the **Administrator: Windows PowerShell ISE** script pane, run the following to remove the locally cached profile of the **ADATUM\\aduser1** account on all Azure VMs serving as session hosts:

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

1. Within the Bastion session to **az140-21-p1-0**, right-click **Start**, in the right-click menu, select **Shut down or sign out** and then, in the cascading menu, select **Sign out**.
1. From the **Disconnected** window, select **Close**.

#### Task 2: Test FSLogix-based profiles with Azure Virtual Desktop

1. Switch to your lab computer, from the lab computer, in the browser window displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, select the **az140-cl-vm11** entry.
1. On the **az140-cl-vm11** blade, select **Connect**, in the drop-down menu, select **Connect via Bastion**.
1. When prompted, provde the following credentials and select **Connect**:

   |Setting|Value|
   |---|---|
   |User Name|**Student@adatum.com**|
   |Password|**Pa55w.rd1234**|

1. Within the Bastion session to **az140-cl-vm11**, click **Start** and, in the **Start** menu, click **Remote Desktop** to start the Remote Desktop client.
1. Within the Bastion session to **az140-cl-vm11**, in the **Remote Desktop** client window, select **Subscribe** and, when prompted, sign in with the **aduser1** credentials.

   >**Note**  If you're not asked to subscribe, you might have to unsubscribe from a previous suscription.

1. in the list of applications, double-click **Command Prompt**, when prompted, provide the password of the **aduser1** account, and verify a **Command Prompt** window opens successfully.
1. In the upper left corner of the **Command Prompt** window, right-click the **Command Prompt** icon and, in the drop-down menu, select **Properties**.
1. In the **Command Prompt Properties** dialog box, select the **Font** tab, modify the size and font settings, and select **OK**.
1. From the **Command Prompt** window, type **logoff** and press the **Enter** key to sign out from the Remote Desktop session.
1. Within the Bastion session to **az140-cl-vm11**, in the **Remote Desktop** client window, in the list of applications, double-click **SessionDesktop** under az140-21-ws1 and verify that it launches a Remote Desktop session. 
1. Within the **SessionDesktop** session, right-click **Start**, in the right-click menu, select **Run**, in the **Run** dialog box, in the **Open** text box, type **cmd** and select **OK** to launch a **Command Prompt** window:
1. Verify that the **Command Prompt** window settings match those you configured earlier in this task.
1. Within the **SessionDesktop** session, minimize all windows, right-click the desktop, in the right-click menu, select **New** and, in the cascading menu, select **Shortcut**. 
1. On the **What item would you like to create a shortcut for?** page of the **Create Shortcut** wizard, in the **Type the location of the item** text box, type **Notepad** and select **Next**.
1. On the **What would you like to name the shortcut** page of the **Create Shortcut** wizard, in the **Type a name for this shortcut** text box, type **Notepad** and select **Finish**.
1. Within the **SessionDesktop** session, right-click **Start**, in the right-click menu, select **Shut down or sign out** and then, in the cascading menu, select **Sign out**.
1. Back in the Bastion session to **az140-cl-vm11**, in the **Remote Desktop** client window, in the list of applications, and double-click **SessionDesktop** to start a new Remote Desktop session. 
1. Within the **SessionDesktop** session, verify that the **Notepad** shortcut appears on the desktop.
1. Within the **SessionDesktop** session, right-click **Start**, in the right-click menu, select **Shut down or sign out** and then, in the cascading menu, select **Sign out**.
1. Switch to your lab computer and, in the Microsoft Edge window displaying the Azure portal, navigate to the **Storage accounts** blade and select the entry representing the storage account you created in the previous exercise.
1. On the storage account blade, in the **File services** section, select **File shares** and then, in the list of file shares, select **az140-22-profiles**. 
1. On the **az140-22-profiles** blade, select **Browse** and verify that its content includes a folder which name consists of a combination of the Security Identifier (SID) of the **ADATUM\\aduser1** account followed by the **_aduser1** suffix.
1. Select the folder you identified in the previous step and note that it contains a single file named **Profile_aduser1.vhd**.

### Exercise 2: Stop and deallocate Azure VMs provisioned and used in the lab

The main tasks for this exercise are as follows:

1. Stop and deallocate Azure VMs provisioned and used in the lab

>**Note**: In this exercise, you will deallocate the Azure VMs provisioned and used in this lab to minimize the corresponding compute charges

#### Task 1: Deallocate Azure VMs provisioned and used in the lab

1. Switch to the lab computer and, in the web browser window displaying the Azure portal, open the **PowerShell** shell session within the **Cloud Shell** pane.
1. From the PowerShell session in the Cloud Shell pane, run the following to list all Azure VMs created and used in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. From the PowerShell session in the Cloud Shell pane, run the following to stop and deallocate all Azure VMs you created and used in this lab:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Note**: The command executes asynchronously (as determined by the -NoWait parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the Azure VMs are actually stopped and deallocated.
