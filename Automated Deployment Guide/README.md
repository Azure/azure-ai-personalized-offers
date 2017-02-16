
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

#### Estimated Provisioning Time: <Guide type="EstimatedTime">X Minutes</Guide>
<Guide type="Description">

**REPLACE THIS SECTION**
The Cortana Intelligence Suite provides advanced analytics tools through Microsoft Azure — data ingestion, data storage, data processing and advanced analytics components — all of the essential elements for building an demand forecasting for energy solution.

This solution combines several Azure services to provide powerful advantages. Event Hubs collects real-time consumption data. Stream Analytics aggregates the streaming data and makes it available for visualization. Azure SQL stores and transforms the consumption data. Machine Learning implements and executes the forecasting model. PowerBI visualizes the real-time energy consumption as well as the forecast results. Finally, Data Factory orchestrates and schedules the entire data flow.

The 'Deploy' button will launch a workflow that will deploy an instance of the solution within a Resource Group in the Azure subscription you specify. The solution includes multiple Azure services (described below) along with a web job that simulates data so that immediately after deployment you have a working end-to-end solution. The sample data of this solution is simulated from publicly available data from the NYISO.

**Solution diagram coming**
## Solution Diagram
![Solution Diagram]()

## Technical details and workflow
1.	The sample data is streamed by newly deployed **Azure Functions**.

2.	This synthetic data feeds into the **Azure Event Hubs** and **Azure SQL** service as data points or events, that will be used in the rest of the solution flow.

3.	**Azure Stream Analytics** analyze the data to provide near real-time analytics on the input stream from the event hub and directly publish to PowerBI for visualization.

4.	The **Azure Machine Learning** service is used to make forecast on the energy demand of particular region given the inputs received.

5.	**Azure SQL Database** is used to store the prediction results received from the **Azure Machine Learning** service. These results are then consumed in the **Power BI** dashboard.

6. **Azure Data Factory** handles orchestration, and scheduling of the hourly model retraining.

7.	Finally, **Power BI** is used for results visualization, so that users can monitor the energy consumption from a region in real time and use the forecast demand to optimize the power generation or distribution process.
</Guide>

