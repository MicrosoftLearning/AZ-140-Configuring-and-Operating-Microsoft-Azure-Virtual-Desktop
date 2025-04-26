---
lab:
    title: 'Lab: Create custom session host images by using image templates'
    module: 'Module 1.5: Create and manage session host images'
---

# Lab - Create custom session host images by using image templates
# Student lab manual

## Lab dependencies

- An Azure subscription you will be using in this lab.
- A Microsoft Entra user account with the Owner role in the Azure subscription you will be using in this lab and with the permissions sufficient to join devices to the Entra tenant associated with that Azure subscription.

## Estimated Time

90 minutes (about 45 minutes is the wait time for the image build to complete)

## Lab scenario

You plan to implement an Azure Virtual Desktop environment. You need to use custom virtual machine images when deploying Azure Virtual Desktop session hosts.

## Objectives
  
After completing this lab, you will be able to:

- Create custom session host images for Azure Virtual Desktop by using image templates

## Lab files

- None

## Instructions

### Exercise 1: Create custom session host images by using image templates

The main tasks for this exercise are as follows:

1. Register required resource providers
1. Create a user-assigned managed identity
1. Create a custom Azure role-based access control (RBAC) role
1. Set permissions on the host image provisioning-related resources
1. Create an Azure Compute Gallery instance and an image definition
1. Create a custom image template
1. Build a custom image
1. Deploy session hosts by using a custom image

> **Note**: Before you can create a custom image template, you need to satisfy a number of prerequisites, including:

- Register all required resource providers
- Create a user-assigned managed identity
- Grant permissions required by the user-assigned managed identity by using a custom Azure role-based access control (RBAC) role
- If you intend to distribute the image by using Azure Compute Gallery, you have to create its instance along with an image definition

#### Task 1: Register required resource providers

1. If needed, from the lab computer, start a web browser, navigate to the Azure portal and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.

    > **Note**: Use the credentials of the `User1-` account listed on the Resources tab on the right side of the lab session window.

1. In the Azure portal, start a PowerShell session in the Azure Cloud Shell.

    > **Note**: If prompted, in the **Getting started** pane, in the **Subscription** drop-down list, select the name of the Azure subscription you are using in this lab and then select **Apply**.

1. In the PowerShell session in the Azure Cloud Shell pane, run the following command to register the **Microsoft.DesktopVirtualization** resource provider:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
    Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
    Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance
    ```

    > **Note**: Do not wait for the registration to complete. This might take about 5 minutes.

1. Close the Azure Cloud Shell pane.

#### Task 2: Create a user-assigned managed identity

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Managed Identities**.
1. On the **Managed Identities** page, select **+ Create**.
1. On the **Basics** tab of the **Create User Assigned Managed Identity** page, specify the following settings and then select **Review + create**:

    > **Note**: When setting the **Name** value, switch to the Resources tab on the right side of the lab session window and identify the string of characters between *User1-* and the *@* character. Use this string to replace the *random* placeholder.

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|The name of a new resource group **az140-15a-RG**|
    |Region|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |Name|**az140**-*random*-**uami**|

1. On the **Review + create** tab, select **Create**.

    >**Note**: Do not wait for the provisioning of the user assigned managed identity to complete. This should take just a few seconds.

#### Task 3: Create a custom Azure role-based access control (RBAC) role

>**Note**: The custom Azure role-based access control (RBAC) role will be used to assign appropriate permissions to the user-assigned managed identity created in the previous task.

1. From the lab computer, in the web browser displaying the Azure portal, start a PowerShell session in the Azure Cloud Shell.

1. In the PowerShell session in the Azure Cloud Shell pane, run the following command to identify the value of the **Id** property of the Azure subscription used for this lab and store it in the **$subscriptionId** variable:

    ```powershell
    $subscriptionId = (Get-AzSubscription).Id
    ```

1. Run the following command to create the role definition of the new custom role including its assignable scope value and store it in the **$jsonContent** variable (make sure to replace the *random* placeholder with the same string you identified in the previous task):

    ```powershell
    $jsonContent = @"
    {
      "Name": "Desktop Virtualization Image Creator (random)",
      "IsCustom": true,
      "Description": "Create custom image templates for Azure Virtual Desktop images.",
      "Actions": [
        "Microsoft.Compute/galleries/read",
        "Microsoft.Compute/galleries/images/read",
        "Microsoft.Compute/galleries/images/versions/read",
        "Microsoft.Compute/galleries/images/versions/write",
        "Microsoft.Compute/images/write",
        "Microsoft.Compute/images/read",
        "Microsoft.Compute/images/delete"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/$subscriptionId",
        "/subscriptions/$subscriptionId/resourceGroups/az140-15b-RG"
      ]
    }
    "@
    ```

1. Run the following command to store the content of the **$jsonContent** variable in a file named **CustomRole.json**:

    ```powershell
    $jsonContent | Out-File -FilePath 'CustomRole.json'
    ```

1. Run the following command to create the custom role:

    ```powershell
    New-AzRoleDefinition -InputFile ./CustomRole.json
    ```

1. Close the Azure Cloud Shell pane.

#### Task 4: Set permissions on the host image provisioning-related resources

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Resource groups** and, on the **Resource groups** page, select **+ Create**.
1. On the **Basics** tab of the **Create a resource group** page, specify the following settings and then select **Review + create**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|The name of a new resource group **az140-15b-RG**|
    |Region|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|

1. On the **Review + create** tab, select **Create**.
1. Refresh the **Resource groups** page and, in the list of resource groups, select **az140-15b-RG**.
1. On the **az140-15b-RG** page, in the vertical navigation menu, select **Access control (IAM)**.
1. On the **az140-15b-RG\|Access control (IAM)** page, select **+ Add** and, in the drop-down menu, select **Add role assignment**.
1. On the **Role** tab of the **Add role assignment** page, ensure that the **Job function roles** tab is selected, in the search textbox, enter **Desktop Virtualization Image Creator** (*random*), in the list of results, select **Desktop Virtualization Image Creator** (*random*), and then select **Next**.

    >**Note**: Make sure to replace the *random* placeholder with the same string you used when defining the new custom RBAC role.

1. On the **Members** tab of the **Add role assignment** page, select the **Managed identity** option, click **+ Select members**, in the **Select managed identities** pane, in the **Managed identity** drop-down list, select **User-assigned managed identity**, in the list of user-assigned managed identities, select **az140**-*random*-**uami** (where the *random* placehodler represents the same string you used when defining the new custom RBAC role), and then click **Select**.
1. Back on the **Members** tab of the **Add role assignment** page, select **Review + assign**.
1. On the **Review + assign** tab, select **Review + assign**. 

#### Task 5: Create an Azure Compute Gallery instance and an image definition

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure compute galleries** and, on the **Azure compute galleries** page, select **+ Create**.
1. On the **Basics** tab of the **Create Azure compute gallery** page, specify the following settings and then select **Next : Sharing method**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**az140-15b-RG**|
    |Name|**az14015computegallery**|
    |Region|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|

1. On the **Sharing** tab of the **Create Azure compute gallery** page, leave the default option **Role based access control (RBAC)** selected and then select **Review + create**.
1. On the **Review + create** tab, select **Create**.

    >**Note**: Wait for the provisioning process to complete. This should take less than 1 minute.

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure compute galleries** and, on the **Azure compute galleries** page, select **az14015computegallery**. 
1. On the **az14015computegallery** page, select **+ Add** and, in the drop-down menu, select **+ VM image definition**. 
1. On the **Basics** tab of the **Create VM image definition** page, specify the following settings (leave other settings with their default values) and then select **Next : Version**:

    |Setting|Value|
    |---|---|
    |Region|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |VM image definition name|**az14015imagedefinition**|
    |OS type|**Windows**|
    |Security type|**Trusted launch supported**|
    |OS state|**Generalized**|
    |Publisher|**MicrosoftWindowsDesktop**|
    |Offer|**Windows-11**|
    |SKU|**win11-23h2-avd-m365**|

    > **Note**: VM generation is automatically set to Gen2, because Gen 1 virtual machines are not supported with Trusted and Confidential security type.

1. On the **Version** tab of the **Create VM image definition** page, leave the settings unchanged and select **Next : Publishing options**.

    > **Note**: You should not create the VM image version at this stage. This will be done by Azure Virtual Desktop.

1. On the **Publishing options** tab of the **Create VM image definition** page, leave the settings unchanged and select **Review + create**.
1. On the **Review + create** tab the **Create VM image definition** page, select **Create**.

    > **Note**: Wait for the provisioning process to complete. This typically takes less than 1 minute.

#### Task 6: Create a custom image template

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** page, in the **Manage** section of the vertical navigation menu, select **Custom image templates** and, on the **Azure Virtual Desktop \| Custom image templates** page, select **+ Add custom image template**. 
1. On the **Basics** tab of the **Create custom image template** page, specify the following settings and select **Next**:

    > **Note**: When setting the **Managed identity** property, make sure to replace the *random* placeholder with the same string you identified earlier in this exercise.

    |Setting|Value|
    |---|---|
    |Template name|**az140-15b-imagetemplate**|
    |Import from existing template|**No**|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**az140-15b-RG**|
    |Location|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |Managed identity|**az140**-*random*-**uami**|

1. On the **Source image** tab of the **Create custom image template** page, specify the following settings and select **Next**:

    |Setting|Value|
    |---|---|
    |Source type|**Platform image (marketplace)**|
    |Select image|**Windows 11 Enterprise multi-session, Version 23H2 + Microsoft 365 Apps**|

1. On the **Distribution targets** tab of the **Create custom image template** page, specify the following settings (leave other settings with their default values) and select **Next**:

    |Setting|Value|
    |---|---|
    |Azure Compute Gallery|enabled|
    |Gallery name|**az14015computegallery**|
    |Gallery image definition|**az14015imagedefinition**|
    |Gallery image version|**1.0.0**|
    |Run output name|**az140-15-image-1.0.0**|
    |Replication regions|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |Exclude from latest|**No**|
    |Storage account type|**Standard_LRS**|

    > **Note**: You can use the **Replication regions** property to accommodate multi-region builds. Setting **Exclude from latest** to **Yes** would prevent this image version from being used when **latest** is specified as the version of the **ImageReference** element during VM creation.

1. On the **Build properties** tab of the **Create custom image template** page, specify the following settings (leave other settings with their default values) and select **Next**:

    |Setting|Value|
    |---|---|
    |Build timeout|**120**|
    |Build VM size|**Standard_DC2s_v3**|
    |OS disk size (GB)|**127**|
    |Staging group|**az140-15c-RG**|
    |VNet|Leave not set|

    > **Note**: **Staging group** is the resource group used to stage resources to build the image and store logs. If you don't provide its name, it will be automatically generated. If the **VNet** name is not set, a temporary one is created, along with a public IP address for the VM used to create the build.

    > **Important**: Ensure that you have sufficient number of available vCPUs for the Build VM size you specified. If not, either choose a different size or request quota increase.

1. On the **Customization** tab of the **Create custom image template** page, select **+ Add built-in script**. 
1. In the **Select built-in scripts** pane, review the available options grouped into operating system specific scripts, Azure Virtual Desktop scripts, MSIX App Attach scripts, Application scripts, and Windows Updates-related scripts, and then select the following entries:

   - **Time zone redirection**: allows the client to use its time zone within a session on session hosts
   - **Disable Storage Sense**: prevents Storage Sense from negatively affecting session hosts by falsely detecting low free disk space conditions
   - **Enable screen capture protection** with **Block Screen capture on client and server**: blocks or hides remote content in screenshots and screen sharing

1. In the **Select built-in scripts** pane, select **Save**.

    > **Note**: You have the option of adding your own scripts. For examples, consider referencing the built-in scripts, such as [Time zone redirection](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/TimezoneRedirection.ps1), [Disable Storage Sense](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/DisableStorageSense.ps1), or [Enable screen capture protection](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/ScreenCaptureProtection.ps1).

1. Back on the **Customization** tab of the **Create custom image template** page, select **Next**.
1. On the **Tags** tab of the **Create custom image template** page, select **Next**.
1. On the **Review + create** tab of the **Create custom image template** page, select **Create**.

    > **Note**: Wait for the template to be created. This might take a few minutes. Refresh the **Azure Virtual Desktop \| Custom image templates** page to review the template status.

#### Task 7: Build a custom image

> **Note**: The remaining tasks of this lab are optional since they involve a fairly extensive wait time. 

1. From the lab computer, in the web browser displaying the Azure portal, on the **Azure Virtual Desktop \| Custom image template** page, select **az140-15b-imagetemplate**.
1. On the **az140-15b-imagetemplate** page, select **Start build**.

    > **Note**: Wait for the build to be created. The actual time to complete the build process might vary, but with the settings provided in the lab instructions, it should complete within 45 minutes. Refresh the page every few minutes and monitor the **Build run state** value in the **Essentials** section of the **az140-15b-imagetemplate** page. 

    > **Note**: The build run state should change at some point from **Running - Building** to **Running - Distributing** and finally to **Succeeded**.

    > **Note**: While waiting for the build to complete, review the content of the staging resource group **az140-15c-RG**, where the build resources, including the bulid virtual machine, a virtual network, network security group, key vault, snapshot, container instance, and storage account are automatically provisioned. 

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Resource groups** and, on the **Resource groups** page, select **az140-15c-RG**.
1. On the **az140-15c-RG** page, in the **Resources** section, note the auto-provisoned resources.
1. Return to the **az140-15b-imagetemplate** page and monitor the build progress. 

    > **Note**: Alternatively, you can use **Activity Log** to keep track of the completion of the build process. The action you should focus on is **Execute a VM image template to produce its output**. Its status should change at some point from **Accepted** to **Succeeded**.

1. Once the build completes, from the lab computer, in the web browser displaying the Azure portal, search for and select **Azure compute galleries** and, on the **Azure compute galleries** page, select **az14015computegallery**. 
1. On the **az14015computegallery**, on the **Definitions** tab, select **az14015imagedefinition**.
1. On the **az14015imagedefinition** page, on the **Versions** tab, review the information about the **1.0.0 (latest version)** image.

#### Task 8: Deploy session hosts by using a custom image

> **Note**: Optionally, consider stepping through the initial stages of deploying Azure Virtual Desktop session hosts by using the custom image you created. 

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Virtual networks** and, on the **Virtual networks** page, select **Create +**
1. On the **Basics** tab of the **Create virtual network** page, specify the following settings and select **Next**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|The name of a new resource group **az140-15d-RG**|
    |Virtual network name|**az140-vnet15d**|
    |Region|The name of the Azure region where you want to deploy the Azure Virtual Desktop environment|

1. On the **Security** tab, accept the default settings and select **Next**.
1. On the **IP addresses** tab, specify the following settings:

    |Setting|Value|
    |---|---|
    |IP address space|**10.30.0.0/16**|

1. Select the edit (pencil) icon next to the **default** subnet entry, in the **Edit** pane, specify the following settings (leave others with their existing values) and select **Save**:

    |Setting|Value|
    |---|---|
    |Name|**hp1-Subnet**|
    |Starting address|**10.30.1.0**|
    |Enable private subnet (no default outbound access)|Disabled|

1. Back on the **IP addresses** tab, select **Review + create** and then select **Create**.

    > **Note**: Wait for the provisioning process to complete. This typically takes less than 1 minute.

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** page, in the **Manage** section of the vertical navigation menu, select **Host pools** and, on the **Azure Virtual Desktop \| Host pools** page, select **+ Create**. 
1. On the **Basics** tab of the **Create a host pool** page, specify the following settings and select **Next : Session hosts >** (leave other settings with their default values):

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**az140-15d-RG**|
    |Host pool name|**az140-15-hp1**|
    |Location|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |Validation environment|**No**|
    |Preferred app group type|**Desktop**|
    |Host pool type|**Pooled**|
    |Create Session Host Configuration|**No**|
    |Load balancing algorithm|**Breadth-first**|

    > **Note**: When using the Breadth-first load balancing algorithm, the max session limit parameter is optional.

1. On the **Session hosts** tab of the **Create a host pool** page, specify the following settings (leave other settings with their default values):

    > **Note**: When setting the **Name prefix** value, switch to the Resources tab on the right side of the lab session window and identify the string of characters between *User1-* and the *@* character. Use this string to replace the *random* placeholder.

    |Setting|Value|
    |---|---|
    |Add virtual machines|**Yes**|
    |Resource group|**Defaulted to same as host pool**|
    |Name prefix|**sh0**_random_|
    |Virtual machine type|**Azure virtual machine**|
    |Virtual machine location|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |Availability options|**No infrastructure redundancy required**|
    |Security type|**Trusted launch virtual machines**|

1. On the **Virtual machines** tab of the **Create a host pool** page, below the **Image** drop-down list, select **See all images**.
1. On the **Select an image** page, select **Shared images** and, in the list of images, select **az14015imagedefinition**. 
1. Back on the **Virtual machines** tab of the **Create a host pool** page, specify the following settings and select **Next : Workspace >** (leave other settings with their default values):

    |Setting|Value|
    |---|---|
    |Virtual machine size|**Standard DC2s_v3**|
    |Number of VMs|**1**|
    |OS disk type|**Standard SSD**|
    |OS disk size|**Default size**|
    |Boot Diagnostics|**Enable with managed storage account (recommended)**|
    |Virtual network|az140-vnet15d|
    |Subnet|**hp1-Subnet**|
    |Network security group|**Basic**|
    |Public inbound ports|**No**|
    |Select which directory you would like to join|**Microsoft Entra ID**|
    |Enroll VM with Intune|**No**|
    |User name|**Student**|
    |Password|Any sufficiently complex string of characters that will be used as the password for the built-in administrator account|
    |Confirm password|The same string of characters you specified previously|

    > **Note**: The password should be at least 12 characters in length and consist of a combination of lower-case characters, upper-case characters, digits, and special characters. For details, refer to the information about [the password requirements when creating an Azure VM](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. On the **Workspace** tab of the **Create a host pool** page, confirm the following setting and select **Review + create**:

    |Setting|Value|
    |---|---|
    |Register desktop app group|**No**|

1. On the **Review + create** tab of the **Create a host pool** page, select **Create**.

    > **Note**: Wait for the deployment to complete. This might take about 10-15 minutes.
