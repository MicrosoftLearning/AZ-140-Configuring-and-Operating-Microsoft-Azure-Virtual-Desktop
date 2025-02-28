---
lab:
    title: 'Lab: Deploy host pools and session hosts by using the Azure portal (Entra ID)'
    module: 'Module 1.4: Implement host pools and session hosts'
---

# Lab - Deploy host pools and session hosts by using the Azure portal (Entra ID)
# Student lab manual

## Lab dependencies

- An Azure subscription you will be using in this lab.
- A Microsoft Entra user account with the Owner role in the Azure subscription you will be using in this lab and with the permissions sufficient to join devices to the Entra tenant associated with that Azure subscription.

## Estimated Time

30 minutes

## Lab scenario

You have a Microsoft Azure subscription. You need to deploy Azure Virtual Desktop environment that uses Microsoft Entra joined session hosts.

## Objectives
  
After completing this lab, you will be able to:

- Deploy Microsoft Entra joined Azure Virtual Desktop session hosts

## Lab files

- None

## Instructions

### Exercise 1: Implement an Azure Virtual Desktop environment using Microsoft Entra joined session hosts
  
The main tasks for this exercise are as follows:

1. Prepare the Azure subscription for deployment of an Azure Virtual Desktop host pool
1. Deploy an Azure Virtual Desktop host pool
1. Create an Azure Virtual Desktop application group
1. Create an Azure Virtual Desktop workspace
1. Grant access to Azure Virtual Desktop host pools

#### Task 1: Prepare the Azure subscription for deployment of an Azure Virtual Desktop host pool

1. From the lab computer, start a web browser, navigate to the Azure portal at [https://portal.azure.com](https://portal.azure.com) and sign in by providing the credentials of a user account with the Owner role in the subscription you will be using in this lab.

    > **Note**: Use the credentials of the `User1-` account listed on the Resources tab on the right side of the lab session window.

1. In the Azure portal, start a PowerShell session in the Azure Cloud Shell.

    > **Note**: If prompted, in the **Getting started** pane, in the **Subscription** drop-down list, select the name of the Azure subscription you are using in this lab and then select **Apply**.

1. In the PowerShell session in the Azure Cloud Shell pane, run the following command to register the **Microsoft.DesktopVirtualization** resource provider:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    ```

    > **Note**: Do not wait for the registration to complete. This might take a couple of minutes.

1. Close the Cloud Shell pane.
1. In the web browser displaying the Azure portal, search for and select **Virtual networks** and, on the **Virtual networks** page, select **Create +**
1. On the **Basics** tab of the **Create virtual network** page, specify the following settings and select **Next**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|The name of a new resource group **az140-11e-RG**|
    |Virtual network name|**az140-vnet11e**|
    |Region|The name of the Azure region where you want to deploy the Azure Virtual Desktop environment|

1. On the **Security** tab, accept the default settings and select **Next**.
1. On the **IP addresses** tab, specify the following settings:

    |Setting|Value|
    |---|---|
    |IP address space|**10.20.0.0/16**|

1. Select the edit (pencil) icon next to the **default** subnet entry, in the **Edit** pane, specify the following settings (leave others with their existing values) and select **Save**:

    |Setting|Value|
    |---|---|
    |Name|**hp1-Subnet**|
    |Starting address|**10.20.1.0**|
    |Enable private subnet (no default outbound access)|Disabled|

1. Back on the **IP addresses** tab, select **Review + create** and then, on the **Review + create** tab, select **Create**.

    > **Note**: Do not wait for the provisioning process to complete. This typically takes less than 1 minute.

1. In the web browser displaying the Azure portal, search for and select **Microsoft Entra ID**.
1. On the **Overview** page of the Microsoft Entra tenant associated with your subscription, in the **Manage** section of the vertical navigation menu, select **Users**.
1. On the **Users** page, in the **Search** text box, enter the name of the `User1-` account listed on the Resources tab on the right side of the lab session window.
1. In the list of results of the search, select the user account entry with the matching name.
1. On the page displaying the proerties of the user account, in the **Manage** section of the vertical navigation menu, select **Groups**.
1. On the **Groups** page, record the name of the group starting with **AVD-DAG** prefix (you will need it later in this lab).
1. Navigate back to the **Users** page, in the **Search** text box, enter the name of the `User2-` account listed on the Resources tab on the right side of the lab session window.
1. In the list of results of the search, select the user account entry with the matching name.
1. On the page displaying the proerties of the user account, in the **Manage** section of the vertical navigation menu, select **Groups**.
1. On the **Groups** page, record the name of the group starting with **AVD-RemoteApp** prefix (you will need it later in this lab).

#### Task 2: Deploy an Azure Virtual Desktop host pool

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop**, on the **Azure Virtual Desktop** page, in the **Manage** section of the vertical navigation menu, select **Host pools** and, on the **Azure Virtual Desktop \| Host pools** page, select **+ Create**. 
1. On the **Basics** tab of the **Create a host pool** page, specify the following settings and select **Next : Session hosts >** (leave other settings with their default values):

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|The name of a new resource group **az140-21e-RG**|
    |Host pool name|**az140-21-hp1**|
    |Location|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |Validation environment|**No**|
    |Preferred app group type|**Desktop**|
    |Host pool type|**Pooled**|
    |Create Session Host Configuration|**No**|
    |Load balancing algorithm|**Breadth-first**|

    > **Note**: When using the Breadth-first load balancing algorithm, the max session limit parameter is optional.

1. On the **Session hosts** tab of the **Create a host pool** page, specify the following settings and select **Next : Workspace >** (leave other settings with their default values):

    > **Note**: When setting the **Name prefix** value, switch to the Resources tab on the right side of the lab session window and identify the string of characters between *User1-* and the *@* character. Use this string to replace the *random* placeholder.

    |Setting|Value|
    |---|---|
    |Add virtual machines|**Yes**|
    |Resource group|**Defaulted to same as host pool**|
    |Name prefix|**sh**-*random*|
    |Virtual machine type|**Azure virtual machine**|
    |Virtual machine location|The name of the Azure region where you want to deploy your Azure Virtual Desktop environment|
    |Availability options|**No infrastructure redundancy required**|
    |Security type|**Trusted launch virtual machines**|
    |Image|**Windows 11 Enterprise multi-session, Version 23H2 + Microsoft 365 Apps**|
    |Virtual machine size|**Standard DC2s_v3**|
    |Number of VMs|**2**|
    |OS disk type|**Standard SSD**|
    |OS disk size|**Default size (128GiB)**|
    |Boot Diagnostics|**Enable with managed storage account (recommended)**|
    |Virtual network|**az140-vnet11e**|
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

    > **Note**: Wait for the deployment to complete. This might take about 20 minutes.

#### Task 3: Create an Azure Virtual Desktop application group

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, select **Application groups**.
1. On the **Azure Virtual Desktop \| Application groups** page, note the existing, auto-generated **az140-21-hp1-DAG** desktop application group, and select it. 
1. On the **az140-21-hp1-DAG** page, in the **Manage** section of the vertical navigation menu, select **Assignments**.
1. On the **az140-21-hp1-DAG \| Assignments** page, select **+ Add**.
1. On the **Select Microsoft Entra users or user groups** page, select **Groups**, in the search box, type the full name of the **AVD-DAG** group you identified in the first task of this exercise, select the checkbox next to the group name, and click **Select**.
1. Navigate back to the **Azure Virtual Desktop \| Application groups** page, select **+ Create**. 
1. On the **Basics** tab of the **Create an application group** page, specify the following settings and select **Next : Applications >**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**az140-21e-RG**|
    |Host pool|**az140-21-hp1**|
    |Application group type|**Remote App**|
    |Application group name|**az140-21-hp1-Office365-RAG**|

1. On the **Applications** tab of the **Create an application group** page, select **+ Add applications**.
1. On the **Add application** page, specify the following settings and select **Review + add**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Application source|**Start menu**|
    |Application|**Word**|
    |Display name|**Microsoft Word**|
    |Description|**Microsoft Word**|
    |Require command line|**No**|

1. Back on the **Applications** tab of the **Create an application group** page, select **+ Add applications**.
1. On the **Add application** page, specify the following settings and select **Review + add**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Application source|**Start menu**|
    |Application|**Excel**|
    |Display name|**Microsoft Excel**|
    |Description|**Microsoft Excel**|
    |Require command line|**No**|

1. Back on the **Applications** tab of the **Create an application group** page, select **+ Add applications**.
1. On the **Add application** page, specify the following settings and select **Review + add**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Application source|**Start menu**|
    |Application|**PowerPoint**|
    |Display name|**Microsoft PowerPoint**|
    |Description|**Microsoft PowerPoint**|
    |Require command line|**No**|

1. Back on the **Applications** tab of the **Create an application group** page, select **Next : Assignments >**.
1. On the **Assignments** tab of the **Create an application group** page, select **+ Add Microsoft Entra users or user groups**.
1. On the **Select Microsoft Entra users or user groups** page, select **Groups**, type the full name of the **AVD-RemoteApp** group you identified in the first task of this exercise, select the checkbox next to the group name, and click **Select**.
1. Back on the **Assignments** tab of the **Create an application group** page, select **Next : Workspace >**.
1. On the **Workspace** tab of the **Create a workspace** page, specify the following setting and select **Review + create**:

    |Setting|Value|
    |---|---|
    |Register application group|**No**|

1. On the **Review + create** tab of the **Create an application group** page, select **Create**.

    > **Note**: Wait for the Application Group to be created. This should take less than 1 minute. 

    > **Note**: Next you will create an application group based on file path as the application source.

1. In the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, select **Application groups**.
1. On the **Azure Virtual Desktop \| Application groups** page, select **+ Create**. 
1. On the **Basics** tab of the **Create an application group** page, specify the following settings and select **Next : Applications >**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**az140-21e-RG**|
    |Host pool|**az140-21-hp1**|
    |Application group type|**RemoteApp**|
    |Application group name|**az140-21-hp1-Utilities-RAG**|

1. On the **Applications** tab of the **Create an application group** page, select **+ Add applications**.
1. On the **Add application** page, on the **Basics** tab, specify the following settings and select **Next**:

    |Setting|Value|
    |---|---|
    |Application source|**File path**|
    |Application path|**C:\Windows\system32\cmd.exe**|
    |Application identifier|**Command Prompt**|
    |Display name|**Command Prompt**|
    |Description|**Windows Command Prompt**|
    |Require command line|**No**|

1. On the **Icon** tab, specify the following settings and select **Review + add**, then select **Add**:

    |Setting|Value|
    |---|---|
    |Icon path|**C:\Windows\system32\cmd.exe**|
    |Icon index|0|

1. Back on the **Applications** tab of the **Create an application group** page, select **Next : Assignments >**.
1. On the **Assignments** tab of the **Create an application group** page, select **+ Add Microsoft Entra users or user groups**.
1. On the **Select Microsoft Entra users or user groups** page, select **Groups**, type the full name of the **AVD-RemoteApp** group you identified in the first task of this exercise, select the checkbox next to the group name, and click **Select**.
1. Back on the **Assignments** tab of the **Create an application group** page, select **Next : Workspace >**.
1. On the **Workspace** tab of the **Create a workspace** page, specify the following setting and select **Review + create**:

    |Setting|Value|
    |---|---|
    |Register application group|**No**|

1. On the **Review + create** tab of the **Create an application group** page, select **Create**.

    > **Note**: Wait for the Application Group to be created. This should take less than 1 minute. 

#### Task 4: Create an Azure Virtual Desktop workspace

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, select **Workspaces**.
1. On the **Azure Virtual Desktop \| Workspaces** page, select **+ Create**. 
1. On the **Basics** tab of the **Create a workspace** page, specify the following settings and select **Next : Application groups >**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**az140-21e-RG**|
    |Workspace name|**az140-21-ws1**|
    |Friendly name|**az140-21-ws1**|
    |Location|The name of the Azure region into which you deployed resources in the first exercise of this lab or a region close to it|

1. On the **Application groups** tab of the **Create a workspace** page, specify the following settings:

    |Setting|Value|
    |---|---|
    |Register application groups|**Yes**|

1. On the **Workspace** tab of the **Create a workspace** page, select **+ Register application groups**.
1. On the **Add application groups** page, select the plus sign next to the **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG**, and **az140-21-hp1-Utilities-RAG** entries and click **Select**. 
1. Back on the **Application groups** tab of the **Create a workspace** page, select **Review + create**.
1. On the **Review + create** tab of the **Create a workspace** page, select **Create**.

#### Task 5: Grant access to Azure Virtual Desktop host pools

> **Note**: When usign Microsoft Entra joined session hosts, you need to assign to Azure Virtual Desktop users and administrators appropriate Azure role-based access control (RBAC) roles. In particular, the *Virtual Machine User Login* role is required to sign in to session hosts and the *Virtual Machine Administrator Login* role is required for the local administrative privileges. 

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Resource groups** and, on the **Resource groups** page, select **az140-21e-RG**.
1. On the **az140-21e-RG** page, in the vertical navigation menu, select **Access control (IAM)**.
1. On the **az140-21e-RG\|Access control (IAM)** page, select **+ Add** and, in the drop-down menu, select **Add role assignment**.
1. On the **Role** tab of the **Add role assignment** page, ensure that the **Job function roles** tab is selected, in the search textbox, enter **Virtual Machine User Login**, in the list of results, select **Virtual Machine User Login**, and then select **Next**.
1. On the **Members** tab of the **Add role assignment** page, ensure that the **User, group, or service principal** option is selected, click **+ Select members**, in the **Select members** pane, locate the **AVD-RemoteApp** group you identified in the first task of this exercise, and click **Select**.
1. Back on the **Members** tab of the **Add role assignment** page, select **Next**.
1. On the **Assignment type** tab of the **Add role assignment** page, set the **Assignment type** to **Active** and then select **Review + assign**.
1. On the **Review + assign** tab of the **Add role assignment** page, select **Review + assign**. 
1. Back on the **az140-21e-RG\|Access control (IAM)** page, select **+ Add** and, in the drop-down menu, select **Add role assignment**.
1. On the **Role** tab of the **Add role assignment** page, ensure that the **Job function roles** tab is selected, in the search textbox, enter **Virtual Machine Administrator Login**, in the list of results, select **Virtual Machine Administrator Login**, and then select **Next**.
1. On the **Members** tab of the **Add role assignment** page, ensure that the **User, group, or service principal** option is selected, click **+ Select memebers**, in the **Select members** pane, locate the **AVD-DAG** group you identified in the first task of this exercise, and click **Select**.
1. Back on the **Members** tab of the **Add role assignment** page, , set the **Assignment type** to **Active**  and then select **Review + assign** and then select **Review + assign**. 
