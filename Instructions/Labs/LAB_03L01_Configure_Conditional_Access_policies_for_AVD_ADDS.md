---
lab:
    title: 'Lab: Configure Conditional Access policies for WVD (AD DS)'
    module: 'Module 3: Manage Access and Security'
---

# Lab - Configure Conditional Access policies for WVD (AD DS)
# Student lab manual

## Lab dependencies

- An Azure subscription
- A Microsoft account or an Azure AD account with the Global Administrator role in the Azure AD tenant associated with the Azure subscription and with the Owner or Contributor role in the Azure subscription
- The completed lab **Prepare for deployment of Azure Virtual Desktop (AD DS)**
- The completed lab **Deploy host pools and session hosts by using the Azure portal (AD DS)**

## Estimated Time

60 minutes

## Lab scenario

You need to control access to a deployment of Azure Virtual Desktop in an Active Directory Domain Services (AD DS) environment by using Azure Active Directory (Azure AD) conditional access.

## Objectives
  
After completing this lab, you will be able to:

- Prepare for Azure Active Directory (Azure AD)-based Conditional Access for Azure Virtual Desktop
- Implement Azure AD-based Conditional Access for Azure Virtual Desktop

## Lab files

- None 

## Instructions

### Exercise 1: Prepare for Azure AD-based Conditional Access for Azure Virtual Desktop

The main tasks for this exercise are as follows:

1. Configure Azure AD Premium P2 licensing
1. Configure Azure AD Multi-Factor Authentication (MFA)
1. Register a user for Azure AD MFA
1. Configure hybrid Azure AD join
1. Trigger Azure AD Connect delta synchronization

#### Task 1: Configure Azure AD Premium P2 licensing

>**Note**: Premium P1 or P2 licensing of Azure AD is required in order to implement Azure AD Conditional Access. You will use a 30-day trial for this lab.

1. From your lab computer, start a web browser, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab and the Global Administrator role in the Azure AD tenant associated with that subscription.
1. In the Azure portal, search for and select **Azure Active Directory** to navigate to the Azure AD tenant associated with the Azure subscription you are using for this lab.
1. On the Azure Active Directory blade, in the vertical menu bar on the left side, in the **Manage** section, click **Users**. 
1. On the **Users | All users (Preview)** blade, select **aduser5**.
1. On the **aduser5 | Profile** blade, in the toolbar, click **Edit**, in the **Settings** section, in the **Usage location** dropdown list, select country where the lab environment is located and, in the toolbar, click **Save**.
1. On the **aduser5 | Profile** blade, in the **Identity** section, identify the user principal name of the **aduser5** account.

    >**Note**: Record this value. You will need it later in this lab.

1. On the **Users | All users (Preview)** blade, select the user account you used to sign at the beginning of this task and repeat the previous step in case your account does not have the **Usage location** assigned. 

    >**Note**: The **Usage location** property must be set in order to assign an Azure AD Premium P2 licenses to user accounts.

1. On the **Users | All users (Preview)** blade, select the **aadsyncuser** user account and identify its user principal name.

    >**Note**: Record this value. You will need it later in this lab.

1. In the Azure portal, navigate back to the **Overview** blade of the Azure AD tenant and, in the vertical menu bar on the left side, in the **Manage** section, click **Licenses**.
1. On the **Licenses \| Overview** blade, in the vertical menu bar on the left side, in the **Manage** section, click **All products**.
1. On the **Licenses \| All products** blade, in the toolbar, click **+ Try/Buy**.
1. On the **Activate** blade, click **Free trial** in the **ENTERPRISE MOBILITY + SECURITY E5** section and then click **Activate**. 
1. While on the **Licenses \| Overview** blade, refresh the browser window to verify that the activation was successful. 
1. On the **Licenses - All products** blade, select the **Enterprise Mobility + Security E5** entry. 
1. On the **Enterprise Mobility + Security E5** blade, in the toolbar, click **+ Assign**.
1. On the **Assign license** blade, click **Add users and groups**, on the **Add users and groups** blade, select **aduser5** and your user accounts, and click **Select**.
1. Back on the **Assign license** blade, click **Assignment options**, on the **Assignment options** blade, verify that all options are enabled, click **Review + assign**, click **Assign**.

#### Task 2: Configure Azure AD Multi-Factor Authentication (MFA)

1. On your lab computer, in the web browser displaying the Azure portal, navigate back to the **Overview** blade of the Azure AD tenant and, in the vertical menu on the left side, in the **Manage** section, click **Properties**.
1. On the **Properties** blade of your Azure AD tenant, at the very bottom of the blade, select the **Manage Security deaults** link.
1. On the **Enable Security defaults** blade, if needed, select **No**, select the **My organization is using Conditional Access** checkbox, and select **Save**.
1. On your lab computer, in the web browser displaying the Azure portal, navigate back to the **Overview** blade of the Azure AD tenant and, in the vertical menu on the left side, in the **Manage** section, click **Security**.
1. On the **Security | Getting started** blade, in the vertical menu on the left side, in the **Manage** section, click **Identity Protection**.
1. On the **Identity Protection | Overview** blade, in the vertical menu on the left side, in the **Protect** section, click **MFA registration policy**.
1. On the **Identity Protection | MFA registration policy** blade, in the **Assignments** section of the **Multi-factor authentication registration policy**, click **All users**, on the **Include** tab, click the **Select individuals and groups** option, on the **Select users**, click **aduser5**, click **Select**, at the bottom of the blade, set the **Enforce policy** switch to **On**, and click **Save**.

#### Task 3: Register a user for Azure AD MFA

1. On your lab computer, open an **InPrivate** web browser session, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing the **user5** user principal name you identified earlier in this exercise and the **Pa55w.rd1234** as the password.
1. When presented with the message **More information required**, click **Next**. This will automatically redirect your browser to the **Microsoft Authenticator** page.
1. On the **Microsoft Authenticator** page, click the link **I want to set up a different method**.
1. In the **Choose a different method** dialog box, in the **Which method would you like to use?** dropdown list, select **Phone** an click **Confirm**.
1. On the **Phone** page, provide your phone number, verify that the **Text me a code** option is selected, and click **Next**.
1. On the **Phone** page, type the code included in the text you received on your phone and click **Next**.
1. Ensure that the SMS was successfully verified and your phone registered successfully and click **Next**.
1. On the **Success!** page, click **Done**.
1. On the Azure portal page, in the upper right corner, click the icon representing the user avatar, click **Sign out**, and close the **In private** browser window. 

#### Task 4: Configure hybrid Azure AD join

> **Note**: This functionality can be leveraged to implement additional security when setting up Conditional Access for devices based on their Azure AD join status.

1. On the lab computer, in the web browser displaying the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **az140-dc-vm11**.
1. On the **az140-dc-vm11** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **az140-dc-vm11 \| Connect** blade, in the **IP address** drop-down list, select the **Load balancer DNS name** entry, and then select **Download RDP File**.
1. When prompted, sign in with the following credentials:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\\Student**|
   |Password|**Pa55w.rd1234**|

1. Within the Remote Desktop session to **az140-dc-vm11**, in the **Start** menu, expand the **Azure AD Connect** folder and select **Azure AD Connect**.
1. On the **Welcome to Azure AD Connect** page of the **Microsoft Azure Active Directory Connect** window, select **Configure**.
1. On the **Additional tasks** page in the **Microsoft Azure Active Directory Connect** window, select **Configure device options** and select **Next**.
1. On the **Overview** page in the **Microsoft Azure Active Directory Connect** window, review the information regarding **Hybrid Azure AD join** and **Device writeback** and select **Next**.
1. On the **Connect to Azure AD** page in the **Microsoft Azure Active Directory Connect** window, authenticate by using the credentials of the **aadsyncuser** user account you created in the previous exercise and select **Next**. 

   > **Note**: Provide the userPrincipalName attribute of the **aadsyncuser** account you recorded earlier in this lab and specify **Pa55w.rd1234** as its password.

1. On the **Device options** page in the **Microsoft Azure Active Directory Connect** window, ensure that the **Configure Hybrid Azure AD join** option is selected and select **Next**. 
1. On the **Device operating systems** page in the **Microsoft Azure Active Directory Connect** window, select the **Windows 10 or later domain-joined devices** checkbox and select **Next**. 
1. On the **SCP configuration** page in the **Microsoft Azure Active Directory Connect** window, select the checkbox next to the **adatum.com** entry, in the **Authentication Service** drop-down list, select **Azure Active Directory** entry, and select **Add**. 
1. When prompted, in the **Enterprise Admin Credentials** dialog box, specify the following credentials, and select **OK**:

   |Setting|Value|
   |---|---|
   |User Name|**ADATUM\Student**|
   |Password|**Pa55w.rd1234**|

1. Back on the **SCP configuration** page in the **Microsoft Azure Active Directory Connect** window, select **Next**.
1. On the **Ready to configure** page in the **Microsoft Azure Active Directory Connect** window, select **Configure** and, once the configuration completes, select **Exit**.
1. Within the Remote Desktop session to **az140-dc-vm11**, start **Windows PowerShell ISE** as administrator.
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console, run the following to move the **az140-cl-vm11** computer account to the **WVDClients** organizational unit (OU):

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, in the **Start** menu, expand the **Azure AD Connect** folder and select **Azure AD Connect**.
1. On the **Welcome to Azure AD Connect** page of the **Microsoft Azure Active Directory Connect** window, select **Configure**.
1. On the **Additional tasks** page in the **Microsoft Azure Active Directory Connect** window, select **Customize synchronization options** and select **Next**.
1. On the **Connect to Azure AD** page in the **Microsoft Azure Active Directory Connect** window, authenticate by using the credentials of the **aadsyncuser** user account you created in the previous exercise and select **Next**. 

   > **Note**: Provide the userPrincipalName attribute of the **aadsyncuser** account you recorded earlier in this lab and specify **Pa55w.rd1234** as its password.

1. On the **Connect your directories** page in the **Microsoft Azure Active Directory Connect** window, select **Next**.
1. On the **Domain and OU filtering** page in the **Microsoft Azure Active Directory Connect** window, ensure that the option **Sync selected domains and OUs** is selected, expand the **adatum.com** node, ensure that the checkbox next to the **ToSync** OU is selected, select the checkbox next to the **WVDClients** OU, and select **Next**.
1. On the **Optional features** page in the **Microsoft Azure Active Directory Connect** window, accept the default settings, and select **Next**.
1. On the **Ready to configure** page in the **Microsoft Azure Active Directory Connect** window, ensure that the checkbox **Start the synchronization process when configuration completes** is selected and select **Configure**.
1. Review the information on the **Configuration complete** page and select **Exit** to close the **Microsoft Azure Active Directory Connect** window.

#### Task 5: Trigger Azure AD Connect delta synchronization

1. Within the Remote Desktop session to **az140-dc-vm11**, switch to the **Administrator: Windows PowerShell ISE** window.
1. Within the Remote Desktop session to **az140-dc-vm11**, from the **Administrator: Windows PowerShell ISE** console pane, run the following to trigger Azure AD Connect delta synchronization:

   ```powershell
   Start-ADSyncSyncCycle -PolicyType Delta
   ```

1. Within the Remote Desktop session to **az140-dc-vm11**, start Microsoft Edge and navigate to the [Azure portal](https://portal.azure.com). When prompted, sign in by using the Azure AD credentials of the user account with the Global Administrator role in the Azure AD tenant associated with the Azure subscription you are using in this lab.
1. Within the Remote Desktop session to **az140-dc-vm11**, in the Microsoft Edge window displaying the Azure portal, search for and select **Azure Active Directory** to navigate to the Azure AD tenant associated with the Azure subscription you are using for this lab.
1. On the Azure Active Directory blade, in the vertical menu bar on the left side, in the **Manage** section, click **Devices**. 
1. On the **Devices | All devices** blade, review the list of devices and verify that the **az140-cl-vm11** device is listed with the **Hybrid Azure AD joined** entry in the **Join Type** column.

   > **Note**: You might have to wait a few minutes for the synchronization to take efect before the device appears in the Azure portal.

### Exercise 2: Implement Azure AD-based Conditional Access for Azure Virtual Desktop

The main tasks for this exercise are as follows:

1. Create an Azure AD-based Conditional Access policy for all Azure Virtual Desktop connections
1. Test the Azure AD-based Conditional Access policy for all Azure Virtual Desktop connections
1. Modify the Azure AD-based Conditional Access policy to exclude hybrid Azure AD joined computers from the MFA requirement
1. Test the modified Azure AD-based Conditional Access policy

#### Task 1: Create an Azure AD-based Conditional Access policy for all Azure Virtual Desktop connections

>**Note**: In this task, you will configure an Azure AD-based Conditional Access policy that requires MFA to sign in to a Azure Virtual Desktop session. The policy will also enforce reauthentication after the first 4 hours following a successful authentication.

1. On your lab computer, in the web browser displaying the Azure portal, navigate back to the **Overview** blade of the Azure AD tenant and, in the vertical menu on the left side, in the **Manage** section, click **Security**.
1. On the **Security \| Getting started** blade, in the vertical menu on the left side, in the **Protect** section, click **Conditional Access**.
1. On the **Conditional Access \| Policies** blade, in the toolbar, click **+ New policy**. 
1. On the **New** blade, configure the following settings:

   - In the **Name** text box, type **az140-31-wvdpolicy1**
   - In the **Assignments** section, click **Users and groups**, click the **Select users and groups** option, click the **Users and groups** checkbox, on the **Select** blade, click **aduser5**, and click **Select**.
   - In the **Assignments** section, click **Cloud apps or actions**, ensure that in the **Select what this policy applies to** switch, the **Cloud apps** option is selected, click the **Select apps** option, on the **Select** blade, select the checkbox next to the **Windows Virtual Desktop** entry, and click **Select**. 
   - In the **Assignments** section, click **Conditions**, click **Client apps**, on the **Client apps** blade, set the **Configure** switch to **Yes**, ensure that both the **Browser** and **Mobile apps and desktop clients** checkboxes are selected, and click **Done**.
   - In the **Access controls** section, click **Grant**, on the **Grant** blade, ensure that the **Grant access** option is selected, select the **Require multi-factor authentication** checkbox and click **Select**.
   - In the **Access controls** section, click **Session**, on the **Session** blade, select the **Sign-in frequency** checkbox, in the first textbox, type **4**, in the **Select units** dropdown list, select **Hours**, leave the **Persistent browser session** checkbox cleared, and click **Select**.
   - Set the **Enable policy** switch to **On**.

1. On the **New** blade, click **Create**. 

#### Task 2: Test the Azure AD-based Conditional Access policy for all Azure Virtual Desktop connections

1. On your lab computer, open an **InPrivate** web browser session, navigate to the [Azure portal](https://portal.azure.com), and sign in by providing the **user5** user principal name you identified earlier in this exercise and the **Pa55w.rd1234** as the password.

   > **Note**: Verify that you are not prompted to authenticate via MFA.

1. In the **InPrivate** web browser session, navigate to the Azure Virtual Desktop HTML5 web client page at [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Note**: Verify that this will automatically trigger authentication via MFA.

1. In the **Enter code** pane, type the code included in the text you received on your phone and click **Verify**.
1. On the **All Resources** page, click **Command Prompt**, on the **Access local resources** pane, clear the **Printer** checkbox, and click **Allow**.
1. When prompted, in the **Enter your credentials**, in the **User name** textbox type the user principal name of **aduser5** and, in the **Password** textbox, type **Pa55word1234** and click **Submit**.
1. Verify that the **Command Prompt** Remote App was launched successfully.
1. In the **Command Prompt** Remote App window, at the command prompt, type **logoff** and press the **Enter** key.
1. Back on the **All Resources** page, in the upper right corner, click **aduser5**, in the dropdown menu, click **Sign Out**, and close the **InPrivate** web browser window.

#### Task 3: Modify the Azure AD-based Conditional Access policy to exclude hybrid Azure AD joined computers from the MFA requirement

>**Note**: In this task, you will modify the Azure AD-based Conditional Access policy that requires MFA to sign in to a Azure Virtual Desktop session such that connections originating from Azure AD joined computers will not require MFA.

1. On your lab computer, in the browser window displaying the Azure portal, on the **Conditional Access | Policies** blade, click the entry representing the **az140-31-wvdpolicy1** policy.
1. On the **az140-31-wvdpolicy1** blade, in the **Assignments** section, click **Conditions**, in the list of conditions, click **Device state**, on the **Device state** blade, set the **Configure** switch to **Yes**, click the **Exclude** tab, on the **Exclude** tab, select **Device Hybrid Azure AD joined**, and click **Done**.
1. On the **az140-31-wvdpolicy1** blade, in the **Access controls** section, click **Grant**, on the **Grant** blade, select the **Require Hybrid Azure AD joined device** checkbox, ensure that the **Require one of the selected controls** option is enabled, and click **Select**.
1. On the **az140-31-wvdpolicy1** blade, click **Save**.

#### Task 4: Test the modified Azure AD-based Conditional Access policy

1. On your lab computer, in the browser window displaying the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, select the **az140-cl-vm11** entry.
1. On the **az140-cl-vm11** blade, select **Connect**, in the drop-down menu, select **RDP**, and then select **Download RDP File**.
1. When prompted, sign in as the **ADATUM\\aduser5** user with **Pa55w.rd1234** as its password.
1. Within the Remote Desktop session to **az140-cl-vm11**, start Microsoft Edge and navigate to navigate to the Azure Virtual Desktop HTML5 web client page at [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Note**: Verify that this time you will not be prompted to authenticate via MFA. This is because **az140-cl-vm11** is Hybrid Azure AD-joined.

1. On the **All Resources** page, click **Command Prompt**, on the **Access local resources** pane, clear the **Printer** checkbox, and click **Allow**.
1. When prompted, in the **Enter your credentials**, in the **User name** textbox type the user principal name of **aduser5** and, in the **Password** textbox, type **Pa55word1234** and click **Submit**.
1. Verify that the **Command Prompt** Remote App was launched successfully.
1. In the **Command Prompt** Remote App window, at the command prompt, type **logoff** and press the **Enter** key.
1. Back on the **All Resources** page, in the upper right corner, click **aduser5**, in the dropdown menu, click **Sign Out**.
1. Within the Remote Desktop session to **az140-cl-vm11**, click **Start**, in the vertical bar directly above the **Start** button, click the icon representing the signed in user account, and, in the pop-up menu, click **Sign out**.
