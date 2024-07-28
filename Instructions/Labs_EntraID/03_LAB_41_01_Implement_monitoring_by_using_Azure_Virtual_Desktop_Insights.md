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
    |Virtual network name|**az140-laworkspace41e**|
    |Region|the name of the Azure region where you deployed the Azure Virtual Desktop environment|

1. On the **Review + Create** page, select **Create**.

    > **Note**: Wait for the provisioning process to complete. This typically takes about 1 minute.

    > **Note**: Next, you need to enable data collection in the newly provisioned Log Analytics workspace of diagnostics from the Azure Virtual Desktop environment, performance counters from the session hosts, and Windows Event Logs from the Azure Virtual Desktop session hosts.

#### Task 3: Set up the Virtual Desktop Insights configuration workbook

> **Note**: When opening Azure Virtual Desktop Insights for the first time, you need to set up Azure Virtual Desktop Insights to target your Azure Virtual Desktop environment.

1. From the lab computer, in the web browser displaying the Azure portal, search for and select **Azure Virtual Desktop** and, on the **Azure Virtual Desktop** page, in the vertical navigation menu, in the **Monitoring** section, select **Workbooks**.
1. in the list of **Windows Virtual Desktop** workbooks, select **Check Configuration**.
1. On the **Resource diagnostics settings** tab, in the **Log Analytics workspace** drop-down list, select **az140-laworkspace41e**.
1. On the **Resource diagnostics settings** tab, select **Configure host pool**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: This effectively enables the following diagnostics tables in the target Log Analytics workspace:
    - Management Activities
    - Feed
    - Connections
    - Errors
    - Checkpoints
    - HostRegistration
    - AgentHealthStatus

    > **Note**: Wait for the deployment process to complete. This typically takes less than 1 minute.

1. Back on the **Resource diagnostics settings** tab, select the **Refresh** toolbar icon (a circular arrow) and then select **Configure workspace**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: This effectively enables the following diagnostics tables in the target Log Analytics workspace (if not already enabled):
    - Management Activities
    - Feed
    - Errors
    - Checkpoints

    > **Note**: Wait for the deployment process to complete. This typically takes less than 1 minute.

1. Refresh the web browser page and verify that the **Resource diagnostics settings** tab displays the target host pool (**az140-21-hp1**) and workspace (**az140-21-ws1**).

    > **Note**: Next, you will configure session host data collection settings by leveraging the capabilities of Azure Monitor Agent.

1. On the **Azure Virtual Desktop \| Workbooks \| Check Configuration** page, select the **Session host data settings** tab.
1. On the **Session host data settings** tab, in the **Log Analytics workspace** drop-down list, select **az140-laworkspace41e**.
1. On the **Session host data settings** tab, in the **Session hosts** section, verify that all session hosts are listed and then select **Add hosts to workspace**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: Wait for the deployment process to complete. This might take about 2 minutes.

1. Back on the **Session host data settings** tab of the **Azure Virtual Desktop \| Workbooks \| Check Configuration** page, select the **Refresh** toolbar icon (a circular arrow).
1. On the **Session host data settings** tab, in the **Performance counters** section, review the missing counters and select **Configure performance counters**.
1. In the **Deploy Template** pane, select **Apply Config**.

    > **Note**: Wait for the deployment process to complete. This should take less than 1 minute.

1. Back on the **Session host data settings** tab of the **Azure Virtual Desktop \| Workbooks \| Check Configuration** page, select the **Refresh** toolbar icon (a circular arrow).
1. On the **Session host data settings** tab, in the **Windows event logs** section, review the missing event logs and select **Configure events**.
1. In the **Deploy Template** pane, select **Deploy**.

    > **Note**: Wait for the deployment process to complete. This should take less than 1 minute.

1. Refresh the web browser page and verify that no issues regarding missing performance counters or events are reported.
1. On the **Azure Virtual Desktop \| Workbooks \| Check Configuration** page, select the **Data generated** tab.

    > **Note**: Use this section to monitor data ingestion. You are responsible for Log Analytics charges for data storage and ingestion.

1. In the web browser displaying the Azure portal, in the **Monitoring** section of the vertical navigation menu, select **Insights**.
1. On the **Azure Virtual Desktop \| Insights** page, review the content of the **Overview** tab, including the **Capacity** section, **Connection diagnostics: % of users able to connect**, **Connection performance: Time to connect (new sessions)**, and **Utilization** telemetry. 
1. Next, review all the remaining tabs on the **Azure Virtual Desktop \| Insights** page, including **Connection Reliability**, **Connection Diagnostics**, **Connection Performance**, **Users**, **Utilization**, **Clients**, and **Alerts**.

    > **Note**: Consider revisiting these tabs of the Insights page once you complete the subsequent labs to review the charts representing collected telemetry.