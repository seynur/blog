---
layout: default
title: "Creating Custom Entity Type with Splunk IT Essentials Work"
summary: "In this part, you will find out how to create custom entity types and associate entities with IT Essentials Work."
author: Merih Bozbura
image: /assets/img/blog/2022-09-26-creating-custom-entity-1.webp
date: 26-09-2022
tags: splunk metrics splunk-it-essentials-work
categories: Splunk

---

# Creating Custom Entity Type with Splunk IT Essentials Work


Splunk [IT Essentials Work](https://splunkbase.splunk.com/app/5403) correlates logs and metrics for each entity and helps you to monitor your infrastructure. It is free, and it consists of several Splunk IT Service Intelligence (ITSI) functionalities.

I’d like to mention two terms before moving on, entity and entity type. An entity is a part of the IT infrastructure that needs administration to deliver IT services. Entity types specify how to categorize a certain kind of data source, or you can think of an entity type as basically a logically correlated group of machines. It is shipped with several out-of-the-box entity types such as *nix, Windows, VMware, etc. Custom entity types are one of the essential things in the IT Essentials Work. You can create a custom entity type for monitoring your critical servers like your web or database servers. When the entities and entity types are ready, you can associate them with IT Essentials Work. The Infrastructure Overview dashboard gives you the ability to monitor the vital metrics of your infrastructure environment.

I tried to explain how to convert event logs into metrics data in the [**first part**](https://medium.com/seynur/converting-event-logs-into-metrics-in-splunk-47f3b52598c6). So, you can ingest your custom metrics, and now you can associate these with your custom entity type in this part.

In this part, you will find out how to create custom entity types and associate entities with IT Essentials Work.

### Creating a custom entity type

As you can see from the screenshot, these little panels in the Infrastructure Overview dashboard are entity types, and they represent the key metrics. Entity types are organized by a key metric, such as average CPU or maximum disk usage. Key metrics provide a snapshot of what is going on in those machines.

As I mentioned above, you can think of entity types as logically correlated groups of machines. So, IIS Servers entity type is created to monitor the IIS server.

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-1.webp)

For creating a custom entity type go to IT Essentials Work → Configuration → Entity Management.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-2.webp)

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-3.webp)

Give a name and description for the entity type.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-4.webp)

Vital metrics are basically Splunk searches, and you can select one of them as the key metric. Create your searches based on your needs. You need to change the field name to “**val**” which IT Essentials Work expects. Also, group by host and specify a span. After creating the search, you will see the result in the display preview.

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-5.webp)

As you can see here, you can choose any vital metric as the key metric.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-6.webp)

You can also add [navigations](https://docs.splunk.com/Documentation/ITSI/4.14.0/Entity/EntityType#Step_3:_Add_navigations_to_your_entity_type), [dashboards](https://docs.splunk.com/Documentation/ITSI/4.14.0/Entity/EntityType#Step_4:_Add_Splunk_dashboards_to_your_entity_type), and [analysis](https://docs.splunk.com/Documentation/ITSI/4.14.0/Entity/EntityType#Step_5:_Add_analysis_data_filters_to_your_entity_type) data filters to your entity type.

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-7.webp)

### Associate entities

Firstly, you need to import your entities that are related to your entity type. There are several [ways](https://docs.splunk.com/Documentation/ITSI/4.13.1/Entity/About) to import your entities, like creating a single one, using a CSV file, or with a Splunk search, etc.

This is the example Splunk search for selecting entities.

```
| mstats count(*) where index=test_metrics by host
```

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-8.webp)

For importing your entities, go to IT Essentials Work → Configuration → Entity Management.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-9.webp)

Check the search results.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-10.webp)

As you can see here, you do not have to import all the search results. Select the host field as the entity title and import only the host field as entities.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-11.webp)

Now, the entities will appear.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-12.webp)

Select your hosts for mapping with your entity type. You can map them all at once with the Bulk Action option.

Bulk Action → Add selected to an entity type.

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-13.webp)

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-14.webp)


When you map the entities with the associated entity type, the Infrastructure Overview dashboard should be populated.


![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-15.webp)

![screenshot](/assets/img/blog/2022-09-26-creating-custom-entity-16.webp)

**NOTE:** There is no frequent data ingestion since it is a demo environment. So, the Current Entity Status Breakdown does not appear.

To sum up, IT Essentials Work is a free application that contains several ITSI functionalities, such as entity integration, entity types, etc. Therefore, it provides the ability to create your infrastructure environment and monitor your critical servers.

---


#### References:

- [Overview of entity types in ITSI](https://docs.splunk.com/Documentation/ITSI/4.13.1/Entity/EntityVisualizations)

- [What is an entity integration?](https://docs.splunk.com/Documentation/ITSI/4.13.1/Entity/About)

- [Create custom entity types in ITSI](https://docs.splunk.com/Documentation/ITSI/4.13.1/Entity/EntityType)

- [IT Essentials Work](https://splunkbase.splunk.com/app/5403)

- [Import entities from a search in ITSI](https://docs.splunk.com/Documentation/ITSI/4.13.1/Entity/ImportSearch)

---
