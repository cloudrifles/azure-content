<properties 
	pageTitle="Copy output data from tutorial to an on-premises SQL Server database" 
	description="The walkthrough in this tutorial extends the data factory tutorial to copy marketing campaign effectiveness data to an on-premises SQL Server database." 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="1/28/2015" 
	ms.author="spelluru"/>


# Walkthrough: Copy campaign effectiveness data to an on-premises SQL Server database
In this walkthrough, you will learn how to set up the environment to enable the pipeline to work with your on-premises data.
 
In the last step of log processing scenario from the first walkthrough with Partition -> Enrich -> Analyze workflow, the marketing campaign effectiveness output was copied to an Azure SQL database. You could also move this data to on-premises SQL Server for analytics within your organization.
 
In order to copy the marketing campaign effectiveness data from Azure Blob to on-premises SQL Server, you need to create additional on-premises Linked Service, Table and Pipeline using the same set of cmdlets introduced in the first walkthrough.

## Pr-requisites

You **must** perform the walkthrough in the [Tutorial: Move and process log files using Data Factory][datafactorytutorial] before performing the walkthrough in this article. 

**(recommended)** Review and practice the walkthrough in the [Enable your pipeline to work with on-premises data][useonpremisesdatasources] article for a walkthrough on creating a pipeline to move data from on-premises SQL Server to an Azure blob store.


In this walkthrough, you will perform the following steps: 

1. [Step 1: Create a data management gateway](#OnPremStep1). The Data Management Gateway  is a client agent that provides access to on-premises data sources in your organization from the cloud. The gateway enables transfer of data between an on premise SQL Server and Azure data stores.	

	You must have at least one gateway installed in your corporate environment as well as register it with the Azure Data Factory before adding on-premises SQL Server database as a linked service to an Azure data factory.

2. [Step 2: Create a linked service for the on-premises SQL Server](#OnPremStep2). In this step, you first create a database and a table on your on-premises SQL Server computer and then create the linked service: **OnPremSqlLinkedService**.  
3. [Step 3: Create table and pipeline](#OnPremStep3). In this step, you will create a table **MarketingCampaignEffectivenessOnPremSQLTable** and pipeline **EgressDataToOnPremPipeline**. 

4. [Step 4: Monitor pipeline and view the result](#OnPremStep4). In this step, you will monitor the pipelines, tables, and data slices by using the Azure Portal.


## <a name="OnPremStep1"></a> Step 1: Create a Data Management Gateway

The Data Management Gateway is a client agent that provides access to on-premises data sources in your organization from the cloud. The gateway enables transfer of data between an on premise SQL Server and Azure data stores.
  
You must have at least one gateway installed in your corporate environment as well as register it with the Azure Data Factory before adding on-premises SQL Server database as a linked service to an Azure data factory.

If you have an existing data gateway that you can use, skip this step.

1.	Create a logical data gateway. In the **Azure Preview Portal**, click **Linked Services** on the **DATA FACTORY** blade.
2.	Click **Add (+) Data Gateway** on the command bar.  
3.	In the **New data gateway** blade, click **CREATE**.
4.	In the **Create** blade, enter **MyGateway** for the Data gateway **name**.
5.	Click **PICK A REGION** and change it if needed. 
6.	Click **OK** in the **Create** blade. 
7.	You should see the **Configure** blade. 
8.	In the **Configure** blade, click **Install directly on this computer**. This will download, install, and configure the gateway on your computer and registers the gateway with the service. If you have an existing gateway installed on your computer that you want to link to this logical gateway on the portal, use the key on this blade to re-register your gateway using Data Management Gateway Configuration Manager (Preview) tool.

	![Data Management Gateway Configuration Manager][image-data-factory-datamanagementgateway-configuration-manager]

9. Click **OK** to close the **Configure** blade and **OK** to close the **Create** blade. Wait until the status of **MyGateway** in the **Linked Services** blade changes to **GOOD**. You can also launch **Data Management Gateway Configuration Manager (Preview)** tool to confirm that the name of the gateway matches the name in the portal and the **status** is **Registered**. You may have to close and reopen the Linked Services blade to see the latest status. It may take a few minutes before the screen is refreshed with the latest status.	

## <a name="OnPremStep2"></a> Step 2: Create a linked service for the on-premises SQL Server

In this step, you first create the required database and table on your on-premises SQL Server computer and then create the linked service.

### Prepare On-Premises database and table

To start with, you need to create the SQL Server database, table, user defined types and stored procedures. These will be used for moving the **MarketingCampaignEffectiveness** results from Azure blob to SQL Server database.

1.	In **Windows Explorer**, navigate to the **OnPremises** sub folder in **C:\ADFWalkthrough** (or the location where you have extracted the samples).
2.	Open **prepareOnPremDatabase&Table.ps1** in your favorite editor, replace the highlighted with your SQL Server information and save the file (please provide **SQL authentication** details). For the purpose of the tutorial, enable SQL Authentication for your database. 
			
		$dbServerName = "<servername>"
		$dbUserName = "<username>"
		$dbPassword = "<password>"

3. In **Azure PowerShell**, navigate to **C:\ADFWalkthrough\OnPremises** folder.
4.	Run **prepareOnPremDatabase&Table.ps1** **(either & in double quotes or as shown below)**.
			
		& '.\prepareOnPremDatabase&Table.ps1'

5. Once the script executes successfully, you will see the following:	
			
		PS E:\ADF\ADFWalkthrough\OnPremises> & '.\prepareOnPremDatabase&Table.ps1'
		6/10/2014 10:12:33 PM Script to create sample on-premises SQL Server Database and Table
		6/10/2014 10:12:33 PM Creating the database [MarketingCampaigns], table and stored procedure on [.]...
		6/10/2014 10:12:33 PM Connecting as user [sa]
		6/10/2014 10:12:33 PM Summary:
		6/10/2014 10:12:33 PM 1. Database 'MarketingCampaigns' created.
		6/10/2014 10:12:33 PM 2. 'MarketingCampaignEffectiveness' table and stored procedure 


### Create the linked service

1.	In the **Azure Preview Portal**, click **Linked Services** tile on the **DATA FACTORY** blade for **LogProcessingFactory**.
2.	In the **Linked Services** blade, click **Add (+) Data Store**.
3.	In the **New data store** blade, enter **OnPremSqlLinkedService** for the **Name**. 
4.	Click **Type (Settings required)** and select **SQL Server**. You should see the **DATA GATEWAY**, **Server**, **Database**, and **CREDENTIALS** settings in the **New data store** blade now. 
5.	Click **DATA GATEWAY (configure required settings)** and select **MyGateway** you had created earlier. 
6.	Enter **name** of the database server that hosts the **MarketingCampaigns** database. 
7.	Enter **MarketingCampaigns** for the Database. 
8.	Click **CREDENTIALS**. 
9.	In the **Credentials** blade, click **Click here to set credentials securely**.
10.	It installs a one-click application for the first time and launches the **Setting Credentials **dialog box. 
11.	In the **Setting Credentials** dialog box, enter **User Name** and **Password**, and click **OK**. Wait until the dialog box closes. 
12.	Click **OK** in the **New data store** blade. 
13.	In the **Linked Services** blade, confirm that **OnPremSqlLinkedService** shows up in the list and the **status** of the linked service is **Good**.

## <a name="OnPremStep3"></a> Step 3: Create table and pipeline

### Ceate the on-premises logical Table

1.	In **Azure PowerShell**, switch to the **C:\ADFWalkthrough\OnPremises** folder. 
2.	Use the cmdlet **New-AzureDataFactoryTable** to create the Tables as follows for **MarketingCampaignEffectivenessOnPremSQLTable.json**.

			
		New-AzureDataFactoryTable -ResourceGroupName ADF -DataFactoryName $df –File .\MarketingCampaignEffectivenessOnPremSQLTable.json
	 
#### Create the pipeline to copy the data from Azure Blob to SQL Server

1.	Use the cmdlet **New-AzureDataFactoryPipeline** to create the Pipeline as follows for **EgressDataToOnPremPipeline.json**.

			
		New-AzureDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName $df –File .\EgressDataToOnPremPipeline.json
	 
2. Use the cmdlet **Set-AzureDataFactoryPipelineActivePeriod** to specify the active period for **EgressDataToOnPremPipeline**.

			
		Set-AzureDataFactoryPipelineActivePeriod -ResourceGroupName ADF -DataFactoryName $df -StartDateTime 2014-05-01Z -EndDateTime 2014-05-05Z –Name EgressDataToOnPremPipeline

	Press **‘Y’** to continue.
	
## <a name="OnPremStep4"></a> Step 4: Monitor pipeline and view the result

You can now use the same steps introduced in [Step 6: Monitoring tables and pipelines](#MainStep6)  to monitor the new pipeline and the data slices for the new on-premises ADF table.
 
When you see the status of a slice of the table **MarketingCampaignEffectivenessOnPremSQLTable** turns into Ready, it means that the pipeline have completed the execution for the slice. To view the results, query the **MarketingCampaignEffectiveness** table in **MarketingCampaigns** database in your SQL Server.
 
Congratulations! You have successfully gone through the walkthrough to use your on-premises data source. 
 
## See Also

Article | Description
------ | ---------------
[Enable your pipelines to work with on-premises data][useonpremisesdatasources] | This article has a walkthrough that shows how to copy data from an on-premises SQL Server database to an Azure blob. 
[Troubleshoot Data Factory issues][troubleshoot] | This article describes how to troubleshoot Azure Data Factory issues
[Azure Data Factory Developer Reference][developer-reference] | The Developer Reference has the comprehensive reference content for cmdlets, JSON script, functions, etc…
[Azure Data Factory Cmdlet Reference][cmdlet-reference] | This reference content has details about all the Data Factory cmdlets. 

[monitor-manage-using-powershell]: ../data-factory-monitor-manage-using-powershell
[use-custom-activities]: ../data-factory-use-custom-activities
[troubleshoot]: ../data-factory-troubleshoot
[cmdlet-reference]: http://go.microsoft.com/fwlink/?LinkId=517456

[datafactorytutorial]: ./data-factory-tutorial
[adfgetstarted]: ../data-factory-get-started
[adfintroduction]: ../data-factory-introduction
[useonpremisesdatasources]: ../data-factory-use-onpremises-datasources
[usepigandhive]: ../data-factory-pig-hive-activities

[azure-preview-portal]: http://portal.azure.com
[azure-purchase-options]: http://azure.microsoft.com/en-us/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/en-us/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/en-us/pricing/free-trial/

[sqlcmd-install]: http://www.microsoft.com/en-us/download/details.aspx?id=35580
[azure-sql-firewall]: http://msdn.microsoft.com/en-us/library/azure/jj553530.aspx

[download-azure-powershell]: http://azure.microsoft.com/en-us/documentation/articles/install-configure-powershell
[adfwalkthrough-download]: http://go.microsoft.com/fwlink/?LinkId=517495
[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908


[image-data-factory-tutorial-end-to-end-flow]: ./media/data-factory-tutorial/EndToEndWorkflow.png

[image-data-factory-tutorial-partition-game-logs-pipeline]: ./media/data-factory-tutorial/PartitionGameLogsPipeline.png

[image-data-factory-tutorial-enrich-game-logs-pipeline]: ./media/data-factory-tutorial/EnrichGameLogsPipeline.png

[image-data-factory-tutorial-analyze-marketing-campaign-pipeline]: ./media/data-factory-tutorial/AnalyzeMarketingCampaignPipeline.png


[image-data-factory-tutorial-egress-to-onprem-pipeline]: ./media/data-factory-tutorial/EgreeDataToOnPremPipeline.png

[image-data-factory-tutorial-set-firewall-rules-azure-db]: ./media/data-factory-tutorial/SetFirewallRuleForAzureDatabase.png

[image-data-factory-tutorial-portal-new-everything]: ./media/data-factory-tutorial/PortalNewEverything.png

[image-data-factory-tutorial-datastorage-cache-backup]: ./media/data-factory-tutorial/DataStorageCacheBackup.png

[image-data-factory-tutorial-dataservices-blade]: ./media/data-factory-tutorial/DataServicesBlade.png

[image-data-factory-tutorial-new-datafactory-blade]: ./media/data-factory-tutorial/NewDataFactoryBlade.png

[image-data-factory-tutorial-resourcegroup-blade]: ./media/data-factory-tutorial/ResourceGroupBlade.png

[image-data-factory-tutorial-create-resourcegroup]: ./media/data-factory-tutorial/CreateResourceGroup.png

[image-data-factory-tutorial-datafactory-homepage]: ./media/data-factory-tutorial/DataFactoryHomePage.png

[image-data-factory-tutorial-create-datafactory]: ./media/data-factory-tutorial/CreateDataFactory.png

[image-data-factory-tutorial-linkedservice-tile]: ./media/data-factory-tutorial/LinkedServiceTile.png

[image-data-factory-tutorial-linkedservices-add-datstore]: ./media/data-factory-tutorial/LinkedServicesAddDataStore.png

[image-data-factory-tutorial-datastoretype-azurestorage]: ./media/data-factory-tutorial/DataStoreTypeAzureStorageAccount.png

[image-data-factory-tutorial-azurestorage-settings]: ./media/data-factory-tutorial/AzureStorageSettings.png

[image-data-factory-tutorial-storage-key]: ./media/data-factory-tutorial/StorageKeyFromAzurePortal.png

[image-data-factory-tutorial-linkedservices-blade-storage]: ./media/data-factory-tutorial/LinkedServicesBladeWithAzureStorage.png

[image-data-factory-tutorial-azuresql-settings]: ./media/data-factory-tutorial/AzureSQLDatabaseSettings.png

[image-data-factory-tutorial-azuresql-database-connection-string]: ./media/data-factory-tutorial/DatabaseConnectionString.png

[image-data-factory-tutorial-linkedservices-all]: ./media/data-factory-tutorial/LinkedServicesAll.png

[image-data-factory-tutorial-datasets-all]: ./media/data-factory-tutorial/DataSetsAllTables.png

[image-data-factory-tutorial-pipelines-all]: ./media/data-factory-tutorial/AllPipelines.png

[image-data-factory-tutorial-diagram-link]: ./media/data-factory-tutorial/DataFactoryDiagramLink.png

[image-data-factory-tutorial-diagram-view]: ./media/data-factory-tutorial/DiagramView.png

[image-data-factory-monitoring-startboard]: ./media/data-factory-tutorial/MonitoringStartBoard.png

[image-data-factory-monitoring-hub-everything]: ./media/data-factory-tutorial/MonitoringHubEverything.png

[image-data-factory-monitoring-browse-datafactories]: ./media/data-factory-tutorial/MonitoringBrowseDataFactories.png

[image-data-factory-monitoring-pipeline-consumed-produced]: ./media/data-factory-tutorial/MonitoringPipelineConsumedProduced.png

[image-data-factory-monitoring-raw-game-events-table]: ./media/data-factory-tutorial/MonitoringRawGameEventsTable.png

[image-data-factory-monitoring-raw-game-events-table-dataslice-blade]: ./media/data-factory-tutorial/MonitoringPartitionGameEventsTableDataSliceBlade.png

[image-data-factory-monitoring-activity-run-details]: ./media/data-factory-tutorial/MonitoringActivityRunDetails.png

[image-data-factory-datamanagementgateway-configuration-manager]: ./media/data-factory-tutorial/DataManagementGatewayConfigurationManager.png

[image-data-factory-new-datafactory-menu]: ./media/data-factory-tutorial/NewDataFactoryMenu.png

[image-data-factory-new-datafactory-create-button]: ./media/data-factory-tutorial/DataFactoryCreateButton.png