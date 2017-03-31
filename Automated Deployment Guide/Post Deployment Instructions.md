# [Personalized Offers for Retail Solution](placeholder for gallery url)

This document is focusing on the post deployment instructions for the automated deployment through [Cortana Intelligence Solutions](https://gallery.cortanaintelligence.com/solutions). The source code of the solution as well as manual deployment instructions can be found [here](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/tree/master/Manual%20Deployment%20Guide).

### Quick links
[Starting the solution](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#starting-the-solution) - see how you can monitor the resources that have been deployed to your subscription.

[Monitor Progress](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#monitor-progress) - see how you can monitor the resources that have been deployed to your subscription.

[Visualization Steps](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#visualization) - instructions to connect up a Power BI dashboard to your deployment that visualized the results.

[Scaling](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md#scaling) - guidance on how to think about scaling this solution according to your needs.

# Architecture
The architecture diagram shows various Azure services that are deployed by [Personalized Offers for Retail Solution](palceholder link) using [Cortana Intelligence Solutions](https://gallery.cortanaintelligence.com/solutions), and how they are connected to each other in the end to end solution.

![Solution Diagram](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Figures/PersonalizedOffersArchitecture.png)


1.	User activity on the website is simulated with an **Azure Function** and a pair of **Azure Storage Queues**, these would not be part of a production solution.

2. Personalized Offer Functionality is implemented as an **Azure Function**. This is the key function that ties everything together to produce an offer and record activity. Data is read in from **Azure Redis Cache** and **Azure DocumentDb**, product popularity probability is returned by **Azure Machine Learning** (if no history for the user exists then cold start values for product popularity are read in from **Azure Redis Cache**). 

3. Raw user activity data (Product and Offer Clicks), Offers made to users, and performance data (for **Azure Functions** and **Azure Machine Learning**) are sent to **Azure Event Hub**.

4. The offer is returned to the User. In our simulation this is done by writing to an **Azure Storage Queue** and picked up by an **Azure Function** in order to produce the next user action.

5.	**Azure Stream Analytics** analyzes the data to provide near real-time analytics on the input stream from the **Azure Event Hub**. The aggregated data is sent to **Azure DocumentDB* and directly published to **PowerBI** for visualization.  The raw data is sent to **Azure Data Lake Storage**. 
</Guide>

All the resources listed above besides Power BI are already deployed in your subscription. The following instructions will guide you on how to start the solution, monitor your solution and create visualizations in Power BI.

# Post Deployment Instructions
Once the solution is deployed to the subscription, you can see the services deployed by clicking the resource group name on the final deployment screen in the CIS.

![CIS resource group link](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Figures/CIS_ResourceGroup.png)

This will show all the resources under this resource group on [Azure management portal](https://portal.azure.com/). This is a good time to pin the resource to your main Azure Portal page to make it easy to return to in any of the following steps. The pin icon can be found at the top right corner of the Resource Group Overview Blade.

## **Starting the solution**
After successful deployment, there are a few steps you need to take to start your solution and to scale it to a reasonable simulation level:

1. From the deployment page or from the Resource Group Page we will need to start each of the **Stream Analytics Jobs**.
	
	a. From the resource group page or from the deployment page click on the link for **StreamProductViewsJob**, this should open it in a new tab or window.
	
	b. Click on the ***Start*** at the top of the over view blade. When prompted select **now** as the time to begin the job. You can now close this tab.
	
	c. From the resource group page or from the deployment page click on the link for **StreamOfferViewsJob**, this should open it in a new tab or window.
	
	d. Click on the ***Start*** at the top of the over view blade. When prompted select **now** as the time to begin the job. You can now close this tab.
	
	e. From the resource group page or from the deployment page click on the link for **StreamClickCountsJob**, this should open it in a new tab or window.
	
	f. Click on the ***Start*** at the top of the over view blade. When prompted select **now** as the time to begin the job. You can now close this tab.
	
	g. From the resource group page or from the deployment page click on the link for **StreamRawDataJob**, this should open it in a new tab or window.
	
	h. Click on the ***Outputs*** section of the menu on the left side. In the **Outputs** blade you will see two Outputs: **clickstreamdl** and **offersdl**. For each of these you will need to click on the output. The new blade that opens with the details will have a ***Renew Authorization*** button on it. Click the button and follow the authentication prompts. Once the output is authenticated, click the ***Save*** button and repeat with the second output.
		
	i. Click the ***Overview*** section of the menu on the left side.
	
	j. Click on the ***Start*** at the top of the over view blade. When prompted select **now** as the time to begin the job. You can now close this tab.


2. From the deployment page or from the Resource Group page click on the Functions App. The name will begin with **functions-** and will be of type **App Service**.

	a. Click on the **Function app settings** link on the lower left side of the blade.
	
	b. Click on **Go to App Service Editor**. This will open a new window with an editor for coding Azure Functions.
	
	c. Find the **hosts.json** file and click on it. Replace the current content with the following:

        {
			"queues": 
			{
				"maxPollingInterval": 700,
				"batchSize": 2,
				"maxDequeueCount": 2
			}
        }
	d. Once this is saved (This can be seen at the top right of the editor) you can close this new tab	
3. On the Azure Functions web page we now need to enable each of the functions that are needed for the application. For the **PersonalizedOfferFunction**, **RedisProductTrigger**, **UpdateTopUsersCache**, **UserSimulation**, and **UserSimulationStartup** the following steps need to be taken:

	a. Click on the function from the left navigation.
	
	b. Select the **Manage** tab for that Function.
	
	c. Set the **Function State** to **Enabled**.
	
5. Now we need to run the **RedisProductTrigger** and **UpdateTopUsersCache** Functions:

	a. Click on the function from the left navigation.
	
	b. Select the **Develop** tab for that Function.
	
	c. Click **Run** at the top of the Function editor.
	
	d. You should see something like the following:
	
        2017-03-18T11:08:20.867 Function started (Id=efacd569-5c8b-4380-b23d-4a18e7fee5cc)
        2017-03-18T11:08:21.164 C# Timer trigger function executed at: 3/18/2017 11:08:21 AM
        2017-03-18T11:08:22.089 Function completed (Success, Id=efacd569-5c8b-4380-b23d-4a18e7fee5cc)

6. The last step is to start the simulation with the **UserSimulationStartup** Function. The steps are the same as the ones in **Step 5** above. The only difference will be this function will take longer to complete.

Now your simulation is running and is active. The information below will help you see how everything is running.

## **Monitor progress**

For each of the services in this solution going to from the resource group page to the service will provide you with some information on how the service is running. Often there is a Metrics blade listed on the left side of that service that will provide more information. For each of the services below a link is provided that explains monitoring each of the services in more detail. 

#### Azure App Service ####
For more information on how to monitor Azure App Service you can take a look [here](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-monitor).

#### Azure DocumentDB ####
For more information on how to monitor DocumentDB take a look at the documentation [here](https://docs.microsoft.com/en-us/azure/documentdb/documentdb-monitor-accounts).

#### Azure Functions ####

For more information on how to monitor Azure Functions take a look at the documentation [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring).

#### Azure Machine Learning Web Service

You can view the machine learning experiment by navigating to your Machine Learning Workspace. The machine learning model is deployed as an Azure Web Service. For more information on monitoring the web service endpoint take a look at the documentation [here](https://docs.microsoft.com/en-us/azure/machine-learning/machine-learning-manage-new-webservice).


#### Azure Redis Cache ####

For more information on how to monitor Azure Redis Cache take a look at the documentation [here](https://docs.microsoft.com/en-us/azure/redis-cache/cache-how-to-monitor).

## **Visualization**
Power BI dashboard can be used to visualize the real-time energy consumption data as well as the updated energy forecast results. The following instructions will guide you to build a dashboard to visualize data from database and from real-time data stream.


### Visualize Personalized Offer Data from Azure DocumentDB

The goal of this part is to get a visual overview of how the Personalized Offers for Retail Solution is running. Power BI can directly connect to an Azure DocumentDB as its data source, where the solution results are stored.

> Note:  1) In this step, the prerequisite is to download and install the free software [Power BI desktop](https://powerbi.microsoft.com/desktop). 2) We recommend you start this process 2-3 hours after you deploy the solution so that you have more data points to visualize.

1.  Get the database credentials.

    You can find your DocumentDB URI and Primary Key. Go to the Document DB page for your solution by going to the Resource page for your solution and selecting the Document DB Service. On the left side of the page there is a **Keys** section where this information can be found.
    
2.	Update the data source of the Power BI file
	
  - Make sure you have installed the latest version of [Power BI desktop](https://powerbi.microsoft.com/desktop).

  -	In this GitHub repository, you can download the **'PersonalizedOffersSolution.pbix'** file under the folder **'Power BI'** and then open it. **Note:** If you see an error massage, please make sure you have installed the latest version of Power BI Desktop.

  - On the top of the file, click **‘Edit Queries’** button.

  - In the pop out window, you will see 9 Queries on the left hand side and on the right side you will see Query Settings. For each of the following queries: **offerCollection**, **products**, **userProductViews**, **referenceCollection**, **users**, **userOfferViews**, and **OfferProducts**, follow the steps below:
  
  	- Select the query on the left
  	- Click the **gear** icon to the right of **Source** in the **Applied Steps** section of the **Query Settings** 
  	- Put in the URI that you got from DocumentDB into the **URL** field and click **OK**
  	- A prompt should ask you for the Account Key enter the Primary Key from DocumentDB in the **Account key** field (You should only have to do this for the first query)
  	- Repeat these steps for the other queries listed above.
  
  - For the these **userProductViews** there are some additional steps to take. Each of the **Applied Steps** must be examined to see that the right values are there.
  	- click on the 6th step **Expanded productviews** gear icon.
  	- in this window click the **Load More** link near the bottom.
  	- verify that all products 1-25 are there in the list and select them all (using the **Select All Columns** checkbox at the top).
  	- click **OK**
  	- click on the next step **Renamed Columns1** and make sure the product number columns at the top are all correct and of type: 1, 2, 3, ... 25 for the columns representing the products. If prompted to okay the change go ahead and click **Insert**.

  - On the top of the screen, you will see a button **'Close & Apply Changes'**,

  - Now the dashboard is updated to connect to your database. You can click **'Refresh'** button on the top to get the latest visualization.

1. (Optional) Publish the dashboard to [Power BI online](http://www.powerbi.com/).
    Note that this step needs a Power BI account (or Office 365 account).

      - Click **"Publish"** on the top panel. Choose **'My Workspace'** and few seconds later a window appears displaying "Publishing succeeded".

      - Click the link on the screen to open it in a browser. On the left panel, go to the **Dataset** section, right click the dataset *'EnergyDemandForecastSolution'*, choose **Dataset Settings**. In the pop out window, click **Enter credentials** and enter your database credentials by following the instructions. To find detailed instructions, please see [Publish from Power BI Desktop](https://support.powerbi.com/knowledgebase/articles/461278-publish-from-power-bi-desktop).

      - Now you can see new items showing under 'Reports' and 'Datasets'. To create a new dashboard: click the **'+'** sign next to the
        **Dashboards** section on the left pane. Enter the name "Energy Demand Forecasting Demo" for this new dashboard.

      - Once you open the report, click   ![Pin](Figures/PowerBI-4.png) to pin all the
		visualizations to your dashboard. To find detailed instructions, see [Pin a tile to a Power BI dashboard from a report](https://support.powerbi.com/knowledgebase/articles/430323-pin-a-tile-to-a-power-bi-dashboard-from-a-report). Here is an example of the dashboard.

      ![Dashboard Example](Figures/PowerBI-11.png)

### Visualize Solution Activity From Real-time Data Stream

The goal of this part is to visualize the real-time information about our solution. Power BI can connect to a real-time data stream through Azure Stream Analytics.

> Note: A [Power BI online](http://www.powerbi.com/) account is required to perform the following steps. If you don't have an account, you can [create one here](https://powerbi.microsoft.com/pricing).

1. Login on [Power BI online](http://www.powerbi.com)

    -   On the left panel Datasets section in My Workspace, you should be able to see a new dataset showing on the left panel of Power BI. This is the streaming data you pushed from Azure Stream Analytics in the previous step.

    -   Make sure the ***Visualizations*** pane is open and is shown on the
    right side of the screen.

2. Now you can directly create a visualization on Power BI online. We will use this example to show you how to create the "Demand by Timestamp" tile:
	-	Click dataset **EnergyForecastStreamData** on the left panel Datasets section.

	-	Click **"Line Chart"** icon.![Line Chart](Figures/PowerBI-3.png)

	-	Click EnergyForecastStreamData in **Fields** panel.

	-	Click **“time”** and make sure it shows under "Axis". Click **“demand”** and make sure it shows under "Values".

	-	Click **'Save'** on the top and name the report as “EnergyStreamDataReport”. The report named “EnergyStreamDataReport” will be shown in Reports section in the Navigator pane on left.

	-	Click **“Pin Visual”**![](Figures/PowerBI-4.png) icon on top right corner of this line chart, a "Pin to Dashboard" window may show up for you to choose a dashboard. Please select "EnergyStreamDataReport", then click "Pin".


## **Customization**
You can reuse the source code in the [Manual Deployment Guide](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/tree/master/Manual%20Deployment%20Guide) to customize and rebuild the solution for your data and business needs.

## **Scaling**
Many of the services used in this solution were selected because they scale up/out, are available in many regions, and have multiple ways they can be further tuned.  For some additional ideas on scaling see the links below:

* **Azure Traffic Manager** - Used to route a user request to the service endpoint nearest to the user. [Documentation Link](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)
* **Azure Application Gateway** - Load Balancing your Application. [Documentation Link](https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-introduction)
* **Azure Machine Learning** - Add more endpoints to handle additional load. [Documentation Link](https://docs.microsoft.com/en-us/azure/machine-learning/machine-learning-scaling-webservice)
* Make use of **Partitioning** all the way from ingestion in **Azure Event Hub**, processing in **Azure Stream Analytics** and **Azure Functions**, and to storage in **Azure Document DB**.
	* Event Hub Partitioning - [Documentation Link 1](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-what-is-event-hubs) and [Documentation Link 2](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-programming-guide#partition-key)
	* [Scaling by partitioning queries in Stream Analytics](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-scale-jobs)
	* [Partitioning output from Stream Analytics](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-documentdb-output)
	* [Partitioning in DocumentDB](https://docs.microsoft.com/en-us/azure/documentdb/documentdb-partition-data)
	* Azure Functions scaling via [Service Plan](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale) and by editing the [host.json](https://github.com/Azure/azure-webjobs-sdk-script/wiki/host.json) file
	
