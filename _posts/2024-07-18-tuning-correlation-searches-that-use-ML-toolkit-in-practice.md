---
layout: default
title: "Tuning correlation searches that use ML toolkit in practice"
summary: "This study aims to ensure that the correlation rules are effectively implemented using MLTK in practice without any errors."
author: Öykü Can Şimşir
image: /assets/img/blog/2024-07-18-MLTK-scheme.webp
date: 18-07-2024
tags: splunk MLTK machine-learning-tool correlation-searches
categories: Splunk

---

# Tuning correlation searches that use ML toolkit in practice

In a rapidly evolving cyber security environment where new threats are encountered every day, effective threat detection and responses to them depend on advanced correlation research. These searches become exponentially more powerful when paired with machine learning (ML) capabilities. Splunk Machine Learning Toolkit (MLTK) offers powerful tools for improving correlation searches, but optimizing their performance requires a nuanced approach. This product, which operates under certain conditions and limits, may sometimes need practical regulation. This article explores practical strategies for tuning correlation searches using Splunk MLTK. Also, if you want to learn more about [Enterprise Security](https://medium.com/r/?url=https%3A%2F%2Fdocs.splunk.com%2FDocumentation%2FES%2F7.3.1%2FUser%2FOverview) and [MLTK](https://medium.com/r/?url=https%3A%2F%2Fdocs.splunk.com%2FDocumentation%2FMLApp%2F5.4.1%2FUser%2FWelcometoMLTK) you can check official documentation pages.

Below, you will find a list of the use cases covered in this blog. Alternatively, you can directly access the preferred method using the provided links. I hope you enjoy reading this article and find it helpful.

* [Correlation Search : Substantial Increase In Intrusion Events](#substantial-increase-in-intrution-events-usecases) 

### Why Correlation Searches?
[Correlation searches](https://medium.com/r/?url=https%3A%2F%2Fdocs.splunk.com%2FDocumentation%2FES%2F7.3.1%2FTutorials%2FCorrelationSearch) are designed to identify relationships and to find all patterns between different events and not only the same data source but also different sources. In this way, security teams can detect complex attack patterns and potential security incidents that might otherwise go unnoticed. The integration of machine learning amplifies this capability by enabling the system to learn from data patterns and improve its detection accuracy over time. As the official Splunk ML app, the Splunk Machine Learning Toolkit can be used to create correlation searches and relevant SPL queries.

### Briefly Splunk Machine Learning Toolkit
Different types of algorithm categories such as Anomaly Detection, Classifiers, Clustering Algorithms, Cross-validation, Feature Extraction, Preprocessing, Regressors, Time Series Analysis, and Utility Algorithms come with the MLTK app, as [documented](https://medium.com/r/?url=https%3A%2F%2Fdocs.splunk.com%2FDocumentation%2FMLApp%2F5.4.1%2FUser%2FAlgorithms). Those familiar with using machine learning methods for calculations understand that the system's load is directly affected by factors such as the duration of fit and predict periods, the number of groups, data, and variables used. When selecting the method for SPL, the effectiveness of the chosen ML method to calculate the desired outcome is not the primary criterion. Sometimes, this method needs to be simplified to be able to calculate the results.

The Splunk MLTK [documentation](https://medium.com/r/?url=https%3A%2F%2Fdocs.splunk.com%2FDocumentation%2FES%2F7.3.0%2FAdmin%2FMLTKtroubleshooting) page mentions some limitations. For example, 1024 is the default group limit of the density function. If your group exceeds this, Splunk will return an error such as "*DensityFunction model: The number of groups cannot exceed &lt;abc&gt;; the current number of groups is &lt;xyz&gt;.*", and this error can be found in the mlspl.log file. This limit can be changed in the configuration files, but that's not the main point I want to highlight. Splunk decided on this limitation to prevent potential server crashes. As previously discussed, ML calculations come with heavy loads, which may overwhelm Splunk. Therefore, changing the default amount can be dangerous if your system fundamentally cannot handle these calculations.

On the other hand, there may be a warning of "*Too few training points in some groups will likely result in poor accuracy for those groups.*" when fitting the MLTK model. This warning indicates that certain signature types lack sufficient data points for accurate calculation, leading to poor accuracy due to statistical limitations.

In order to address the points mentioned above, I would like to share some relatively simple, practical solutions from our previous work related to Enterprise Security Use-Cases.

<div id="substantial-increase-in-intrution-events-usecases"></div>

### Correlation Search : Substantial Increase In Intrusion Events

#### Original
The correlation search was extracted from the SA-NetworkProtection add-on and is described as *"Alerts when a statistically significant increase in a particular intrusion event is observed."*. The SPL of the search can be found below. This search is executed for last 1 hour.

````
| tstats `summariesonly` count as ids_attacks,values(IDS_Attacks.tag) as tag from datamodel=Intrusion_Detection.IDS_Attacks by IDS_Attacks.signature 
| `drop_dm_object_name("IDS_Attacks")` 
| `mltk_apply_upper("app:count_by_signature_1h", "high", "ids_attacks")`
````

Model (MLTK: Network - Event Count By Signature Per Hour - Model Gen) can be seen below, generating a normal distribution with yesterday's *"ids_attacks"* values at one-hour intervals. The ***mltk_apply_upper*** macro is used in the correlation search to call the model and calculate the upper bound with a 95% confidence interval.

````
| tstats `summariesonly` count as ids_attacks from datamodel=Intrusion_Detection.IDS_Attacks by _time,IDS_Attacks.signature span=1h 
| `drop_dm_object_name("IDS_Attacks")` 
| fit DensityFunction ids_attacks by signature partial_fit=true dist=norm into app:count_by_signature_1h
````

Since correlation and related searches are easily explained, alternative calculations can be mentioned instead of default ones. The outcome of these calculations can vary based on the statistical expertise of the individuals conducting the calculations, the complexity of the data, and how the data changes over time. It is not accurate to assume that the data follows a specific distribution without conducting statistical tests in theorical. Each distribution has its own set of assumptions and tests. In practical terms, especially in fields like cybersecurity, analyzing data can be time-consuming and may delay the triggering of alarms. Experts generally prefer to set a fixed upper/lower bound to find outlier data. However, with rapidly developing technology, this approach is becoming insufficient for many areas. Even the advancement of AI and technology cannot guarantee the detection of outliers at the desired speed, especially in big data, as not everyone has access to high technology yet. Therefore, it would be beneficial for the security of alarms and controls to switch to methods that are not static but do not impose too much load on the system.

As previously mentioned, under some circumstances, it will not be possible to use correlation searches containing MLTK distribution model calculations directly. Below, you will find correlation and model searches for "*Brute Force Access Behavior Detected Over One Day*". Encountered issues when running this correlation search due to more than 1024 signature groups in Splunk, with over 50,000 events and 1024 signature types, in this study (**Figure 1**). 

| ![screenshot](/assets/img/blog/2024-07-18-mltk-error-info.webp) |
|:--:| 
| *Figure 1:* The information about exceeding 1024 signature groups and error images. |


#### New
The search was executed with time filter both from 4 hours ago to 1 hour ago, 8 days ago to 1 day ago and from 31 days ago to 1 day ago with hourly data, resulting in **Figure 2** all together as a table. Also, all spls, and all results can be found below (**Figure 3**, **Figure 4**, **Figure 5**) individually. There is a slight difference between the upper bound values in both searches. It cannot be said which SPL is way better than the other due to a lack of information about different causes such as the behavioral pattern of the events or the importance of the event's signatures. For instance, "Custom Attack Type 1001" is counted as 1 in the last 1 hour.  All upper bound values are valid, except the one that searches the last 4 hours of data, for which there is no data for that signature in this time period. If the last 4 hours filter is selected, this signature will not be seen in this table. Besides, there are alerts for 101, 3, 2 individual signatures for 4 hours, 7 days, and 30 days filters respectively. This is the main point that I want to share.

| ![screenshot](/assets/img/blog/2024-07-18-new-filter-all-in-one.webp) |
|:--:| 
| *Figure 2:* Calculating upper bounds for each signature and different time ranges (4 hours, 7 days, 30 days). |

| ![screenshot](/assets/img/blog/2024-07-18-new-filter-4h.webp) |
|:--:| 
| *Figure 3:* Calculating upper bounds for each signature for 4 hours. |


| ![screenshot](/assets/img/blog/2024-07-18-new-filter-7d.webp) |
|:--:| 
| *Figure 4* Calculating upper bounds for each signature for 7 days. |


| ![screenshot](/assets/img/blog/2024-07-18-new-filter-30d.webp) |
|:--:| 
| *Figure 5:* Calculating upper bounds for each signature for 30 days. |

There can be different statistical analysis before decide which SPL will be used. There may be developed different filters according to event analysis. 

### Conclution
To sum up, the MLTK tool is designed for analyzing data distribution and performing various data analyses for a specified period. If additional configurations are not defined, the MLTK will assume that the specified event has a normal distribution and perform calculations accordingly. However, in cases where quick action is required due to constraints of the MLTK tool, simpler statistical methods may provide better results. Before analysis, it is important to determine the desired methods by analyzing events and, if possible, using methods such as EDA to examine events in detail and calculate filters for a time period. This can improve the performance of both analysts reviewing the alarms and the SPLs run in Splunk.



---


## References:

- [[1]](https://docs.splunk.com/Documentation/ES/7.3.2/Admin/MLTKsearches#SA-NetworkProtection) Splunk. (2024). *Machine Learning Toolkit Searches in Splunk Enterprise Security.* *https://docs.splunk.com/Documentation/ES/7.3.2/Admin/MLTKsearches#SA-NetworkProtection*
- [[2]](https://splunkbase.splunk.com/app/2890) Splunk. (2024). *Splunk Machine Learning Toolkit*. *https://splunkbase.splunk.com/app/2890*
- [[3]](https://docs.splunk.com/Documentation/MLApp/5.4.1/User/AboutMLTK) Splunk. (2024). *About the Splunk Machine Learning Toolkit.* *https://docs.splunk.com/Documentation/MLApp/5.4.1/User/AboutMLTK*



---