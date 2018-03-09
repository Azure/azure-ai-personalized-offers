# Personalized Offers - Manual Deployment Guide

# Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Setup Steps](#setup-steps)
   - [General](#setup-steps)
   - [Azure Storage](#storage)
   - [Azure Machine Learning](#aml)
   - [Azure Event Hubs](#eventhub)
   - [Create Azure Cosmos DB](#acdb)
   - [Create Azure Data Lake Store](#adls)
   - [Create Azure Redis Cache](#redis)
   - [Create Azure Functions](#af)
   - [Azure Stream Analytics](#asa)    
- [Starting the Solution](#startup)
- [More information](#moreinfo)

## Introduction

This guide will walk you through the procedure to manually create the Personalized Offers Solution. You can get an overview of the project [here](https://github.com/Azure/cortana-intelligence-personalized-offers/blob/master/README.md) and see some details on how to work with the deployed solution [here](https://github.com/Azure/cortana-intelligence-personalized-offers/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md). 

The necessary materials are included in the ***src*** folder in this [repository](https://github.com/Azure/cortana-intelligence-personalized-offers/blob/master/Manual%20Deployment%20Guide/src). 

## Prerequisites

The steps described later in this guide require the following prerequisites:

1. An [Azure subscription](https://azure.microsoft.com/en-us/) with login credentials
2. The creation of:
   * 1 Data Lake Store
   * 4 Stream Jobs with a total of 43 Streaming Units
   * 1 Event Hub with 20 Throughput Units, 16 partitions and 4 Consumer Groups
   * 1 DocumentDB database with 6 collections, each provisioned with 10000 RUs, 10GB (3 are Partitioned). 

Ensure adequate Data Lake Stores and Stream Processing units are available before provisioning. Please consider deleting any unused Data Lake Store from your subscription. You may contact Azure Support if you need to increase the limit.

## Architecture

The architecture diagram shows various Azure services that are deployed by [Personalized Offers Solution](https://github.com/Azure/cortana-intelligence-personalized-offers) using [Cortana Intelligence Solutions](https://gallery.cortanaintelligence.com/solutions), and how they are connected to each other in the end-to-end solution.

![Solution Diagram](https://cloud.githubusercontent.com/assets/16085124/24881519/084cd072-1e0c-11e7-9093-7eaf48d4d513.png)

## Setup Steps (Estimated Time: 3 hours)

The following are the steps to deploy the end-to-end solution.

### Accessing Files in the Git Repository

This tutorial will refer to files available in the Technical Deployment Guide section of the [Cortana Intelligence Personalized Offers Git repository](https://github.com/Azure/cortana-intelligence-personalized-offers/). You can download all of these files at once by clicking the "Clone or download" button.

You can download or view individual files by navigating through the repository folders. If you choose this option, be sure to download the "raw" version of each file by clicking the filename to view it, then clicking "Download". You will also find a settings.txt file in the ***src*** folder that can be used to keep track of settings you will need for configuring the Azure Functions. The names provided in the settings.txt file correspond to the names of the settings, and the entries you add will be the values.

### Choose a Unique String

You will need a unique string to identify your deployment because some Azure services such as Azure Storage require a unique name for each instance. We suggest you use only letters and numbers in this string. The length of your unique string should not be greater than 9 characters.

We suggest you use "[UI]poffer[N]" where [UI] are the user's initials, N is a random integer that you choose and characters are lowercase. Please open your settings.txt and write down your unique string.

### Create an Azure Resource Group for the solution
1. Log into the [Azure Management Portal](https://ms.portal.azure.com).
1. Click the **Resource groups** button, and then click the **+ Add** button to add a resource group.
1. Enter your **unique string** for the resource group and choose your subscription.
1. For **Resource group location**, you should choose one of the following as they are the locations that offer all of the Azure services used in this guide (with the exception of Azure Data Factory, which need not be located in the same location):
  - South Central US
  - West Europe
  - Southeast Asia
 - Click **Create**

Please open your settings.txt file and save the information in the form of the following table, replacing the content in [] with the actual values.  

| **Azure Resource Group** |              |
|--------------------------|--------------|
| resourceGroupName        | **[unique]** |
| region                   | **[region]** |

### Instruction for Finding Your Resource Group Overview

In this tutorial, all resources will be created in the resource group you just created. You can easily access these resources from the resource group overview page, which can be accessed as follows:

1. Log into the [Azure Management Portal](https://ms.portal.azure.com).
2. If you pinned the Resource Group when creating it, you will find your resource group here and can click on it to see all of the associated resources.
3. Click the **Resource groups** button.
4. Choose the subscription your resource group resides in.
5. Search for (or directly select) your resource group from the list of resource groups.

Note that you may need to close the resource description page to add new resources.

In the following steps, if any entry or item is not mentioned in the instructions, please leave it as the default value.

<a name="storage"></a>
### Create an Azure Storage Account

In this section we will go through the steps necessary to create the storage account and a blob associated with it, and upload some files to the blob. Along the way we will note down the values that we will need later in our settings.txt file.

1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to the resource group you just created.
2. In the **Overview** panel, click **+ Add** to add a new resource. Enter **Storage account** and hit "Enter" to search.
3. Click on **Storage account - blob, file, table, queue** offered by Microsoft (in the "Storage" category).
4. Click **Create** at the bottom of the description panel.
5. Enter your **unique string** for "Name".
6. Make sure the selected resource group is the one you just created. If not, choose the resource group you created for this solution.
7. Click the **Create** button at the bottom.
8. Go back to your resource group overview and wait until the storage account is deployed. To check the deployment status, refresh the page or the list of resources in the resource group as needed.

#### Get the Primary Key for the Azure Storage Account
These are the steps to get the access key that will be used in later steps.

1. Click the created storage account. In the new panel, click on **Access keys**.
2. In the new panel, click the "Click to copy" icon next to `key1`, and paste the key into your settings.txt file as detailed below.

| **Azure Storage Account**      |                                                                                       |
|--------------------------------|---------------------------------------------------------------------------------------|
| storageAccountName             | **[unique string]**                                                                   |
| storageAccountKey              | **[key1]**                                                                            |
| storageAccountConnectionString | DefaultEndpointsProtocol=https;AccountName=**[uniques string]**;AccountKey=**[key1]** |

#### Add Blob Storage and Upload Resources
These are the steps to create the **Blob storage**.

1. After getting the primary key, click on **Overview** on the left to return to the main panel for the Storage Account.
2. Click on **Blobs** in the central area of the main panel.
3. Click on **+ Container** at the top of the new panel.
4. Enter **[unique string]blob** for "Name".
5. Select **Blob** for "Public access level".
6. Click **OK** at the bottom of the panel.
7. Click on the blob that you just created.

At this time, if you haven't already, make sure to download the following files from the **src** directory of the [GitHub repository](https://github.com/Azure/cortana-intelligence-personalized-offers/blob/master/Manual%20Deployment%20Guide/src): 

* OfferPriority.txt
* offers.txt
* OfferThreshold.txt
* products.txt
* redisSeed.txt
* users.txt

To upload these files:

1. Click **Upload** at the top of the panel.
2. In the panel that opens, click the **folder icon** to the right of the "Files" field.
3. Navigate to where you saved the files on your computer.
4. Pressing the 'Control Key' on your keyboard and clicking on each of the files (single-click) will allow you to select all the files at once. 
5. Click the **Open** button at the bottom of the window.
6. Click **Upload** at the bottom of the panel.
7. Click the **x** at the top right of the panel for uploading files to dismiss it.
8. You should now see a list of files in the blob container.
9. Click on the **Container properties** button at the top of the panel.
10. Click the **Copy** icon to the right of the URL field in the properties panel that opened.
11. Paste this value into your settings.txt as shown in the table below.

| **Azure Storage Blob Container Files** |                                             |
|----------------------------------------|---------------------------------------------|
| referenceCollectionFile1               | **[Blob Container URL]**/OfferPriority.txt  |
| referenceCollectionFile2               | **[Blob Container URL]**/OfferThreshold.txt |
| offerFile                              | **[Blob Container URL]**/offers.txt         |
| productFile                            | **[Blob Container URL]**/products.txt       |
| userFile                               | **[Blob Container URL]**/users.txt          |
| redisCacheSeedFile                     | **[Blob Container URL]**/redisSeed.txt      |

<a name="aml"></a>
### Set up Azure Machine Learning
		
#### The Azure ML Model
The model used in this guide is based on the [Personalized Offers Solution How To Guide](https://gallery.azure.ai/Experiment/personalized-offers-solution-how-to-guide-4) from the Azure AI Gallery. The experiment used to train the model can be found [here](https://gallery.azure.ai/Experiment/Personalized-Offers-Solution-How-To-Guide-2).

#### Create Azure Machine Learning Workspace
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to the resource group you created.
2. In the **Overview** panel, click **+ Add** to add a new resource. Enter **Machine Learning Studio Workspace** and hit "Enter" to search.
3. Click on **Machine Learning Studio Workspace** offered by Microsoft in the "Data + Analytics" category.
4. Click the **Create** button at the bottom of the description panel.
5. In the Machine Learning Studio workspace panel:
    1. Enter your **unique string** for "Workspace name".
    2. Choose **Use existing** for "Storage account" and select the storage account you created earlier.  
    3. Choose **Create new** for "Web service plan".
    5. Click on **Web service plan pricing tier**, choose **S1 Standard** and click **Select** at the bottom.
    6. Click **Create** at the bottom.

#### Deploy Azure Machine Learning Predictive Web Service
1. Go to the [Personalized Offers Solution How To Guide](https://gallery.azure.ai/Experiment/personalized-offers-solution-how-to-guide-4) in the Azure AI Gallery.
2. Click the **Open in Studio** button on the right. Log in if needed.
3. Choose the region and workspace. For the region, you should choose the region that your resource group resides in. For the workspace, choose the workspace you just created.
4. Wait until the experiment is copied.
5. Click **Run** at the bottom of the page. It takes around three minutes to run the experiment.
6. Click **Deploy Web Service** at the bottom of the page, then click **Deploy Web Service [Classic]** to publish the web service. This will lead you to the web service page.  This page can also be found by clicking the **Web services** button on the left-hand menu bar in your workspace.
7. Copy the **API key** and save it in your settings.txt as per the table given below.
8. Click the **REQUEST/RESPONSE** link under the **API HELP PAGE** section. On the help page that opens, copy the **Request URI** under the **Request** section, and save it in your settings.txt as per the table given below.

| **Machine Learning Web Service** |                   |
|----------------------------------|-------------------|
| mlPublishedExperimentKey         | **[API key]**     |
| mlPublishedExperimentEndpoint    | **[Request URI]** |

<a name="eventhub"></a>
### Create an Azure Event Hub
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Event Hubs** and hit "Enter" to search.
3. Click on **Event Hubs** offered by Microsoft in the "Internet of Things" category.
4. Click **Create** at the bottom of the description panel.
5. In the new panel for creating a namespace, enter your **unique string** for "Name".
6. Click the **Create** button at the bottom of the panel.
7. Return to your resource group's overview page. When it has finished deploying, click on the resource of type "Event Hub".
8. Click **Scale** in the left-hand menu bar, slide the **Throughput Units** slider to 20, and click *Save* at the top.
9. Click ***Shared access policies*** in the left-hand menu bar.
	1. In the new panel click **RootManageSharedAccessKey**
	2. Copy the **Primary key** using the copy button to the right of the field, and add it to your settings.txt file.
	3. Copy the **Connection string–primary key** using the copy button to the right of the field, and add it to your settings.txt file.
	4. Click the **x** in the top right to dismiss this panel.
9. Click **Overview** on the left-hand menu bar, and then on the **+ Event Hub** button to add an event hub.
10. In the new panel:
    1. Enter **personalizedofferseh** for "Name".
    2. Enter **16** for "Partition Count".
    3. Enter **1** for "Message Retention".
    4. Click **Create** at the bottom.
11. Click on the ***Event Hubs*** option in the left-hand menu bar.
12. Click on the event hub named **personalizedofferseh** created through the previous steps. In the new panel:
    1. Click **+ Consumer group** at the top of the panel
    	a. Enter **clickactivityaggcg** for the 'name' field.
        b. Click **Create** at the bottom.
    2. Repeat 2 more times creating the following Consumer Groups:
    	- **clickactivitydbcg**
    	- **clickactivitydlcg**

| **Azure Event Hub**                 |                                     |
|-------------------------------------|-------------------------------------|
| serviceBusNamespace                 | **[unique string]**                 |
| eventHubSharedAccessPolicyKeyName   | RootManageSharedAccessKey           |
| eventHubSharedAccessPolicyKey       | **[Primary key]**                   |
| eventHubConnectionString            | **[Connection string–primary key]** |
| eventHubName                        | personalizedofferseh                |
| eventHubGroup1                      | clickactivityaggcg                  |
| eventHubGroup2                      | clickactivitydbcg                   |
| eventHubGroup3                      | clickactivitydlcg                   |
 
<a name="acdb"></a>
### Create Azure Cosmos DB
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Azure Cosmos DB** and hit "Enter" to search.
3. Click on **Azure Cosmos DB** offered by Microsoft in the "Storage" category.
4. Click **Create** at the bottom of the description panel.
5. In the Azure Cosmos DB panel:
   1. Enter your **unique string** for "ID".
   2. Choose **SQL** for "API".
   3. Click **Create** at the bottom.
6. Navigate back to the resource you have just created, then:
   1. Click on **Keys** on the left.
   2. Select **Read-write Keys** at the top of the new panel.
   3. Use the copy button to the right of the fields **URI**, **PRIMARY KEY**, and **PRIMARY CONNECTION STRING**, and add their values to the settings.txt file.
			 
| **Azure Cosmos DB**    |                                                             |
|------------------------|-------------------------------------------------------------|
| docDbUri               | **[URI]**                                                   |
| docDbKey               | **[PRIMARY KEY]**                                           |
| docDbConnectionString  | **[PRIMARY CONNECTION STRING]** (remove the ";" at the end) |

<a name="adls"></a>
## Create Azure Data Lake Store
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Data Lake Store** and hit "Enter" to search.
3. Click on **Data Lake Store** offered by Microsoft in the "Storage" category.
4. Click **Create** at the bottom of the description panel.
5. In the Data Lake Store panel:
   1. Enter your **unique string** for "Name".
   2. Click **Create** at the bottom.
 
| **Azure Data Lake Store** |                     |
|---------------------------|---------------------|
| adlStoreAccount           | **[unique string]** |

<a name="redis"></a>
## Create Azure Redis Cache
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Redis Cache** and hit "Enter" to search.
3. Click on **Redis Cache** offered by Microsoft in the "Databases" category.
4. Click **Create** at the bottom of the description panel.
5. In the Redis Cache panel:
   1. Enter your **unique string** for "DNS name".
   2. Choose **Standard C2 (2.5 GB Cache, Replication)** for "Pricing tier".
   3. Click **Create** at the bottom.
6. Navigate back to the resource you have just created, then:
   1. Click on **Access keys** on the left.
   2. Use the copy button to the right of the field **Primary**, and add the value to the settings.txt file.

| **Azure Redis Cache** |                     |
|-----------------------|---------------------|
| redisCacheName        | **[unique string]** |
| redisCacheKey         | **[Primary]**       |

<a name="af"></a>
## Create Azure Functions

### Create App Service plan
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **App Service Plan** and hit "Enter" to search.
3. Click on **App Service Plan** offered by Microsoft.
4. Click **Create** at the bottom of the description panel.
5. In the New App Service Plan panel:
   1. Enter your **unique string** for "App Service plan".
   2. Choose **Windows** for "Operating System".
   3. Click on **Pricing tier**, choose **S3 Standard** and click **Select** at the bottom.
   4. Click **Create** at the bottom.

### Set up Azure Functions settings
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Function App** and hit "Enter" to search.
3. Click on **Function App** offered by Microsoft in the "Web + Mobile" category.
4. Click **Create** at the bottom of the description panel.
5. In the Function App panel:
   1. Enter your **unique string** for "App name".
   2. Choose **App Service Plan** for "Hosting Plan".
   3. Click on **App Service plan/Location**, then click on the plan you created previously.
   4. Choose **Use existing** for "Storage", then enter your **unique string**.
   5. Click **Create** at the bottom.
6. Navigate back to the resource you have just created, then:
   1. Click on **Platform features** from the menu across the top.
   2. Click on **Application settings**.
   3. Choose **64-bit** for "Platform".
   4. Choose **On** for "Always On".
   5. Add all key-value pairs you have been storing in your settings.txt under the "Application settings" section (starting with storageAccountName).
   6. Click on **Save** at the top of the page.
   7. Close the tab by clicking on **x**.

### Upload the Azure Functions code 
1. From the "Application settings" tab, click on **Advanced tools (Kudu)**.
2. In the new window that opens, click on **Debug console**, then click on **CMD**.
3. Click on the **site** folder in the top half of the window.
4. Click on the **wwwroot** folder.
5. Find the functions.zip file you downloaded from GitHub, then drag it from your machine to the page onto the section on the right-hand side that reads **Drag here to upload and unzip**. Alternatively, unzip the file locally and transfer all its contents by dragging them onto the page.
6. Close the page to return to the Function Apps screen.
7. Refresh the list of functions by clicking on the "Refresh" icon to the right of your Function Apps instance.

### Populate Cosmos DB and Redis Cache
1. Select **SeedDocumentDb** from the list of functions.
2. Click **Run** at the top of the page, and wait for the function to complete. You may check the execution status by clicking on **Logs** at the bottom of the page.
3. Repeat for the function **SeedRedisCache**.

<a name="asa"></a>
## Create Azure Stream Analytics (ASA) Jobs
For this solution we will be creating 4 separate stream jobs, to better understand query performance.

### Create Product Views Stream Job
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Stream Analytics job** and hit "Enter" to search.
3. Click on **Stream Analytics job** offered by Microsoft in the "Internet of Things" category.
4. Click **Create** at the bottom of the description panel.
5. In the New Stream Analytics job panel:
   1. Enter **productViewsJob** for "Job name".
   2. Click **Create** at the bottom.
6. Navigate back to the resource you have just created, then:
    1. Click on **Inputs** on the left.
    2. Click on **+ Add stream input** at the top, then click on **Event Hub**.
    3. In the panel that opens:
       1. Enter **ClickActivity** for "Input alias".
       2. Select your **unique string** for "Event Hub namespace".
       3. Click on **Use existing** under "Event Hub name", then select **personalizedofferseh**.
       4. Select **RootManageSharedAccessKey** for "Event Hub policy name".
       5. Enter **clickactivityaggcg** for "Event Hub consumer group".
       6. Click **Save** at the bottom.
    4. Click on **Functions** on the left.
    5. Click on **+ Add** at the top, then click on **Javascript UDF**.
    6. In the panel that opens:
       1. Enter **productViewsJson** for "Function alias".
       2. Copy the contents of the file **ProductViewsUDF.txt**, and replace the current function definition displayed on the right side of the screen.
       3. Click **Save** at the bottom.
    7. Click on **Outputs** on the left.
    8. Click on **+ Add** at the top, then click on **Cosmos DB**.
    9. In the panel that opens:
       1. Enter **ProductViews** for "Output alias".
       2. Select your **unique string** for "Account id".
       3. Click on **Use existing** under "Database", then select **personalizedOffers**.
       4. Enter **productViewsCollection** for "Collection name pattern".
       5. Enter **id** for "Document id".
       6. Click **Save** at the bottom.
    9. Click on **Query** on the left.
   10. In the panel that opens:
       1. Copy the contents of the **ProductViewsQuery.txt** file, and replace the current query displayed on the right side of the screen.
       2. Click **Save** at the top, then confirm by clicking on **Yes**.
   11. Click on **Scale** on the left.
   12. In the panel that opens:
       1. Slide the "Streaming units" slider to **12**.
       2. Click **Save** at the top, then confirm by clicking on **Yes**.
   13. Click on **Overview** on the left.
   14. Click **Start** at the top.
   15. In the panel that opens, click **Start** at the bottom to start the stream job.

### Create Offer Views Stream Job
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Stream Analytics job** and hit "Enter" to search.
3. Click on **Stream Analytics job** offered by Microsoft in the "Internet of Things" category.
4. Click **Create** at the bottom of the description panel.
5. In the New Stream Analytics job panel:
   1. Enter **offerViewsJob** for "Job name".
   2. Click **Create** at the bottom.
6. Navigate back to the resource you have just created, then:
    1. Click on **Inputs** on the left.
    2. Click on **+ Add stream input** at the top, then click on **Event Hub**.
    3. In the panel that opens:
       1. Enter **clickactivitydbcg** for "Input alias".
       2. Select your **unique string** for "Event Hub namespace".
       3. Click on **Use existing** under "Event Hub name", then select **personalizedofferseh**.
       4. Select **RootManageSharedAccessKey** for "Event Hub policy name".
       5. Enter **clickactivitydbcg** for "Event Hub consumer group".
       6. Click **Save** at the bottom.
    4. Click on **Functions** on the left.
    5. Click on **+ Add** at the top, then click on **Javascript UDF**.
    6. In the panel that opens:
       1. Enter **offerViewsJson** for "Function alias".
       2. Copy the contents of the file **OfferViewsUDF.txt**, and replace the current function definition displayed on the right side of the screen.
       3. Click **Save** at the bottom.
    7. Click on **Outputs** on the left.
    8. Click on **+ Add** at the top, then click on **Cosmos DB**.
    9. In the panel that opens:
       1. Enter **OfferViews** for "Output alias".
       2. Select your **unique string** for "Account id".
       3. Click on **Use existing** under "Database", then select **personalizedOffers**.
       4. Enter **offerViewsCollection** for "Collection name pattern".
       5. Enter **id** for "Document id".
       6. Click **Save** at the bottom.
    9. Click on **Query** on the left.
   10. In the panel that opens:
       1. Copy the contents of the **OfferViewsQuery.txt** file, and replace the current query displayed on the right side of the screen.
       2. Click **Save** at the top, then confirm by clicking on **Yes**.
   11. Click on **Scale** on the left.
   12. In the panel that opens:
       1. Slide the "Streaming units" slider to **18**.
       2. Click **Save** at the top, then confirm by clicking on **Yes**.
   13. Click on **Overview** on the left.
   14. Click **Start** at the top.
   15. In the panel that opens, click **Start** at the bottom to start the stream job.

### Create Click Counts Stream Job
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Stream Analytics job** and hit "Enter" to search.
3. Click on **Stream Analytics job** offered by Microsoft in the "Internet of Things" category.
4. Click **Create** at the bottom of the description panel.
5. In the New Stream Analytics job panel:
   1. Enter **clickCountsJob** for "Job name".
   2. Click **Create** at the bottom.
6. Navigate back to the resource you have just created, then:
    1. Click on **Inputs** on the left.
    2. Click on **+ Add stream input** at the top, then click on **Event Hub**.
    3. In the panel that opens:
       1. Enter **clickactivitydbcg** for "Input alias".
       2. Select your **unique string** for "Event Hub namespace".
       3. Click on **Use existing** under "Event Hub name", then select **personalizedofferseh**.
       4. Select **RootManageSharedAccessKey** for "Event Hub policy name".
       5. Enter **clickactivitydbcg** for "Event Hub consumer group".
       6. Click **Save** at the bottom.
    4. Click on **Outputs** on the left.
    5. Click on **+ Add** at the top, then click on **Cosmos DB**.
    6. In the panel that opens:
       1. Enter **ProductCounts** for "Output alias".
       2. Select your **unique string** for "Account id".
       3. Click on **Use existing** under "Database", then select **personalizedOffers**.
       4. Enter **productCollection** for "Collection name pattern".
       5. Enter **id** for "Document id".
       6. Click **Save** at the bottom.
    7. Click on **+ Add** at the top, then click on **Cosmos DB**.
    8. In the panel that opens:
       1. Enter **UserCounts** for "Output alias".
       2. Select your **unique string** for "Account id".
       3. Click on **Use existing** under "Database", then select **personalizedOffers**.
       4. Enter **userCollection** for "Collection name pattern".
       5. Enter **id** for "Document id".
       6. Click **Save** at the bottom.
    9. Click on **Query** on the left.
   10. In the panel that opens:
       1. Copy the contents of the **ClickCountsQuery.txt** file, and replace the current query displayed on the right side of the screen.
       2. Click **Save** at the top, then confirm by clicking on **Yes**.
   11. Click on **Scale** on the left.
   12. In the panel that opens:
       1. Slide the "Streaming units" slider to **12**.
       2. Click **Save** at the top, then confirm by clicking on **Yes**.
   13. Click on **Overview** on the left.
   14. Click **Start** at the top.
   15. In the panel that opens, click **Start** at the bottom to start the stream job.

### Create Raw Data Stream Job
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In the **Overview** panel, click **+ Add** to add a new resource. Type **Stream Analytics job** and hit "Enter" to search.
3. Click on **Stream Analytics job** offered by Microsoft in the "Internet of Things" category.
4. Click **Create** at the bottom of the description panel.
5. In the New Stream Analytics job panel:
   1. Enter **rawDataJob** for "Job name".
   2. Click **Create** at the bottom.
6. Navigate back to the resource you have just created, then:
    1. Click on **Inputs** on the left.
    2. Click on **+ Add stream input** at the top, then click on **Event Hub**.
    3. In the panel that opens:
       1. Enter **clickactivitydlcg** for "Input alias".
       2. Select your **unique string** for "Event Hub namespace".
       3. Click on **Use existing** under "Event Hub name", then select **personalizedofferseh**.
       4. Select **RootManageSharedAccessKey** for "Event Hub policy name".
       5. Enter **clickactivitydlcg** for "Event Hub consumer group".
       6. Click **Save** at the bottom.
    4. Click on **Outputs** on the left.
    5. Click on **+ Add** at the top, then click on **Data Lake Store**.
    6. In the panel that opens:
       1. Enter **clickstreamdl** for "Output alias".
       2. Select your **unique string** for "Account name".
       3. Enter **personalizedoffers/clickstream/{date}** for "Path prefix pattern".
       4. Click on **Authorize**.
       5. Click **Save** at the bottom.
    7. Click on **+ Add** at the top, then click on **Data Lake Store**.
    8. In the panel that opens:
       1. Enter **offersdl** for "Output alias".
       2. Select your **unique string** for "Account name".
       3. Enter **personalizedoffers/offers/{date}** for "Path prefix pattern".
       4. Click on **Authorize**.
       5. Click **Save** at the bottom.
    9. Click on **Query** on the left.
   10. In the panel that opens:
       1. Copy the contents of the **RawDataQuery.txt** file, and replace the current query displayed on the right side of the screen.
       2. Click **Save** at the top, then confirm by clicking on **Yes**.
   11. Click on **Overview** on the left.
   12. Click **Start** at the top.
   13. In the panel that opens, click **Start** at the bottom to start the stream job.

<a name="startup"></a>
## Starting the Solution
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. Select the resource with type "App Service".
3. Repeat the following steps for the functions **PersonalizedOfferFunction**, **RedisProductTrigger**, **UpdateTopUsersCache**,  **UserSimulation**, and **UserSimulationStartup**:
   1. Click on the function name on the left.
   2. Click on **Manage** on the left.
   3. Set the "Function State" to **Enabled**.
4. Repeat the following steps for the functions **RedisProductTrigger** and **UpdateTopUsersCache**:
   1. Click on the function name on the left.
   2. Click **Run** at the top of the page, and wait for the function to complete. You may check the execution status by clicking on **Logs** at the bottom of the page.
5. Finally, start the simulation by running the **UserSimulationStartup** function:
   1. Click on the function name on the left.
   2. Click **Run** at the top of the page, and wait for the function to complete. You may check the execution status by clicking on **Logs** at the bottom of the page.

<a name="moreinfo"></a>
## More Information

The following links provide information on [monitoring](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#monitor-progress), [scaling](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#scaling) and [visualizing](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#visualization) the output of the deployed solution. Details for [stopping the solution](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#stopping) can also be found on the same page.

