# Module: Design a solution to log and monitor Azure resources

**Module on Learn:** [Design a solution to log and monitor Azure resources](https://learn.microsoft.com/en-us/training/modules/design-solution-to-log-monitor-azure-resources/)

**Status:** Stub — add sections as you complete each unit on Learn.

---

## Your notes

### Logging & monitoring design

- **Summary:**
- **What to log vs what to alert on:**
- **Remember for the exam:**

---

_Add unit headings here to mirror the module TOC once you start the module._

Azure Monitor
Data Collection Rules (DCRs), which define what data to collect, how to filter and transform it, and where to send it

write log queries using Kusto Query Language (KQL) to analyze your collected data. KQL supports filtering, aggregation, joins, and time-series analysis.

create DCRs in Azure Monitor and assign them to your VMs and hybrid machines using resource associations. Use Azure Policy to enforce DCR assignment at scale.

Define metrics to monitor about your Tailwind Traders resources, such as peak usage rates, access information, workloads, or incident scenarios. Use the Metrics Explorer to investigate the collected data.

While you can deploy one or more workspaces in your Azure subscription, you should ensure your initial deployment follows Microsoft guidelines.

The workspace is also a container where you collect and aggregate data.

Most data sources write to their own tables in an Azure Monitor Logs workspace.
you can set billing and retention for each workspace.

Most IT organizations use a centralized, decentralized, or hybrid model for their architecture. 

centralise = administered by a single team, Extra administrative overhead is needed to maintain access control to different users. This model is known as hub and spoke.

decentralize = Each team has their own workspace created in a resource group they own and manage. A disadvantage of this module is that it can be difficult to cross-correlate logs.

hybrid = The hybrid design commonly results in a complex, expensive, and hard-to-maintain configuration with gaps in logs coverage.

access mode
Workspace-context = Queries are scoped to all data in all tables in the workspace. 
Resource-context = A user accesses the workspace for a particular resource, resource group, or subscription. 

Azure Workbooks is a feature of Azure Monitor. Workbooks provide a flexible canvas for data analysis and the creation of rich visual reports within the Azure portal. = usage of an app, to do root cause analysis, put together an operational playbook, and many other tasks. and the ability to combine data from disparate sources within a single report. 


Azure insights

Azure insights can help you identify performance issues in the Tailwind Traders architecture. Consider these characteristics about insights:

Azure insights provide a customized monitoring experience for particular applications and services.

Azure insights collect and analyze both logs and metrics.

Consider Azure Workbooks. Explore how Tailwind Traders apps can be used with Azure Workbooks. Investigate the root cause analysis of incidents, and put together an operational playbook for your team.

Consider Azure insights and data analysis. Include Azure insights for a custom monitoring experience for Tailwind Traders apps and services. Review insights about your network, VMs, and other Azure resources. Collect Logs and Metrics data from Workbooks and analyze the data.

Azure Data Explorer
Azure Data Explorer is a fast and highly scalable data exploration service for log and telemetry data.
handle multiple data streams, so you can collect, store, and analyze your data from all resources. such as websites, applications, IoT devices, and more.

You can use Azure Data Explorer to provide monitoring support for all aspects and for all types of logs for Tailwind Traders.
Azure Data Explorer provides greater flexibility for building quick and easy near-real-time analytics dashboards, pattern recognition, and time series analysis. The tool supports granular role-based access control, anomaly detection and forecasting, and machine learning.