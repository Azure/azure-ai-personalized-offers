# Personalized Offers - An Azure AI Solution How-to Guide

In today’s highly competitive and connected environment, modern businesses can no longer survive with generic, static online content. Furthermore, marketing strategies using traditional tools are often expensive, hard to implement, and do not produce the desired return on investment. These systems often fail to take full advantage of the data collected to create a more personalized experience for the user. 
Surfacing offers that are customized for the user has become essential to build customer loyalty and remain profitable. On a retail website, customers desire intelligent systems which provide offers and content based on their unique interests and preferences. Today’s digital marketing teams can build this intelligence using the data generated from all types of user interactions. By analyzing massive amounts of data, marketers have the unique opportunity to deliver highly relevant and personalized offers to each user. However, building a reliable and scalable big data infrastructure, and developing sophisticated machine learning models that personalize to each user is not trivial. 

## Solution Dashboard
The snapshot below shows an example PowerBI dashboard that gives insights into the offers being shows to customers and predicted customer affinity to those offers.
[Insights](placeholder for dashboard image)

## Solution Architecture
![Solution Diagram Picture](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/blob/master/Automated%20Deployment%20Guide/Figures/PersonalizedOffersArchitecture.png)

## What's Under the Hood
Azure AI, formerly known as Cortana Intelligence, provides advanced analytics tools through Microsoft Azure — data ingestion, data storage, data processing and advanced analytics components — all of the essential elements for analyzing the customer demand and making personalized offers. 
This solution combines several Azure services to provide powerful advantages. Event Hubs collects real-time consumption data. Stream Analytics aggregates the streaming data and updates the data used in making personalized offers to the customer. Azure DocumentDB stores the customer, product and offer information. Azure Storage is used to manage the queues that simulate user interaction. Azure Functions are used as a coordinator for the user simulation and as the central portion of the solution for generating personalized offers. Azure Machine Learning implements and executes the product recommendations and when no user history is available Azure Redis Cache is used to provide pre-computed product recommendations for the customer. PowerBI visualizes the activity of the system with the data from DocumentDB.

## Getting Started

This solution package contains materials to help both technical and business audiences understand our Personalized Offers solution for Retail built on [Azure Platform](https://partner.microsoft.com/en-us/solutions/microsoft-azure-platform).

## Business Audiences

In this repository you will find a folder labeled [*Solution Overview for Business Audiences*](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/tree/master/Solution%20Overview%20for%20Business%20Audiences) which contains a  presentation covering this solution and benefits of using Azure AI.

For more information on how to tailor Azure AI to your needs [connect with one of our partners](https://azure.microsoft.com/en-us/partners/).

## Technical Audiences

See the [*Manual Deployment Guide*](https://github.com/Azure/cortana-intelligence-personalized-offers-retail-2/tree/master/Manual%20Deployment%20Guide) folder for a full set of instructions on how to put together and deploy a Personalized Offers solution using Azure AI. For technical problems or questions about deploying this solution, please post in the issues tab of the repository.

## Related Resources
A playbook for approaching personalization problems, which can be considered to include use cases such as that discussed within this solution, is published [here](https://github.com/Azure/cortana-intellligence-personalization-data-science-playbook).


# Contributing

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
