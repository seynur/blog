---
layout: default
title: "Detecting Cyber Threats with MITRE ATT&CK App for Splunk — Part 2"
summary: "In this part of the blog series the goal is to utilize MITRE ATT&CK App for Splunk and associate custom/new correlation ..."
author: Selim Seynur
image: /assets/img/blog/2020-04-17-mitre-attack-maprule-1.webp
date: 17-04-2020
tags: splunk mitre siem
categories: Splunk

---

# Detecting Cyber Threats with MITRE ATT&CK App for Splunk — Part 2

![screenshot](/assets/img/blog/2020-04-17-mitre-attack-maprule-1.webp)

In this part of the blog series the goal is to utilize [MITRE ATT&CK App for Splunk](https://splunkbase.splunk.com/app/4617/) and associate custom/new correlation searches with MITRE ATT&CK techniques. This way we can develop rules that are pertinent to our environment and requirements which also align with MITRE ATT&CK Framework.

Part 1 of this blog post can be found [here](/splunk/2020/03/12/detecting-cyber-threats-with-mitre-attack-app-for-splunk-part1).

So, now you have the setup and as it is very likely that you have many other correlation searches that are specific to your environment. Some of these correlation searches are applicable to MITRE ATT&CK techniques/tactics and it would be nice to view those within this context.

## How do we associate a rule with a technique?

We initially started this rule-to-technique mappings with Analytic Stories (more on that below) but based on usage and customer feedback it was much more efficient to do the mappings with a simple lookup. Hence, we have 2 different approaches for associating a rule with a technique.

1. The lookup file:
We added a simple lookup file (`mitre_user_rule_technique_lookup.csv`) that has the `rule_name` and `technique_id` list (separated by space). You can always edit this file manually:

```
rule_name, technique_id "MyApp - My Rule 1 - Rule" , "T1078"
```

However, since this is not ideal and prone to human error, we created a view that does this for you.

### Map Rule to Technique View:
Go to "**Configuration** -> **Map Rule to Technique**" from MITRE ATT&CK Framework App menu. Initially it should appear something similar to following image.

![screenshot](/assets/img/blog/2020-04-17-mitre-attack-maprule-2.webp)

There are 2 main panels in this view: the left panel shows you the contents of the lookup file (existing mappings) and the right panel shows you what you have added once you hit the “Submit” button.

Next, select the rule name form Rule Name dropdown menu item and associate with technique IDs from **MITRE ATT&CK Technique** multi-select then hit **Submit**. Both panels will be updated accordingly.

![screenshot](/assets/img/blog/2020-04-17-mitre-attack-maprule-3.webp)

And that’s it! Depending on your lookup generator search schedules (default is at midnight daily), you may need to run "**Configuration -> Setup**" manually in order to repopulate the lookup files.

Note: I personally feel that the “setup” is not the best name for that view and it will probably change in the upcoming versions since we’re planning on an actual configuration setup to be in place

2. The Analytic Story:
If you’d like to add more information around your searches and perhaps make them a part of a specific use-case/story, it would be a good idea to utilize [Analytic Stories](https://docs.splunk.com/Documentation/ESSOC/1.0.53/stories/Intro) within Splunk.

Splunk ES Content Update comes with [Analytic Stories](https://docs.splunk.com/Documentation/ESSOC/1.0.53/stories/Intro). Here’s the definition from Splunk documentation:

> Splunk Analytic Stories are security guides that provide you with tactics, techniques, and methodologies to assist with detection, investigation, and response.

Within each analytic story, we can review the description and details that provide insights into the security methodology in order to detect, investigate, and respond to incidents. Splunk ES Content Update comes with various out-of-the-box searches to achieve this goal and also comes with annotations for applicable searches. These annotations provide mappings for frameworks, such as CIS controls, Kill Chain phase, and as you may have already guessed MITRE ATT&CK.

MITRE ATT&CK App for Splunk utilizes these annotations in order to populate the lookups it uses. You will need to create a new Analytic Story (or use an existing one), add your search to that Analytic Story and provide mitre_attack annotation and appropriate mapping (Technique Name) for this to work. Step by step instructions for this can be found within the documentation:

[https://seynur.github.io/DA-ESS-MitreContent/user_guide.html#match-with-analytic-story](https://seynur.github.io/DA-ESS-MitreContent/user_guide.html#match-with-analytic-story)

What’s Next?
In this part of the series I wanted to briefly introduce how to add mappings for correlation searches and MITRE ATT&CK framework techniques for MITRE ATT&CK App for Splunk. This way users are not limited to what’s provided out-of-the-box with Splunk but can also add their custom rules to match with the framework. Moving forward, we have some new features (and bug fixes :) ) lined up and I hope to share some more information in upcoming months as we release them.

Part 3 of the series can be found [here](/splunk/2020/06/10/detecting-cyber-threats-with-mitre-attack-app-for-splunk-part3).

---

#### References:

- [MITRE ATT&CK](https://attack.mitre.org/)
- [Splunk Enterprise](https://www.splunk.com/)
- [Splunk Enterprise Security](https://splunkbase.splunk.com/app/263/)
- [Splunk ES Content Update](https://splunkbase.splunk.com/app/3449/)
- [MITRE ATT&CK App for Splunk](https://splunkbase.splunk.com/app/4617/)
- [Documentation for MITRE ATT&CK App for Splunk](https://seynur.github.io/DA-ESS-MitreContent/)
- [Splunk Analytic Story](https://docs.splunk.com/Documentation/ESSOC/1.0.53/stories/Intro)

---