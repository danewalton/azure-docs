---
title: 'Register and scan Azure SQL DB'
description: This article outlines the process to register an Azure SQL database in Azure Purview including instructions to authenticate and interact with the Azure SQL DB source
author: athenads
ms.author: athenadsouza
ms.service: purview
ms.topic: how-to
ms.date: 11/02/2021
ms.custom: template-how-to, ignite-fall-2021
---
# Connect to Azure SQL Database in Azure Purview

This article outlines the process to register an Azure SQL data source in Azure Purview including instructions to authenticate and interact with the Azure SQL database source

## Supported capabilities

|**Metadata Extraction**|  **Full Scan**  |**Incremental Scan**|**Scoped Scan**|**Classification**|**Access Policy**|**Lineage**|
|---|---|---|---|---|---|---|
| [Yes](#register) | [Yes](#scan)|[Yes](#scan) | [Yes](#scan)|[Yes](#scan)| No |[Data Factory Lineage](how-to-link-azure-data-factory.md)|

### Known limitations

* Azure Purview doesn't support over 300 columns in the Schema tab and it will show "Additional-Columns-Truncated".

## Prerequisites

* An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* An active [Purview resource](create-catalog-portal.md).

* You will need to be a Data Source Administrator and Data Reader to register a source and manage it in the Purview Studio. See our [Azure Purview Permissions page](catalog-permissions.md) for details.

## Register

This section will enable you to register the Azure SQL DB data source and set up an appropriate authentication mechanism to ensure successful scanning of the data source.

### Steps to register

It is important to register the data source in Azure Purview prior to setting up a scan for the data source.

1. Go to the [Azure portal](https://portal.azure.com), and navigate to the **Purview accounts** page and select your _Purview account_

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-purview-acct.png" alt-text="Screenshot that shows the Purview account used to register the data source":::

1. **Open Purview Studio** and navigate to the **Data Map**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-open-purview-studio.png" alt-text="Screenshot that navigates to the Sources link in the Data Map":::

1. Create the [Collection hierarchy](./quickstart-create-collection.md) using the **Collections** menu and assign permissions to individual sub-collections, as required

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-collections.png" alt-text="Screenshot that shows the collection menu to assign access control permissions to the collection hierarchy":::

1. Navigate to the appropriate collection under the **Sources** menu and select the **Register** icon to register a new Azure SQL DB

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-data-source.png" alt-text="Screenshot that shows the collection used to register the data source":::

1. Select the **Azure SQL Database** data source and select **Continue**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-select-ds.png" alt-text="Screenshot that allows selection of the data source":::

1. Provide a suitable **Name** for the data source, select the relevant **Azure subscription**, **Server name** for the SQL server and the **collection** and select on **Apply**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-ds-details.png" alt-text="Screenshot that shows the details to be entered in order to register the data source":::

1. The Azure SQL Server Database will be shown under the selected Collection

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-ds-collections.png" alt-text="Screenshot that shows the data source mapped to the collection to initiate scanning":::

## Scan

### Authentication for a scan

In order to have access to scan the data source, an authentication method in the Azure SQL Database needs to be configured.
The following options are supported:

* **SQL Authentication**

* **Managed Identity** - As soon as the Azure Purview Account is created, a system **Managed Identity** is created automatically in Azure AD tenant. Depending on the type of resource, specific RBAC role assignments are required for the Azure Purview MSI to perform the scans.
If you are using managed identity, your Purview account has its own managed identity which is basically your Purview name when you created it

* **Service Principal** - In this method, you can create a new or use an existing service principal in your Azure Active Directory tenant.

##### Configure Azure AD authentication in the database account

The service principal or managed identity must have permission to get metadata for the database, schemas and tables. It must also be able to query the tables to sample for classification.

- [Configure and manage Azure AD authentication with Azure SQL](../azure-sql/database/authentication-aad-configure.md)
- You must create an Azure AD user in Azure SQL Database with the exact Purview's managed identity or your own service principal by following tutorial on [Create the service principal user in Azure SQL Database](../azure-sql/database/authentication-aad-service-principal-tutorial.md#create-the-service-principal-user-in-azure-sql-database). You need to assign proper permission (e.g. `db_datareader`) to the identity. Example SQL syntax to create user and grant permission:

    ```sql
    CREATE USER [Username] FROM EXTERNAL PROVIDER
    GO
    
    EXEC sp_addrolemember 'db_datareader', [Username]
    GO
    ```

    > [!Note]
    > The `Username` is your own service principal or Purview's managed identity. You can read more about [fixed-database roles and their capabilities](/sql/relational-databases/security/authentication-access/database-level-roles#fixed-database-roles).

#### Using SQL Authentication for scanning

> [!Note]
> Only the server-level principal login (created by the provisioning process) or members of the `loginmanager` database role in the master database can create new logins. It takes about **15 minutes** after granting permission, the Purview account should have the appropriate permissions to be able to scan the resource(s).

You can follow the instructions in [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-current&preserve-view=true#examples-1) to create a login for Azure SQL Database if you don't have this available. You will need **username** and **password** for the next steps.

1. Navigate to your key vault in the Azure portal

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-key-vault.png" alt-text="Screenshot that shows the key vault":::

1. Select **Settings > Secrets** and select **+ Generate/Import**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-secret.png" alt-text="Screenshot that shows the key vault option to generate a secret":::

1. Enter the **Name** and **Value** as the *password* from your Azure SQL Database

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-secret-sql.png" alt-text="Screenshot that shows the key vault option to enter the sql secret values":::

1. Select **Create** to complete

1. If your key vault is not connected to Purview yet, you will need to [create a new key vault connection](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)

1. Finally, [create a new credential](manage-credentials.md#create-a-new-credential) using the key to setup your scan

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-credentials.png" alt-text="Screenshot that shows the key vault option to set up credentials":::

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-key-vault-options.png" alt-text="Screenshot that shows the key vault option to create a secret":::

#### Using Managed Identity for scanning

It is important to give your Purview account the permission to scan the Azure SQL DB. You can add the Catalog's MSI at the Subscription, Resource Group, or Resource level, depending on what you want it to have scan permissions on.

> [!Note] 
> You need to be an owner of the subscription to be able to add a managed identity on an Azure resource.

1. From the [Azure portal](https://portal.azure.com), find either the subscription, resource group, or resource (for example, an Azure SQL Database) that you would like to allow the catalog to scan.

1. Select **Access Control (IAM)** in the left navigation and then select **+ Add** --> **Add role assignment**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-sql-ds.png" alt-text="Screenshot that shows the Azure SQL database":::

1. Set the **Role** to **Reader** and enter your _Azure Purview account name_ under **Select** input box. Then, select **Save** to give this role assignment to your Purview account.

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-access-managed-identity.png" alt-text="Screenshot that shows the details to assign permissions for the Purview account":::

#### Using Service Principal for scanning

##### Creating a new service principal

If you need to [Create a new service principal](./create-service-principal-azure.md), it is required to register an application in your Azure AD tenant and provide access to Service Principal in your data sources. Your Azure AD Global Administrator or other roles such as Application Administrator can perform this operation.

##### Getting the Service Principal's Application ID

1. Copy the **Application (client) ID** present in the **Overview** of the [_Service Principal_](./create-service-principal-azure.md) already created

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-sp-appln-id.png" alt-text="Screenshot that shows the Application (client) ID for the Service Principal":::

##### Granting the Service Principal access to your Azure SQL Database

1. Navigate to your key vault in the Azure portal

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-key-vault.png" alt-text="Screenshot that shows the key vault to add a secret for for Service Principal":::

1. Select **Settings > Secrets** and select **+ Generate/Import**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-secret.png" alt-text="Screenshot that shows the key vault option to generate a secret for Service Principal":::

1. Enter the **Name** of your choice and **Value** as the **Client secret** from your Service Principal

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-create-secret.png" alt-text="Screenshot that shows the key vault option to enter the secret values":::

1. Select **Create** to complete

1. If your key vault is not connected to Purview yet, you will need to [create a new key vault connection](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)

1. Finally, [create a new credential](manage-credentials.md#create-a-new-credential) using the key to setup your scan

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-credentials.png" alt-text="Screenshot that shows the key vault option to add a credentials for Service Principal":::

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-sp-cred.png" alt-text="Screenshot that shows the key vault option to create a secret for Service Principal":::

### Firewall settings

If your database server has a firewall enabled, you will need to update the firewall to allow access in one of two ways:

1. Allow Azure connections through the firewall.
1. Install a Self-Hosted Integration Runtime and give it access through the firewall.

#### Allow Azure Connections

Enabling Azure connections will allow Azure Purview to reach and connect the server without updating the firewall itself. You can follow the How-to guide for [Connections from inside Azure](../azure-sql/database/firewall-configure.md#connections-from-inside-azure).

1. Navigate to your database account
1. Select the server name in the **Overview** page
1. Select **Security > Firewalls and virtual networks**
1. Select **Yes** for **Allow Azure services and resources to access this server**
:::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-firewall.png" alt-text="Allow Azure services and resources to access this server." border="true":::

#### Self-Hosted Integration Runtime

A self-hosted integration runtime (SHIR) can be installed on a machine to connect with a resource in a private network.

1. [Create and install a self-hosted integration runtime](./manage-integration-runtimes.md) on a personal machine, or a machine inside the same VNet as your database server.
1. Check your database server firewall to confirm that the SHIR machine has access through the firewall. Add the IP of the machine if it does not already have access.
1. If your Azure SQL Server is behind a private endpoint or in a VNet, you can use an [ingestion private endpoint](catalog-private-link-ingestion.md#deploy-self-hosted-integration-runtime-ir-and-scan-your-data-sources) to ensure end-to-end network isolation.

### Creating the scan

1. Open your **Purview account** and select the **Open Purview Studio**
1. Navigate to the **Data map** --> **Sources** to view the collection hierarchy
1. Select the **New Scan** icon under the **Azure SQL DB** registered earlier

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-new-scan.png" alt-text="Screenshot that shows the screen to create a new scan":::

#### If using SQL Authentication

1. Provide a **Name** for the scan, select **Database selection method** as _Enter manually_, enter the **Database name** and the **Credential** created earlier, choose the appropriate collection for the scan and select **Test connection** to validate the connection. Once the connection is successful, select **Continue**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-sql-auth.png" alt-text="Screenshot that shows the SQL Authentication option for scanning":::

#### If using Managed Identity

1. Provide a **Name** for the scan, select the **Purview MSI** under **Credential**, choose the appropriate collection for the scan

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-managed-id.png" alt-text="Screenshot that shows the Managed Identity option to run the scan":::

1. Select **Test connection**. On a successful connection, select **Continue**

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-test.png" alt-text="Screenshot that allows the Managed Identity option to run the scan":::

#### If using Service Principal

1. Provide a **Name** for the scan, choose the appropriate collection for the scan and select the **Credential** dropdown to select the credential created earlier

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-sp.png" alt-text="Screenshot that shows the option for service principal to enable scanning":::

1. Select **Test connection**. On a successful connection, select **Continue**

### Scoping and running the scan

1. You can scope your scan to specific folders and sub-folders by choosing the appropriate items in the list.

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-scope-scan.png" alt-text="Scope your scan":::

1. Then select a scan rule set. You can choose between the system default, existing custom rule sets, or create a new rule set inline.

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-scan-rule-set.png" alt-text="Scan rule set":::

1. If creating a new _scan rule set_

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-new-scan-rule-set.png" alt-text="New Scan rule set":::

1. You can select the **classification rules** to be included in the scan rule

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-classification.png" alt-text="Scan rule set classification rules":::

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-sel-scan-rule.png" alt-text="Scan rule set selection":::

1. Choose your scan trigger. You can set up a schedule or run the scan once.

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-scan-trigger.png" alt-text="scan trigger":::

1. Review your scan and select **Save and run**.

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-review-scan.png" alt-text="review scan":::

### View Scan

1. Navigate to the _data source_ in the _Collection_ and select **View Details** to check the status of the scan

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-view-scan.png" alt-text="view scan":::

1. The scan details indicate the progress of the scan in the **Last run status** and the number of assets _scanned_ and _classified_

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-view-scan-details.png" alt-text="view scan details":::

1. The **Last run status** will be updated to **In progress** and subsequently **Completed** once the entire scan has run successfully

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-scan-complete.png" alt-text="view scan completed":::

### Manage Scan

Scans can be managed or run again on completion

1. Select the **Scan name** to manage the scan

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-manage scan.png" alt-text="manage scan":::

1. You can _run the scan_ again, _edit the scan_, _delete the scan_  

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-manage-scan-options.png" alt-text="manage scan options":::

1. You can _run an incremental scan_ or a _full scan_ again

    :::image type="content" source="media/register-scan-azure-sql-database/register-scan-azure-sql-db-full-inc.png" alt-text="full or incremental scan":::

## Next steps

Now that you have registered your source, follow the below guides to learn more about Purview and your data.

- [Data insights in Azure Purview](concept-insights.md)
- [Lineage in Azure Purview](catalog-lineage-user-guide.md)
- [Search Data Catalog](how-to-search-catalog.md)
