# Consumer behavior prediction, based on Taobao data
## Team Member
Name|Student ID|GitHub
---|--|---
Zhang Ping|1901212672|[Parametric3](https://github.com/Parametric3)
Xu Chenqi|1901212653|[XuChenqi](https://github.com/XuChenqi)
Lin Haoru|1901212609|[HalinaLin](https://github.com/HalinaLin)
Luo Chaojing|1901212618|[crystallake27](https://github.com/crystallake27)
## Background
E-commerce has gradually become an essential part of our daily life. In 2019, China's online consumption accounted for more than 25% of total consumption. Under this situation, taking advantage of the big data generated by online transactions, we can analyze and reasonably predict consumer behavior.</p>
Our team obtained Taobao's consumer behavior data from Tianchi, AliCloud, starting from **November 18, 2014** to **December 18, 2014.** Based on that, we derived a series of other features which help interprete the consumer behaviors, and performed corresponding data preprocessing. After that, we comprehensively conducted LR, SVM, Decision Tree, Random Forest. 
## Data
Our data is from Tianchi, AliCloud, starting from November 18, 2014 to December 18, 2014. The original data includ two files. One is about all the users' bahaviors on all the items in that period, inclouding **6 basic indicators**, user id, item id, behavior type (browse, collect, purchase, etc.), user geohash (location), item category and time.</p>
Name|Meaning|Explain
:---:|:---:|:---:
user_id|the unique user identidication|
item_id|the unique item identidication|
behavior_type|the classification of user behavior|browing, collecting, adding into cart and purcahsing, corresponding to 1, 2, 3 and 4 respectively
user_geohash|the location of the user|can be null
item_category|the category of one certain item|
time|time when behaving| precised to hour

For example, **141278390,282725298,1,95jnuqm,5027,2014-11-18 08**.</p>
The other file is about all the items, inclouding **3 basic indicators**, item id, item geohash (location), item category.</p>
Name|Meaning|Explain
:---:|:---:|:---:
item_id|the unique item identidication|
item_geohash|the location of the item|can be null
item_category|the category of one certain item|

For example, **117151719,96ulbnj,7350**.</p>
For detailed data, see our [data](https://disk.pku.edu.cn:443/link/2B3214E55199700FDB7D21C86F93A9E7) in PKU Cloud.
## Overall Analysis
### Problem Type: Classification
**'Prucahse'** correspodning to **'1'**, and **'Not Purchase'** corresponding to **'0'**</p>
Particularity of the problem:
1. The label is not just for one user or one item, but for one user-item pair **(user, item)**, which influence the selection of training set and test set
2. This is multi-period problem. For example, for one user-item pair, its label is derived from the bahevior of 18th, while its features are derived from days before 18th. That is because we can only the past to predict the future.
### Days that really matter
Since we will use the information before 18th to predict the user behavior in 18th, the next question is how many days should be considered. Intuitively, the behavior one month ago definetely has nothing to do with whether the user will buy or not. So this actually a hyperparameter that we should decide first, and then find its best value through fine-tuning. Using $\Delta$ to denote it, and we let $\Delta$ = 2 at first. Later, we will also implement the same preprocessing and model on the datasets with $\Delta$ = 3 and 4 respectively.</p>
### Test Set
Three ways to choose test set:</p>
1. ~~All users and all items:~~
![](https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/Test_Set_Selection_1.png)
2. ~~Users who have certain behaviors and the items that users has interactions with in the two days before :~~
![](https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/Test_Set_Selection_2.png)
3. For each user who was active before, only condider the items that he has interactions with:
<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/Test_Set_Selection_3.png" height="330" width="450"/>
</div>

### Train Set
The basic rules all the same for train set. Two things to consider:</p>
1. As customary, we set the ratio between the training set and the test set to be 4:1. So since we have choose **(16, 17 → 18)** as the test set (before '→', the features of test set; after '→', the labels of the test set), we need to choose another four groups.
2. If we count back from 18th, the abnormal comsumption in 12th, December will distrurb our model greatly, so we need to skip that when choose the train set.

The overall process is shown below, and related data that has been processed is in our [Data](https://github.com/Parametric3/PHBS_MLF_2019/tree/master/Data) folder.
![OveralL Process](https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/OveralL_Process.png)
## Feature Generating
All the features can be dicided into two levels:
1. Basic features, which usually involve one indicator in the original file;
2. Interactive features, which involve two or more original indicators.

And for each feature class, like user features, the features it contains can be divided into three types:
1. Satistic features: Obtained by directly counting the number of certain event.
2. Ratio features: Ratio between two satistic features.
3. Time features: Features that involve time.
### Basic Features
1. **User features** (Feature type = 1):

Feature name| type | Explaination
---|---|---
1_user_activity|statistic|number of user actions in the two days
1_number_of_items_related|statistic|number of items that the user had interactions with in the two days
1_number_of_browsing_actions|statistic|number of browsing actions in the two days
1_number_of_collecting_actions|statistic|number of collecting actions in the two days
1_number_of_carting_actions|statistic|number of adding into the cart actions in the two days
1_number_of_buying_actions|statistic|number of purchasing actions in the two days
1_behavior_pattern|statistic|1 as dirctly buying, 0 as collecting or adding into cart before buying and -1 as not buying
1_ratio_of_browsing_actions|ratio|ratio between number of browsing actions and number of all actions
1_ratio_of_collecting_actions|ratio|ratio between number of collecting actions and number of all actions
1_ratio_of_carting_actions|ratio|ratio between number of adding into cart actions and number of all actions
1_ratio_of_buying_actions|ratio|ratio between number of purchasing actions and number of all actions
1_conveting_rate|ratio|ratio between number of items finally purchased and number of all items related
1_first_time_online|time|the time lag between first online and the 0 o'clock on the prediction day
1_last_time_online|time|the time lag between last online and the 0 o'clock on the prediction day
1_time_lag|time|the time lag between first online and last time online
1_the_behavior_frequency|time|time lag over the total number of activities

2. **Item features** (Feature type = 2):

Feature name| type | Explaination
---|---|---
2_item_buy|statistic| number of times the product was purchased in the two days
2_item_view|statistic| number of times the product was viewed in the two days
2_item_collect|statistic| number of times the product was collected in the two days
2_item_add|statistic| number of times the product was carted in the two days
2_item_buypeople|statistic| number of users who purchased the product in the two days(the number of people who have been deduplicated)
2_item_viewpeople|statistic| number of users who viewed the product in the two days(the number of people who have been deduplicated)
2_item_collectpeople|statistic| number of users who collected the product in the two days(the number of people who have been deduplicated)
2_item_addpeople|statistic| number of users who carted the product in the two days(the number of people who have been deduplicated)
2_item_frequentbuypeople|statistic|number of users who make multiple purchases in the two days
2_item_frequentviewpeople|statistic|number of users who viewed the product multiple times in the two days
2_item_frequentcollectpeople|statistic|number of users who collected the product multiple times in the two days
2_item_frequentaddpeople|statistic|number of users who carted the product multiple times in the two days
2_item_buy_view|ratio|ratio of number of times the product was purchased in the two days to number of times the product was viewed in the two days
2_item_buy_collect|ratio|ratio of number of times the product was purchased in the two days to number of times the product was collected in the two days
2_item_buy_add|ratio|ratio of number of times the product was purchased in the two days to number of times the product was carted in the two days
2_item_buypeople_viewpeople|ratio|ratio of number of users who purchased the product in the two days to number of users who viewed the product in the two days (the number of people who have been deduplicated)
2_item_buypeople_collectpeople|ratio|ratio of number of users who purchased the product in the two days to number of users who collected the product in the two days (the number of people who have been deduplicated)
2_item_buypeople_addpeople|ratio|ratio of number of users who purchased the product in the two days to number of users who carted the product in the two days (the number of people who have been deduplicated)
2_item_frequentbuypeople_buypeople|ratio|ratio of number of users who make multiple purchases in the two days to number of users who purchased the product in the two days (the number of people who have been deduplicated)
2_item_frequentviewpeople_viewpeople|ratio|ratio of number of users who viewed the product multiple times in the two days to number of users who viewed the product in the two days (the number of people who have been deduplicated)
2_item_frequentcollectpeople_collectpeople|ratio|ratio of number of users who collected the product multiple times in the two days to number of users who collected the product in the two days (the number of people who have been deduplicated)
2_item_frequentaddpeople_addpeople|ratio|ratio of number of users who carted the product multiple times in the two days to number of users who carted the product in the two days (the number of people who have been deduplicated)

3. **Category features** (Feature type = 3):

Feature name| type | Explaination
---|---|---
3_number_of_categories_related|statistic|number of categories that user had interaction with
3_category_concentration_rate|ratio|number of items related over number of categories related

4. **Geo features** (Feature type = 4):

Feature name| type | Explaination
---|---|---
4_geo_purchasepower|ratio| ratio of total number of products purchased in the area to total number of users in the area
4_geo_buyview|ratio|ratio of total number of products purchased in the area to total number of products viewed in the area
4_geo_buycollect|ratio|ratio of total number of products purchased in the area to total number of products collected in the area
4_geo_buyadd|ratio|ratio of total number of products purchased in the area to total number of products carted in the area

5. **UC(User and Category) features** (Feature type = 5):

Feature name| type | Explaination
---|---|---
5_Number_of_items|statistic| Numbers of items the user had interactions with in that category
5_Repurchasing_pattern|statistic| 1 if the user had repurchasing behavior
5_Number_of_purchasing|statistic| Number of purchasing behaviors in that category
5_Category_prefernce|ratio|Numbers of items the user had interactions with in that category divided by number of all items the user had interactions with
5_Category_purchase_power|ratio|Number of purchasing behaviors in that category divided by numbers of items the user had interactions with in that category
5_Overnight_purchase_pattern|ratio|whether the user purchase the item one day after browsing, collecting or adding into cart

Note: some ratio-based indicators in this article have missing indicator data because the denominator is 0. For such indicators, the missing value is filled with 0.

### Interactive Features


## Data preprocessing
### Dealing with missing data
We eliminate samples with missing values.
### Standardization
In order to eliminate the model result error caused by the size of the data itself, we standardize the data.
### Imbalanced Sample: Up&downsampling</p>
Through statistics, we have a total of 279,525 samples, while the number of samples with the "label=1"(**'Purchase'**) is only 1,529. The ratio of samples with "label=1" and 'label=0' is around 1:190. In order to eliminate the impact of data imbance on the model results, we upscaled the data with "label=1" and also downscaled the data with "label=0" in the training set. In the end, the ratio of samples with "label=1" and "label=0" is around 1:10.</p>

## Model Building
### 1. Lasso Logistic Regression</p>
Use L1 regularization to achieve variable selection</p>
When C=0.01, there're 25 features selected, including 'number of buying actions', 'ratio of purchases to collects', which quite make sense. We get the training accuracy: 0.914 and test accuracy: 0.481. There's obvious overfitting problem and the accuracy is below 0.5.Therefore, we don't adopt this model.</p>

### 2. PCA + Logistic Regression
Choose 2 principle components can explain more than 90% of the variance in the model, then do Logistic Regression. We get the training accuracy: 0.901 and test accuracy: 0.995. We wonder why 1&2 both use logistic regression but get totally different results. Also, PCA reflects that the variables have multicollinearity problem.</p>
<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/PCA.png" height="450" width="650"/>
</div>

### 3. SVM</p>
We get the training accuracy: 0.945 and test accuracy: 0.995.

### 4. Random Forest--bagging</p>
We get the training accuracy: 0.966 and test accuracy: 0.972. Important features include 'item_buy_add' and 'item_viewpeople'.</p>
<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/FI.png" height="400" width="650"/>
</div>

Through 5-folds cross-validation, we get F1- score 80.53%, which is quite nice. As the ROC curve shows, AUC is 0.91.

<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/ROC for RF.jpg" height="450" width="650"/>
</div>

### 5. GBRT (Gradient Boost Regression Tree)--boosting</p>
We get the training accuracy: 0.925 and test accuracy: 0.989. AUC is 0.65.

<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/ROC for GBC.jpg" height="450" width="650"/>
</div>

### Conclusion
SVM and Random Forest are suitable method for our data.

## Future Work
1. Introduce interaction terms;</p>
2. Try combination of models;</p>
### Example (Source: Tianchi Bigdata)
### RF+GBRT</p>
**Combination1**</p>
a. Use RF to train, output is y_rf;</p>
b. Use GBRT to train under y-y_rf;</p>
c. Use the average of a&b when predict.</p>
<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/RF+GBRT1.png" height="300" width="850"/>
</div>

**Combination2**</p>
a. Use RF to get random features;</p>
b. Use GBRT for those features;</p>
c. Use the average of multiple GBRT outputs when predict.</p>
<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/RF+GBRT2.png" height="300" width="600"/>
</div>

### GBRT+LR</p>
Stucture/depth/learning rate/iteration are adjusted accordingly</p>
<div align="center">
<img src="https://raw.githubusercontent.com/Parametric3/PHBS_MLF_2019/master/Figs/LR+GBRT.png" height="330" width="750"/>
</div>

3. Optimize the parameters in the model using gradient search;</p>
