---
title: Enable automatic tuning
description: You can enable automatic tuning on your database easily using the Azure portal.
author: NikaKinska
ms.author: nnikolic
ms.reviewer: wiassaf, mathoma
ms.date: 06/06/2022
ms.service: sql-db-mi
ms.subservice: performance
ms.topic: how-to
ms.custom: sqldbrb=1
---
# Enable automatic tuning in the Azure portal to monitor queries and improve workload performance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database automatically manages data services that constantly monitor your queries and identifies the action that you can perform to improve performance of your workload. You can review recommendations and manually apply them, or let Azure SQL Database automatically apply corrective actions - this is known as **automatic tuning mode**.

Automatic tuning can be enabled at the server or the database level through:

- The [Azure portal](automatic-tuning-enable.md#azure-portal)
- [REST API](automatic-tuning-enable.md#rest-api) calls
- [T-SQL](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current&preserve-view=true) commands

> [!NOTE]
> For Azure SQL Managed Instance, the supported option FORCE_LAST_GOOD_PLAN can only be configured through [T-SQL](https://azure.microsoft.com/blog/automatic-tuning-introduces-automatic-plan-correction-and-t-sql-management). The Azure portal based configuration and automatic index tuning options described in this article do not apply to Azure SQL Managed Instance.

> [!NOTE]
> Configuring automatic tuning options through the ARM (Azure Resource Manager) template is not supported at this time.

## Enable automatic tuning on server

On the server level you can choose to inherit automatic tuning configuration from "Azure Defaults" or not to inherit the configuration. Azure defaults are FORCE_LAST_GOOD_PLAN enabled, CREATE_INDEX disabled, and DROP_INDEX disabled.

### Azure portal

To enable automatic tuning on a [server](logical-servers.md) in Azure SQL Database, navigate to the server in the Azure portal and then select **Automatic tuning** in the menu.

![Screenshot shows Automatic tuning in the Azure portal, where you can apply options for a server.](./media/automatic-tuning-enable/server.png)

Select the automatic tuning options you want to enable and select **Apply**.

Automatic tuning options on a server are applied to all databases on this server. By default, all databases inherit configuration from their parent server, but this can be overridden and specified for each database individually.

### REST API

To find out more about using a REST API to enable automatic tuning on a **server**, see [Server automatic tuning UPDATE and GET HTTP methods](/rest/api/sql/serverautomatictuning).

## Enable automatic tuning on an individual database

Azure SQL Database enables you to individually specify the automatic tuning configuration for each database. On the database level you can choose to inherit automatic tuning configuration from the parent server, "Azure Defaults" or not to inherit the configuration. Azure Defaults are set to FORCE_LAST_GOOD_PLAN is enabled, CREATE_INDEX is disabled, and DROP_INDEX is disabled.

> [!TIP]
> The general recommendation is to manage the automatic tuning configuration at **server level** so the same configuration settings can be applied on every database automatically. Configure automatic tuning on an individual database only if you need that database to have different settings than others inheriting settings from the same server.

### Azure portal

To enable automatic tuning on a **single database**, navigate to the database in the Azure portal and select **Automatic tuning**.

Individual automatic tuning settings can be separately configured for each database. You can manually configure an individual automatic tuning option, or specify that an option inherits its settings from the server.

![Screenshot shows Automatic tuning in the Azure portal, where you can apply options for a single database.](./media/automatic-tuning-enable/database.png)

Once you have selected your desired configuration, click **Apply**.

### REST API

To find out more about using a REST API to enable automatic tuning on a single database, see [Azure SQL Database automatic tuning UPDATE and GET HTTP methods](/rest/api/sql/2022-02-01-preview/database-automatic-tuning).

### T-SQL

To enable automatic tuning on a single database via T-SQL, connect to the database and execute the following query:

```SQL
ALTER DATABASE current SET AUTOMATIC_TUNING = AUTO | INHERIT | CUSTOM
```

Setting automatic tuning to AUTO will apply Azure Defaults. Setting it to INHERIT, automatic tuning configuration will be inherited from the parent server. Choosing CUSTOM, you will need to manually configure automatic tuning.

To configure individual automatic tuning options via T-SQL, connect to the database and execute the query such as this one:

```SQL
ALTER DATABASE current SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON, CREATE_INDEX = ON, DROP_INDEX = OFF)
```

Setting the individual tuning option to ON will override any setting that database inherited and enable the tuning option. Setting it to OFF will also override any setting that database inherited and disable the tuning option. Automatic tuning option for which DEFAULT is specified, will inherit the automatic tuning configuration from the server level settings.

> [!IMPORTANT]
> In the case of [active geo-replication](auto-failover-group-sql-db.md), Automatic tuning needs to be configured on the primary database only. Automatically applied tuning actions, such as for example index create or delete will be automatically replicated to geo-secondaries. Attempting to enable Automatic tuning via T-SQL on the read-only secondary will result in a failure as having a different tuning configuration on the read-only secondary is not supported.
>

To find out more abut T-SQL options to configure automatic tuning, see [ALTER DATABASE SET Options (Transact-SQL)](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current&preserve-view=true).

## Troubleshooting

### Automated recommendation management is disabled

In case of error messages that automated recommendation management has been disabled, or simply disabled by system, the most common causes are:
- Query Store is not enabled, or
- Query Store is in read-only mode for a specified database, or
- Query Store stopped running because it ran out of allocated storage space.

The following steps can be considered to rectify this issue:
- Clean up the Query Store, or modify the data retention period to "auto" by using T-SQL, or increase Query Store maximum size. See how to [configure recommended retention and capture policy for Query Store](./query-performance-insight-use.md#recommended-retention-and-capture-policy).
- Use SQL Server Management Studio (SSMS) and follow these steps:
  - Connect to the Azure SQL Database
  - Right-click on the database
  - Go to Properties and click on Query Store
  - Change the Operation Mode to Read-Write
  - Change the Store Capture Mode to Auto
  - Change the Size Based Cleanup Mode to Auto

### Permissions

For Azure SQL Database, managing Automatic tuning in Azure portal, or using PowerShell or REST API requires membership in Azure built-in RBAC roles.

To manage automatic tuning, the minimum required permission to grant to the user is membership in the [SQL Database contributor](/azure/role-based-access-control/built-in-roles#sql-db-contributor) role. You can also consider using higher privilege roles such as SQL Server Contributor, Contributor, and Owner.

For permissions required to manage Automatic tuning with T-SQL, see [Permissions](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current&preserve-view=true#permissions-1) for **ALTER DATABASE**.

## Configure automatic tuning e-mail notifications

To receive automated email notifications on recommendations made by the automatic tuning, see the [automatic tuning e-mail notifications](automatic-tuning-email-notifications-configure.md) guide.

## Next steps

- Read the [Automatic tuning article](automatic-tuning-overview.md) to learn more about automatic tuning and how it can help you improve your performance.
- See [Performance recommendations](database-advisor-implement-performance-recommendations.md) for an overview of Azure SQL Database performance recommendations.
- See [Query Performance Insights](query-performance-insight-use.md) to learn about viewing the performance impact of your top queries.
