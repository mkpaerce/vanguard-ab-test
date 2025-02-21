# **Objective:** 

Analyze the performance of an A/B test to determine if a new, more intuitive User Interface (UI), along with in-context prompts, significantly enhances user experience for Vanguard's clients by increasing the completion rates of the online process.


# **Datasets:**


## **Demographic Data:**



* **Numerical Columns:**  \
‘bal’ - balance in their account
* **Categorical Columns: ** \
‘clnt_age’ - age of client \
‘clnt_tenure_yr’  - number of years the client has held account \
‘Clnt_tenure_mnth’ - number of months the client has held account \
‘Gendr’ - client gender \
‘Num_accts’ - number of accounts owned

    ‘Calls_6_mnth’ - number of log ons in the last 6 months


    ‘Logons_6_mnth’ - number of log ons in the last 6 months


    ‘Client_id’ - unique identifier for clients



## **Website Data 1 & 2:**



* **Numerical Columns:**

    ‘date_time’ - includes the day and exact time when each step occurs

* **Categorical Columns:**

    ‘client_id’ - represents the individual client - unique (e.g. “Prince”)


    ‘visitor_id’ - represents the device and client (e.g. “Prince on Mobile Phone” or “prince on Web”)


    ‘visit_id’ - represents the session (e.g. “Prince visits at 3pm”)


    ‘process_step’ - represents the stages in the online process



## **Experiment Clients:**



* **Categorical Columns:**

    Client_id - unique client id


    Variation - test or control group



# **Cleaning & Transformations:**

**Website Data:**



1. Merging website data 1 & 2 via ‘concat’ into one file
2. Adding the column ‘variation’ indicating whether it’s a part of the control or test group
3. Dropping duplicates (1148 duplicated values)
4. Split date and time into two separate columns
5. Change the values of process_step into integers [0 - 4]

**Demographic Data:**



1. Adding the column ‘variation’ indicating whether it’s a part of the control or test group
2. Dropped rows with null values (10-18 rows)


# **Client Behavior Analysis:**

**Client Tenure:**



* The clients using the online process are primarily long-term clients  - **75% of the control group have held accounts for 6+ years.**
* The **median tenure of clients** using the online process is **11 years** (based on control)
* The distribution for client tenure in the control group is **multimodal** suggesting that the control group might have been selected based on** three different groups**, e.g. New customers with tenure **5–10 years**. Mid-tenured customers **15–20 years**. Long-term customers **25+ years**

**Client Age:**



* **Clients using the online process are typically middle aged,** the median age is 48 years old. (based on control)
* The distribution is **multimodal**, suggesting that maybe the control group was selected based on **two groups 18-40 and 40+.**

**Other demographic indicators:**



* Even distribution of M, F, and Unknown gender
* Majority of clients have 2 accounts
* Larger share of clients have 6 calls
* Larger share of clients have 9 log-ons, minimum 3 log-ons


# **KPIs & Testing**

We will use **three KPIs** to determine the significance and success of the new process on a test group (new process) and control group (baseline, old process): 


## Completion Rate

The proportion of total users who reach the final ‘confirm’ step  (numerical - discrete**)**

**Significance Level:** 0.05 (standard)

**Hypothesis: **“Completion rate for the test group is higher than the control group”



* Null hypothesis: completion rate, control >= completion rate, test
* Alternative hypothesis: completion rate, control &lt; completion rate, test

**NOTE** - This is a one-tail test (since we are testing for an increase).

**Type of Distribution: **Binomial** **because it’s measuring multiple users and completion rates are discrete.

**Test for Statistical Significance:** 



* Use **proportions test (Z-test)** because we are comparing two proportions (i.e. The data is discrete - binary - true or false)
* **If the p-value is less than 0.05,** we reject the null hypothesis. The difference we observed is not due to random chance.


### **How we do this with Python:**

Two-sample Z-test for proportions (one-tailed, "larger").



1. Get the number of users who complete for the test group. (x_test)
2. Get the number of users who complete for the control group. (x_control)
3. Get the total number of users in each group. (n_test, n_control)
4. Perform a **Z-test for two proportions** to compare completion rates.
5. >> proportions_ztest([x_test, x_control], [n_test, n_control], alternative="larger")


## Process Completion Time: 

The average duration for users to complete the online process (numerical - continuous).

**Significance Level:** 0.05 (standard)

**Hypothesis:** “The mean completion time for the test group is lower than the control.”



* **Null Hypothesis:** mean completion time, control =&lt; mean completion time, test
* **Alternative Hypothesis:** mean completion time, control > mean completion time, test

**NOTE:** This is a one-tail test (since we are testing for a decrease).

**Type of Distribution:**



* Right-skewed, so we apply a **log transformation** to normalize the data before analysis.
* After transformation, the data is approximately **normally distributed**, which allows us to use a **T-test**.

**Test for Statistical Significance:**



* Use a **t-test** because we are comparing two samples with continuous variables.
* If the p-value is less than 0.05, we reject the null hypothesis. The difference we observed is not due to random chance.

**How we do this with Python:**



* Two-sample **t-test** (one-tailed, "less").
* Get a numpy array with group times for test group
* Get a numpy array with group times for control group
* Apply log transformation for test group and control group (log_test, log_control)
* >> ttest_ind({log_test}, {log_control}, alternative="less") 


## Error Rates: 

The proportion of process steps where users moved backward, relative to the total number of steps taken. (discrete - categorical).

> Error rate = Number of steps back / total number of steps taken

**Significance Level:** 0.05 (standard)

**Hypothesis:** “The error rates for the test group are lower than the control group.”

**Null Hypothesis:** error rate, control =&lt; error rate, test \
**Alternative Hypothesis:** error rate, control > error rate, test

**NOTE:** This is a one-tail test (since we are testing for a decrease).

**Type of Distribution:**



* **Binomial distribution** because error rates represent **discrete binary outcomes** (users either go back a step or they don’t).

**Test for Statistical Significance:**



* Use a **proportions test **(**z-test) **because we are comparing two proportions (error rates) from different groups.
* If the p-value is less than 0.05, we reject the null hypothesis. The difference we observed is not due to random chance.


### **How we do this with Python:**

Two-sample Z-test for proportions (one-tailed, "less").



6. Get the number of users who went back a step (**errors**) for the test group. (x_test)
7. Get the number of users who went back a step (**errors**) for the control group. (x_control)
8. Define the total number of users in each group. (n_test, n_control)
9. Perform a **proportions test (Z-test)** to compare error rates.
10. >> proportions_ztest([x_test, x_control], [n_test, n_control], alternative="smaller")


# **Results & Findings**


## *Completion Rate*

**Hypothesis: **“Completion rate for the test group is higher than the control group” (greater)

	Completion Rate = # of People Reaching Step 4 / # of People Reaching Step 0

	Completion Rate for Test:  **45.5% **

Completion Rate for Control: **36.8%**

Difference: +**8.7%**

(Z-statistic: 28.6203, P-value: 0.0000)

**Summary: **We were able to reject the null hypothesis, the test group has a higher completion rate.

**Recommendations: **The difference of 8.7% in completion rate satisfies the cost effectiveness threshold of 5%.


## *Completion Time, Overall*

**Hypothesis:** “The mean completion time for the test group is lower than the control.”

	Completion Time = Time to get from step 1 to step 4

	Mean Completion Time, Test Group =** 58.68 seconds**

	Mean Completion Time, Control Group = **64.98 seconds**

**	**Difference in mean completion times**: 6.3 seconds**

**	**(T-statistic: -21.6127, P-value: 0.0000000000)

**Summary: **We were able to reject the null hypothesis, the mean completion time for the test group was significantly lower than the control group (6.3 seconds.)

**Note, we performed a log transformation to deal with extreme outliers in both groups


## *Completion time, By Step*

**Hypothesis:** The mean completion time per step for the test group is lower than the control.


<table>
  <tr>
   <td><strong>Step</strong>
   </td>
   <td><strong>Completion time, Test</strong>
   </td>
   <td><strong>Completion time, Control</strong>
   </td>
   <td><strong>Z-Stat, P-Value</strong>
   </td>
   <td><strong>Outcome</strong>
   </td>
  </tr>
  <tr>
   <td>Step 0-1
   </td>
   <td>31,08 s
   </td>
   <td>38,03 s
   </td>
   <td>Z-statistic: -40.0813
<p>
P-value: 0.0000
   </td>
   <td>Yes, avg test group is significant lower.
   </td>
  </tr>
  <tr>
   <td>Step 1-2
   </td>
   <td>37,04 s
   </td>
   <td>34,21 s
   </td>
   <td>Z-statistic: 20.2266
<p>
P-value: 1.0000
   </td>
   <td>No, avg test group is significant higher.
   </td>
  </tr>
  <tr>
   <td>Step 2-3
   </td>
   <td>86,39 s
   </td>
   <td>87,25 s
   </td>
   <td>Z-statistic: 7.3753 P-value: 1.0000
   </td>
   <td>No, avg test group is significant higher.
   </td>
  </tr>
  <tr>
   <td>Step 3-4
   </td>
   <td>105,12 s
   </td>
   <td>126,10 s
   </td>
   <td>Z-statistic: -22,2068
<p>
P-value: 0.0000
   </td>
   <td>Yes, avg test group is significant lower
   </td>
  </tr>
</table>


**Summary: **We see that the reduction in completion time comes due to a significant reduction in average time from step 0-1 and 3-4. 

**Recommendation: **The web should focus on improving steps 1-2 and 2-3 as the average time has increased in those steps. The web could have some errors in those steps.


## *Error Rates, Overall*

**Hypothesis:** “The error rates for the test group are lower than the control group.” (smaller)


    Error rate = proportion of process steps where users moved backward, relative to the total number of steps taken.


    Error Rate, Test Group: **6.6%**


    Error Rate, Control Group: **4.3%**


    (Z-statistic: 27.6021, P-value: 1.0000) \


**Summary: **We failed to reject the null hypothesis, the error rate for the test group was larger than the control group.

**Recommendation: **It’s possible that users were confused by the new process, causing more errors. If possible, go back and look to see if the error rates were significantly different between test group users with 1 visit and users +1 visits.


## *Error Rates, By Step*

**Hypothesis:** The error rates for the test group do not differ significantly from those of the control group at each step (two-sided).


<table>
  <tr>
   <td><strong>Step</strong>
   </td>
   <td><strong>Error Rate, Test</strong>
   </td>
   <td><strong>Error Rate, Control</strong>
   </td>
   <td><strong>Z-Stat, P-Value</strong>
   </td>
   <td><strong>Outcome</strong>
   </td>
  </tr>
  <tr>
   <td>Step 1
   </td>
   <td>19.1%
   </td>
   <td>9.7%
   </td>
   <td>Z-statistic: 31.9364
<p>
P-value: 0.0000
   </td>
   <td>Yes, significant diff.
   </td>
  </tr>
  <tr>
   <td>Step 2
   </td>
   <td>11.3%
   </td>
   <td>6.2%
   </td>
   <td>Z-statistic: 19.6670
<p>
P-value: 0.0000
   </td>
   <td>Yes, significant diff.
   </td>
  </tr>
  <tr>
   <td>Step 3
   </td>
   <td>9.3%
   </td>
   <td>11.1%
   </td>
   <td>Z-statistic: -6.3238
<p>
P-value: 0.0000
   </td>
   <td>Yes, significant diff.
   </td>
  </tr>
  <tr>
   <td>Step 4
   </td>
   <td>0%
   </td>
   <td>0%
   </td>
   <td>Z-statistic: 1.4152
<p>
P-value: 0.1570
   </td>
   <td>No significant difference.
   </td>
  </tr>
</table>


**Summary: **We see that Step 1 and 2 were especially problematic for test group users. Where users in the test group actually performed better in step 3.

**Recommendation: **Focus on improving step 1 and 2 in the new design.


# **Recommendations**

**Go for Implementation** – The **+8.7% completion rate** exceeds the 5% cost-effectiveness threshold. The company should **proceed with the web flow improvements**.

**Key Efficiency Gain** – Faster completion time was mainly driven by a **significant improvement in Step 0-1**.

**Error Rate Consideration** – No major reduction in errors, likely due to a learning curve. A **longer test** may help determine if errors stabilize over time.

**Final Recommendation** – Since **Step 0-1** had the biggest impact, most benefits can likely be achieved **by optimizing just this step**, minimizing disruption while maximizing results.
