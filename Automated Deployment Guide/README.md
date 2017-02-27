
# [Personalized Offers for Retail Solution](placeholder for gallery url)

This folder contains the post-deployment instructions for the deployable Personalized Offers for Retail solution in the Cortana Intelligence Gallery. To start a new solution deployment, visit the gallery page [here](placeholder for gallery url).

<Guide type="PostDeploymentGuidance" url="https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Post%20Deployment%20Instructions.md"/>

## <a name="Summary"></a>Summary
<Guide type="Summary">
In today’s highly competitive and connected environment, modern businesses can no longer survive with generic, static online content. Furthermore, marketing strategies using traditional tools are often expensive, hard to implement, and do not produce the desired return on investment. These systems often fail to take full advantage of the data collected to create a more personalized experience for the user. 

Surfacing offers that are customized for the user has become essential to build customer loyalty and remain profitable. On a retail website, customers desire intelligent systems which provide offers and content based on their unique interests and preferences. 
Today’s digital marketing teams can build this intelligence using the data generated from all types of user interactions. By analyzing massive amounts of data, marketers have the unique opportunity to deliver highly relevant and personalized offers to each user. However, building a reliable and scalable big data infrastructure, and developing sophisticated machine learning models that personalize to each user is not trivial. 
</Guide>

## <a name="Description"></a>Description

#### Estimated Provisioning Time: <Guide type="EstimatedTime">45 Minutes</Guide>
<Guide type="Description">

**REPLACE THIS SECTION**
The Cortana Intelligence Suite provides advanced analytics tools through Microsoft Azure — data ingestion, data storage, data processing and advanced analytics components — all of the essential elements for building an demand forecasting for energy solution.

This solution combines several Azure services to provide powerful advantages. Event Hubs collects real-time consumption data. Stream Analytics aggregates the streaming data and makes it available for visualization, as well as updating the data used in making personalized offers to the customer. Azure DocumentDB stores the customer, product and offer information. Azure Storage is used to manage the queues that simulate user interaction and for the archival storage of training data if the extension to this solution is built. Azure Functions are used as a coordinator for the user simulation and as the central portion of the solution for generating personalized offers. Machine Learning implements and executes the product recommendations and when no user history is available Azure Redis Cache is used to provide pre-computed product recommendations for the customer. PowerBI visualizes the real-time activity for the system and with the data from DocumentDB the performance of the various offers.

The 'Deploy' button will launch a workflow that will deploy an instance of the solution within a Resource Group in the Azure subscription you specify. The solution includes multiple Azure services (described above) and provides at the end a few short instructions necessary to have a working end-to-end solution with simulated user behavior. 

**Solution diagram coming**
## Solution Diagram
![Solution Diagram](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Figures/PersonalizedOffersArchitecture.png)

## Technical details and workflow
1.	An **Azure Function** begins the flow of the system by taking users from **Azure DocumentDB** and placing them in an **Azure Storage Queue**.

2. An **Azure Function** reads the initial user list and starts generating user activity that is placed in another **Azure Storage Queue**.

3. This simulated user activity is read by the central **Azure Function** and the process for producing a personalized offer begins here.

4. The user activity is used to bring in data from **Azure DocumentDB** providing some history for the user, either **Azure Machine Learning** or **Azure Redis Cache** are used to provide product recommendations and finally the offer is produced with logic in the function and information provided by **Azure DocumentDB** and **Azure Redis Cache**.

5. The user response is put back on an **Azure Storage Queue** and the user activity, performance data and, potentially, training data are sent to an **Azure Event Hub** for further processing.

6.	**Azure Stream Analytics** analyze the data to provide near real-time analytics on the input stream from the **Azure Event Hub**. The aggregated data is sent to **Azure DocumentDB* and directly published to **PowerBI** for visualization.  The raw data is sent to **Azure Data Lake Storage**. If the training extension is completed, training data is written to **Azure Blog Storage**.
</Guide>

#### Disclaimer

©2017 Microsoft Corporation. All rights reserved.  This information is provided "as-is" and may change without notice. Microsoft makes no warranties, express or implied, with respect to the information provided here.  Third party data was used to generate the solution.  You are responsible for respecting the rights of others, including procuring and complying with relevant licenses in order to create similar datasets.