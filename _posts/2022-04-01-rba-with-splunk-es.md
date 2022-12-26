---
layout: default
title: "Risk-Based Alerting (RBA) with Splunk Enterprise Security"
summary: "In this blog, you will find out the abilities of Splunk ES Risk Framework and an idea of how to integrate Risk-Based Alerting into your SOC environment."
author: Merih Bozbura
image: /assets/img/blog/2022-04-01-rba-with-splunk-es-1.webp
date: 01-04-2022
tags: splunk risk-based-alerting rba risk-analysis siem
categories: Splunk

---

# Risk-Based Alerting (RBA) with Splunk Enterprise Security

Alert fatigue and false-positive results are the most common problems in a Security Operation Center (SOC) environment. The correlation searches are generally based on static thresholds, and if a particular point exceeds, an alert is triggered. No matter how we try to tune these searches, sometimes this can cause the noise to increase even more. As a result, we usually tend to suppress the correlation searches so that SOC analysts do not face alert fatigue, but this can reduce the coverage in some contexts without realizing it. At this point, one should consider implementing risk-based alerting since it is just not triggering several alerts based on a set of rules. Risk-based alerting aggregates risky events through time and attributes risk scores to the related risk object (assets or identities).

> "An effective enterprise risk management program promotes a common understanding for recognizing and describing potential risks that can impact an agency’s mission and the delivery of services to the public. Such risks include, but are not limited to, strategic, market, cyber, legal, reputational, political, and a broad range of operational risks such as information security, human capital, business continuity, and related risks. — [OMB Memorandum M-17–25](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-37r2.pdf)"

Splunk Enterprise Security (Splunk ES) is a security information and event management (SIEM) solution installed on Splunk Enterprise. It uses the search and correlation capabilities of Splunk Enterprise. Also, it populates out-of-the-box data models, dashboards, alerts, etc., to capture, monitor, and report on data from security devices, systems, and applications. You can find more information about Splunk ES [here](https://docs.splunk.com/Documentation/ES/7.0.0/User/Overview).

Another feature of Splunk ES is that it includes a Risk Framework. It allows us to find out “low and slow” attacks while contributing to the risk score for suspicious events that spread over the weeks or months. Besides, it helps us to deal with alert fatigue while doing this.

In this blog, you will find out the abilities of Splunk ES Risk Framework and an idea of how to integrate Risk-Based Alerting into your SOC environment.

### How Does Risk-Based Alerting Work?

#### Risk Framework

As you can see from the below image, Risk-Based Alerting starts by including the Risk adaptive response action to the correlation searches. Risk actions do not trigger the alerts directly. They contribute to the risk score of the related risk object. The risk score, risk object, and risk object type are the **risk modifiers** since they change the risk in Splunk Enterprise Security.

![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-1.webp)

These risk modifiers are stored in the **risk index**. A **system**, a **user**, and unrecognized devices, or users as **others** are the default risk objects; however, ES Admins can create risk objects like files or URLs.

![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-2.webp)


Risk Analysis Adaptive response is not the only way to do risk attributions, and you can create an **ad-hoc risk score** for any risk object. Also, threat objects can be included.


![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-3.webp)

Along with the risk score, we can take advantage of the [**risk factor**](https://docs.splunk.com/Documentation/ES/7.0.0/Admin/Createriskfactors) to adjust the risk score. Risk factors help to prioritize suspicious behavior since they increase the risk score. For instance, you can create or edit a risk factor that increases the risk score of a user who has a privileged or administrative identity.


![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-4.webp)


After risk attributions to the related objects, we can examine the risk index now. The source field represents the correlation search name. You can see the “Excessive Failed Logins” correlation search, its annotations, and risk modifiers in the risk index as an example. These out-of-the-box searches can be found in [Splunk Security Essentials](https://splunkbase.splunk.com/app/3435), [Splunk ES Content Update](https://splunkbase.splunk.com/app/3449), and [Splunk Enterprise Security](https://splunkbase.splunk.com/app/263) applications.


![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-5.webp)

The default risk incident rules in Splunk Enterprise Security use the risk data model and make risk analysis for us. These [out-of-the-box rules](https://docs.splunk.com/Documentation/ES/7.0.0/Admin/Usedefaultcorrelationsearches) are “ATT&CK Tactic Threshold Exceeded for Object Over Previous 7 days” and “Risk Threshold Exceeded for Object Over 24 Hour Period”. They create a [**risk notable events**](https://docs.splunk.com/Documentation/ES/7.0.0/User/Triagenotableevents#Use_custom_risk_notables_to_identify_threats) in the Incident Review dashboard.


![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-6.webp)

![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-7.webp)


These incidents can be assigned to a SOC analyst to defend, detect, and respond to the incidents. Also, you can select incidents that you want to examine and create [**investigations**](https://docs.splunk.com/Documentation/ES/7.0.0/User/Startaninvestigation) in the Splunk Enterprise Security application.


![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-8.webp)


The [**Risk Analysis dashboard**](https://docs.splunk.com/Documentation/ES/7.0.0/User/RiskAnalysis) is where you will see the overall attributions of the risk score. The dashboard allows us to see the most suspicious risky objects by sorting the risk scores. Also, you can find out the tendency of annotations. Such as, you may realize that all of the MITRE ATT&CK techniques/sub-techniques in the Risk Scores or Risk Modifiers by Annotations Panels under the same tactic. This can lead to notice if there are any risky events that we missed to examine, or maybe we have less coverage for that tactic or lack of data sources.


![screenshot](/assets/img/blog/2022-04-01-rba-with-splunk-es-9.webp)

To sum up, Risk-Based Alerting can help you chase slow and suspicious movements in your environment without drowning you in a sea of alerts.

Note: All of the events are shown in the screenshots are test events.

---


#### References:

- [Embark on Your Risk-Based Alerting Journey With Splunk ](https://www.splunk.com/en_us/pdfs/resources/solution-guide/embark-on-your-risk-based-alerting-journey-with-splunk.pdf)

- [Risk Management Framework for Information Systems and Organizations](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-37r2.pdf)

- [Risk Analysis framework in Splunk ES](https://dev.splunk.com/enterprise/docs/devtools/enterprisesecurity/riskanalysisframework/)

- [Risk Scoring](https://docs.splunk.com/Documentation/ES/7.0.0/User/RiskScoring)

- [Risk Objects](https://docs.splunk.com/Documentation/ES/7.0.0/Admin/Createriskobjects)

- [Ad Hoc Risk Entry](https://docs.splunk.com/Documentation/ES/7.0.0/User/Createadhocriskentry)

- [Risk Incident Rules](https://docs.splunk.com/Documentation/ES/7.0.0/Admin/Usedefaultcorrelationsearches)

- [Investigation](https://docs.splunk.com/Documentation/ES/7.0.0/User/Startaninvestigation)

- [Risk Factors](https://docs.splunk.com/Documentation/ES/7.0.0/Admin/Createriskfactors)

- [Risk Analysis](https://docs.splunk.com/Documentation/ES/7.0.0/User/RiskAnalysis)

- [Splunk Enterprise Security](https://www.splunk.com/pdfs/product-briefs/splunk-enterprise-security.pdf)

- [About Splunk Enterprise Security](https://docs.splunk.com/Documentation/ES/7.0.0/User/Overview)


---
