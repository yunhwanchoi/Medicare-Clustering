# K-Means Clustering Lines of Services in the Emergency Department Provided by Medicare

The following README.md file includes the analysis without code. The full walk-through of the analysis and the Python code are available [here](https://nbviewer.org/github/yunhwanchoi/Medicare-Clustering/blob/main/Medicare%20Providers%20Clustering.ipynb).

# Table of Contents

[I. Introduction](#1)


[II. Data Cleaning/Preparation](#2)
  - [A. Choosing a Subset of the Data](#2a)
  - [B. Visuals/Removing Outliers](#2b)
    
[III. K-means Clustering](#3)


[IV. Clustering Results](#4)


[V. Conclusion](#5)


# I. Introduction<a class="anchor" id="1"></a>
  

Centers for Medicare and Medicaid Services (CMS) has provided a dataset “Physician and Other
Supplier PUF” with information on services and procedures provided under Medicare from 2012
to 2018. With almost 10 million rows and 26 features, a wealth of information and trends can be
extracted from the dataset. One class of useful information that can be drawn from this dataset is
the healthcare industry’s pattern of spending, where insights can be drawn to optimize healthcare
costs. Healthcare expenditure in the United States by far surpasses any other nation, as in 2018 it
amounted to 3.6 trillion dollars per year, accounting for 19.4\% of the GDP. Understanding patterns within expenditure, such as in which
types of segments of the healthcare industry the expenditure is concentrated in, would be beneficial
for Medicare and other public and private healthcare entities. 

The analysis with the full code can be found [here](https://nbviewer.org/github/yunhwanchoi/Medicare-Clustering/blob/main/Medicare%20Providers%20Clustering.ipynb).

Source of the Data: Center for Medicare and Medicaid Services (CMS)\
Link: https://www.cms.gov/Research-Statistics-Data-and-Systems/Statistics-Trends-and-Reports/Medicare-Provider-Charge-Data/Physician-and-Other-Supplier

## Goals

The emergency department is chaotic in nature and requires a lot of resources, and therefore is often expensive to manage. Therefore, understanding the pattern of expenditure in ED services is crucial for healthcare entities and systems to efficiently operate. The goal of this analysis is to generate insights about spending patterns by Emergency Departments via segmenting lines of services with similar characteristics into different groups by using K-means clustering.

Because the dataset contains many
different types of services, **I decided to focus on services provided occurring in the Emergency
Department (ED)**.

Through K-means clustering, I produced a dataset that grouped each line of services (by a given provider)
relevant to the Emergency Department into 11 different clusters. Each cluster is associated with a certain characteristic
and has information about Medicare’s expenditure patterns for the specific cluster. Healthcare
system managers can get a brief overview of whether the expenses are being allocated correctly,
and whether the services that tend to be more expensive and dealing with more severe cases are being covered
proportionately more by Medicare. 

## Dataset

**Each row of the dataset represents a specific line of service given by the specified
provider.**

The dataset has 26 features, including information of the provider such as name,
credentials, location, and field of practice. It also includes information about the type of service
provided, as well as the number of times that specific line of service has been provided, and its
average cost to Medicare and the beneficiaries.

Some notable features include the following:
- name, address/country, and identifier code for the performing provider 
- `nppes_entity_code`: Type of entity of the provider. 
    - `nppes_entitycode = I`: identifies providers registered as an individual (e.g. a doctor)
    - `nppes_entitycode = O`: identifies providers registered as an organization
- `nppes_credentials`: Credentials of the provider.
    - This field is blank when `nppes_entity_code` is `O`. 
- `provider_type`: The specialty of the field that the physician or professional (provider) is associated with
    - Because there were a lot of different provider types, I took a subset of five of the most common ones.
    - Emergency Medicine, Physician Assistant, Nurse Practitioner, Internal Medicine, Cardiology
- `hcpcs_code`: Represents the HCPCS code that identifies the `provider_description`
- `hcpcs_description`: Provides the type of the service provided
- `line_srvc_cnt`: Represents the number of services provided per provider (2012-2018)
- `average_Medicare_allowed_amt`: Represents the average of the sum of the following for each service provided 
    - Amount paid by Medicare 
    - Amount paid by beneficiaries (copays/deductibles)
    - Amount paid by third party
- `average_Medicare_payment_amt`: Average amount that Medicare paid (after deductible and coinsurance have been deducted for the service). 
- `average_Medicare_standardized_amt`: Standardized average amount that Medicare paid 
    - Adjusts for geographic differences in payment rates for individual services
    - Available starting on data collected from 2014 

I also added a new feature
- `average_perc_covered`: Percentage of the total bill paid by Medicare (standardized). 

# II. Data Cleaning/Preparation <a class="anchor" id="2"></a>

![image](https://user-images.githubusercontent.com/52474489/142739338-388346f5-7ddc-470f-af41-171011c2127b.png)

**There are 9,961,865 instances in the dataset. There are 8 numeric features and 17 categorical features.** 

## A. Choosing a Subset of the Data<a class="anchor" id="2a"></a>

To conduct meaningful analysis about spending patterns in the Emergency Department, I needed to take a small subset of this dataset. I decided to narrow down my rows into the following criteria:
-	Provider must participate in Medicare
-	Provider must be based in the United States
-	Provider must be registered in NPPES as an individual (entity type code `I`)
-	The type of service (`hcpcs_description`) must be in one of five top categories
    - Emergency department visit, problem with significant threat to life or function (**HCPCS Code 99285**)
    - Emergency department visit, problem of high severity (**HCPCS Code 99284**)
    - Emergency department visit, moderately severe problem (**HCPCS Code 99283**)
    - Emergency department visit, low to moderately severe problem (**HCPCS Code 99282**)
    - Emergency department visit, self limited or minor problem (**HCPCS Code 99281**)
        - There are **5512** values for `hcpcs_description`, or the different types of services there are provided. To see the most relevant `hcpcs_description` to the emergency department, I identified every `hcpcs_description` with the word "Emergency Department." This isn't an exhaustive list of services in the ED, but it ensures that all of them occur within ED.

The dataset is almost entirely composed of the visits of top 3 severity levels: 
- **Life threatening Severity**
- **High Severity**
- **Moderate Severity**

After selecting out the data based on `hcpcs_description`, I decided to further narrow down the dataset to lines of services that have `provider_type` within the five most common categories. 
- Emergency Medicine
- Physician Assistant
- Nurse Practitioner
- Internal Medicine
- Cardiology

*Note: I didn't require `provider_type` to be Emergency Medicine. Other providers like Internal Medicine or General Surgery can also provide services in the Emergency Department*

After dropping features that are unnecessary for clustering (provider name, address, etc) **the final cleaned dataset was left with the following features:**
- Provider Type (Emergency Medicine, PA, NP, Internal Med, Cardiology)
- HCPCS Description (With 5 varying severity levels for visits to the ED)
- `line_srvc_cnt`: Represents the number of services provided by a provider
- `average_Medicare_allowed_amt`: total sum of the cost of the service
- `average_Medicare_standardized_amt`: Standardized average amount that Medicare paid 
- `average_perc_covered`: % that Medicare covered on average for the line of service by the given provider
   - Obtained the new feature by calculating `average_Medicare_standardized_amt`\*100/`average_Medicare_allowed_amt`

![image](https://user-images.githubusercontent.com/52474489/142739664-44a1400c-448e-4d8b-8445-9d713a63de3c.png)


## B. Visuals/Removing Outliers <a class="anchor" id="2b"></a>

Clustering can be rather sensitive to outliers. After examining the data, I decided to remove any instances with `line_srvc_cnt` > 1000, and ensured `average_perc_covered` was equal or less than 100% to remove errors potentially caused by data entry mistakes or unusual circumstances. I then plotted a histogram for each numerical variable.

![image](https://user-images.githubusercontent.com/52474489/142739885-1d6fa016-3cb5-4a3b-b293-4cb2445e8e92.png)

Above represents the frequency histograms of all four of the numeric features. The variable `line_service_cnt` follows an exponential-like distribution, which makes sense as they are whole number “counts” of how many each service has been provided. For the two variables that indicate Medicare expenses, they seem to follow an irregular distribution where certain numbers have a spike. This may be due to “rounding” of expenses to the nearest whole dollar, but it is interesting to note that there are three distinct spikes for both histograms, where each spike is preceded by a similar pattern. This may mean that certain types of services within the ED were overrepresented, where maybe those specified services have distinct costs. I was able to use clustering to gain more insights on how these numeric variables are related to the categorical variables.

Another interesting thing to note was that the percentage of the bill covered by Medicare mostly fall between 60% and 80%, with a slight skew towards 80%.

![image](https://user-images.githubusercontent.com/52474489/142739907-9715f0b1-64b6-4095-adfc-4aa853f4c8dc.png)

A high correlation is observed between total cost `average_Medicare_allowed_amt` and how much medicare is paying for the service `average_Medicare_standard_amt`, which makes sense. These variables are also moderately positively correlated with the number of services (frequently conducted services cost more). Relatively little correlation exists between cost of service and coverage % by Medicare, which is an interesting insight because you would expect service cost to be a major factor in how much of the total cost Medicare is willing to pay. Clustering can reveal other underlying factors.

# III. K-Means Clustering<a class="anchor" id="3"></a>

### SCREE Plot

After one-hot encoding and normalizing the dataset, I ran multiple instances of K-means and obtained the following **Scree Plot** which plots the number of clusters to its respective **SSE**, a measure of variation within each cluster (the sum of the squared differences between each observation and its cluster's mean). SSE is expected to constantly decrease as the number of clusters k increases. I used from *k=2* to *k=24*.

![image](https://user-images.githubusercontent.com/52474489/142739927-b5caee60-66fc-4989-8459-a8b5a04fb27b.png)

The plot noticeably has a kink at *k = 9*, and slowly declines after, which suggests that the difference in SSE is relatively marginal after this point. *k = 9* may be a good number to choose. 

### Silhouette Plot 

In addition to the SCREE Plot, I calculated the **silhouette value** for each possible number of cluster k. The silhouette value is a measure of how similar (or cohesive) an instance is to its own cluster. The higher the silhouette value, the tighter the clusters are for the given k. 
I generated a silhouette score for each k used on the dataset and plotted it from *k=2* to *k=15*.

![image](https://user-images.githubusercontent.com/52474489/142739942-58840fa7-2a1e-4f3e-84e1-6d83407a266a.png)

The silhouette plot suggests a higher range of k, with 14 being the value that generates the greatest silhouette score. Similarly to the SCREE plot, it is suggested to choose a *k* around where the silhouette score drops off. 

**I will choose *k=11* because the increase in the silhouette score seems to taper off after this point.**

# IV. Clustering Results<a class="anchor" id="4"></a>

After clustering with *k = 11*, I added two more variables to get more information about our clusters, `size` and `Total Average Medicare Cost`. 
- `size` represents the number of instances in each cluster
- `Total Average Medicare Cost` represents average Medicare cost multiplied by the number of lines of services.

After parsing through the clusters, I created an intuitive dataframe that provides characteristics about each cluster, and sorted the clusters in descending order of Average Medicare Cost per Service (for a given cluster).

![image](https://user-images.githubusercontent.com/52474489/142740046-d3f0ee37-b264-4c1b-8444-832b9f144c1d.png)

*Each numeric value represents the average for each observation in the cluster (except Size)*

**Notable patterns**

- **Cluster 2** 
    - Biggest cluster
    - Highest average cost per service/total cost
    - Highest average coverage rate 
    - Highest freqeuncy
    - Almost entirely composed of instances associated with life threatening visits (Highest severity)


- All of the instances with minor ED visits were grouped in **Cluster 6**, and is entirely composed of minor ED visits. It also has the lowest costs and lowest coverage rate. It is also the smallest cluster. 



- **Cluster 0**, **Cluster 2**, and **Cluster 4** are composed entirely of Emergency Medicine services, with visit severity levels that are almost uniform across the cluster. To define these clusters
    - Cluster 2 - EM/Life threatening severity
    - Cluster 4 - EM/High severity
    - Cluster 0 - EM/Moderate severity
  - **Average cost per service/total cost, coverage rate, frequency of service all increase with increasing severity level of ED Visit**
  - Because all of these clusters are composed of entirely EM instances, this suggests that severity is very important in determining these features. 
  - These clusters are also very similar in size.

- We see a similar pattern as above for **Cluster 1**, **Cluster 9**, **Cluster 10**, except instead of EM, these are instances associated with services provided by Physician Assistants. 
    - Cluster 1 - PA/Life threatening severity
    - Cluster 10 - PA/High severity
    - Cluster 9 - PA/Moderate severity
    - **They exhibit similar patterns of increasing cost, coverage rate, and frequency with increasing severity.**
    - **It's important to note that the numbers are significantly lower than instances associated with EMs. This suggests that services provided by Emergency Medicine is more costly and more frequent. Total bill is more covered by Medicare in greater proportion than compared to Physician Assistant types.**
    
    
- Often times, instances in clusters will entirely compose of a certain Provider Type, especially if these types are not very represented in the dataset. 
    - This suggests that the clustering algorithm detected that some of these numeric metrics are determined by Provider Type.
    - We can make a rough ranking for Provider Type based on their associated features
        - **Most expensive lines of services: Emergency Medicine > Physician Assistant > Cardiology > Internal Medicine > Nurse Practitioners**
        - **Frequency of service per provider: EM > Internal Medicine> PA > NP > Cardiology**

- **Severity Level** is the greatest determinant of coverage rate, regardless of provider type
    - In the table below, sorted by coverage rate, the severity clearly trends down with decreasing coverage rate. One exception is **Cluster 9**, where with moderate severity, the coverage rate is the lowest. 

![image](https://user-images.githubusercontent.com/52474489/142740470-94e5eed7-df57-417d-82ed-45c677ed0e12.png)


# V. Conclusion<a class="anchor" id="5"></a>

- In terms of lines of services that are associated with Emergency Department visits, **expenditure and coverage rate depends on the severity level of the visit**.
    - This makes sense as severe procedures are likely to cost more and require more attention.
    - Severity level is the most distinct determinant factor of coverage rate. 
        - This is worth investigating. High cost makes sense as determinant of coverage rate as beneficiaries are likely to demand more coverage as cost of the service becomes very high. However, the results show this is not necessarily the case, as the above table shows that EM/life threatening services are covered at comparable rates as PA services, even though the former is much more expensive and frequently occurring. Investigating the processes in determining coverage rate and comparing with any existing benchmarks may be useful in making future decisions.
        
- Expenditure depends on the type of the provider. 
    - Provider determining expenditure makes sense as certain professions within the medical setting make more money than others (Doctors who practice emergency medicine make more than Nurse Practitioners). 
    
- Frequently occurring instances of services cost the most. 
    - Services associated with EM and that deal with life threatening issues are the most commonly occurring in their respective types, and are also both most expensive. 
    - The fact that frequency and cost per service are highly correlated with each other is something to note. 
    - Identifying in any waste or inefficiencies in insuring EM/Life threatening cases will be beneficial in terms of minimizing total Medicare expenditure.  

Possible future analysis can be done by using previous or future sets of data on the CMS website to compare how some of these patterns have changed over time. This may generate more insights about how the nature of the healthcare industry has shifted over time, over the course of different government officials taking office. 








