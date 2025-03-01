---
title: Monitoring Azure ExpressRoute
description: Start here to learn how to monitor Azure ExpressRoute.
services: expressroute
author: duongau
ms.author: duau
ms.service: expressroute
ms.topic: how-to
ms.custom: subject-monitoring
ms.date: 06/22/2021
---

# Monitoring Azure ExpressRoute

When you have critical applications and business processes relying on Azure resources, you want to monitor those resources for their availability, performance, and operation. 

This article describes the monitoring data generated by Azure ExpressRoute. Azure ExpressRoute uses [Azure Monitor](../azure-monitor/overview.md). If you're unfamiliar with the features of Azure Monitor common to all Azure services that use it, read [Monitoring Azure resources with Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).
 
## ExpressRoute insights

Some services in Azure have a special focused pre-built monitoring dashboard in the Azure portal that provides a starting point for monitoring your service. These special dashboards are called "insights".

ExpressRoute uses Network insights to provide a detailed topology mapping of all ExpressRoute components (peerings, connections, gateways) in relation with one another. Network insights for ExpressRoute also have preloaded metrics dashboard for availability, throughput, packet drops, and gateway metrics. For more information, see [Azure ExpressRoute Insights using Networking Insights](expressroute-network-insights.md).

## Monitoring data 

Azure ExpressRoute collects the same kinds of monitoring data as other Azure resources that are described in [Monitoring data from Azure resources](../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data). 

See [Monitoring Azure ExpressRoute data reference](monitor-expressroute-reference.md) for detailed information on the metrics and logs metrics created by Azure ExpressRoute.

## Collection and routing

Platform metrics and the Activity log are collected and stored automatically, but can be routed to other locations by using a diagnostic setting.  

Resource Logs aren't collected and stored until you create a diagnostic setting and route them to one or more locations.

See [Create diagnostic setting to collect platform logs and metrics in Azure](../azure-monitor/essentials/diagnostic-settings.md) for the detailed process for creating a diagnostic setting using the Azure portal, CLI, or PowerShell. When you create a diagnostic setting, you specify which categories of logs to collect. The categories for *Azure ExpressRoute* are listed in [Azure ExpressRoute monitoring data reference](monitor-expressroute-reference.md#resource-logs).

> [!IMPORTANT]
> Enabling these settings requires additional Azure services (storage account, event hub, or Log Analytics), which may increase your cost. To calculate an estimated cost, visit the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator).

The metrics and logs you can collect are discussed in the following sections.

## Analyzing metrics

You can analyze metrics for *Azure ExpressRoute* with metrics from other Azure services using metrics explorer by opening **Metrics** from the **Azure Monitor** menu. See [Getting started with Azure Metrics Explorer](../azure-monitor/essentials/metrics-getting-started.md) for details on using this tool. 

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/metrics-page.png" alt-text="Screenshot of the metrics dashboard for ExpressRoute.":::

For reference, you can see a list of [all resource metrics supported in Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

* To view **ExpressRoute** metrics, filter by Resource Type *ExpressRoute circuits*. 
* To view **Global Reach** metrics, filter by Resource Type *ExpressRoute circuits* and select an ExpressRoute circuit resource that has Global Reach enabled. 
* To view **ExpressRoute Direct** metrics, filter Resource Type by *ExpressRoute Ports*. 

Once a metric is selected, the default aggregation will be applied. Optionally, you can apply splitting, which will show the metric with different dimensions.

## Analyzing logs

Data in Azure Monitor Logs is stored in tables where each table has its own set of unique properties.  

All resource logs in Azure Monitor have the same fields followed by service-specific fields. The common schema is outlined in [Azure Monitor resource log schema](../azure-monitor/essentials/resource-logs-schema.md#top-level-common-schema). The schema for ExpressRoute resource logs is found in the [Azure ExpressRoute Data Reference](monitor-expressroute-reference.md#schemas). 

The [Activity log](../azure-monitor/essentials/activity-log.md) is a platform logging that provides insight into subscription-level events. You can view it independently or route it to Azure Monitor Logs, where you can do much more complex queries using Log Analytics.

ExpressRoute stores data in the following tables.

| Table | Description |
| ----- | ----------- |
| AzureDiagnostics | Common table used by multiple services to store Resource logs. Resource logs from ExpressRoute can be identified with `MICROSOFT.NETWORK`. |
| AzureMetrics | Metric data emitted by ExpressRoute that measure their health and performance. 

To view these tables, navigate to your ExpressRoute circuit resource and select **Logs** under *Monitoring*.

### Sample Kusto queries

Here are some queries that you can enter into the Log search bar to help you monitor your Azure ExpressRoute resources. These queries work with the [new language](../azure-monitor/logs/log-query-overview.md).

* To query for BGP route table learned over the last 12 hours.

    ```Kusto
    AzureDiagnostics
    | where TimeGenerated > ago(12h)
    | where ResourceType == "EXPRESSROUTECIRCUITS"
    | project TimeGenerated, ResourceType , network_s, path_s, OperationName
    ```

* To query for BGP informational messages by level, resource type, and network.

    ```Kusto
    AzureDiagnostics
    | where Level == "Informational"
    | where ResourceType == "EXPRESSROUTECIRCUITS"
    | project TimeGenerated, ResourceId , Level, ResourceType , network_s, path_s
    ```

* To query for Traffic graph BitInPerSeconds in the last one hour.
    
    ```Kusto
    AzureMetrics
    | where MetricName == "BitsInPerSecond"
    | summarize by Average, bin(TimeGenerated, 1h), Resource
    | render timechart
    ```

* To query for Traffic graph BitOutPerSeconds in the last one hour.
    
    ```Kusto
    AzureMetrics
    | where MetricName == "BitsOutPerSecond"
    | summarize by Average, bin(TimeGenerated, 1h), Resource
    | render timechart
    ```

* To query for graph of ArpAvailability in 5-minute intervals.
    
    ```Kusto
    AzureMetrics
    | where MetricName == "ArpAvailability"
    | summarize by Average, bin(TimeGenerated, 5m), Resource
    | render timechart
    ```

* To query for graph of BGP availability in 5-minute intervals.

    ```Kusto
    AzureMetrics
    | where MetricName == "BGPAvailability"
    | summarize by Average, bin(TimeGenerated, 5m), Resource
    | render timechart
    ```

## Alerts

Azure Monitor alerts proactively notify you when important conditions are found in your monitoring data. They allow you to identify and address issues in your system before your customers notice them. You can set alerts on [metrics](../azure-monitor/alerts/alerts-metric-overview.md), [logs](../azure-monitor/alerts/alerts-unified-log.md), and the [activity log](../azure-monitor/alerts/activity-log-alerts.md). Different types of alerts have benefits and drawbacks.

The following table lists common and recommended alert rules for ExpressRoute.

| Alert type | Condition | Description  |
|:---|:---|:---|
| ARP availability down | Dimension name: Peering Type, Aggregation type: Avg, Operator: Less than, Threshold value: 100% | When ARP availability is down for a peering type. |
| BGP availability down | Dimension name: Peer, Aggregation type: Avg, Operator: Less than, Threshold value: 100% | When BGP availability is down for a peer. |

### Alerts for ExpressRoute gateway connections

1. To configure alerts, navigate to **Azure Monitor**, then select **Alerts**.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/eralertshowto.jpg" alt-text="alerts":::
2. Select **+ Select Target** and select the ExpressRoute gateway connection resource.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alerthowto2.jpg" alt-text="target":::
3. Define the alert details.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alerthowto3.jpg" alt-text="action group":::
4. Define and add the action group.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/actiongroup.png" alt-text="add action group":::


### Alerts based on each peering

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/basedpeering.jpg" alt-text="each peering":::

### Configure alerts for activity logs on circuits

In the **Alert Criteria**, you can select **Activity Log** for the Signal Type and select the Signal.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alertshowto6activitylog.jpg" alt-text="activity logs":::

## Next steps

* See [Monitoring ExpressRoute data reference](monitor-expressroute-reference.md) for a reference of the metrics, logs, and other important values created by ExpressRoute.
* See [Monitoring Azure resources with Azure Monitor](../azure-monitor/overview.md) for details on monitoring Azure resources.
