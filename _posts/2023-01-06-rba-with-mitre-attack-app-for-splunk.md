---
layout: default
title: "Risk-Based Alerting (RBA) with MITRE ATT&CK App for Splunk"
summary: "..."
author: Selim Seynur
image: /assets/img/blog/2023-01-06-rba-mitre.webp
date: 06-01-2023
tags: splunk risk-based-alerting rba risk-analysis siem
categories: Splunk

---

# Risk-Based Alerting (RBA) with MITRE ATT&CK App for Splunk

In this post, I'd like to review Risk-Based Alerting (RBA) in the context of the [MITRE ATT&CK App for Splunk](https://splunkbase.splunk.com/app/4617/) with a sample usage.  The goal is to reduce the number of alerts to a meaningful and manageable quantity by defining risky entities while tracking threats via the well-known and popular MITRE ATT&CK framework.  I'll start with an introduction of concepts and common terminology, which will be followed by a quick and (hopefully) simple demo of the idea.


### About RBA:
A great introduction to RBA and Splunk is already available at [Risk-Based Alerting (RBA) with Splunk Enterprise Security](https://medium.com/seynur/risk-based-alerting-rba-with-splunk-enterprise-security-430ada24c119) by [Merih Bozbura](https://mbozbura.medium.com/); this post explains how to implement RBA using Splunk Enterprise Security, set up risk scores for different types of security events, and use these scores to prioritize alerts and take appropriate action.  In addition, [Implementing risk-based alerting](https://lantern.splunk.com/Security/Product_Tips/Enterprise_Security/Implementing_risk-based_alerting) by [Justin Bull](https://www.linkedin.com/in/justinbull/) and [The Essential Guide to Risk Based Alerting (RBA)](https://www.splunk.com/en_us/form/the-essential-guide-to-risk-based-alerting.html) by [Haylee Mills](https://www.splunk.com/en_us/blog/author/hmills.html) provide excellent information on RBA methodology, maturity levels, and why it's needed as another solution layer on top of already existing SIEM ([Splunk Enterprise Security](https://splunkbase.splunk.com/app/263/)).  

As per the above links, RBA maturity journey can be summarized in 4 levels:
1. Level 0: Prepare & build
2. Level 1: Monitory and tweak
3. Level 2: Operationalize
4. Level 3: Implement & curate

It will not be an easy one-click solution but this iterative process will be worth the effort once (and if) implemented properly.  RBA provides an additional perspective to security monitoring; it uses risk scores to prioritize alerts based on the potential impact of the associated security event. These risk scores are typically calculated based on a variety of factors, such as the type of entity involved (importance of assets and identities), the severity of the event, and the potential consequences of the event. By taking an entity-based approach, RBA can help organizations more effectively identify and prioritize the most significant security threats, and take appropriate action to address them. Perhaps RBA can be considered as an entity-based approach as opposed to the classical SIEM rule-based alerting approach.


### About MITRE ATT&CK App:
The MITRE ATT&CK framework is a comprehensive model for understanding and analyzing cyber threats. It provides a common language and a common approach for describing and understanding the tactics, techniques, and procedures (TTPs) used by attackers. Integrating RBA with the MITRE ATT&CK framework can help organizations more effectively identify and prioritize the most significant security threats by taking into account the potential impact of different TTPs.  I previously wrote a series of posts, [Detecting Cyber Threats with MITRE ATT&CK App for Splunk](https://medium.com/seynur/detecting-cyber-threats-with-mitre-att-ck-app-for-splunk-a6627439a9e3), on how to utilize [MITRE ATT&CK App for Splunk](https://splunkbase.splunk.com/app/4617/).  As of this writing, the latest version (3.8.0) supports dynamic annotations to match MITRE ATT&CK techniques; this feature enables better integration with RBA-based correlation searches.



### MITRE ATT&CK Tactics and Techniques
I think a brief explanation of MITRE ATT&CK tactics and techniques would be useful here. Tactics are goals (why an adversary performs each action) and can be considered as an intermediate objective of the adversary.  Techniques are the means by which adversaries achieve their tactical goals (how an adversary performs each action).  Subtechniques provide more specific descriptions of adversarial behaviour, fundamentally subtechniques are the same as techniques for our purposes.

### Entities (Assets and Identities):
[Asset and Identity framework](https://dev.splunk.com/enterprise/docs/devtools/enterprisesecurity/assetandidentityframework/) is a key component for RBA to work properly since having up-to-date entity information with somewhat accurate importance levels determines the total risk score associated with those entities (e.g. risk factors for entities).  

![screenshot](/assets/img/blog/2023-01-06-assetandidentity-framework.webp)

Tuning some of the fields in these lookups turn out to be pretty important moving forward.  One approach we perform with our clients is to come up with a Data Flow Diagram (DFD) for the purposes of Splunk.  We start with a high-level approach to understanding the underlying organizational architecture, business model, data workflow, and importance levels of entities.  This information gives us a good starting point to tune existing out-of-the-box correlation searches, adjust risk scores/factors, and come up with new use cases pertaining to the organization's security goals.


### Demo
For this demo, I'll create 2 simple correlation (risk) rules by generating commands that populate the `risk` index.  In addition, I'll create a single RIR (Risk Incident Rule) that performs an aggregated search on the `risk` index and populates the `notable` index so that both MITRE ATT&CK App and ES Incident Review can utilize the results based on entity analysis.  Instead of each triggered correlation search, we will be focusing on risky entities and possible activities along the kill chain according to MITRE ATT&CK framework.

Here's a quick snapshot of the activity:

![screenshot](/assets/img/blog/2023-01-06-rba-mitre.webp)

2 rules are created with the following information and they both utilize "Risk Analysis" Adaptive Response Action to increment `src` and `user` risk scores.  Here are the relevant parts from `savedsearches.conf`:

```properties
[Threat - Demo1 - Rule]
action.correlationsearch.annotations = {"mitre_attack":["T1133","T1110","T1110.001"]}
action.correlationsearch.enabled = 1
action.correlationsearch.label = Demo1
...
action.risk = 1
action.risk.param._risk = [{"risk_object_field":"src","risk_object_type":"system","risk_score":20},{"risk_object_field":"user","risk_object_type":"user","risk_score":15},{"threat_object_field":"app","threat_object_type":"app_info"}]
action.risk.param._risk_message = Demo1 Risk
...
search = | makeresults | eval src="10.10.1.1", dest="10.20.40.50", user="demo1_user", action="failure", tag="authentication", app="demo_app"

[Threat - Demo2 - Rule]
action.correlationsearch.annotations = {"mitre_attack":["T1059.008","T1040"]}
action.correlationsearch.label = Demo2
...
action.risk = 1
action.risk.param._risk = [{"risk_object_field":"src","risk_object_type":"system","risk_score":10},{"risk_object_field":"user","risk_object_type":"user","risk_score":5},{"threat_object_field":"app","threat_object_type":"app_info"}]
action.risk.param._risk_message = Demo2 Risk
...
search = | makeresults | eval src="10.10.1.1", dest="10.20.30.40", user="demo2_user", action="allowed", tag=split("network,communicate",","), app="demo_app"

```

Both these rules will populate the `risk` every few minutes.  Since we don't have anything on the `notable` index, we will not be alerted on any of these events as an analyst.  In order to be notified, I cloned "ATT&CK Tactic Threshold Exceeded For Object Over Previous 7 Days" correlation search and named it as "RIR - Demo - ATT&CK Tactic Threshold Exceeded For Object Over 12 Hours"

I updated the earliest time to `-730m@m` to match 12 hours and the threshold counts in the search as follows (last line):
```
...
mitre_tactic_id_count >= 2 and source_count >= 1
```

![screenshot](/assets/img/blog/2023-01-06-rir-rule-1.webp)

Note that this rule does not have any static MITRE ATT&CK annotations specified, it simply gathers this information from the other rules (namely Demo1 and Demo2) and populates the annotations dynamically.  Here's the list of all 3 demo rules we have.

![screenshot](/assets/img/blog/2023-01-06-demo-rules.webp)

MITRE ATT&CK views will be populated when our demo RIR triggers and populate the `notable` index.  We can see multiple techniques being triggered and when we do a drill-down, we can see our Demo - RIR rule with both `user` and `src` entity information.  In this case, instead of having to dive into multiple rules with possible false positives, we can focus on risky entities and investigate further.

![screenshot](/assets/img/blog/2023-01-06-mitre-matrix.webp)

![screenshot](/assets/img/blog/2023-01-06-incident-review.webp)

A similar view is also provided within MITRE ATT&CK App.  Triggered Tactics & Techniques view displays 2 Sankey diagrams for analyzing kill-chain based on the entity (asset or identity).  This visualization can be powerful for investigations to determine the activity of risky objects and perhaps new alerting rules can be generated from this as well (e.g. A technique under "Privilege Escalation" is triggered for the same user that had previously triggered one or more techniques under "Initial Access").

![screenshot](/assets/img/blog/2023-01-06-mitre-triggered-tactics.webp)

For example, in our demo rules, we have both `demo1_user` and `demo2_user` firing correlation searches and populating the `risk` index.  Our Demo - RIR rule is looking for risky objects that trigger 2 or more tactics within the past 12 hours and only one of the users falls under this category.  Such a view enables an analyst to focus directly on the high-risk object (`demo1_user` in this case) for further investigation.


# Summary
Splunk ES and ESCU come with many useful correlation searches and while most of these alerts need to be triggered the end users (analysts) do not need to be notified by all of them.  Utilizing RBA and MITRE ATT&CK framework in order to determine risky activity instead of focusing on rule-based alerts can help manage false positives and enable teams to focus on organizational security goals.  I tried to provide a very simple example with the demo above on how one might utilize both RBA and MITRE ATT&CK app.  Note that this is another layer of detection mechanism for your data-driven security journey and should definitely help not only with SIEM-specific needs but also with SOAR implementations and related teams.


> More than 40% of organizations receive 10,000+ alerts per day, with 50%+ of those alerts turning out to be false positives.
<br/> - [The Essential Guide to Risk Based Alerting (RBA)](https://www.splunk.com/en_us/form/the-essential-guide-to-risk-based-alerting.html)

Risk-Based Alerting (RBA) is a method for prioritizing security alerts based on risk scores that are calculated using a variety of factors such as the type of entity involved, the severity of the event, and the potential consequences of the event. This helps organizations more effectively identify and prioritize the most significant security threats. It is very likely that teams will need to iteratively review, tune, and improve their RBA implementations.  Especially integrating RBA with the MITRE ATT&CK framework to effectively identify and prioritize threats by taking into account the potential impact of different tactics and techniques.  Overall, the best approach for integrating RBA with the MITRE ATT&CK framework will depend on the specific needs and resources of the organization but I feel that adding this

---


#### References:

- [MITRE ATT&CK](https://attack.mitre.org/)
- [Risk-Based Alerting (RBA) with Splunk Enterprise Security](https://medium.com/seynur/risk-based-alerting-rba-with-splunk-enterprise-security-430ada24c119)
- [Embark on Your Risk-Based Alerting Journey With Splunk ](https://www.splunk.com/en_us/pdfs/resources/solution-guide/embark-on-your-risk-based-alerting-journey-with-splunk.pdf)
- [Risk Analysis framework in Splunk ES](https://dev.splunk.com/enterprise/docs/devtools/enterprisesecurity/riskanalysisframework/)
- [Implementing risk-based alerting](https://lantern.splunk.com/Security/Product_Tips/Enterprise_Security/Implementing_risk-based_alerting)
- [The Essential Guide to Risk Based Alerting (RBA)](https://www.splunk.com/en_us/form/the-essential-guide-to-risk-based-alerting.html)
- [MITRE ATT&CK App for Splunk](https://splunkbase.splunk.com/app/4617/)
- [Splunk Enterprise Security](https://splunkbase.splunk.com/app/263/)
- [Detecting Cyber Threats with MITRE ATT&CK App for Splunk](https://medium.com/seynur/detecting-cyber-threats-with-mitre-att-ck-app-for-splunk-a6627439a9e3)
- [Asset and Identity framework](https://dev.splunk.com/enterprise/docs/devtools/enterprisesecurity/assetandidentityframework/)

---
