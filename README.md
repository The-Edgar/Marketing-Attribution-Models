# Marketing Attribution Models

## 1. About the Class
Python class developed to solve media attribution problems in Digital Marketing.

## 2. About Multichannel Attribution
In the digital context, before conversion, the user is impacted by several points of contact, which can generate increasingly long and complex journeys.

* How to allocate credit for conversions and optimize online media investment? *

To solve this problem, we use **Attribution Models **.


### 2.1 Types of Models

#### Heuristic Models

- **Last Interaction **:
    - Standard attribution model for both Google Analytics and media tools such as Google Ads and Facebook Business manager;
    - Assigns the entire conversion result to the last touchpoint.

- **Last non-Direct Click **:
    - All direct traffic is ignored, and 100% of the sale credit goes to the last channel through which the customer reached the site before completing the conversion.

- **First Interaction **:
    - Assigns the entire conversion result to the first point of contact.

- **Linear **:
    - Each touch point in the conversion path.

- **Time Decay **:
    - The closest points of contact in terms of time of sale or conversion receive most of the credit.

- **Position Based **:
    - In the attribution model Based on the position, 40% of the credit is attributed to each first and last interaction, and the remaining 20% ​​of credit is distributed evenly for the intermediary interactions.


#### Algorithmic Models

#### **Shapley Value **

Concept coming from Game Theory, to distribute the contribution of each player in a cooperation game.

Attributes credits for conversions by calculating the contribution of each channel present in the journey, using permutations of journeys with and without the channel in question.


**For example **, how can we attribute the 19 conversions on the journey below?

Natural Search> Facebook> Direct> **$ 19 **


The Shapley Value of each channel is calculated based on observations, that is, for each journey, it is necessary to have the conversion values ​​for all the combinations that compose it.


Natural Search> **$ 7 **<br/>
Facebook> **$ 6 **<br/>
Direct> **$ 4 **<br/>
Natural Search> Facebook> **$ 15 **<br/>
Natural Search> Direct> **$ 7 **<br/>
Facebook> Direct> **$ 9 **<br/>
Natural Search> Facebook> Direct> **$ 19 **<br/>


The number of iterations increases exponentially with the number of channels: on the order of 2 ^ N, where N is the number of channels.

Thus, for a journey with 3 channels, 8 calculations are required. **For journeys with more than 15 channels, it is practically impossible. **


Shapley Value by default does not consider the order of the channels, but the contribution of his presence on the journey.
To take this into account, it is necessary to increase the order of the number of combinations.

From this comes the difficulty in using a method that considers the * order of the channels * for a large number N, because, in addition to the 2 ^ N interactions for the calculation of the Shapley Value of a given channel i, **we need the * observation * of the channel i in all possible positions. **


**Negative points about Shapley Value **
- Limits the number of points of contact since the number of iterations is in the order of 2 ^ N;
- If not ordered, Shapley Value considers that the contribution of channel A is the same whether preceded by B or C;
- If ordered, the number of combinations increases a lot and the days must be available, otherwise zero is assigned to that day;
- Channels that are few present or present on long journeys will have small contributions;


#### **Markov chains **
A Markov chain is a particular case of a stochastic process with the property that the probability distribution of the next state depends only on the current state and not on the sequence of events that preceded it.


Using markov chains in the context of multichannel attribution, we can calculate the probability of interactions between media channels using the **Transition Matrix **.


To find the contribution of each channel, we use **Removal Effect **: the channel in question is removed from the journey and the conversion probability is calculated.

The attribution is given by the ratio of the difference between the total conversion probability and the conversion probability without the channel, and the original total conversion probability.

The greater the removal effect, the greater the contribution of the channel to the conversion.


**Markovian processes do not have any type of restriction in relation to the number or order of channels and
considers the channel sequence as a fundamental part of the algorithm **.


### 2.2 References
- [Attribution Models in Marketing] (https://data-science-blog.com/blog/2019/04/18/attribution-models-in-marketing/)
- [Attribution Theory: The Two Best Models for Algorithmic Marketing Attribution - Implemented in Apache Spark and R] (http://datafeedtoolbox.com/attribution-theory-the-two-best-models-for-algorithmic-marketing-attribution- implemented-in-apache-spark-and-r /)
- [Game Theory Attribution: The Model You’ve Probably Never Heard Of] (https://clearcode.cc/blog/game-theory-attribution/)
- [Marketing Channel Attribution With Markov Models In R] (https://www.bounteous.com/insights/2016/06/30/marketing-channel-attribution-markov-models-r/?ns=l)
- [Multi-Channel Funnels Data-Driven Attribution] (https://support.google.com/analytics/topic/3180362?hl=en&ref_topic=3205717)
- [Multi-Channel Marketing Attribution model with R (part 1: Markov chains concept)] (https://analyzecore.com/2016/08/03/attribution-model-r-part-1/)
- [Multi-Channel Marketing Attribution model with R (part 2: practical issues)] (https://analyzecore.com/2017/05/31/marketing-multi-channel-attribution-model-r-part-2-practical -issues /)
- [ml-book / shapley] (https://christophm.github.io/interpretable-ml-book/shapley.html)
- [Overview of Attribution modeling in MCF] (https://support.google.com/analytics/answer/1662518?hl=en)

## 3. Importing the Class


`` python
>> pip install marketing_attribution_models
``


`` python
from marketing_attribution_models import MAM
``

## 4. Demo

### **Creating the MAM object **

**The creation of the MAM object **is based on **two Data Frame formats **and that is guided by the group_channels parameter:

* **group_channels = True **. It is expected to receive a basis on which **each line would be a session of the user's journey **.
  * This data frame must contain columns representing User ID, Boolean indication whether or not there was a transaction during the session, session timestamp and the channel on which the user generated the session;
* **group_channels = False **. It receives the basis on which the **journey has already been grouped **and that each line represents a complete journey for a given user until conversion. For Google Analytics users, this base can be generated by exporting the Top Conversion Paths report in the Conversions tab.
  * In this case, the channel column and time_till_conv_colname would receive in each line a journey separated by a separator, '>' as a default and which can be changed in the path_separator parameter.

In our case, we will present an example in which the journeys are not yet grouped, that each row represents a journey and that we do not yet have an ID for Each Journey.

**Attention point:**
The class already includes a function represented by the create_journey_id_based_on_conversion parameter, which if True, a Journey ID will be created based on the User ID columns, passed in the group_channels_id_list parameter and the column that represents whether or not there was a conversion, passed in the journey_with_conv_colname parameter.

In this case, the sessions of each user will be ordered and each transaction will create a new Journey ID. However, **we encourage you to create a Journey ID based on the business knowledge of each base explored **. Being able to create specific conditions of time so that there is a break of workday, as for example if it is identified that the average workday of a certain business lasts 1 week until the conversion, we can adopt a criterion that if a certain user does not interact with the site for a week, your journey will be broken, as there may be a break in interest.



Exemplifying how the parameters would be configured in the scenario described above with group_channels = True.

1. The **Pandas Data Frame **containing the database to be analyzed must be passed;
2. Indicate the base format in **group_channels **= True
3. Name of the column containing the channel groupings in **channels_colname **;
4. Boolean column indicating whether or not the session was converted to **journey_with_conv_colname **;
5. List containing the names of the columns that represent the Journey ID, which can be a combination of columns in **group_channels_by_id_list **. But in this case, as we are indicating that we will create a Journey ID in the parameter **create_journey_id_based_on_conversion = True **, just indicate the User ID column;
6. Column representing the date that the session in **group_timestamp_colname **occurred. Column that can receive in addition to the days of the year, the time the session took place;
7. Finally, in our case, we indicate that we will generate a Journey ID from the columns indicated in the group_channels_by_id_list and journey_with_conv_colname parameters, in **create_journey_id_based_on_conversion **= True



`` python
attributions = MAM (df,
    group_channels = True,
    channels_colname = 'channels',
    journey_with_conv_colname = 'has_transaction',
    group_channels_by_id_list = ['user_id'],
    group_timestamp_colname = 'visitStartTime',
    create_journey_id_based_on_conversion = True)
``

For exploratory and learning purposes, we have implemented a way to generate a **random database **using the **random_df = True **parameter. It is not necessary to fill in the others.


`` python
attributions = MAM (random_df = True)
``

As soon as the object was created, we can check how the **base looked after the creation of journey_id and the grouping of sessions **in joranadas through the **attribute .DataFrame. **


`` python
attributions.DataFrame
``
| | journey_id | channels_agg | time_till_conv_agg | converted_agg | conversion_value |
| - | - | - | - | - | - |
| 0 | id: 0_J: 0 | Facebook | 0.0 | True | 1 |
| 1 | id: 0_J: 1 | Google Search | 0.0 | True | 1 |
| 2 | id: 0_J: 10 | Google Search> Organic> Email Marketing | 72.0> 24.0> 0.0 | True | 1 |
| 3 | id: 0_J: 11 | Organic | 0.0 | True | 1 |
| 4 | id: 0_J: 12 | Email Marketing> Facebook | 432.0> 0.0 | True | 1 |
| ... | ... | ... | ... | ... | ... |
| 20341 | id: 9_J: 5 | Direct> Facebook | 120.0> 0.0 | True | 1 |
| 20342 | id: 9_J: 6 | Google Search> Google Search> Google Search | 48.0> 24.0> 0.0 | True | 1 |
| 20343 | id: 9_J: 7 | Organic> Organic> Google Search> Google Search | 480.0> 480.0> 288.0> 0.0 | True | 1 |
| 20344 | id: 9_J: 8 | Direct> Organic | 168.0> 0.0 | True | 1 |
| 20345 | id: 9_J: 9 | Google Search> Organic> Google Search> Emai ... | 528.0> 528.0> 408.0> 240.0> 0.0 | True | 1 |

This **attribute is updated for each model generated **and in the case of heuristic results, a column containing the attribution given by a given model will be added at the end.

**Attention:**
Model calculations are not calculated based on the .DataFrame parameter, if it is changed, the results will not be affected.


`` python
attributions.attribution_last_click ()
attributions.DataFrame
``

| | journey_id | channels_agg | time_till_conv_agg | converted_agg | conversion_value |
| - | - | - | - | - | - |
| 0 | id: 0_J: 0 | Facebook | 0.0 | True | 1 |
| 1 | id: 0_J: 1 | Google Search | 0.0 | True | 1 |
| 2 | id: 0_J: 10 | Google Search> Organic> Email Marketing | 72.0> 24.0> 0.0 | True | 1 |
| 3 | id: 0_J: 11 | Organic | 0.0 | True | 1 |
| 4 | id: 0_J: 12 | Email Marketing> Facebook | 432.0> 0.0 | True | 1 |
| ... | ... | ... | ... | ... | ... |
| 20341 | id: 9_J: 5 | Direct> Facebook | 120.0> 0.0 | True | 1 |
| 20342 | id: 9_J: 6 | Google Search> Google Search> Google Search | 48.0> 24.0> 0.0 | True | 1 |
| 20343 | id: 9_J: 7 | Organic> Organic> Google Search> Google Search | 480.0> 480.0> 288.0> 0.0 | True | 1 |
| 20344 | id: 9_J: 8 | Direct> Organic | 168.0> 0.0 | True | 1 |
| 20345 | id: 9_J: 9 | Google Search> Organic> Google Search> Emai ... | 528.0> 528.0> 408.0> 240.0> 0.0 | True | 1 |


As we work with a large volume of data, we know that it is not possible to evaluate the results attributed for each journey that resulted in a transaction. Thus, by consulting the **attribute group_by_channels_models we bring the results of the models grouped by each channel **.

**Attention:**
The grouped results are not overwritten if the same model is calculated more than once and both results will be present in the group_by_channels_models attribute.


`` python
attributions.group_by_channels_models
``




<div>

| channels | attribution_last_click_heuristic |
| - | - |
| Direct | 2133 |
| Email Marketing | 1033 |
| Facebook | 3168 |
| Google Display | 1073 |
| Google Search | 4255 |
| Instagram | 1028 |
| Organic | 6322 |
| Youtube | 1093 |



And as with .DataFrame, **group_by_channels_models is also updated for each new model run **and without the limitation of not bringing the algorithmic results


`` python
attributions.attribution_shapley ()
attributions.group_by_channels_models
``

| | channels | attribution_last_click_heuristic | attribution_shapley_size4_conv_rate_algorithmic |
| - | - | - | - |
| 0 | Direct | 109 | 74.926849 |
| 1 | Email Marketing | 54 | 70.558428 |
| 2 | Facebook | 160 | 160.628945 |
| 3 | Google Display | 65 | 110.649352 |
| 4 | Google Search | 193 | 202.179519 |
| 5 | Instagram | 64 | 72.982433 |
| 6 | Organic | 315 | 265.768549 |
| 7 | Youtube | 58 | 60.305925 |

### **About models **

All heuristic models exhibit the same behavior regarding the update of **attributes .DataFrame and .group_by_channels_models **and also regarding **method output **that will return a **tuple containing two Series pandas **.


`` python
attribution_first_click = attributions.attribution_first_click ()
``

**The first output **of the tuple corresponds to the results in the **journey granularity **, similar to the result found in the .DataFrame


`` python
attribution_first_click [0]
``




    0 [1, 0, 0, 0, 0]
    1 [1]
    2 [1, 0, 0, 0, 0, 0, 0, 0, 0]
    3 [1, 0]
    4 [1]
                           ...
    20512 [1, 0]
    20513 [1.0, 0]
    20514 [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    20515 [1.0, 0]
    20516 [1, 0, 0, 0]
    Length: 20517, dtype: object



**The second **corresponds to the results in **channel granularity **, similar to the result found in .DataFrame


`` python
attribution_first_click [1]
``

| | channels | attribution_first_click_heuristic |
| - | - | - |
| 0 | Direct | 2078 |
| 1 | Email Marketing | 1095 |
| 2 | Facebook | 3177 |
| 3 | Google Display | 1066 |
| 4 | Google Search | 4259 |
| 5 | Instagram | 1007 |
| 6 | Organic | 6361 |
| 7 | Youtube | 1062 |

#### **Customizing the models **

Among the models present in the class only Last Click, First Click and Linear do not have customizable parameters beyond the **parameter group_by_channels_models **, which receives a **boolean value **and if **false **, **no will return the results of the models grouped by channels **.

##### **Last Click Non **model

It was created to replicate the standard behavior of Google Analytics in which **Direct traffic is superimposed **if it occurs after some interaction from another source within a certain period.

By default, the but_not_this_channel parameter receives the value 'Direct', but can be changed to other channels / values ​​according to your channels and groupings.




`` python
attributions.attribution_last_click_non (but_not_this_channel = 'Direct') [1]
``

| channels | attribution_last_click_non_Direct_heuristic |
| - | - | - |
| 0 | Direct | 11 |
| 1 | Email Marketing | 60 |
| 2 | Facebook | 172 |
| 3 | Google Display | 69 |
| 4 | Google Search | 224 |
| 5 | Instagram | 67 |
| 6 | Organic | 350 |
| 7 | Youtube | 65 |

##### **Position Based Model **

You can receive a list in the parameter **list_positions_first_middle_last **determining the percentages that will be allocated for the beginning, middle and end of the journey according to the business context of your customer / data. And that **by default **is distributed with the percentages **40% for the introducer channel, 20% distributed for the intermediate channels and 40% for the converter. **


`` python
attributions.attribution_position_based (list_positions_first_middle_last = [0.3, 0.3, 0.4]) [1]
``

| | channels | attribution_position_based_0.3_0.3_0.4_heuristic |
| - | - | - |
| 0 | Direct | 95.685085 |
| 1 | Email Marketing | 57.617191 |
| 2 | Facebook | 145.817501 |
| 3 | Google Display | 56.340693 |
| 4 | Google Search | 193.282305 |
| 5 | Instagram | 54.678557 |
| 6 | Organic | 288.148896 |
| 7 | Youtube | 55.629772 |

##### **Time Decay Model **

It can be shortened as to **decay percentage **in parameter **decay_over_time **and as to **time in hours in which this percentage will be applied **in parameter **frequency **.

However, it is worth noting that if there are more points of contact between the decay time spaces, the value will be distributed equally to these channels;

Example of model operation:
- **Channels: **Facebook> Organic> Paid Search
- **Time to Conversion: **14> 12> 0
- **Decay frequency: **7
- **Attributed results: **
  - 25% for Facebook;
  - 25% for Organic;
  - 50% for Paid Search;



`` python
attributions.attribution_time_decay (
    decay_over_time = 0.6,
    frequency = 7) [1]
``

| | channels | attribution_time_decay0.6_freq7_heuristic |
| - | - | - |
| 0 | Direct | 108.679538 |
| 1 | Email Marketing | 54.425914 |
| 2 | Facebook | 159.592216 |
| 3 | Google Display | 64.350107 |
| 4 | Google Search | 192.838884 |
| 5 | Instagram | 64.611414 |
| 6 | Organic | 314.920082 |
| 7 | Youtube | 58.581845 |

##### **Markov Chains **

**Attribution Model **based on **Markov Chains **helps us to solve the media attribution problem with an **algorithmic approach **based on data that calculates the probability of transition between channels.

This model behaves like the others when it comes to updating the .DataFrame and .group_by_channels_models, in addition to **returning a tuple **with the first two results representing the same ones previously described in the heuristic models. However, we obtain two outputs, the **transition matrix **and the **removal effect **.

As an input parameter, we have, in principle, how to indicate whether or not the probability of transition to the same state will be considered.


`` python
attribution_markov = attributions.attribution_markov (transition_to_same_state = False)
``

| | channels | attribution_markov_algorithmic |
| - | - | - |
| 0 | Direct | 2305.324362 |
| 1 | Email Marketing | 1237.400774 |
| 2 | Facebook | 3273.918832 |
| 3 | Youtube | 1231.183938 |
| 4 | Google Search | 4035.260685 |
| 5 | Instagram | 1205.949095 |
| 6 | Organic | 5358.270644 |
| 7 | Google Display | 1213.691671 |

This setting **does not affect the aggregated results **and that are attributed to each channel, **but the values ​​observed in the transition matrix **. And as we start **transition_to_same_state = False **the diagonal line, which represents the auto-transition, appears zeroed.


`` python
ax, fig = plt.subplots (figsize = (15.10))
sns.heatmap (attribution_markov [2] .round (3), cmap = "YlGnBu", annot = True, linewidths = .5)
``




! [png] (readme-images / output_37_1.png)


**Removal Effect **, fourth output of the attribution_markov results, is given by the ratio of the difference between the total conversion probability and the conversion probability without the channel, and the original total conversion probability.


`` python
ax, fig = plt.subplots (figsize = (2,5))
sns.heatmap (attribution_markov [3] .round (3), cmap = "YlGnBu", annot = True, linewidths = .5)
``




! [png] (readme-images / output_39_1.png)


##### **Shapley Value **

Finally, we have the second algorithmic model of the MAM class, the **Shapley Value **, a concept that comes from **Game Theory **, to distribute the contribution of each player in a cooperation game.

Model assigns credits for conversions by calculating the contribution of each channel present in the journey, using combinations of journeys with and without the channel in question.

**size parameter limits the number of unique channels in the journey **, by default **is defined as the last **4 **. This is because the number of iterations increases exponentially with the number of channels. 2N, where N is the number of channels.

The methodology for calculating marginal contributions can vary through the **parameter order **, which by default calculates the contribution of the **combination of channels regardless of the order in which they appear **in the different days.



`` python
attributions.attribution_shapley (size = 4, order = True, values_col = 'conv_rate') [0]
``

| | combinations | conversions | total_sequences | conversion_value | conv_rate | attribution_shapley_size4_conv_rate_order_algorithmic |
| - | - | - | - | - | - | - |
| 0 | Direct | 909 | 926 | 909 | 0.981641 | [909.0] |
| 1 | Direct> Email Marketing | 27 | 28 | 27 | 0.964286 | [13.948270234099155, 13.051729765900845] |
| 2 | Direct> Email Marketing> Facebook | 5 | 5 | 5 | 1.000000 | [1.6636366232390172, 1.5835883671498818, 1,752 ... |
| 3 | Direct> Email Marketing> Facebook> Google D ... | 1 | 1 | 1 | 1.000000 | [0.2563402919193473, 0.2345560799963515, 0.259 ... |
| 4 | Direct> Email Marketing> Facebook> Google S ... | 1 | 1 | 1 | 1.000000 | [0.2522517802130265, 0.2401286956930936, 0.255 ... |
| ... | ... | ... | ... | ... | ... | ... |
| 1278 | Youtube> Organic> Google Search> Google Dis ... | 1 | 2 | 1 | 0.500000 | [0.2514214624662836, 0.24872101523605275, 0.24 ... |
| 1279 | Youtube> Organic> Google Search> Instagram | 1 | 1 | 1 | 1.000000 | [0.2544401477637237, 0.2541071889956603, 0.253 ... |
| 1280 | Youtube> Organic> Instagram | 4 | 4 | 4 | 1.000000 | [1.2757196742326997, 1.4712839059493295, 1,252 ... |
| 1281 | Youtube> Organic> Instagram> Facebook | 1 | 1 | 1 | 1.000000 | [0.2357631944623868, 0.2610913781266248, 0.247 ... |
| 1282 | Youtube> Organic> Instagram> Google Search | 3 | 3 | 3 | 1.000000 | [0.7223482210689489, 0.7769049003203142, 0.726 ... |

Finally, the parameter on which the Shapley Value will be calculated can be changed to **values_col **, which by default uses the **conversion rate **which is a way of **considering non-conversions in the model calculation **. However, we can also consider in the calculation the total conversions or the value generated by the conversions, as shown below.


`` python
attributions.attribution_shapley (size = 3, order = False, values_col = 'conversions') [0]
``

| | combinations | conversions | total_sequences | conversion_value | conv_rate | attribution_shapley_size3_conversions_algorithmic |
| - | - | - | - | - | - | - |
| 0 | Direct | 11 | 18 | 18 | 0.611111 | [11.0] |
| 1 | Direct> Email Marketing | 4 | 5 | 5 | 0.800000 | [2.0, 2.0] |
| 2 | Direct> Email Marketing> Google Search | 1 | 2 | 2 | 0.500000 | [-3.1666666666666665, -7.666666666666666, 11.8 ... |
| 3 | Direct> Email Marketing> Organic | 4 | 6 | 6 | 0.666667 | [-7.833333333333333, -10.833333333333332, 22.6 ... |
| 4 | Direct> Facebook | 3 | 4 | 4 | 0.750000 | [-8.5, 11.5] |
| ... | ... | ... | ... | ... | ... | ... |
| 75 | Instagram> Organic> Youtube | 46 | 123 | 123 | 0.373984 | [5.833333333333332, 34.33333333333333, 5.83333 ... |
| 76 | Instagram> Youtube | 2 | 4 | 4 | 0.500000 | [2.0, 0.0] |
| 77 | Organic | 64 | 92 | 92 | 0.695652 | [64.0] |
| 78 | Organic> Youtube | 8 | 11 | 11 | 0.727273 | [30.5, -22.5] |
| 79 | Youtube | 11 | 15 | 15 | 0.733333 | [11.0] |

### Preview
And now that we have the results attributed by the different models stored in our object **. Group_by_channels_models **according to our business context we can plot a graph and compare the results.


`` python
attributions.plot ()
``



! [png] (readme-images / output_45_1.png)


If you want to select only the algorithmic models, we can specify it in the **model_type **parameter.


`` python
attributions.plot (model_type = 'algorithmic')
``




! [png] (readme-images / output_47_1.png)

