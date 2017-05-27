# Personalized Offers - Manual Deployment Guide


## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Setup Steps](#setup-steps)
   - [General](#setup-steps)
   - [Azure Storage](#storage)
   - [Azure Machine Learning](#aml)
   - [Azure Event Hubs](#eventhub)
   - [Create Azure NoSQL (DocumentDB)](#docdb)
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

1.  An [Azure subscription](https://azure.microsoft.com/en-us/) with login credentials
2.  This solution requires the creation of:
	* 1 Data Lake Store
	* 4 Stream Jobs with a total of 43 Streaming Units
	* 1 Event Hub with 20 Throughput Units, 16 partitions and 4 Consumer Groups
	* 1 DocumentDB database with 6 collections each provisioned with 10000 RUs, 10GB (3 are Partitioned). 
	
Ensure adequate Data Lake Stores and Stream Processing units are available before provisioning. Please consider deleting any unused Data Lake Store from your subscription. You may contact Azure Support if you need to increase the limit.

## Architecture

The architecture diagram shows various Azure services that are deployed by [Personalized Offers Solution](placeholder link) using [Cortana Intelligence Solutions](https://gallery.cortanaintelligence.com/solutions), and how they are connected to each other in the end to end solution.

![Solution Diagram](https://cloud.githubusercontent.com/assets/16085124/24881519/084cd072-1e0c-11e7-9093-7eaf48d4d513.png)

## Setup Steps (Estimated Time: 3 hours)

The following are the steps to deploy the end-to-end solution.

### Accessing Files in the Git Repository

This tutorial will refer to files available in the Technical Deployment Guide section of the [Cortana Intelligence Churn Prediction git repository](https://github.com/Azure/cortana-intelligence-personalized-offers/). You can download all of these files at once by clicking the "Clone or download" button on the repository.

You can download or view individual files by navigating through the repository folders. If you choose this option, be sure to download the "raw" version of each file by clicking the filename to view it, then clicking Download. You will also find a settings.txt file in the ***src*** folder that can be used to keep track of settings you will need for configuring the Azure Functions. The names provided in the settings.txt file are the ones that will be used as the name of the setting and the entries you add will be the value for each setting.

### Choose a Unique String

You will need a unique string to identify your deployment because some Azure services, e.g. Azure Storage requires a unique name for each instance across the service. We suggest you use only letters and numbers in this string and the length should not be greater than 9.
 
We suggest you use "[UI]poffer[N]"  where [UI] is the user's initials,  N is a random integer that you choose and characters must be entered in in lowercase. Please open your settings.txt and write down "unique:[unique]" with "[unique]" replaced with your actual unique string.

		
### Create an Azure Resource Group for the solution
1. Log into the [Azure Management Portal](https://ms.portal.azure.com).
1. Click **Resource groups** button on upper left, and then click **+** button to add a resource group.
1. Enter your **unique string** for the resource group and choose your subscription.
1. For **Resource Group Location**, you should choose one of the following as they are the locations that offer all of the Azure services used in this guide (with the exception of Azure Data Factory, which need not be located in the same location):
  - South Central US
  - West Europe
  - Southeast Asia
 - Check the *Pin to dashboard* checkbox to make it easy to return to this Resource group. 
 - Click *Create*
 - This will bring you back to the resource group.

Please open your settings.txt file and save the information in the form of the following table. Please replace the content in [] with its actual value.  

| **Azure Resource Group** |                     |
|------------------------|---------------------|
| resource group name    |[unique]|
| region              |[region]||

### Instruction for Finding Your Resource Group Overview

In this tutorial, all resources will be generated in the resource group you just created. You can easily access these resources from the resource group overview page, which can be accessed as follows:

1. Log into the [Azure Management Portal](https://ms.portal.azure.com).
2. If you pinned the Resource Group when creating it, you will find your resource group here and can click on it to see all of the assoicated resources.
3. Click the **Resource groups** button on the upper-left of the screen.
4. Choose the subscription your resource group resides in.
5. Search for (or directly select) your resource group in the list of resource groups.
 
Note that you may need to close the resource description page to add new resources.

In the following steps, if any entry or item is not mentioned in the instruction, please leave it as the default value.

<a name="storage"></a>
### Create an Azure Storage Account

In this section we will go through the steps necessary to create the storage account, a blob associated with it, and upload some files to the blob. Along the way we will note down those values that we need for later in our settings.txt file.

1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to the resource group you just created.
2. In ***Overview*** panel, click **+** to add a new resource. Enter **Storage Account** and hit "Enter" key to search.
3. Click on **Storage Account** offered by Microsoft (in the "Storage" category).
4. Click **Create** at the bottom of the description panel.
5. Enter your **unique string** for "Name".
6. Make sure the selected resource group is the one you just created. If not, choose the resource group you created for this solution.
7. Leaving the default values for all other fields, click the **Create** button at the bottom.
8. Go back to your resource group overview and wait until the storage account is deployed. To check the deployment status, refresh the page or the list of the resources in the resource group as needed.

#### Get the Primary Key for the Azure Storage Account
These are the steps to get the access key that will be used in later steps.

1. Click the created storage account. In the new panel, click on **Access keys**.
2. In the new panel, click the "Click to copy" icon next to `key1`, and paste the key into your settings.txt file.
3. See below for the values needed and the

| **Azure Storage Account** |                     |
|------------------------|---------------------|
| storageAccountName        | **[unique string]** |
| storageAccountKey     | **[key1]**             ||
| storageAccountConnectionString | DefaultEndpointsProtocol=https;AccountName=**[uniques string]**;AccountKey=**[key1]**


#### Add Blob Storage and Upload Resources
These are the steps to create the **Blob storage** 

1. After getting the primary key, click the **Overview** on the left to return to the main panel for the Storage Account.
2. Click on **Blobs** in the central area of the main panel.
3. Click the **+Container** at the top of the new panel.
4. Enter the **[unique string]blob** for "Name".
5. Select **Blob** for "Access type".
6. Click **Create** at the bottom of the panel.
7. In the list of Blobs for this storage account click on the blob that you just created.

At this time if you haven't already make sure to download the following files from the **src** directory of the [github repository](https://github.com/Azure/cortana-intelligence-personalized-offers/blob/master/Manual%20Deployment%20Guide/src): 

* OfferPriority.txt
* offers.txt
* OfferThreshold.txt
* products.txt
* redisSeed.txt
* users.txt

To upload these files:

1. Click **Upload** at the top of the panel
2. In the panel that opens to the right click the **folder icon** to right of the "Files" field.
3. Navigate to where you saved the files on your computer.
4. Pressing the 'Control Key' on your keyboard and clicking on each of the files (single-click) will allow you to select all the files at once. 
5. Then at the bottom of the window click the **open** button.
6. Click **Upload** at the bottom of the panel.
7. Click the **X** at the top right of the panel for uploading files to dismiss it.
8. You should now see a list of files in the blob container
9. Click on the **Properties** button at the top of the panel.
10. Click the **Copy Icon** to the right of the URL field in the properties panel that opened.
11. Paste this into your settings.txt in place of the **[Blob Container URL]** as shown in the table below:

| **Azure Storage Blob Container Files** |					|
|------------------------------------------|--------------------|
| referenceCollectionFile1 | **[Blob Container URL]**/OfferPriority.txt|
| referenceCollectionFile2 | **[Blob Container URL]**/OfferThreshold.txt|
| offerFile | **[Blob Container URL]**/offers.txt|
| productFile | **[Blob Container URL]**/products.txt|
| userFile | **[Blob Container URL]**/users.txt|
| redisCacheSeedFile | **[Blob Container URL]**/redisSeed.txt|


<a name="aml"></a>
### Set up Azure Machine Learning
		
#### The Azure ML Model
The model used in this Guide is based on the [Personalized Offers Solution How To Guide](https://gallery.cortanaintelligence.com/Details/personalized-offers-solution-how-to-guide-4) from the gallery. The experiment used to train the model can be found [here](https://gallery.cortanaintelligence.com/Experiment/Personalized-Offers-Solution-How-To-Guide-2).


#### Create Azure Machine Learning Workspace

1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to the resource group you created.
2. In ***Overview*** panel, click **+** to add a new resource. Enter **Machine Learning Workspace** and hit "Enter" key to search.
3. Click on **Machine Learning Workspace** offered by Microsoft in the "Intelligence + analytics" category.
4. Click the **Create** button at the bottom of the description panel.
5. In the Machine Learning workspace panel:
    1. Enter your **unique string** for "Workspace name".
    2. Leave "Subscription", "Resource group", and "Location" as the default.
    3. Choose **Use existing** for "Storage account" and select the storage account you created earlier.  
    4. Choose **Standard** for "Workspace pricing tier".
    5. Choose **Create new** for "Web service plan".
    6. Click on **Web service plan tier**, choose **S1 Standard** and click **Select** at the bottom.
    7. Click **Create** at the bottom.

#### Deploy Azure Machine Learning Predictive Web Service
1. Go to the [Personalized Offers Solution How To Guide](https://gallery.cortanaintelligence.com/Details/personalized-offers-solution-how-to-guide-4) web page in the Cortana Intelligence Gallery.
2. Click the **Open in Studio** button on the right. Log in if needed.
3. Choose the region and workspace. For region, you should choose the region that your resource group resides. You can get the information from table "Azure Resource Group".  For workspace, choose the workspace you just created.
4. Wait until the experiment is copied.
5. Click **Run** at the bottom of the page. It takes around three minutes to run the experiment.
6. Click **Deploy Web Service** at the bottom of the page, choose **classic** web service, and click "Yes" to publish the web service. (You may receive a warning message that the web service input has not been selected: you can ignore this message.) This will lead you to the web service page.  The web service home page can also be found by clicking the ***WEB SERVICES*** button on the left-hand menu bar in your workspace.
7. Copy the ***API key*** from the web service home page and save it in the memo table given below.
8. Click the link ***REQUEST/RESPONSE*** under the ***API HELP PAGE*** section. On the REQUEST RESPONSE help page, copy the
***Request URI*** under the ***Request*** section and add it to the table below as you will need this information later

|**Machine learning Web Service** |      |
| --------------------------- |--------------------------|
| mlPublishedExperimentKey    | [API key from API help page] |
| mlPublishedExperimentEndpoint       |  [REQUEST/RESPONSE URI]                   ||


<a name="eventhub"></a>
### Create an Azure Event Hub
1. Go to the [Azure Portal](https://ms.portal.azure.com) and navigate to your resource group.
2. In "Overview" panel, click **+ Add** to add a new resource. Type **Event Hubs** and hit "Enter" key to search.
3. Click on **Event Hubs** offered by Microsoft in the "Internet of Things" category.
4. Click **Create** at the bottom of the description panel.
5. In the new panel for creating a namespace, enter your **unique string** for "Name".
6. Leaving the default values for all other fields, click the **Create** button at the bottom of the panel.
7. Return to your resource group's overview page. When it has finished deploying, click on the resource of type "Event hubs".
8. Click **Scale**, and slide the *Throughput Units* slider to 20 and click *Save* at the top
9. Click ***Shared access policies*** in the left-hand menu bar (under the ***SETTINGS*** heading).
	1. In the new panel click **RootManageSharedAccessKey**
	2. Copy the **PRIMARY KEY** using the copy button on the right of the field and add it to your settings.txt file.
	3. Copy the **CONNECTION STRING–PRIMARY KEY** using the copy button on the right and add it to your settings.txt file.
	4. Click the **X** in the top right to dismiss this panel.
9. Click **Overview** on the left and then the **+ Event Hub** button to add an event hub.
10. In the new panel:
    1. Enter **personalizedofferseh** for "Name".
    2. Enter **16** for "Partition Count".
    3. Enter **1** for "Message Retention".
    4. Click **Create** at the bottom.
11. Click on the ***Event Hubs*** option in the menu bar at left (under the "Entities" heading).
12. Click on the event hub named **personalizedofferseh** created through the previous steps. In the new panel:
    1. Click **+ Consumer Group** at the top of the panel
    	a. Enter **clickactivityaggcg** for the 'name' field.
    2. Repeat 2 more times creating the following Consumer Groups:
    	- **clickactivitydbcg**
    	- **clickactivitydlcg**
    	

| **Azure Event Hub** |                        |
|---------------------|------------------------|
| serviceBusNamespace | [unique string]  |
| eventHubSharedAccessPolicyKeyName  |     RootManageSharedAccessKey                 |
| eventHubSharedAccessPolicyKey       |  [PRIMARY KEY]     |
| eventHubConnectionString | [CONNECTION STRING–PRIMARY KEY]
| eventHubName | personalizedofferseh |
| eventHubGroup1 | clickactivityaggcg |
| eventHubGroup2 | clickactivitydbcg |
| eventHubGroup3 | clickactivitydlcg |
 
<a name="docdb"></a>
### Create Azure NoSQL (DocumentDB)
1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
2. In the left hand menu select *Resource groups* or if you pinned it to the Dashboard during the first step you should find it on the Dashboard as a tile.
3. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
4. At the top of the "Overview" blade in Resource Group blade click __+Add__.
5. In the *Search Everything* search box enter **DocumentDB**
6. Choose **NoSQL (DocumentDB)** from the results, then click *Create*
	1. Enter **unique string** for *ID*
	2. Pick the same *Subscription*, *Resource Group*, and *Location* as the other resources that have been created.
7. Return back to *Resource groups* and choose the resource group for this solution.
8. Click on the resource of type **NoSQL (DocumentDB) account**, then on the subsequent blade perform the following steps:
	1. Click on **Keys** under the SETTINGS heading on the left
	2. From the top of the new panel select **Read-write Keys** 
	3. From this panel use the copy button to the right of the fields **URI**, **PRIMARY KEY** and **PRIMARY CONNECTION STRING** and add them to the settings.txt file.
			 
| **Azure NoSQL (DocumentDb)** |                        |
|---------------------|------------------------|
| docDbUri | [URI]  |
| docDbKey | [PRIMARY KEY]
| docDbConnectionString  |     [PRIMARY CONNECTION STRING, **remove the ';' at the end**]                 |


<a name="adls"></a>
## Create Azure Data Lake Store
 1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 2. In the left hand menu select *Resource groups* or if you pinned it to the Dashboard during the first step you should find it on the Dashboard as a tile.
 3. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 4. At the top of the "Overview" blade in Resource Group blade click __+Add__.
 5. In the *Search Everything* search box enter **Data Lake Store**
 6. Choose **Data Lake Store** from the results, then click *Create*
 7. Enter **unique string** for the *Name*
 8. Pick the same *Subscription* and *Resource Group* as used for other services in this solution
 9. Pick a *Location* near the same region as the other services in this solution
 10. Select **Pay-as-you-go** for *Pricing*
 11. Click __Create__ to create the Data Lake Store
 
| **Azure NoSQL (DocumentDb)** |                        |
|---------------------|------------------------|
| adlStoreAccount | [unique string]  |

<a name="redis"></a>
## Create Azure Redis Cache
 1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 2. In the left hand menu select *Resource groups*
 3. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 4. At the top of the Resource Group blade click __+Add__.
 5. In the *Search Everything* search box enter **Redis Cache**
 6. Choose **Redis Cache** from the results then click *Create*
 7. Enter **unique string** for *DNS name*
 8. Keep *Subscription*, *Resource group*, and *Location* same as previous services
 9. For the *Pricing tier* click on it:
 	1. Select *View all* from the top right and select **C2 Standard**
 	2. Click __Select__ at the bottom.
 10. Click __Create__
 11. Return to the Resource Group and locate the item of type **Redis Cache**
 12. In the new panel select **Access Keys** under SETTINGS not the left.
 13. Use the copy button to the right of the **Primary** field to copy the key and add it to the settings.txt file

| **Azure Redis Cache** |                        |
|---------------------|------------------------|
| redisCacheName | [unique string]  |
| redisCacheKey | [Primary] |

<a name="af"></a>
## Create Azure Functions

### Create App Service plan
 1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 2. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 3. At the top of the Resource Group blade click __+Add__.
 4. In the *Search Everything* search box enter **App Service Plan**
 5. Choose **App Service Plan** from the results then click *Create*
 6. Enter **unique string** for *App Service plan*
 7. Keep the *Subscription*, *Resource Group*, and *Location* the same as the previous services
 8. Select **Windows** for *Operating System*
 9. For *Pricing Tier* select ***S3 Standard***
 10. Click __Create__


### Setup Azure Functions Settings
 1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 2. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
 3. At the top of the Resource Group blade click __+Add__.
 4. In the *Search Everything* search box enter **Function App**
 5. Choose **Function App** from the results then click *Create*
 	1. Enter **unique string** for *App name*
 	2. Keep *Subscription* and *Resource Group* the same as the previous services
 	3. For *Hosting Plan* select **App Service Plan** then select **unique string** for *App Service plan/Location*
 	4. Select **unique string** for the *Storage Account*
 	5. Click _Create_
 6. Return to the Resource Group and select the line of type **App Service**
 7. From the menu across the top select *Platform Features*, then select *Application Settings* from GENERAL SETTINGS
 8. In the new panel configure the following:
 	1. *Platform* : **64-bit**
 	2. *Always On* : **On**
 	3. *App Settings* is where we want to take all the key, value pairs from our settings.txt file and add them to this section. The first column in each row is the *key* and the second column is the *value* (these are the items you have been keeping track of from each of the previous sections).
	4. Click __Save__ at the top to save these changes and close the Blade with the **X** after the changes have saved.
	
	
### Upload the Azure Functions Code 
1. Now select *Advanced tools (Kudu)* from the DEVELOPMENT TOOLS Section
	1. In the new window that opens select **Debug Console -> CMD**
 	2. Click on the **site** folder in the top half of the window
 	3. Click on the **wwwroot** folder
	
 		a. Find the folder on your machine where you downloaded the contents of the **src** folder from GitHub
		
 		b. Find the functions.zip file
		
 		c. Drag this file from your machine to the web page to the area below where it shows the **Size** column. It should highlight and say: **Drag here to upload and unzip**
		
		d. This should create the following folders: PersonalizedOfferFunction, PersonalizedOffersUtils, RedisProductTrigger, SeedDocumentDb, SeedRedisCache, UpdateTopUsersCache, UserSimulation, and UserSimulationStartup
		
	4. Close the tab opened by the App Service Editor and return to the Azure Functions Screen.

### Populate DocumentDB and Redis Cache
1. From the main Azure Functions page:
	1. Select **SeedDocumentDb**
	
		a. Click **Run** at the top of the page
		
		b. Wait for the function to complete (Click **Logs** at the bottom to see the function executing)
		
	2. Select **SeedRedisCache**
	
		a. Click **Run** at the top of the page
		
		b. Wait for the function to complete (Click **Logs** at the bottom to see the function executing) 

<a name="asa"></a>
## Create Azure Stream Analytics (ASA) Jobs
For this solution we will be creating 4 separate stream jobs so we can more easily see how each stream job is doing and better understand query performance.

### Create Product Views Stream Job
This stream job will take the data generated by the PersonalizedOfferFunction and aggregate it to write to the productViewsCollection in DocumentDb.

1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
2. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
3. At the top of the Resource Group blade click __+Add__.
4. In the *Search Everything* search box enter **Stream Analytics job**
5. Choose **Stream Analytics job** from the results then click *Create*
6. Enter **productViewsJob** as the name and choose the subscritpion, resource group and location using previous choices.
7. Click *Create* and return to the *Resource groups* blade. 
Select the **productViewsJob** resource to open the Stream Analytics job to modify the job settings.
8. Click *Inputs* on the left to open the Inputs Blade  
    1. At the top of the *Inputs* page click __+ADD INPUT__
    2. In the new panel:
    	
    	a. *Input Alias* : **ClickActivity**
    	
    	b. *Source Type* : **Data stream**
    2. *Source* : **Event hub**
    3. *Subscription* : Use event hub from current subscription
    4. *Service bus namespace* : **unique string**
    5. *Event hub name* : **personalizedofferseh**
    6. *Event hub policy name* : **RootManageSharedAccessKey**
    7. *Event hub consumer group* : **clickactivityaggcg**
    8. *Event serialization format* : JSON
    9. *Encoding* : UTF-8
    10. Click __Create__
9. Click **Functions** on the left to open the Functions Blade
 	1. Click __Add__ at the top
    2. Enter **productViewsJson** for *Function Alias*
    3. Select ***Javascript*** for *Function Type*
    4. Select ***Any*** for *Output Type*
    5. On the right side of the screen copy the contents of the file **ProductViewsUDF.txt** from the **src** directory and replace the contents currently in the function.
   	6. Click __Create__
10. Click *Outputs* on the left to open the Outputs Blade
    1. Click __Add__ at the top
    2. In the new panel:
    
    	a. *Output Alias* : **ProductViews**
    	
    	b. *Sink* : **DocumentDB**
    	
    	c. *Account id*: **unique string**
    	
    	d. *Database* : **personalizedOffers**
    	
    	e. *Collection name pattern* : **productViewsCollection**
		
		f. *Document id* : **id**
		
		g. Click __Create__

11. Click *Query* on the left to open the Query Blade
    1. Copy the contents of the **ProductViewsQuery.txt** file from the **src** directory
    2. Paste it into the query area replacing the current contents
    3. Click *Save*
12. Click *Scale* on the left and slide the *Streaming units* slider to ***12*** and click __Save__
13. Click *Overview* on the left

14. Click *Start* at the top to begin the stream job

### Create Offer Views Stream Job
This stream job will take the data generated by the PersonalizedOfferFunction and aggregate it to write to the offerViewsCollection in DocumentDb.

1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
2. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
3. At the top of the Resource Group blade click __+Add__.
4. In the *Search Everything* search box enter **Stream Analytics job**
5. Choose **Stream Analytics job** from the results then click *Create*
6. Enter **offerViewsJob** as the name and choose the subscritpion, resource group and location using previous choices.
7. Click *Create* and return to the *Resource groups* blade. 
Select the **offerViewsJob** resource to open the Stream Analytics job to modify the job settings.
8. Click *Inputs* on the left to open the Inputs Blade  
    1. At the top of the *Inputs* page click __+ADD INPUT__
    2. In the new panel:
    	
    	a. *Input Alias* : **clickactivitydbcg**
    	
    	b. *Source Type* : **Data stream**
    2. *Source* : **Event hub**
    3. *Subscription* : Use event hub from current subscription
    4. *Service bus namespace* : **unique string**
    5. *Event hub name* : **personalizedofferseh**
    6. *Event hub policy name* : **RootManageSharedAccessKey**
    7. *Event hub consumer group* : **clickactivitydbcg**
    8. *Event serialization format* : JSON
    9. *Encoding* : UTF-8
    10. Click __Create__
9. Click **Functions** on the left to open the Functions Blade
 	1. Click __Add__ at the top
    2. Enter **offerViewsJson** for *Function Alias*
    3. Select ***Javascript*** for *Function Type*
    4. Select ***Any*** for *Output Type*
    5. On the right side of the screen copy the contents of the file **OfferViewsUDF.txt** from the **src** directory and replace the contents currently in the function.
   	6. Click __Create__
10. Click *Outputs* on the left to open the Outputs Blade
    1. Click __Add__ at the top
    2. In the new panel:
    
    	a. *Output Alias* : **OfferViews**
    	
    	b. *Sink* : **DocumentDB**
    	
    	c. *Account id*: **unique string**
    	
    	d. *Database* : **personalizedOffers**
    	
    	e. *Collection name pattern* : **offerViewsCollection**
		
		f. *Document id* : **id**
		
		g. Click __Create__

11. Click *Query* on the left to open the Query Blade
    1. Copy the contents of the **OfferViewsQuery.txt** file from the **src** directory
    2. Paste it into the query area replacing the current contents
    3. Click *Save*
12. Click *Scale* on the left and slide the *Streaming units* slider to ***18*** and click __Save__
13. Click *Overview* on the left

14. Click *Start* at the top to begin the stream job

### Create Click Counts Stream Job
This stream job will take the data generated by the PersonalizedOfferFunction and aggregate it to write to the productCollection and the userCollection in DocumentDb.

1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
2. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
3. At the top of the Resource Group blade click __+Add__.
4. In the *Search Everything* search box enter **Stream Analytics job**
5. Choose **Stream Analytics job** from the results then click *Create*
6. Enter **clickCountsJob** as the name and choose the subscritpion, resource group and location using previous choices.
7. Click *Create* and return to the *Resource groups* blade. 
Select the **clickCountsJob** resource to open the Stream Analytics job to modify the job settings.
8. Click *Inputs* on the left to open the Inputs Blade  
    1. At the top of the *Inputs* page click __+ADD INPUT__
    2. In the new panel:
    	
    	a. *Input Alias* : **clickactivitydbcg**
    	
    	b. *Source Type* : **Data stream**
    2. *Source* : **Event hub**
    3. *Subscription* : Use event hub from current subscription
    4. *Service bus namespace* : **unique string**
    5. *Event hub name* : **personalizedofferseh**
    6. *Event hub policy name* : **RootManageSharedAccessKey**
    7. *Event hub consumer group* : **clickactivitydbcg**
    8. *Event serialization format* : JSON
    9. *Encoding* : UTF-8
    10. Click __Create__
9. Click *Outputs* on the left to open the Outputs Blade
    1. Click __Add__ at the top
    2. In the new panel:
    
    	a. *Output Alias* : **ProductCounts**
    	
    	b. *Sink* : **DocumentDB**
    	
    	c. *Account id*: **unique string**
    	
    	d. *Database* : **personalizedOffers**
    	
    	e. *Collection name pattern* : **productCollection**
		
		f. *Document id* : **id**
		
		g. Click __Create__
	3. Click __Add__ at the top
    4. In the new panel:
    
    	a. *Output Alias* : **UserCounts**
    	
    	b. *Sink* : **DocumentDB**
    	
    	c. *Account id*: **unique string**
    	
    	d. *Database* : **personalizedOffers**
    	
    	e. *Collection name pattern* : **userCollection**
		
		f. *Document id* : **id**
		
		g. Click __Create__

10. Click *Query* on the left to open the Query Blade
    1. Copy the contents of the **ClickCountsQuery.txt** file from the **src** directory
    2. Paste it into the query area replacing the current contents
    3. Click *Save*
11. Click *Scale* on the left and slide the *Streaming units* slider to ***12*** and click __Save__
12. Click *Overview* on the left

13. Click *Start* at the top to begin the stream job

### Create Raw Data Stream Job
This stream job will take the data generated by the PersonalizedOfferFunction and aggregate it to write to the productCollection and the userCollection in DocumentDb.

1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
2. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
3. At the top of the Resource Group blade click __+Add__.
4. In the *Search Everything* search box enter **Stream Analytics job**
5. Choose **Stream Analytics job** from the results then click *Create*
6. Enter **rawDataJob** as the name and choose the subscritpion, resource group and location using previous choices.
7. Click *Create* and return to the *Resource groups* blade. 
Select the **rawDataJob** resource to open the Stream Analytics job to modify the job settings.
8. Click *Inputs* on the left to open the Inputs Blade  
    1. At the top of the *Inputs* page click __+ADD INPUT__
    2. In the new panel:
    	
    	a. *Input Alias* : **clickactivitydlcg**
    	
    	b. *Source Type* : **Data stream**
    2. *Source* : **Event hub**
    3. *Subscription* : Use event hub from current subscription
    4. *Service bus namespace* : **unique string**
    5. *Event hub name* : **personalizedofferseh**
    6. *Event hub policy name* : **RootManageSharedAccessKey**
    7. *Event hub consumer group* : **clickactivitydlcg**
    8. *Event serialization format* : JSON
    9. *Encoding* : UTF-8
    10. Click __Create__
9. Click *Outputs* on the left to open the Outputs Blade
    1. Click __Add__ at the top
    2. In the new panel:
    
    	a. *Output Alias* : **clickstreamdl**
    	
    	b. *Sink* : **Data Lake Store**
    	
		c. At this point you have to click **Authorize Connection**
    	    	
    	d. *Subscription* : This should be the subscription that the Data Lake you created earlier is in (same as your resource group)
    	
    	e. *Account Name : **unique string**
		
		f. *Path prefix pattern*: **personalizedoffers/clickstream/{date}**
		
		g. Click __Create__
	3. Click __Add__ at the top
    4. In the new panel:
    
    	a. *Output Alias* : **offersdl**
    	
    	b. *Sink* : **Data Lake Store**
    	
		c. At this point you have to click **Authorize Connection**
    	    	
    	d. *Subscription* : This should be the subscription that the Data Lake you created earlier is in (same as your resource group)
    	
    	e. *Account Name : **unique string**
		
		f. *Path prefix pattern*: **personalizedoffers/offers/{date}**
		
		g. Click __Create__

10. Click *Query* on the left to open the Query Blade
    1. Copy the contents of the **RawDataQuery.txt** file from the **src** directory
    2. Paste it into the query area replacing the current contents
    3. Click *Save*
    
11. Click *Overview* on the left

12. Click *Start* at the top to begin the stream job


<a name="startup"></a>
## Starting the Solution
1. Log into the [Azure Management Portal](https://ms.portal.azure.com) 
2. Locate the resource group  you created for this project and click on it displaying the resources associated with the group in the resource group blade.
3. Select the line with the type **App Service**
4. On the Azure Functions web page we now need to enable each of the functions that are needed for the application. For the **PersonalizedOfferFunction**, **RedisProductTrigger**, **UpdateTopUsersCache**, **UserSimulation**, and **UserSimulationStartup** the following steps need to be taken:

	i. Click on the function from the left navigation.
	
	ii. Select the **Manage** tab for that Function.
	
	iii. Set the **Function State** to **Enabled**.
	
3. Now we need to run the **RedisProductTrigger** and **UpdateTopUsersCache** Functions:

	i. Click on the function from the left navigation.
	
	ii. Click **Run** at the top of the Function editor.
	
	iii. You should see something like the following:
	
        2017-03-18T11:08:20.867 Function started (Id=efacd569-5c8b-4380-b23d-4a18e7fee5cc)
        2017-03-18T11:08:21.164 C# Timer trigger function executed at: 3/18/2017 11:08:21 AM
        2017-03-18T11:08:22.089 Function completed (Success, Id=efacd569-5c8b-4380-b23d-4a18e7fee5cc)

4. The last step is to start the simulation with the **UserSimulationStartup** Function. The steps are the same as the ones in **Step 3** above. The only difference will be this function will take longer to complete.


<a name="moreinfo"></a>
## More Information

The following links provide information on [monitoring](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#monitor-progress), [scaling](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#scaling) and [visualizing](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#visualization) the output of the deployed solution. Details for [stopping the solution](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#stopping) can also be found on the same page.

