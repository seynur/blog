---
layout: default
title: "How to Become A Certified Splunk Enterprise Admin?"
summary: "In this blog, I will talk about the stages of becoming a certified Splunk admin. Splunk is a data (The Data-to-Everything™) platform that allows you to collect any data from any source and analyze it intelligently and generate value from the data."
author: Enes Oğuzhan Alataş
image: /assets/img/blog/2020-05-05-how-to-become-splunk-admin-1.webp
date: 05-05-2020
tags: splunk splunk-administration
categories: Splunk

---

# How to Become A Certified Splunk Enterprise Admin?

In this blog, I will talk about the stages of becoming a certified [Splunk](https://www.splunk.com/) admin. Splunk is a data (The Data-to-Everything™) platform that allows you to collect any data from any source and analyze it intelligently and generate value from the data. As a certified Splunk admin, you can not only contribute to the more effective use of Splunk in your organization, but also make an important contribution to your career. This blog aims to summarize the topics to be considered during the training and examination process to the candidates.

Splunk offers a [certification track](https://www.splunk.com/en_us/training.html#l2-name-22) as picture for each certificate you can get. In a nutshell, certification tracks require some trainings and exams, and suggest some of them. I recommend candidates, who do not have enough hands-on experience, to complete all exams and trainings offered on the related path.

![screenshot](/assets/img/blog/2020-05-05-how-to-become-splunk-admin-1.webp)

There are four trainings and two exams on the track Splunk offers for its admin certification. In order to become a certified admin, users are advised to complete their user and power user trainings. Although it is not **compulsory to take** the user certification exam, it is compulsory to have the **power user certificate** in order to take the admin certification exam. After this stage, Splunk **strongly recommends** users to complete **System and Data Administration** trainings. The image above summarizes this certification path simply. Next, I will touch upon the points I suggest to pay attention to for each stage.

### FUNDAMENTALS PART

Fundamentals trainings are composed of contents that Splunk generally targets end users and that does not have a high technical intensity. These trainings are intended to teach all Splunk operations that users with **User and Power User** privileges will frequently operate. You can easily review the actions that users with these authorizations can do in the content [here](https://docs.splunk.com/Documentation/Splunk/7.3.1/Security/Rolesandcapabilities).

After completing these trainings, it is necessary to be successful in the Power User exam. So how? Although a demo environment is offered during the trainings, I recommend candidates to **practice**, as much as they can, by installing a **Splunk Standalone** on their personal computers. The practices that the candidates will do in their own environment and, if possible with their own data, will significantly benefit to have grasp of the operations described in the training.

![screenshot](/assets/img/blog/2020-05-05-how-to-become-splunk-admin-2.webp)

### ADMINISTRATION PART

Splunk offers its trainings at admin level in two different parts: system administration and data administration. The topics covered in these trainings can be summarized easily through the figure below.

![screenshot](/assets/img/blog/2020-05-05-how-to-become-splunk-admin-3.webp)

Thanks to the labs applied at the end of each module, the trainees also get hands-on practice. Remember, you will need to **complete all the labs** in order to complete the trainings successfully. For this reason, it is useful to take a look at some [CLI commands](https://files.fosswire.com/2007/08/fwunixref.pdf) and a CLI text editor such as vi or nano before training. Although all these give you enough information to be successful in the certification exam, I strongly recommend that you practice before the exam. At this point, as same in the Fundamentals party, I recommend you to create a Splunk deployment on your own environment, but of course this should be a little more complex ;)

![screenshot](/assets/img/blog/2020-05-05-how-to-become-splunk-admin-4.webp)


And there are some **final tips for Admin Certification Exam:**

- Define at least three different data inputs (you can try the [eventgen](http://splunk.github.io/eventgen/)),
- Create different user groups with different capabilities,
- Practice all the operations in [props](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Propsconf) and [transforms](https://docs.splunk.com/Documentation/Splunk/8.0.3/Admin/Transformsconf) configurations,
- Practice index-time and search-time [precedences](https://docs.splunk.com/Documentation/Splunk/8.0.3/Admin/Wheretofindtheconfigurationfiles).

Let’s ingest data, define users, assign roles, create some apps and what ever you learned during the training. Do not avoid break something, you are not in the production. Then take the exam as **the first step to be a Splunk Ninja!**

---
