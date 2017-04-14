#Retraining the Personalized Offer Model
In some cases, the user of Personalized Offers might want to demonstrate other types of realistic behavior in the [Power BI dashboard](https://powerbi.microsoft.com/en-us/). This section outlines how Data Generator rules can be enhanced to alter trained behavior of the machine learning algorithm which result in different simulated user behavior in the Power BI dashboard.

The process is divided into the following logical steps:
1. first we setup the retraining pipeline to collect training data
2. we alter the Data Generator with new rules which alter simulated user behaviour
3. we run the whole pipeline again and record this new user behaviour as new training data
4. we re-train the Azure Machine Learning Studio experiment with this new training data
5. we re-deploy the experiment and link it to the existing Personalized Offers deployment which is also now running the same altered user behaviour
6. the altered user behaviour is now visible in Power BI dasboard

### Prerequisites
 - Log into the [Azure Management Portal](https://ms.portal.azure.com) 
 - In the left hand menu select Resource Groups
 - Locate the resource group you created for this project and click on it displaying the resources associated with the group in the resource group blade
 - Make sure you see the Overview panel on the left hand side selected - you should see the deployed resources in your resource group.
 - Click the App Service resource which starts with "functions-" name to select the deployed Azure Functions code editor
 - Select the PersonalizedOfferFunction on the left
 - Select Manage on the left and click Disabled in Function State: you have now successfully paused the Personalized Offers user simulation and can make changes to its pipeline

### EventHub
 - Go back to your Resource Group view in the previous step
 - Select existing EventHub in your Resource Group
 - Select EventHubs on the left
 - Click the Plus sign Event Hub button to add a new Event Hub
 - Enter name personaloffereh2, increase partition count to "16", and click Create

### Azure Stream Analytics Job
- Go back to Resource Group view, select Add, search for Steam Analytics Job, select it and click Create.
- Input "trainingdata" as the Job Name and select the Subscription, Resource group and Location to match your existing Resource group and click Create
- Go back to Resource Group, select the deployed Stream Analytics Job
- Click Inputs under Job Topology, and then click Add. Fill in "trainingdata" as Input alias, select the appropriate Event Hub name (prefixed with your resource group name) and then the created "personaloffereh2" Event hub name. Click Source and then select "Blob storage", wait for the GUI to register the selection, then go back to Source and select "Event hub" - this should trigger a GUI update. Make sure that the settings which were entered earlier are still present in the GUI - they may have been overwritten. Now Event hub policy name field has been cleared and you can click Create
- Click Output on the left and create the Output named "trainingout", select "Blob storage" in Sink, associated storage account from your Resource group, then create new container named "trainingdata", input Path pattern "trainingdata/{date}/{time}" and click Create.
- Navigate to the Query tab on the left and then copy-paste the following query and click save, overwriting the existing template query:
```
SELECT
    *
INTO
    trainingout
FROM
    trainingdata
```
- go back to Overview and click Start at the top to initialize the ASA job

### Azure Function
 - Go back to the Resource Group overview
 - Click the App Service resource which starts with "functions-" name to select the deployed Azure Functions code editor
 - Select the PersonalizedOfferFunction 
 - Click View Files in the top right hand corner to navigate to the rest of the files. Modify AMLWrapper.csx to include trainingOutput as the last parameter in `AMLWrapper.GenerateMLInput` function with `ICollector<string> trainingOutput`
 - add `trainingOutput.Add(mlInputList[i-1,0]);` after `mlInputList[i-1,0] = JsonConvert.SerializeObject(mlInputData).Replace("\\", "");` in the for-loop just before the return statement and click Save.
 - Add `ICollector<string> trainingOutput` in run.csx main Run method, just before outputUserResponseQueue argument and pass trainingOutput in the function call to `AMLWrapper.GenerateMLInput` method call.
 - Click save and make sure the function compiles - you can view the log output at the bottom by clicking Logs button in the bottom panel.
 - Select function.json and add
    ```,
    {
      "type": "eventHub",
      "name": "trainingOutput",
      "path": "personaloffereh2",
      "connection": "eventHubConnectionString",
      "direction": "out"
    }
    ```
    just after the DocumentDB "dbClient" definition - this sets up the new EventHub connection - click Save. 

### Rule Alteration

 - The pipeline is now in place and you can start gathering training data for the new rules.
 - Click the User Simulation function on the left, then View Files on top right and modify the code inside UserSimulator.csx to add new user behavior to the data generator. The entry point for user behaviour generation is located inside `UserSimulator.UserClick` method.
 - Make sure that any code alteration works before proceeding further. 

### Post-processing steps

 - Navigate back to the PersonalizedOfferFunction Azure Function view, click Manage and enable the function again
 - You should see the function execute with no errors in the run.csx function log
 - Training data is available in the storage account in your resource group in the "trainingdata" container

### Possible Improvements

You may want to increase the rate at which you're cycling the user messages through the queues to increase the rate of data throughput to gather training data faster. For this, you'll need to:
 - Navigate back to the PersonalizedOfferFunction Azure Function view, click Manage and disable the function again
 - On the left, click "Function app settings", then select "Go to App Service Editor" and replace the contents of host.json in the left panel with
```
{
"queues": {
      "maxPollingInterval": 2000,
      "batchSize": 32,
      "maxDequeueCount": 5,
      "newBatchThreshold": 16
    }
}
```
 - Go back to Resource group Overview, select the "App Service plan"
 - Click Scale Up on the left, choose S3 instance and click Select
 - Click Scale Out on the left, and move the slider to 10 Instances and select Save in the top left corner
 - Navigate back to the PersonalizedOfferFunction Azure Function view, click Manage and enable the function again

### Experiment Retraining

		
The training data is generated into the "trainingdata" container in the storage account as described above.
- Grab the [pre-published experiment](https://gallery.cortanaintelligence.com/Experiment/Personalized-Offers-Solution-How-To-Guide-2) for Personalized Offers
- Click Open in Studio and select the Azure ML Studio workspace which you want the experiment to be deployed to
- In the Training experiment panel, go to Import Data module in the experiment and point it to the storage account and the container and full data path location which we've setup above. Location and paths support wildcards. 
- You will also have to re-poppulate the storage account key for the import module and the storage account name
- Click Run to re-train the experiment with this new data
- Select Set Up Web Service below and choose Update Predictive Experiment
- Press Run again to re-run the predictive experiment
- Next choose Deploy Web Service and then Deploy Web Service [Classic]
- Copy the API key
- Select Request Response API link and copy the POST Method Request URI
- Navigate back to the Resource Group and then to Azure Functions view
- Select "Function app settings" at the bottom left corner, then "Configure app settings" and scroll all the way down
- Replace the API key of the published Azure Machine Learning experiment into `mlPublishedExperimentKey`
- Replace the POST Method Request URI into `mlPublishedExperimentEndpoint`
- Navigate back to PersonalizedOfferFunction and start the function through Manage tab on the left.

You have now successfully updated your experiment with the new training data and machine learning model trained on this new dataset.

## User Simulation

The `UserSimulation` Azure Function implements an individual user, who is clicking either an offer with 1% likelihood or a particular product - this is decided randomly each time a user views a page.

For product transitions, we implemented multiple random walks, which are switched by taking into consideration static user profile information:
1. Random Behaviour 1: represent a shopper who are interested in products typically bought by the male population.
2. Random Behaviour 2: same as above, but represents a shopper primarily interested in products used by females.
3. Affinity Behaviour: heavy pull towards product category which is given by user's affinity vector (decided by user profile).

Please note that we don't correlate gender with (1) and (2) above, so for instance we could have a male interested in female beauty products (maybe they're buying a gift).

Random walks 1 and 2 can be viewed as noise - they don't correlate to any user profile information and are only driven by recent user behaviour of the user - which page the user is viewing. This gives the machine learning model something to learn (current page has high predictive power), yet at the same time doesn't make it seem like the user is only driven by a static random walk - the behavior is more realistic.

We randomly switch to state (3) with 80% likelihood. If we are not in state (3), states (1) or (2) are then picked with 50% likelihood each, i.e. likelihoods for behaviour (1) through (3) are 0.2*0.5, 0.2*05 and 0.8 respectively.

## Machine Learning examples

This section explaing how AMLS generates product suggestions for the Offer Logic mechanism. Consider the profile of Amy Curtis, one of our generated users, coupled with a Desktop Computer product:

```
{
  "userId": "86",
  "eventType": "TrainingData",
  "name": "Amy Curtis",
  "email": "Amy.Curtis@contoso.com",
  "gender": "female",
  "latitude": 41.764586479302,
  "longitude": -71.608070226874,
  "distance": 49.866696097547,
  "creditCardType": "None",
  "autoRenew": 0,
  "categories": {
    "Apparel": 0,
    "Photography": 0.6,
    "Computers": 0.96110683809442,
    "Beauty": 0,
    "Fitness": 0
  },
  "currentPage": "2",
  "productId": "8",
  "productName": "Desktop Computer",
  "totalClicks": 1159,
  "cost": 650,
  "weight": 10000,
  "productCategories": {
    "Apparel": 0,
    "Photography": 0,
    "Computers": 1,
    "Beauty": 0,
    "Fitness": 0
  },
  "day": 35,
  "hour": 7,
  "minute": 1,
  "totalProductViews": 205,
  "action": "p",
  "click": 0,
  "eventTime": "2017-03-06T23:00:01.4377354",
  "EventProcessedUtcTime": "2017-03-06T23:00:06.0249033Z",
  "PartitionId": 0,
  "EventEnqueuedUtcTime": "2017-03-06T23:00:04.2240000Z"
}
```

We can see that Amy has a strong affinity for Computer products of about 0.96 and weaker affinity to Photography of 0.6, as specified by the "categories" attribute. We also see other user information, such as her address (given as map coordinates), distance to the nearest store, etc. For the product, we see a similar set of product features, product affinity vector, number of times Amy viewed this product in the past day, hour and minut,the total number of times the product was viewed by all users and the current page which Amy is browsing. Hence, we have the following set of features which we use for AMLS suggestions:
* user features
* user product category affinities (user tags)
* product features
* product category features (product tags)
* recency features: current page, number of product views in a certain time frame for user and across other users

All this information about Amy and Desktop Computer is fed into AMLS to arrive at a suggested probability of 0.5217. Similarly, we take another product "Zoom Lens DSLR 18-55mm", join it with Amy's profile and feed it through AMLS to obtain a likelihood of 0.0430:

```
{
  "userId": "86",
  "eventType": "TrainingData",
  "name": "Amy Curtis",
  "email": "Amy.Curtis@contoso.com",
  "gender": "female",
  "latitude": 41.764586479302,
  "longitude": -71.608070226874,
  "distance": 49.866696097547,
  "creditCardType": "None",
  "autoRenew": 0,
  "categories": {
    "Apparel": 0,
    "Photography": 0.6,
    "Computers": 0.96110683809442,
    "Beauty": 0,
    "Fitness": 0
  },
  "currentPage": "2",
  "productId": "3",
  "productName": "Zoom Lens DSLR 18-55mm",
  "totalClicks": 1865,
  "cost": 350,
  "weight": 150,
  "productCategories": {
    "Apparel": 0,
    "Photography": 1,
    "Computers": 0,
    "Beauty": 0,
    "Fitness": 0
  },
  "day": 0,
  "hour": 0,
  "minute": 0,
  "totalProductViews": 205,
  "action": "p",
  "click": 0,
  "eventTime": "2017-03-06T23:00:01.4377354",
  "EventProcessedUtcTime": "2017-03-06T23:00:06.0249033Z",
  "PartitionId": 0,
  "EventEnqueuedUtcTime": "2017-03-06T23:00:04.2240000Z"
}
```

We feed all products in the inventory, coupled with Amy's information through AMLS, obtaining a likelihood score for each one. The magnitude of the likehood isn't important - it's the relative magnitude which allows us to order products by interest. In this case, we end up with top 5 product suggestions for Amy:

| Top 5 Product Names | Likelihood |
|:------------:|:----------:|
| Desktop Computer | 0.5217 |
| Zoom Lens DSLR 18-55mm | 0.0430 |
| DSLR Lens Filter 77mm | 0.0070 |
| Compact Camera | 0.0052 |
| Foundation Brush | 0.0046 |
