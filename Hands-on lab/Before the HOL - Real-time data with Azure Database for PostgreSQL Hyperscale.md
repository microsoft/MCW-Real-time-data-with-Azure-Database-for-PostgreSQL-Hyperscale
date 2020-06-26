![Microsoft Cloud Workshops](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png 'Microsoft Cloud Workshops')

<div class="MCWHeader1">
Real-time data with Azure Database for PostgreSQL Hyperscale
</div>

<div class="MCWHeader2">
Before the hands-on lab setup guide
</div>

<div class="MCWHeader3">
June 2020
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2020 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

<!-- TOC -->

- [Real-time data with Azure Database for PostgreSQL Hyperscale before the hands-on lab setup guide](#real-time-data-with-azure-database-for-postgresql-hyperscale-before-the-hands-on-lab-setup-guide)
  - [Requirements](#requirements)
  - [Before the hands-on lab](#before-the-hands-on-lab)
    - [Task 1: Create an Azure resource group using Azure Cloud Shell](#task-1-create-an-azure-resource-group-using-azure-cloud-shell)
    - [Task 2: Create Cloud Shell variables](#task-2-create-cloud-shell-variables)
    - [Task 3: Create an Azure Key Vault](#task-3-create-an-azure-key-vault)
    - [Task 4: Create an event hub with Kafka enabled](#task-4-create-an-event-hub-with-kafka-enabled)
    - [Task 5: Create an Azure Data Lake Storage Gen2 account](#task-5-create-an-azure-data-lake-storage-gen2-account)
    - [Task 6: Create Azure Databricks workspace](#task-6-create-azure-databricks-workspace)
    - [Task 7: Deploy Azure Database for PostgreSQL](#task-7-deploy-azure-database-for-postgresql)
    - [Task 8: Install Npgsql](#task-8-install-npgsql)
    - [Task 9: Download the starter files](#task-9-download-the-starter-files)

<!-- /TOC -->

# Real-time data with Azure Database for PostgreSQL Hyperscale before the hands-on lab setup guide

## Requirements

1. Microsoft Azure subscription must be pay-as-you-go or MSDN.
   - Trial subscriptions will not work.
   - **IMPORTANT**: To complete the OAuth 2.0 access components of this hands-on lab you must have permissions within your Azure subscription to create an App Registration and service principal within Azure Active Directory.
2. Install [pgAdmin](https://www.pgadmin.org/download/) 4 or greater.
3. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).

## Before the hands-on lab

Duration: 30 minutes

In this exercise, you will set up your environment for use in the rest of the hands-on lab. You should follow all steps provided before attending the Hands-on lab.

> **IMPORTANT**: Many Azure resources require unique names. Throughout these steps you will see the word "SUFFIX" as part of resource names. You should replace this with your Microsoft alias, initials, or another value to ensure resources are uniquely named.

### Task 1: Create an Azure resource group using Azure Cloud Shell

In this task, you will use the Azure Cloud shell to create a new Azure Resource Group for this lab.

1. In the [Azure portal](https://portal.azure.com), select the Azure Cloud Shell icon from the top menu.

   ![The Azure Cloud Shell icon is highlighted in the Azure portal's top menu.](media/cloud-shell-icon.png 'Azure Cloud Shell')

2. In the Cloud Shell window that opens at the bottom of your browser window, select **Bash**.

   ![In the Welcome to Azure Cloud Shell window, Bash is highlighted.](media/cloud-shell-select-bash.png 'Azure Cloud Shell')

3. If prompted that you have no storage mounted, select the subscription you are using for this hands-on lab and select **Create storage**.

   ![In the You have no storage mounted dialog, a subscription has been selected, and the Create Storage button is highlighted.](media/cloud-shell-create-storage.png 'Azure Cloud Shell')

   > **Note**: If creation fails, you may need to select **Advanced settings** and specify the subscription, region and resource group for the new storage account.

4. After a moment, you will receive a message that you have successfully requested a Cloud Shell, and be presented with bash Azure prompt.

   ![The Azure Cloud Shell is displayed with its default prompt.](media/cloud-shell-prompt.png 'Azure Cloud Shell')

5. If you have multiple subscriptions, choose the appropriate subscription in which the resource should be billed. List all your subscriptions by entering the following into the shell:

   ```bash
   az account list
   ```

6. Select the specific subscription ID under your account using `az account set` command. Copy the `id` value from the output of the previous command for the subscription you wish to use into the `subscription id` placeholder:

   ```bash
   az account set --subscription <subscription id>
   ```

7. Create a variable to hold your resource group name. This will be used when creating other resources. Replace SUFFIX with your Microsoft alias, initials, or another value to ensure uniqueness.

   ```bash
   resourcegroup=hands-on-lab-SUFFIX
   ```

8. Create a variable to hold your resource group location name. Replace the `westus2` location with a location closest to you. This same location will be used when provisioning other Azure resources. **Please note** that currently, the only regions available for deploying to the Azure Database for PostgreSQL Hyperscale (Citus) deployment option are East US 2, West US 2, West Europe, Southeast Asia. Consider using one of these: `westus2`, `eastus2`, `westeurope`, or `southeastasia`.

   ```bash
   location=westus2
   ```

   > For a list of valid location names, execute: `az account list-locations -o table`.

9. Enter the following command to create a subscription group.

   ```bash
   az group create --name $resourcegroup --location $location
   ```

### Task 2: Create Cloud Shell variables

Azure Cloud Shell allows you to create variables to store values that can be referenced when executing scripts. In this task, you will create variables in addition to the two you have already created. These variables will be used in the tasks that follow.

1. Create a variable to hold your Event Hubs namespace value. This will be used as a reference when creating your event hub. Be sure to replace SUFFIX with your unique value.

   ```bash
   namespace=wwi-namespace-SUFFIX
   ```

2. Create a variable to hold your storage account name. Be sure to replace SUFFIX with your unique value.

   ```bash
   storagename=wwiadlsSUFFIX
   ```

3. Create a variable to hold your Azure Databricks workspace name. Be sure to replace SUFFIX with your unique value.

   ```bash
   workspace=wwi-databricks-SUFFIX
   ```

4. Create a variable to hold your Azure Key Vault name. Be sure to replace SUFFIX with your unique value.

   ```bash
   keyvault=wwi-keyvault-SUFFIX
   ```

### Task 3: Create an Azure Key Vault

Azure Key Vault is a cloud service that works as a secure secrets store. You can securely store keys, passwords, certificates, and other secrets. In this task, you will create an Azure Key Vault that will be used to securely store secrets, such as your PostgreSQL database and Azure Data Lake Storage Gen2 credentials. These secrets will be accessed by Azure Databricks.

1. Enter the following to create a Key Vault:

   ```bash
   az keyvault create --name $keyvault --resource-group $resourcegroup --location $location
   ```

### Task 4: Create an event hub with Kafka enabled

In this task, you will first create an Event Hubs namespace with Kafka enabled. An Event Hubs namespace provides a unique scoping container, referenced by its fully qualified domain name, in which you create one or more event hubs.

1. Enter the following to create your Kafka-enabled Event Hubs namespace:

   ```bash
   az eventhubs namespace create --name $namespace --resource-group $resourcegroup --enable-kafka true -l $location
   ```

2. Enter the following to add an event hub named "clickstream" to your namespace:

   ```bash
   az eventhubs eventhub create --name clickstream --resource-group $resourcegroup --namespace-name $namespace
   ```

### Task 5: Create an Azure Data Lake Storage Gen2 account

Azure Data Lake Storage Gen2 provides a very fast native directory-based file system optimized for streaming workloads and tailored to work with the Hadoop Distributed File System (HDFS). You will access ADLS Gen2 data from Azure Databricks, using the [ABFS driver](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-abfs-driver).

1. Enter the following to create a general-purpose v2 storage account with locally-redundant storage:

   ```bash
   az storage account create \
    --name $storagename \
    --resource-group $resourcegroup \
    --location $location \
    --sku Standard_LRS \
    --kind StorageV2
   ```

### Task 6: Create Azure Databricks workspace

In this task, you will use the Azure Cloud Shell to create a new Azure Databricks workspace with an Azure Resource Management (ARM) template. During the lab, you will create a Spark cluster within your Azure Databricks workspace to perform real-time stream processing against website clickstream data that is sent through Event Hubs using the Kafka protocol.

1. Execute the following command to create your Azure Databricks workspace with an ARM template:

   ```bash
   az deployment group create \
       --name DatabricksWorkspaceDeployment \
       --resource-group $resourcegroup \
       --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-databricks-workspace/azuredeploy.json" \
       --parameters workspaceName=$workspace pricingTier=premium location=$location
   ```

### Task 7: Deploy Azure Database for PostgreSQL

In this task, you will deploy a new Azure Database for PostgreSQL, selecting the Hyperscale (Citus) option.

1. In the [Azure portal](https://portal.azure.com), select **+ Create a resource** and search for "azure database for postgresql". Select **Azure Database for PostgreSQL** from the search results. Select **Create** on the Azure Database for PostgreSQL overview blade.

   ![Create a resource is highlighted as well as the search term and result.](media/search-azure-db-for-postgresql.png 'Create a resource')

   >**Note:** If you do not see the portal menu, select the menu icon in the upper left-hand corner of the portal to display.

   ![The show portal menu icon is highlighted.](media/portal-menu.png 'Show portal menu')

2. Select **Create** under the **Hyperscale (Citus) server group** deployment option.

   ![The Hyperscale (Citus) server group option is highlighted.](media/select-hyperscale.png 'Select Azure Database for PostgreSQL deployment option')

3. Fill out the new server details form with the following information:

   - **Resource group**: Select the resource group you created earlier.
   - **Server group name**: Enter a unique name for the new server group, such as **wwi-postgres-SUFFIX**, which will also be used for a server subdomain.
   - **Admin username**: Currently required to be the value **citus**, and can't be changed.
   - **Password**: _Enter a valid password you will remember_.
   - **Location**: Use the location you provided when creating the resource group, or the closest available.

   > **Note**: The server admin password that you specify here is required to log in to the server and its databases. Remember or record this information for later use.

4. Select **Configure server group**. Leave the settings in that section unchanged and select **Save**.

   ![The form fields are filled out using the previously defined values.](media/create-hyperscale-server-group.png 'Hyperscale server group')

5. Select **Review + create** and then **Create** to provision the server. Provisioning takes **up to 10** minutes.

6. The page will redirect to monitor deployment. When the live status changes from **Your deployment is underway** to **Your deployment is complete**, select the **Outputs** menu item on the left of the page. The outputs page will contain a coordinator hostname with a button next to it to copy the value to the clipboard. Record this information for later use.

   ![The deployment output shows the Coordinator Hostname value after deployment is complete.](media/postgres-coordinator-hostname.png 'Outputs')

7. Select **Overview** to view the deployment details, then select **Go to resource**.

   ![The Overview menu item and Go to resource button are both highlighted.](media/postgres-deployment-overview.png 'Deployment overview')

   If you are redirected to the Resource Group instead of the Azure Database for PostgreSQL server group, select the server group to continue to the next step.

   ![The Azure Database for PostgreSQL server group is highlighted.](media/postgres-server-group-resource.png 'Resource Group')

8. Select **Networking** in the left-hand menu underneath Security. In the Firewall rules blade, select **Yes** to *allow Azure services and resources to access this server group*, then enter the following to create a new firewall rule to allow all connections (from your machine and Azure services):

   - **Firewall rule name**: Enter `ALL`
   - **Start IP**: Enter `0.0.0.0`
   - **End IP**: Enter `255.255.255.255`

   ![The Firewall rules blade is displayed.](media/postgres-firewall.png 'Firewall rules')

9. Select **Save** to apply the new firewall rule.

### Task 8: Install Npgsql

Npgsql is a .NET data provider for PostgreSQL and is required to connect Power BI Desktop to your PostgreSQL database cluster. Make sure you have installed [Power BI Desktop](https://powerbi.microsoft.com/desktop/) before continuing.

1. Navigate to <https://github.com/npgsql/npgsql/releases> and download then run the **.msi** file for the latest version that includes an `msi` file in its assets.

   ![The .msi file is highlighted for the latest release.](media/npgsql-latest-release-msi.png 'Npgsql releases')

2. During installation, select **Npgsql GAC Installation** when given the option to select features you want to install. Select the **Entire feature will be installed on local hard drive** option.

   ![The Npgsql GAC Installation feature is selected.](media/npgsql-features.png 'Npgsql Custom Setup')

### Task 9: Download the starter files

Download a starter project that includes a payment data generator that sends real-time payment data for processing by your lab solution, as well as data files used in the lab.

1. From your lab computer, download the starter files by downloading a .zip copy of the Cosmos DB real-time advanced analytics GitHub repo.

2. In a web browser, navigate to the [Visualizing real-time data with Azure Database for PostgreSQL Hyperscale MCW repo](https://github.com/microsoft/MCW-Real-time-data-with-Azure-Database-for-PostgreSQL-Hyperscale).

3. On the repo page, select **Clone or download**, then select **Download ZIP**.

   ![Download .zip containing the repository](media/github-download-repo.png 'Download ZIP')

4. Unzip the contents to your root hard drive (i.e. `C:\`). This will create a folder on your root drive named `MCW-Real-time-data-with-Azure-Database-for-PostgreSQL-Hyperscale`.

You should follow all steps provided _before_ performing the Hands-on lab.
