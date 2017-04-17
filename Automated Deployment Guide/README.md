
# [Personalized Offers](placeholder for gallery url)

This folder contains the post-deployment instructions for the deployable Personalized Offers solution in the Cortana Intelligence Gallery. To start a new solution deployment, visit the gallery page [here](placeholder for gallery url).

<Guide type="PostDeploymentGuidance" url="https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md"/>

## <a name="Summary"></a>Summary
<Guide type="Summary">
In today’s highly competitive and connected environment, modern businesses can no longer survive with generic, static online content. Furthermore, marketing strategies using traditional tools are often expensive, hard to implement, and do not produce the desired return on investment. These systems often fail to take full advantage of the data collected to create a more personalized experience for the user. 

Surfacing offers that are customized for the user has become essential to build customer loyalty and remain profitable. On a retail website, customers desire intelligent systems which provide offers and content based on their unique interests and preferences. 

Today’s digital marketing teams can build this intelligence using the data generated from all types of user interactions. By analyzing massive amounts of data, marketers have the unique opportunity to deliver highly relevant and personalized offers to each user. However, building a reliable and scalable big data infrastructure, and developing sophisticated machine learning models that personalize to each user is not trivial. 
</Guide>

## Prerequisites
<Guide type="Prerequisites">

This solution requires the creation of:

* 1 Data Lake Store
* 4 Stream Jobs with a total of 43 Streaming Units
* 1 Event Hub with 20 Throughput Units, 16 partitions and 4 Consumer Groups
* 1 DocumentDB database with 6 collections each provisioned with 10000 RUs, 10GB (3 are Partitioned). 
	
Ensure adequate Data Lake Stores and Stream Processing units are available before provisioning. Please consider deleting any unused Data Lake Store from your subscription. You may contact Azure Support if you need to increase the limit.

</Guide>

## <a name="Description"></a>Description

#### Estimated Provisioning Time: <Guide type="EstimatedTime">45 Minutes</Guide>
<Guide type="Description">
Cortana Intelligence provides advanced analytics tools through Microsoft Azure — data ingestion, data storage, data processing and advanced analytics components — all of the essential elements for building a personalized offer solution.

This solution combines several Azure services to provide powerful advantages. Event Hubs collects real-time consumption data. Stream Analytics aggregates the streaming data and makes it available for visualization, as well as updating the data used in making personalized offers to the customer. Azure DocumentDB stores the customer, product and offer information. Azure Storage is used to manage the queues that simulate user interaction. Azure Functions are used as a coordinator for the user simulation and as the central portion of the solution for generating personalized offers. Azure Machine Learning implements and executes the user to product affinity scoring and when no user history is available Azure Redis Cache is used to provide pre-computed product affinities for the customer. PowerBI visualizes the real-time activity for the system and with the data from DocumentDB the behavior of the various offers.

The 'Deploy' button will launch a workflow that will deploy an instance of the solution within a Resource Group in the Azure subscription you specify. The solution includes multiple Azure services (described above) and provides at the end a few short instructions necessary to have a working end-to-end solution with simulated user behavior. 

## Solution Diagram
![Solution Diagram](https://cloud.githubusercontent.com/assets/16085124/24881519/084cd072-1e0c-11e7-9093-7eaf48d4d513.png)

## Technical details and workflow
1.	User activity on the website is simulated with an **Azure Function** and a pair of **Azure Storage Queues**.

2. Personalized Offer Functionality is implemented as an **Azure Function**. This is the key function that ties everything together to produce an offer and record activity. Data is read in from **Azure Redis Cache** and **Azure DocumentDb**, product affinity scores are computed from **Azure Machine Learning** (if no history for the user exists then pre-computed affinities are read in from **Azure Redis Cache**). 

3. Raw user activity data (Product and Offer Clicks), Offers made to users, and performance data (for **Azure Functions** and **Azure Machine Learning**) are sent to **Azure Event Hub**.

4. The offer is returned to the User. In our simulation this is done by writing to an **Azure Storage Queue** and picked up by an **Azure Function** in order to produce the next user action.

5.	**Azure Stream Analytics** analyzes the data to provide near real-time analytics on the input stream from the **Azure Event Hub**. The aggregated data is sent to **Azure DocumentDB**.  The raw data is sent to **Azure Data Lake Storage**. 
</Guide>

#### Disclaimer

©2017 Microsoft Corporation. All rights reserved.  This information is provided "as-is" and may change without notice. Microsoft makes no warranties, express or implied, with respect to the information provided here.  Third party data was used to generate the solution.  You are responsible for respecting the rights of others, including procuring and complying with relevant licenses in order to create similar datasets.
