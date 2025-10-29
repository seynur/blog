---
layout: default
title: "MITRE ATT&CK App for Splunk 3.14.0 — Release Highlights"
summary: "This guide shows how to set up a highly available Syslog-ng environment with Keepalived, ensuring continuous log collection and automatic failover via a shared Virtual IP."
author: Hilal Gevrek
date: 30-10-2025
tags: keepalived syslog-ng vrrp failover ha-cluster
categories: Keepalived

---

# MITRE ATT&CK App for Splunk 3.14.0 — Release Highlights

## 1. What is MITRE ATTACK App for Splunk?

[**MITRE ATTACK App for Splunk**](https://splunkbase.splunk.com/app/4617) extends the **MITRE ATT&CK framework** by providing **compliance and triage dashboards**, **correlation search mapping**, and **drill-down analysis**. These capabilities help security teams effectively **operationalize ATT&CK** within their **SIEM workflows**.

**Key Features:**

* **ATT&CK Matrix Visualization:** Interactive dashboard showing adversary techniques.

* **Integration with Splunk ES & ESCU:** Maps correlation searches to ATT&CK techniques.

* **Detection Coverage Overview:** Highlights strengths and gaps in detection coverage.

* **Custom Rule Mapping:** Allows manual or lookup-based ATT&CK alignment for custom searches.

* **Drill-Down Analysis:** Enables quick investigation from matrix to Analyst Queue.

* **Automated Data Lookups:** Keeps visualizations and mappings up to date via scheduled searches.
> 📚 **Learn more & download:**
> - **Documentation**: Official setup and usage guide.
[https://seynur.github.io/mitre-attack-app-for-splunk-docs/v3.14.0/](https://seynur.github.io/mitre-attack-app-for-splunk-docs/v3.13.0/)
> - **Medium posts:** A detailed walkthrough by Selim Seynur explaining detection use cases and setup.
[https://medium.com/seynur/detecting-cyber-threats-with-mitre-att-ck-app-for-splunk-a6627439a9e3](https://medium.com/seynur/detecting-cyber-threats-with-mitre-att-ck-app-for-splunk-a6627439a9e3)
> - **Splunkbase app:** Official page to download and install the app on Splunk.
[https://splunkbase.splunk.com/app/4617](https://splunkbase.splunk.com/app/4617)
> - **Github page:** Source code, and issue tracking.
[https://github.com/seynur/DA-ESS-MitreContent](https://github.com/seynur/DA-ESS-MitreContent)

## 2. What’s New in the MITRE ATTACK App for Splunk (3.14.0)

**Release:** 3.14.0 — **Date:** Oct 30, 2025

**Version 3.14.0** introduces a refreshed layout and enhanced visualization experience, making it easier to navigate between tactics, techniques, and detection mappings.

The redesigned interface improves usability and provides a more intuitive workflow for analysts exploring ATT&CK coverage and investigation data. In this release, we’ve also introduced a **new lookup populated with externally provided threat actor data**, which serves as the foundational source for a **new dashboard**. This dashboard enables analysts to perform investigations and analyses based on threat actor information, offering deeper visibility into adversary behaviors and related detections.

Together, these updates enhance the **MITRE ATT&CK App for Splunk** with a refreshed UI and a new threat actor lookup, helping security teams operationalize the ATT&CK framework with greater precision and efficiency.

Let’s take a closer look at the new updates:

### **2.1. Layout Redesign:**

The **Configuration** and **Setup** sections remain unchanged. However, the main dashboards have been reorganized under two new **dropdown menus**: **Compliance** and **Incidents**.

<p align="center">
  <img src="/assets/img/blog/2025-10-30-new-panels.webp" width="600"/>
</p>

**Compliance:**

* **MITRE ATT&CK Compliance** → Now located under the **Compliance** dropdown as **ATT&CK Matrix**.

* Two new panels have been added under this section, powered by newly introduced lookup: **Threat Actor** and **Threat Actor (Table)**. The next section provides a closer look at how these panels work.

**Incidents:**

* **MITRE ATT&CK Matrix** → Now accessible under the **Incidents** dropdown as **ATT&CK Matrix**.

* **MITRE ATT&CK Triggered Tactics & Techniques** → Now available under the **Incidents** dropdown as **Tactics & Techniques**.

This redesign streamlines navigation and enhances usability by grouping related dashboards under clear dropdown menus, allowing analysts to switch seamlessly between **compliance** and **incident** views.

### **2.2. New Lookup:**

As part of this release, a new lookup has been introduced to support threat actor–based analysis within the MITRE ATT&CK App. This lookup provides a structured way to associate **threat actors** with the **ATT&CK techniques and sub-techniques** they are known to use. The data is externally provided and serves as the foundation for the new **Threat Actor** and **Threat Actor (Table)** dashboards.

**Lookup Path:**

DA-ESS-MitreContent/lookups/mitre_threat_actor_lookup.csv

**Example Lookup:**

    Agrius,AjaxSecurityTeam,Whitefly
    T1583,T1555.003,T1059
    T1059.003,,T1588.002
    ,,T1003.001

**Explanation:**

* The **header row** (first line) contains the names of the threat actors (e.g., *Agrius*, *AjaxSecurityTeam*, *Whitefly*).

* ⚠️ The header values should **not contain any whitespace**.

* The **subsequent rows** list the **MITRE ATT&CK techniques** and **sub-techniques** associated with each threat actor.

**Example Interpretation:**

* The group **Agrius** is mapped to techniques **T1583** and **T1059.003**.

* **AjaxSecurityTeam** is mapped to **T1555.003**.

* **Whitefly** is mapped to **T1059**, **T1588.002**, and **T1003.001**.

This structure allows the dashboards to **correlate threat actors with the ATT&CK techniques they employ**, providing a **clear, technique-based perspective on adversary behavior and coverage**.

The lookup dynamically feeds the new **Threat Actor** and **Threat Actor (Table)** panels, allowing analysts to explore detections and correlations based on known adversary profiles.

### **2.3. New Dashboard:**

**Threat Actor:**
This dashboard provides a **threat actor–centric compliance view** built from a dedicated lookup table, where each entry maps an actor (via threat actor name) to the **MITRE ATT&CK techniques and sub-techniques** they are known to use. Unlike the standard MITRE ATT&CK Compliance dashboard, which focuses on tactics and overall coverage, **this dashboard emphasizes threat actor behavior**, helping security teams assess which adversary techniques are most relevant to their environment and identify potential gaps in detection and defense. To use it, create a lookup based on your own threat actor database or the provided seed list, then explore the visuals to analyze technique and sub-technique coverage.

Here’s a simple example dashboard that uses the lookup mentioned above. The color scheme follows the same logic as the MITRE ATT&CK Compliance dashboard (now renamed ATT&CK Matrix).

<p align="center">
  <img src="/assets/img/blog/2025-10-30-threat-actor.webp" width="600"/>
</p>

The **“Filter by Threat Actor”** option lets you display only the correlation searches and MITRE ATT&CK techniques associated with a specific threat actor (e.g., Agrius, AjaxSecurityTeam, Whitefly). The other filtering options work the same way as in the previous dashboards, and the **Filter by Threat Actor** field defaults to **ALL**.

<p align="center">
  <img src="/assets/img/blog/2025-10-30-threat-actor-panel.webp" width="600"/>
</p>

**Threat Actor (Table):** 
This table displays the rule names, associated MITRE ATT&CK technique IDs, and the count of rules based on the underlying query results.

<p align="center">
  <img src="/assets/img/blog/2025-10-30-threat-actor-table.webp" width="600"/>
</p>

The **Filter by Technique ID** option allows filtering the results by specific MITRE ATT&CK techniques. By default, it is set to **ALL**.

<p align="center">
  <img src="/assets/img/blog/2025-10-30-threat_actor_table_panel.webp" width="600"/>
</p>

⚠️ **Note:** You can change the default configuration from the **Edit** menu on the right side of the screen. You will see a screen like the one below, where you can specify your options.

<p align="center">
  <img src="/assets/img/blog/2025-10-30-default-rules.webp" width="600"/>
</p>

## References:

* [**Splunkbase — MITRE ATT&CK App for Splunk**](https://splunkbase.splunk.com/app/4617)

* [**MITRE ATT&CK Official Framework**](https://attack.mitre.org/)

* [**MITRE ATT&CK App for Splunk (Official Documentation)**](https://seynur.github.io/mitre-attack-app-for-splunk-docs/v3.13.0/)

* [**Seynur Medium Series — “Detecting Cyber Threats with MITRE ATT&CK App for Splunk” (Parts 1–3)**](https://medium.com/seynur/detecting-cyber-threats-with-mitre-att-ck-app-for-splunk-a6627439a9e3)

* [**Splunk Enterprise Security — Official User Guide (v8.2)**](https://help.splunk.com/en/splunk-enterprise-security-8/user-guide/8.2/introduction/about-splunk-enterprise-security)

* [**Splunk Enterprise Security — Release Notes**](https://help.splunk.com/en/splunk-enterprise-security-8/release-notes-and-resources/8.1/splunk-enterprise-security-release-notes/release-notes-for-splunk-enterprise-security)

* [**Splunk Education — ES 8.0: Updates for the Splunk SOC**](https://www.splunk.com/en_us/pdfs/training/es80-updates-for-the-splunk-soc-course-description.pdf)