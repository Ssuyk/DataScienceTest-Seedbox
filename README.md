# Report for Seedbox technical test
​														**Vincent Gruson (date: 2018/01/28)**

## Introduction

The purpose of this test is to extract relevant information regarding an A/B test performed on a website hosted by Seedbox. More particularly, the impact of the modification of the cancellation procedure is analysed. Initially, members were able to cancel their subscription by filling a web form. What is proposed here is to force them to call-in to the customer service line in order to cancel. 

To perform the analysis, two files are provided, testSamples.csv and transData.csv: 

* testSamples gives a list of unique users who participated in the A/B test (59721 unique ID, 14835 (24.8%) in the Test/call-in group, 44886 (75.2%) in the Control/web form group). This file aims at knowing the group (Control/Test) in which the unique ID belongs.
* The second file provides the number and type of transactions of randomly selected users, picked up from the testSamples.csv file.

From this test, four questions are asked: 

1. What is the approximate probability distribution between the test group and the control group?
2. Is a user that must call-in to cancel more likely to generate at least 1 additional rebill?
3. Is a user that must call-in to cancel more likely to generate more revenues?
4. Is a user that must call-in more likely to produce a higher chargeback rate?

**Note**: methods are not discussed intensively in this report. For more information, see the Jupyter Notebook (or the .py file) called "SeedBox_DataScienceApplicationTest".

## Results summary

Before providing a deep analysis, here are the answers of the above questions given in a concise way. 

1. The probability distributions of the different transactions are :

   |  Group  | Rebill | Refund | Chargeback | Total |
   | :-----: | :----: | :----: | :--------: | :---: |
   | Control | 0.505  | 0.025  |   0.014    | 0.545 |
   |  Test   |  0.43  | 0.016  |   0.008    | 0.454 |
   |  Total  | 0.935  | 0.041  |   0.022    |   1   |

   The probability distributions of the two groups in term of unique sample ID inside transData.csv are: 

   |  Group  | Rebill | Refund | Chargeback | Total |
   | :-----: | :----: | :----: | :--------: | :---: |
   | Control | 0.345  | 0.032  |   0.020    | 0.397 |
   |  Test   | 0.571  | 0.021  |   0.011    | 0.603 |
   |  Total  | 0.916  | 0.054  |   0.030    |   1   |

2. Users who must call-in are more likely to generate at least 1 additional rebill. The probability that users in the control group generate at least 1 additional rebill is around 87%, while being up to 95% in the test group. 

3. The mean revenue from the control group is about 83\$ and about 58\$ for the test group, with respective standard deviation of 103\$ and 55\$. This means that people from the test group are less likely to generate more revenues than users from the control group. However, the high standard deviation of the control group indicates that even if they generate more revenues on average, they can also ask for more refunds or chargebacks.

4. Users that must call-in are less likely to produce a higher chargeback rate. The chargeback rate is defined as the ratio between the number of "chargeback" transactions over the number of "rebill" transactions. In the control group, this chargeback rate is about $2.7\%$, while being about $1.6\%$  in the test group, thus leading to an absolute $0.9\%$ percent chargeback rate decrease ($35\%$ relative decrease).

**Now, a more in-depth study is proposed.** 

## 1. What is the approximate probability distribution between the test group and the control group?

Relevant information for this question have been given in the Results summary section. Here, one can find the table and Figure of the different populations observed during this study. 

![TotalTransactions](C:\Users\Vincent\Documents\GitHub\datasciencetest\Figures\TotalTransactions.png)

**Figure 1**:Distribution of the different transactions of the studied samples. *Left* : comparison of the total number of transactions between the control and the test group. *Right*: comparison between the different types of transactions, independently of the group of origin.




The table below is a summary of what can be found in Figure 1.

|  Group  | Rebill | Refund | Chargeback | Total |
| :-----: | :----: | :----: | :--------: | :---: |
| Control |  3756  |  188   |    106     | 4050  |
|  Test   |  3205  |  118   |     57     | 3380  |
|  Total  |  6961  |  306   |    163     | 7430  |

![CompareUniqueID](C:\Users\Vincent\Documents\GitHub\datasciencetest\Figures\CompareUniqueID.png)
**Figure 2**: Unique user distribution. *Left*: comparison and repartition of the two groups. *Right*: comparison between the different types of transactions, independently of the group of origin.



The table below is a summary of the results appearing in Figure 2:

| Group   | Rebill | Refund | Chargeback | Total |
| ------- | ------ | ------ | ---------- | ----- |
| Control | 941    | 88     | 53         | 1082  |
| Test    | 1556   | 58     | 29         | 1643  |
| Total   | 2497   | 146    | 82         | 2725  |

## 2. Is a user that must call-in to cancel more likely to generate at least 1 additional rebill?

The objective is to know if users from the test group are more inclined to occasion at least one rebill. To do that, one can compare the "REBILL" distribution of the two groups. Basically, we count the number of transactions per sample ID. In **Figure 3**, the number of time each unique ID generates a "REBILL" of the Control (left, red) and Test (right, blue) groups is displayed.

The main observation here is that both distributions follow a decay law (either a exponential or a Poisson distribution), meaning that most of the time, people will only generate one or two "REBILL". This is particularly true for the Test case, where $65\%$ of the population only generates up to 2 "REBILL", while being up to $45\%$ for the control group. 

![RebillDistribution](C:\Users\Vincent\Documents\GitHub\datasciencetest\Figures\RebillDistribution.png)
**Figure 3**: Number of REBILL transactions per unique user ID, for the Control group (left) and for the Test group (right).



From this curve, one can extract the probability that a unique user generates **at least** 1 "REBILL". Basically, it corresponds to the number of unique ID who did at least one REBILL transaction over the total number of unique ID in a given group.  For the control group, $87\%$ are inclined to do so, while the value goes up to $95\%$ for the test group, leading to an absolute difference of $8\%$ ($8.5\%$ relative difference). 

To check the statistical significance of this A/B test, a Bayesian approach is used [^1]. While being easy to implement, it offers a intuitive view of the mean value and the certainty of the measurement related to the sample size (by looking at the spread of the distribution). It also eases the extraction of the p-value by measuring the overlap between the two group distributions. Results are displayed in Figure 4, where one can extract two informations: 

* the mean value, *i.e* the probability that a unique user generates at least one REBILL, is represented by the black vertical lines, for each group.
* the p-value, defined in this case as the overlap between the two distributions. As there is no overlap between the two, this means that the p-value tends to zero, meaning the test is statistically significant. 

[^1]: For more information about the way it is implemented, one can go to this simple How-to https://www.countbayesie.com/blog/2015/4/25/bayesian-ab-testing 

![Bayesian_Question2](C:\Users\Vincent\Documents\GitHub\datasciencetest\Figures\Bayesian_Question2.png)
**Figure 4**: Extraction of the probability that an user generates at least 1 "REBILL" and verification of the statistical significance of this study, using Bayesian statistical inference method. 



**To conclude**, asking the client to call-in for cancellation of their subscriptions leads to a higher REBILL rate. 


## 3. Is a user that must call-in to cancel more likely to generate more revenues? 

To answer this question, the total expense of each unique user has to be determined, without considering the associated transaction type. By observing the TransData.csv file, one can see that the different users always do the same operation. Thus, to extract the total expense of each users, one just needs to multiply the "transaction_amount" by the number of operations. 

The associated results are displayed Figure 5, as well as a fit attempt using a Normal distribution. From this, one can extract the values of interest, summarized in the following table: 

|         | mean $\mu$ | standard deviation $\sigma$ |
| :-----: | :--------: | :-------------------------: |
| Control |  83.03\$   |          103.14\$           |
|  Test   |  58.08\$   |           54.90\$           |

![CompareTotalExpense_question3](C:\Users\Vincent\Documents\GitHub\datasciencetest\Figures\CompareTotalExpense_question3.png)

**Figure 5**: Total expense distribution and fit (black curve) based on a Normal distribution, for the control group (left,red) and for the test group (right, blue).



When considering only the mean value, one tends to say that users from the test group spend less than those from the control group ($\mu_{test} = 58.08\$ < \mu_{control} = 83.03\$$). However, users from the test group seem to be more consistent, as the standard deviation is lower than the control group one ($\sigma_{test} = 54.90\$ > \sigma_{control} = 103.14\$$), meaning that they are less likely to ask for refunds or chargebacks.

Regarding the statistical significance of this test, a t-test comparing the two distributions gives a p-value of 0, meaning that these results are trustworthy (considering the commonly accepted threshold value $\alpha = 0.05$).

**While not being as obvious as in the previous question, asking user to call-in seems beneficial.**

## 4. Is a user that must call-in more likely to produce a higher chargeback rate?

Here, I got a bit confused with the definition of chargeback rate. The question asks to calculate the chargeback rate defined as the number of chargeback transactions over the number of rebill transactions. Looking at the definition online, it appears that the chargeback rate can also be defined as the number of chargebacks over the total number of transactions (in our case : rebill + refund + chargeback).

To take a decision, a comparison of the two cases is done.

* **Case 1**: Using the question definition, the control group chargeback rate is about $2.6\%$, and about $1.7\%$ for the test group. Using a Bayesian approach, we see this time that the two distributions slightly overlap, leading to a p-value of $0.003$. This is still way below the $\alpha =0.05$ threshold, leading to statistical significance. Results are displayed in Figure 6.

![Bayesian_Question4_case1](C:\Users\Vincent\Documents\GitHub\datasciencetest\Figures\Bayesian_Question4_case1.png)
**Figure 6**: Extraction of the chargeback rate considering **Case 1** definition and verification of the statistical significance of this study, using Bayesian statistical inference method. 



* **Case 2**: Using the other definition $\left( \frac{\#Chargeback}{\#Total} \right)$, the control group chargeback rate is about $2.8\%$, and about $1.8\%$ for the test group. A p-value of $0.002$ is obtained. Again, these results are significant, considering a threshold value $\alpha = 0.05$.

![Bayesian_Question4_case2](C:\Users\Vincent\Documents\GitHub\datasciencetest\Figures\Bayesian_Question4_case2.png)
**Figure 7**: Extraction of the chargeback rate considering **Case 2** definition and verification of the statistical significance of this study, using Bayesian statistical inference method. 




As in our situation, there is no major difference between the two cases ($\Delta_{CR} \approx 0.1-0.3\%$), the result from the question definition will be used. 

Thus, the chargeback rate of the test group is $35\%$ lower than the control group chargeback rate ($\Delta_{CR} = 2.6-1.7 = 0.9\%$), meaning that the call-in setup is limiting chargeback requests.

## Conclusion

From this study, the call-in system appears as a promising alternative to ensure:

* a higher rebill rate ($95\%$ for the test group against $87\%$ for the control group)
* a more reproducible behavior in term of total expense (lower mean value of the total expense but also lower standard deviation)
* a good way to limit chargeback rate. This point is of great importance as chargeback rate can be associated to fraud. Moreover, a high chargeback rate can lead to penalties or supplementary fees from bank organizations. 

It should be applied permanently.