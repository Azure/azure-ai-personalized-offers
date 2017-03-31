# Personalized Offer Logic

This purpose of this document is to outline the Offer Logic component of the Personalized Offers solution. 

Offer Logic is the main differentiating factor between a classic recommendation approach of recommending ads (offers) to users and recommending offers based on the inventory of the retailer. The former case is commonly termed as Real Time Bidding. The latter case requires an additional logical layer, where given the products which the user desires most, the system suggests an offer to the user based on retail manager's settings. 

In this implementation, Offer Logic takes as input the list of relevant products along with their popularity probability, as suggested by a Machine Learning model (trained on personalized shopping and browsing behaviors of many of retailer's customers) and outputs the most relevant offer based on user's affinity to the products advertised in the offer and campaign manager's settings.

Please note that in future revisions of this project, Offer Logic can also be replaced by a machine learning model. In this case, data generator would become a nested Monte Carlo - first layer is used to generate training data for the product recommender, and second layer trains the offer recommender, taking product recommender's predictions as input.

## Offer Specifics

Offers themselves are conceptually split into two categories:
1. Offers which have product information (**specific offers**), for example, “10% off on fitness shoes” targets all fitness shoe products, and the user has to have a high affinity to fitness shoes in order to qualify for this offer.
2. Offers which are not targeting specific offers (**general offers**): for example, “10% off store-wide Black Friday sale”. These offers can also have other attributes - in the case of Black Friday, an offer would have a calendar attribute. We could also have other attributes which are computed dynamically, such as "10% off to top 20 shoppers" (user IDs for top 20 shoppers, based on the number of purchases, would have to be compute dynamically).

## Offer Logic Pseudocode

Offer Logic has the following pseudocode (hyperparameters are shown in capital letters - they can be adjusted by the campaign manager):
1. First we recommend the most likely products for a user, and take the top N such products along with predicted probabilities. For users with shopping history, this is done by Azure Machine Learning Studio (AMLS) recommender implementation; for users with no history (cold users), a static Redis cache is used. An FBT recommender from recommendation API can be used to pre-populate the cache.
2. For every **specific** offer, for every product which the offer links to, we compute the cosine similarity between the user Category Vector (tag vector) and product Category Vector, weighted by the product popularity probability returned from AMLS, if the product which the offer links to is among the top N recommended products; zero is scored if the intersection of the set of offer products and top N products is empty.
3. We rank the offers by the weighted cosine similarity and if an offer is above the certain threshold E, show the offer with highest weighted cosine similarity is shown.
4. Else, we continue and start considering offers in the order specified by offerPriority table (please see the next section). Last offer in offerPriority table is always shown if all other conditions for the previous offers are not met.

## Data Structures

Data is stored in Document DB Collections.

General offers come with “condition” label, which is inputted at the time of creation. Current conditions are: date, topUsers, distance & specificProducts. 

Each offer triggers a mathematical function (stored in “functionName” attribute), which if evaluated to true will show the offer to the user. Each function may use the threshold parameter E if needed. Each function is implemented separately by the engineering team. If the function is not found during code execution, next priority offer function is executed.

### Reference Collection

#### offerThreshold

Contains static information from the campaign manager, such as user affinity threshold, top N number of users, etc

#### offerPriority

Contains an ordered list of offers to consider, in case user affinity for the offers offers is below threshold E.

#### topUsers

Stores top website users (dynamically updated).

### Offer Collection

Contains offer information. Each offer has a list of products which it references, as well as function reference and function parameters which execute offer logic in the code.

