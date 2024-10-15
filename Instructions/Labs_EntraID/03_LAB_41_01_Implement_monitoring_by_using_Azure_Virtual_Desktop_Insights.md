---
lab:
    title: 'Lab: Implement monitoring by using Azure Virtual Desktop Insights'
    module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Lab - Implement monitoring by using Azure Virtual Desktop Insights
# Student lab manual

## Lab dependencies

- An Azure subscription you will be using in this lab.
- A Microsoft Entra user account with the Owner or Contributor role in the Azure subscription you will be using in this lab and with the permissions sufficient to join devices to the Entra tenant associated with that Azure subscription.
- The lab *Deploy host pools and session hosts by using the Azure portal (Entra ID)* completed
- The lab *Manage host pools and session hosts by using the Azure portal (Entra ID)* completed

## Estimated Time

25 minutes

## Lab scenario

You have an existing Azure Virtual Desktop environment. You want to monitor the status and activities of the environment.

## Objectives
  
After completing this lab, you will be able to:

- implement monitoring of an Azure Virtual Desktop environment

## Lab files

- None

## Instructions

### Exercise 1: Implement monitoring of an Azure Virtual Desktop environment
  
The main tasks for this exercise are as follows:

1. Register the Azure subscription with the Microsoft.Insights resource provider
1. Create an Azure Log Analytics workspace
1. Set up the Virtual Desktop Insights configuration workbook

#### Task 1: Register the Azure subscription with the Microsoft.Insights resource provider

> **Note**: Azure Virtual Desktop Insights rely on the Microsoft.Insights resource provider, so you need to first register it within the Azure subscription you are using for this lab. In the *Deploy host pools and session hosts by using the Azure portal (Entra ID)* lab, you performed this task by using Azure PowerShell. In this lab, you will accomplish it by using the Azure portal (either method is supported and available).

1. If needed, from the lab computer, start a web browser, navigate to the Azure portal and sign in by providing credentials of a user account with the Owner role in the subscription you will be using in this lab.

    > **Note**: Use the credentials of the `User1-` account listed on the Resources tab on the right side of the lab session window.

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Subscriptions**, on the **Subscriptions** page, select the Azure subscription you are using in this lab, and, in the vertical navigation menu, in the **Settings** section, select **Resource providers**.
1. On the **Resource providers** tab, in the search text box, enter **Microsoft.Insights**, in the list of results, select the small circle to the left of the **Microsoft.Insights** entry, and then select **Register**.

    > **Note**: Wait for the registration process to complete. This typically takes about 1 minute. Use the **Refresh** toolbar button to display the up-to-date value of the registration status.

#### Task 2: Create an Azure Log Analytics workspace

> **Note**: Azure Virtual Desktop Insights is a dashboard built on Azure Monitor Workbooks that facilitates monitoring of Azure Virtual Desktop environments. 

> **Note**: You can only monitor Autoscale operations with Insights with pooled host pools. 

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Log Analytics workspaces**, on the **Log Analytics workspaces** page, select **+ Create**.
1. On the **Basics** tab of the **Create Log Analytics workspace** page, specify the following settings and select **Review + create**:

    |Setting|Value|
    |---|---|
    |Subscription|the name of the Azure subscription you are using in this lab|
    |Resource group|the name of a new resource group **az140-411e-RG**|
    |Name|**az140-laworkspace41e**|
    |Region|the name of the Azure region where you deployed the Azure Virtual Desktop environment|

1. On the **Review + Create** page, select **Create**.

    > **Note**: Wait for the provisioning process to complete. This typically takes about 1 minute.

    > **Note**: Next, you need to enable data collection in the newly provisioned Log Analytics workspace of diagnostics from the Azure Virtual Desktop environment, performance counters from the session hosts, and Windows Event Logs from the Azure Virtual Desktop session hosts.

#### Task 3: Set up the Virtual Desktop Insights configuration workbook

> **Note**: When opening Azure Virtual Desktop Insights for the first time, you need to set up Azure Virtual Desktop Insights to target your Azure Virtual Desktop environment.

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, in the vertical navigation menu, in the **Monitoring** section, select **Workbooks**.
1. In the list of **Windows Virtual Desktop** workbooks, in the **Windows Virtual Desktop** section, select the **Insights** workbook.
1. On the **Azure Virtual Desktop \| Workbooks \| Insights** page, review the warning messages indicating that the workspace and session hosts are not sending data to the workspace and then select the **Configuration workbook** link to repair the issue.
1. On the **CheckAMAConfiguration** page, on the **Resource diagnostics settings** tab, in the **Log Analytics workspace** drop-down list, select **az140-laworkspace41e**.
1. On the **CheckAMAConfiguration** page, on the **Resource diagnostics settings** tab, in the **Host pool az140-21-hp1** section, note the warning message indicating that no existing diagnostic configuration was found for the selected host pool and then select **Configure host pool**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: This effectively enables the following diagnostics tables in the target Log Analytics workspace:
    - Management Activities
    - Feed
    - Connections
    - Errors
    - Checkpoints
    - HostRegistration
    - AgentHealthStatus

    > **Note**: Wait for the deployment to complete. This typically takes less than 1 minute.

1. On the **CheckAMAConfiguration** page, on the **Resource diagnostics settings** tab, select the **Refresh** icon (a circular arrow) in the toolbar.
1. Review the **Host pool az140-21-hp1** section and verify that the diagnostic settings are enabled for **allLogs**.
1. While on the **Resource diagnostics settings** tab, scroll down to the **Workspace az140-21-ws1** section and then select **Configure workspace**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: This effectively configures the workspace for **allLogs**.

    > **Note**: Wait for the deployment to complete. This typically takes less than 1 minute.

1. On the **CheckAMAConfiguration** page, on the **Resource diagnostics settings** tab, select the **Refresh** icon (a circular arrow) in the toolbar.
1. Review the **Workspace az140-21-ws1** section and verify that the diagnostic settings are enabled for **allLogs** and that thre are no remaining warning messages.
1. Navigate to the top of the **CheckAMAConfiguration** page and switch to the **Select host data settings** tab.
1. On the **Select host data settings** tab, in the **Create DCR** section, in the **Workspace destination** drop-down list, select **az140-laworkspace41e** and then select **Create data collection rule**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: Wait for the deployment to complete. This typically takes less than 1 minute.

1. On the **CheckAMAConfiguration** page, on the **Select host data settings** tab, select the **Refresh** icon (a circular arrow) in the toolbar.

    > **Note**: Before you proceed, make sure that the newly created DCR is listed in the **Available DCRs** subsection of the **Create DCR** section. If that is not the case, wait for another minute and refresh the page again.

1. On the **Select host data settings** tab, in the **Selected DCR** drop-down list, select the entry starting with **microsoft-avdi-** prefix.
1. On the **Select host data settings** tab, in the **DCR associations** section, select **Deploy Association**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: This effectively associates the newly created DCR with the session hosts in the **az140-21-hp1** host pool.

    > **Note**: Wait for the deployment to complete. This typically takes less than 1 minute.

1. On the **CheckAMAConfiguration** page, on the **Select host data settings** tab, select the **Refresh** icon (a circular arrow) in the toolbar.
1. On the **Select host data settings** tab, in the **Session hosts missing Azure Monitor extension** section, select **Add extension**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: This effectively installs the Azure Monitor extension on the session hosts in the **az140-21-hp1** host pool.

    > **Note**: Wait for the deployment to complete. This might take about 1 minute.

1. On the **CheckAMAConfiguration** page, on the **Select host data settings** tab, select the **Refresh** icon (a circular arrow) in the toolbar.
1. Verify that there are no error or warning messages displayed. 
1. Navigate to the top of the **CheckAMAConfiguration** page, select the **Data generated** tab, and then select the **Refresh** icon (a circular arrow) in the toolbar.
1. Review the sections displaying graphs representing collected data, including **Billed data over last 24hrs**, **Performance Counters**, and **Events**.

    > **Note**: Use the **Billed data over last 24hrs** section to monitor data ingestion. You are responsible for Log Analytics charges for data storage and ingestion.

1. In the web browser displaying the Azure portal, navigate back to the **Azure Virtual Desktop** page and, in the **Monitoring** section of the vertical navigation menu, select **Insights**.
1. On the **Azure Virtual Desktop \| Insights** page, review the content of the **Overview** tab, including the **Capacity** section, **Connection diagnostics: % of users able to connect**, **Connection performance: Time to connect (new sessions)**, and **Utilization** telemetry. 
1. Next, review all the remaining tabs on the **Azure Virtual Desktop \| Insights** page, including **Connection Reliability**, **Connection Diagnostics**, **Connection Performance**, **Users**, **Utilization**, **Clients**, and **Alerts**.

    > **Note**: Consider revisiting these tabs of the Insights page once you complete the subsequent labs to review the charts representing collected telemetry.
