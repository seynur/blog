---
layout: default
title: "Upgrading Splunk ES from 7.x to 8.1.1 - Our Journey and Lessons Learned"
summary: "Upgrading from Splunk ES 7.x to 8.1.1 isn’t just a version change — it’s a structured transition that, with the right preparation and automation, can be smooth, controlled, and operationally safe."
author: Öykü Can Şimşir
image: /assets/img/blog/2026-02-18-upgrading-es-from-es7x-to-es811.webp
date: 18-02-2026
tags: splunk es upgrade 8.1.1
categories: Splunk 

---

# Upgrading Splunk ES from 7.x to 8.1.1 – Our Journey and Lessons Learned
## 🌸 1. Overview
### 1.1. Why We Decided to Upgrade (and What This Blog is All About)

As February 2026 rolled around, many of us felt the push to upgrade since support for Splunk Enterprise Security 7.3 was about to wrap up on February 28, 2026 [[1]](https://www.splunk.com/en_us/legal/splunk-software-support-policy.html). It was time to take action!

If you're looking for a solid overview of upgrading to ES 8.0.x, you’ll want to check out Splunk's guide on [Splunk Lantern](https://lantern.splunk.com/Get_Started_with_Splunk_Software/Upgrading_to_Enterprise_Security_8.0.x_-_Overview?mt-learningpath=es8x). It covers everything from differences to expectations and all the changes in version 8.x.

But don't worry; this blog isn't just going to be a dry list of commands for the upgrade! Instead, we’re excited to share the insights that made a real difference for us during this process:

- Pre-flight checks (health assessments, readiness evaluations, and those important "are we really ready to upgrade today?" conversations)
- Operational controls (what we looked at before and after the upgrade, and how we checked that everything was running smoothly)
- Handy scripts that helped us cut down on manual work
- The little hiccups that popped up only during the actual upgrades (like configuration conflicts and those pesky messages saying, "`Failed to migrate the following ES detections: [...`")

While Splunk has documented the upgrade steps well, what really helps is having a "real-life upgrade kit" to go along with them. That’s the scoop we’re here to share with you!

### 1.2. Why ES 8.x Creates Extra Work (and Why It Catches People Off Guard)

One of the biggest changes in ES 8.x is that Risk-Based Alerting (RBA) becomes a key focus. With this shift, the cleanliness of our correlation search metadata becomes super important!

Starting with ES 8.0, some fields we used to think of as "nice to have" are now must-haves for event-based detections, especially the risk modifier and risk message. If they’re missing (or if the risk object field isn’t there in the results), you might find that findings/notables don’t get created after the upgrade. There are also a few new fields to keep in mind, which we’ll touch on later.

So, yes, upgrades can bring about an extra workload:
- Lots of correlation searches to manage
- A bunch of parameters to check and complete
- Consistency requirements that we didn't have to worry about in older versions 

And if you’re looking at the direction of ES 8.3+, Splunk is diving deeper into Entity Risk Scoring and broader risk workflows. That means embracing the "RBA mindset" is becoming even more crucial!

### 1.3. What We Did to Make It Easier (What You’ll Learn Next)

Instead of manually tweaking hundreds of detections, we decided to approach this like an upgrade project:

1. **Control Points First**  
   We figured out what "healthy and ready" looks like before upgrading, and we defined what "success" means on the other side of it.

2. **Treating Correlation Searches as Data**  
   We gathered configurations, checked them out in bulk, identified missing mandatory parameters, and made consistent updates.

3. **Using Simple Scripts for Filling Required Fields**  
   We didn’t go for fancy tools-just some practical scripts that:
   - Identify any missing mandatory fields (like risk message and risk modifiers)
   - Optionally come up with sensible defaults
   - Steer clear of overriding anything that’s already customized (unless we intentionally want to)

4. **Documenting the "Gotchas"**  
   The upgrade UI might look smooth, but the internal logs tell the real story. We’ll point out where to look and what patterns to pay attention to.

That’s the heart of this blog: it’s less about "how to click upgrade," and more about "how to upgrade safely without getting swamped by post-upgrade fixes."

---

## 2. Pre-Upgrade 

### 2.1. Health Check (Keep It Simple, But Don’t Skip It)

Before we dive into the upgrade, it's important to take a step back and ask ourselves: Is our environment stable? Instead of jumping straight into commands, we wanted to make sure everything is nice and calm-boring, even!

Here’s what we’re looking for:
- No active issues to worry about.
- No strange warnings popping up.
- No half-broken features that we've learned to live with.

Keep in mind that once we upgrade, any existing problem can suddenly become "an upgrade issue." 

Depending on your setup, we can check things from:
- The Monitoring Console (MC) in clustered environments 
- The standalone server (or its local MC) in single-node setups

We’re not getting into the nitty-gritty here; we simply make sure that:
- The overall platform health looks good.
- The scheduler is in a comfortable place.
- There aren’t too many skipped searches.
- Search Head Cluster activity (if applicable) is stable.
- The KV Store has no obvious errors or replication hiccups.
- The internal logs are clean and free of unusual warnings.

For those using a Search Head Cluster (SHC), it’s especially important to ensure everything is calm:
- No rolling restarts.
- No captain instability.
- No pending bundle replication.

A calm cluster before the upgrade means a better chance of staying calm afterward!

Speaking of the KV Store, we want to do a quick sanity check-there’s no need for a deep dive. Just confirm that we don’t have any active errors or unhealthy states. The Enterprise Security (ES) component depends heavily on the KV Store, so let’s catch those issues early!

Another crucial step too often overlooked: double-check your app versions and compatibility between ES and Splunk before hitting "upgrade." You want to ensure that your target ES version works with your version of Splunk Enterprise, as well as any dependent apps. You can find the official Splunk-ES compatibility matrix here:
[Compatibility Matrix](https://help.splunk.com/en/splunk-enterprise/release-notes-and-updates/compatibility-matrix/splunk-products-version-compatibility/splunk-products-version-compatibility-matrix).

We’re keeping it simple here and not getting into specific monitoring commands, as Splunk Lantern has plenty of great resources to guide you through the preparation. Check it out for a full checklist:
[Upgrade Overview for Enterprise Security 8.0.x](https://lantern.splunk.com/Get_Started_with_Splunk_Software/Upgrading_to_Enterprise_Security_8.0.x_-_Overview).

At this stage, we’re not looking to reinvent anything-we just want to eliminate as many variables as possible. If something doesn’t work right after the upgrade, we can confidently say:
> It was running smoothly before we touched it!

Next, we’ll talk about some key changes that will really affect operations in ES 8.x: embracing Risk-Based Alerts (RBA), the must-have risk parameters, and how we prepared our correlation searches before the upgrade.

### 2.2. Pre-Upgrade Operations (Backups, Configuration Prep & Index Readiness)

Now that our health check is done and our environment is calm, it’s time to focus on an important step we absolutely can’t skip: *pre-upgrade operational preparation*. Think of this as your *rollback insurance*-definitely not optional!

1. **Backup Strategy**
   
   Before upgrading ES, it’s a good idea to back up each Search Head individually. Make sure to back up:
   - **KV Store**
   - **`$SPLUNK_HOME/etc` directory**

   This step is important because ES will make changes to:
   - Correlation searches
   - Risk configurations
   - Notable parameters
   - KV-backed objects
   - Internal app configurations

   Having a clean snapshot from before the upgrade makes a world of difference if anything goes haywire afterward. And if you run into tricky errors related to knowledge objects or the KV Store, Splunk’s documentation has great resources to help!

2. **SHC-Deployer/Search Head Configurations**

   Depending on your architecture, it’s a smart move to make a few controlled adjustments before installing ES 8.1. 

   If you’re in an SHC environment, these configurations should go into the `server.conf` and `web.conf` files on the **deployer**; otherwise, put them on the **search head**:

   ```ini
   # server.conf
   [httpServer]
   max_content_length = 5000000000
   ```

   ```ini
   # web.conf
   [settings]
   splunkdConnectionTimeout = 300
   max_upload_size = 2048
   ```

2. **New Index Requirements (Don’t Forget the Indexers)**

   Splunk Enterprise Security (ES) version 8.x might bring some new indexes your way, especially when it comes to:

   - Risk
   - Findings
   - Summary-related data

   Before we upgraded our Search Heads, we took a few important steps to ensure a smooth transition:

   - We made sure all required indexes were set up on the indexers.
   - If we were using an indexer cluster, we confirmed that cluster replication was complete.
   - We checked that the indexes were both searchable and writable.

   It’s key to know that if you upgrade ES first and then realize an index is missing, it can lead to correlation searches either failing without notice or causing some puzzling errors. So, being prepared on both the indexer side and the Search Head side is super important!

   In our situation, we simply added the necessary indexes to our indexers. Just remember to double-check your versions and specific needs. You can find the indexes list below.

   ```
   ## missioncontrol
   ###### MC aux incidents ######

   [mc_aux_incidents]
   repFactor = auto
   coldPath = $SPLUNK_DB/mc_aux_incidents/colddb
   homePath = $SPLUNK_DB/mc_aux_incidents/db
   thawedPath = $SPLUNK_DB/mc_aux_incidents/thaweddb

   ###### MC artifacts ######

   [mc_artifacts]
   repFactor = auto
   coldPath = $SPLUNK_DB/mc_artifacts/colddb
   homePath = $SPLUNK_DB/mc_artifacts/db
   thawedPath = $SPLUNK_DB/mc_artifacts/thaweddb

   ###### MC investigations ######

   [mc_investigations]
   repFactor = auto
   coldPath = $SPLUNK_DB/mc_investigations/colddb
   homePath = $SPLUNK_DB/mc_investigations/db
   thawedPath = $SPLUNK_DB/mc_investigations/thaweddb

   ###### MC events ######

   [mc_events]
   repFactor = auto
   coldPath = $SPLUNK_DB/mc_events/colddb
   homePath = $SPLUNK_DB/mc_events/db
   thawedPath = $SPLUNK_DB/mc_events/thaweddb

   ###### MC old incidents ######

   [mc_incidents_backup]
   repFactor = auto
   coldPath = $SPLUNK_DB/mc_incidents_backup/colddb
   homePath = $SPLUNK_DB/mc_incidents_backup/db
   thawedPath = $SPLUNK_DB/mc_incidents_backup/thaweddb

   ## SA-ContentVersioning

   [cms_main]
   homePath   = $SPLUNK_DB/cms_main/db
   coldPath   = $SPLUNK_DB/cms_main/colddb
   thawedPath = $SPLUNK_DB/cms_main/thaweddb

   ```

---

## 3. Performing the Upgrade (We’re All Set!)

Fantastic job getting to this point! Here’s a quick recap of what we’ve accomplished together:

- Completed health checks ✅
- Taken backups 📦
- Ensured the indexers are ready 🛠️
- Reviewed correlation searches 🔍

Now it’s time for the exciting part: the upgrade! 🎉

1. **Upgrading with a Search Head Cluster (SHC)**

If you’re working in a clustered environment, we’ll use the SHC Deployer to handle the upgrade, and it’s easier than you might think!

You have a couple of fun options:

- **Option 1:** Upload and install the ES 8.1 app directly on the Deployer.
- **Option 2:** If you’re feeling adventurous, you can also use the CLI for the installation.

Once you’ve uploaded the app, it will be found in `$SPLUNK_HOME/etc/shcluster/apps`. After that, just push the bundle and let the magic happen! ✨

Once the bundle is pushed:

- Take a moment to let the cluster catch its breath. 💤
- Give replication some time to do its thing.
- Let the captain of the ship (that’s your cluster leader!) coordinate everything smoothly. ⚓

With all the prep work you’ve done, the upgrade should go off without a hitch! So, no need to rush—grab a cup of coffee or tea and relax while you wait! ☕️

2. **If You Don’t Have a Cluster**

No worries at all—upgrading is super simple and straightforward for you! You can:

- Upgrade using the friendly UI, or
- Opt for the CLI if you prefer that route.

After you’ve installed the upgrade, you can:

- Sit back and let ES finish its internal upgrade routines.
- Keep an eye on the `_internal` logs.
- Make sure everything is initialized properly. 

And that’s really all there is to it! You’re doing absolutely great! 🎈

---

## 4. After Upgrade - The First Reality Check

Now we’re entering an interesting phase! 🧐

Before we dive into fixing any detections, our priority will be ensuring that the system itself is fully operational. So here’s what we’ll want to check:

- Look for any messages indicating issues, except for the `Failed to migrate the following ES detections: [` error.
- Ensure that the ES app loads without any errors.
- Make sure Mission Control (the new name for the Incident Review panel) opens smoothly and shows old triggered events. 🎉
- Confirm that risk, notable, and other new indexes are searchable.
- Check that correlation searches are enabled and scheduled. 
- Verify there are no critical errors in `_internal`.
- If you’re in a clustered environment (SHC), check that everything is stable and the captain is healthy. 🚦
- Watch out for any bundle replication issues.
- Keep an eye on KV Store issues.
- Ensure all ES-related apps are on the same version.
- Remember to review the [ES configuration health](<insert-link>) dashboards.
- Check for any modular input errors with this command:

  ```
  index=_internal sourcetype=splunkd log_level=ERROR component=ModularInputs 
  | table _time host source component log_level event_message
  | sort -_time
  ```

Give the system a little time to breathe. ES has some background initialization tasks to complete after the upgrade, especially in SHC environments. Rushing into configuration changes too quickly can create some confusion.

If everything looks stable and operational, only then do we move on to reviewing detections! 🙌

*Even if all seems clean*, you might see a message in the UI that says:
```
Failed to migrate the following ES detections: [...]
```

Don’t worry—this is pretty common when upgrading from ES 7.x to 8.x. Why does this happen?

Because ES 8.x enforces stricter requirements around:

- `risk_message`
- `rule_description`
- `investigation_type`
- `description`

Some of these fields were optional before, but now they’re required!

To find out which correlation rules (now called event-based detections) were not converted properly, you can use the following SPL. It will return a simple table showing which of your detections are missing any required fields. 
```
| rest /servicesNS/-/-/saved/searches splunk_server=local
```

To find out what changes to correlation searches are needed to pass ES 8 Content Management validation checks after migration—run this on your ES 7.x system:

```
| rest /servicesNS/-/-/saved/searches splunk_server=local
``` Determine what Correlation search changes are required to pass ES8 Content Management validation checks after migration - run on ES7.x system ```
| search disabled=0 is_scheduled=1 action.correlationsearch.enabled=1
| rename "eai:acl.app" as app
| eval actions=split(actions,",")
| eval will_create_a_new_risk_event=if(actions in ("risk","notable"," notable"),0,1)
| eval needs_risk_object=if(isnull('action.risk.param._risk_message'),1,0)
| eval needs_description=if(isnull(description) OR description="",1,0)
| eval needs_notable_title=if (isnull('action.notable.param.rule_title'),1,0)
| eval needs_notable_description=if(isnull('action.notable.param.rule_description') OR 'action.notable.param.rule_description'="" ,1,0)
| eval score=will_create_a_new_risk_event+needs_risk_object+needs_description+needs_notable_title+needs_notable_description
| eval actions=split(actions,",")
| sort - score 
| where score > 0
| table app, title, description, score, will_create_a_new_risk_event, needs_description, needs_risk_object, needs_notable_title, needs_notable_description, actions, action.notable.param.rule_title, action.notable.param.security_domain, action.risk.param._risk action.risk.param._risk_message, action.notable.param.rule_description, description
```
 We recently discovered some issues with our correlation searches, and we’re looking for ways to make things easier and less manual!

Instead of manually updating each correlation search, we decided to use our handy script:

👉 [https://github.com/seynur/seynur-tools/splunk_es8_config_updater](https://github.com/seynur/seynur-tools/splunk_es8_config_updater)

**Here’s what the script does:**
- It scans through the `savedsearches.conf` file.
- It finds any missing elements, such as:
  - Description
  - Rule description
  - Risk message
  - Investigation type (This is the new for us)
- It fills those gaps with helpful generic names:
  - `"description = Generic Desc - <stanza_name>`
  - `"action.notable.param.rule_description = Generic Rule Desc - <stanza_name>`
  - `"action.notable.param.investigation_type = default"`
  - `"action.risk.param._risk_message = Generic Risk Msg - <stanza_name>`
- Plus, it makes sure not to overwrite anything unnecessarily.
- From now on, after February 28, 2026, you can freely change your descriptions and other mandatory fields ​​as you wish. 😊

**An Important Tip:**
Make sure:
1. Create a `default` *investigation type* value. There is no default value for required field. 😊

2. All your correlation searches are in a single app and its `savedsearches.conf` file. This isn’t just a good idea—it’s best practice! It helps with manageability and version control, keeping everything tidy and organized.

If you have several `savedsearches.conf` files scattered in different `$SPLUNK_HOME/etc/.../local` directories, no worries! You can easily combine all the configurations with this command:
```
find $SPLUNK_HOME/etc -path "*/local/savedsearches.conf" -type f -exec cat {} \; > /tmp/merged_savedsearches.conf
```

**OR**

if you want to check which app contains this search, you can use the command below:
```
find $SPLUNK_HOME/etc -path "*/local/savedsearches.conf" -type f -exec sh -c '
echo "### FILE: $1"
cat "$1"
echo ""
' _ {} \; > /tmp/merged_savedsearches.conf
```

**How to Run It Safely**

1. **For Search Head Cluster (SHC) Environments**

   Here’s the plan:
   1. Stop all Search Heads at the same time.
   2. Run the script.
   3. Update the `savedsearches.conf` file.
   4. Push the changes with the deployer.
   5. Bring all Search Head members back online together.

   This ensures everything stays consistent across the cluster!

2. **For Standalone Environments**

   It’s a bit simpler:
   - Run the script.
   - Update the file.
   - Trigger a debug/refresh.
   - If needed, restart Splunk.

   Sometimes, a refresh is all you need, but if things feel off, a restart will do the trick!

**A Friendly Reminder** → Always take care when editing production files. Here’s what I suggest:

   - Run the script in the `/tmp` directory.
   - Create the updated `savedsearches.conf` file.
   - Review the changes at your leisure.
   - Finally, replace the original file.

   This method gives you:
   - Complete visibility
   - Extra control
   - A safe way to roll back if needed

Once we’ve patched up those missing parameters and ensured that our Enterprise Security (ES) detections meet the 8.x requirements:
- Migration errors will vanish.
- Correlation searches will run smoothly.
- Risk-based detections will operate as they should.
- Incident Review will be stable and reliable.
- And, we will make an ES upgrade operation succesfully! 🧚🏽‍♀️

---

## 5. Summary & Final Thoughts

Wow, we've made it! The heavy lifting is done, and here's where we stand:
- The platform is stable and running smoothly.
- The upgrade is complete—yay!
- Our detections now align perfectly with the ES 8.x requirements.

What started as just a "version upgrade" turned into an opportunity for a major structural fine-tuning, especially with RBA and detection hygiene. And honestly, that's a fantastic outcome!

With ES 8.x, we're moving towards some great improvements:
- Cleaner and more efficient correlation searches
- Well-defined risk parameters
- More organized detection content
- Enhanced long-term maintainability

Sure, the transition might involve some extra effort upfront. But once everything is in sync, you'll find that the environment is much more predictable and easier to handle.

If you tackle the upgrade step by step, you can keep things manageable:
- Start with a health check
- Make sure you back everything up
- Go for a clean upgrade
- Check system stability
- Then fix detections in bulk

By doing it this way, you turn what could feel like a chaotic scramble into a controlled project instead of a firefighting exercise.

I hope sharing our approach can help make your journey from ES 7.x to 8.1.1 smoother. Plus, you'll be all set to move on to 8.3.0 too!

---

💬 I’m eager to hear your thoughts, ideas, or any questions you might have!

Feel free to connect with me on [*LinkedIn*](https://www.linkedin.com/in/%C3%B6yk%C3%BC-can/) or drop a comment on my blog.  

Thanks for reading! And remember, if you’re diving into Splunk internals, you’re definitely not alone! 😊  

Until next time, Happy Splunking! 👋🔍

---

# References:
- [[1]](https://www.splunk.com/en_us/legal/splunk-software-support-policy.html) Splunk. (n.d.). *Splunk software support policy*. Retrieved from https://www.splunk.com/en_us/legal/splunk-software-support-policy.html

- [[2]](https://help.splunk.com/en/splunk-enterprise/release-notes-and-updates/compatibility-matrix/splunk-products-version-compatibility/splunk-products-version-compatibility-matrix) Splunk. (n.d.). *Splunk products version compatibility matrix*. Splunk Documentation. Retrieved from
https://help.splunk.com/en/splunk-enterprise/release-notes-and-updates/compatibility-matrix/splunk-products-version-compatibility/splunk-products-version-compatibility-matrix

- [[3]](https://lantern.splunk.com/Get_Started_with_Splunk_Software/Upgrading_to_Enterprise_Security_8.0.x_-_Overview) Splunk. (n.d.). *Upgrading to Enterprise Security 8.0.x – Overview*. Splunk Lantern. Retrieved from https://lantern.splunk.com/Get_Started_with_Splunk_Software/Upgrading_to_Enterprise_Security_8.0.x_-_Overview

- [[4]](https://github.com/seynur/seynur-demos/tree/main/splunk/splunk_es8_config_updater) Seynur (2025). seynur-demos/splunk_es8_config_updater [GitHub repository]. GitHub. Retrieved from https://github.com/seynur/seynur-demos/tree/main/splunk/splunk_es8_config_updater