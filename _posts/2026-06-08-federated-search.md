---
layout: default
title: "Splunk Talking to Splunk: Federated Search in 3 Minutes"
summary: "Federated search lets one Splunk instance query another remotely: no forwarders, no pipelines. Just two config files, a restart, and you're done."
author: Öykü Can
image: /assets/img/blog/2026-06-08-federated-search.png
date: 08-06-2026
tags: splunk federated-search remote search
categories: Splunk 

---

# Splunk Talking to Splunk: Federated Search in 3 Minutes

Think of two different Splunks: Splunk-1 for production use and Splunk-2 for tests. Getting access to the information stored in both would need some work like setting up a pipeline, using forwarders, or something else, wouldn’t it? Well, here comes the idea that might surprise you – how about just getting your second Splunk’s info instantly?

This is where federated search helps, starting version ***Splunk Enterprise 8.2***, and ***Splunk Cloud Platform 8.1.2103*** [[1]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk) [[2]](https://www.splunk.com/en_us/blog/platform/introducing-splunk-federated-search.html). In other words, by using this functionality, one Splunk (say, `splunk_s2`) gets access to the other (`splunk_s1`) at query time and shows you its information as if it was already stored in your first Splunk starting version 8.2! Isn’t it great? ✨

---

# Getting Started

Now imagine that we have two new Splunks standing next to each other. The `splunk_s2` can easily look into `splunk_s1`’s data without configuring anything else except connecting to port `8089`.

This is what our scheme looks like:
```
[ splunk_s1 ] <- federated search (port 8089) ->  [ splunk_s2 ]
  remote                                               local
(lives the data)                                  (makes the search)
```

You will need to make some settings on the `splunk_s2` side. For the `splunk_s1`, nothing should be done except it being alive and ready to accept connections to port 8089.

> **Note:** Tested on Splunk Enterprise 9.4.11 with two standalone instances.

## Option A - Config Files (Preferred)

The good thing about config files is that they are replicable, manageable in a version control system, and platform-agnostic for all Splunk versions. 

### Step 1a - Configure the Federation Provider: 
Let's start by creating (and modifying) a file named `federated.conf` in `splunk_s2`, which will refer to the location and authentication methods on `splunk_s1`.

This is where you should place it [[3]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk):
`$SPLUNK_HOME/etc/system/local/federated.conf`

The following information should be added here:

```
[provider://splunk_s1]
hostPort                = x.x.x.x:8089
mode                    = standard
type                    = splunk
serviceAccount          = admin
password                = topSecretAdminPassword
use
```
> **Note:** Although the password will be saved as a plaintext initially, there is no need to fear, as it will get encrypted by Splunk after its first reboot.

### Step 2a – Federate an Index:
We will now specify the index that should be federated to `splunk_s2`. Let’s begin by specifying the `_internal` index, yes, we can federate the `_internal` index as well!

Place this configuration under [[3]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk):
`$SPLUNK_HOME/etc/system/local/indexes.conf`

```
[federated:internal]
federated.dataset = index:_internal
federated.provider = splunk_s1
```

> **Note:** It is vital to note that `federated:internal` is our local alias and `federated.dataset = index:_internal` is the actual index name on the remote side.

Then restart `splunk_s2` and jump to [Run & Verify](#run--verify).

## Option B - Using the UI

For those who find it easier to click buttons rather than type commands, here is how you can get everything done using the UI!

### Step 1b - Add a Federated Provider:
Go to **Settings → Federated Search → Federated Providers**, then follow these steps [[4]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/define-a-splunk-platform-federated-provider#ariaid-title3):

1. Go to **Settings → Federated Search → Federated Providers**, and click **Add federated provider**.
2. Enter the required information as follows:

    | Field                      | Value            |
    |----------------------------|------------------|
    | Provider mode              | Standard         |
    | Provider name              | splunk_s1      |
    | Remote host                | x.x.x.x:8089     |
    | Service account username    | admin            |
    | Service account password     | your password     |

3. Don’t forget to check **Test connection**, which will give you a reassuring notification like **"Connection successful."**

| ![screenshot](/assets/img/blog/2026-06-08-sto2_splunk_provider.png) |
|:--:|
| *Figure 1*: Add Federated Splunk Provider screen successfully. |

### Step 2b – Add a Federated Index:
Let's proceed with adding a federated index [[4]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/define-a-splunk-platform-federated-provider#ariaid-title3):

1. Navigate to **Federated Indexes**, and click **Add federated index**.
2. Enter the following information in the fields:

    | Field                      | Value                  |
    |----------------------------|------------------------|
    | Federated index name       | internal               |
    | Federated provider         | splunk_s1 (Splunk)  |
    | Dataset type               | Index                  |
    | Dataset name               | _internal              |

| ![screenshot](/assets/img/blog/2026-06-08-sto2_index.png) |
|:--:|
| *Figure 2*: Create a Federated Index screen. |

## Run & Verify
Whichever path you took, head to Search & Reporting on `splunk_s2` and run:

```
index=federated:internal 
| stats count values(index) as index values(sourcetype) as sourcetype by splunk_federated_provider
```

You should have received `splunk_s1` as well as a number of events!

| ![screenshot](/assets/img/blog/2026-06-08-sto2_spl.png) |
|:--:|
| *Figure 3*: Remote search results from `splunk_s2`. |

> **Note:** whether you’re working with the UI or the configuration files, the result will be the same! The user interface simply makes it more convenient for documentation and management.

--- 

# Keep in Mind

- **Port Availability**: Ensure management port is accessible. This port will be used for federated search; note that federated search does not utilize the management input port [[4]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/define-a-splunk-platform-federated-provider). When faced with connection issues, make sure to review the firewall settings first. Use the command `curl https://x.x.x.x:<mgtport>/services` on `splunk_s2` to test port availability.

- **Permissions for Accessing Indexes**: In case you’re working with an ordinary service account, make sure that the user has permissions to read the target index, including internal indexes, which usually aren’t accessible by default [[1]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk) .

- **SSL Verification for Testing Purposes**: When using self-signed certificates, you may encounter SSL handshake issues [[5]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/service-accounts-and-security-for-federated-search-for-splunk).  In order to turn off verification (only for testing purposes), simply include the following line in your provider stanza:
    ``` 
    sslVerifyServerCert = false
    ```

--- 

# Conclusion
Federated search sounds intimidating on paper but is easy enough to do — simply two configuration files and a restart. It has nothing like forwarders, pipelines, or any other kind of mess.

💬 I'd love to hear your feedback, suggestions or even questions on it! Feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/%C3%B6yk%C3%BC-can/)! 🧚🏽‍♀️

Till next time! 👋

---

# References

[[1]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk) Splunk. (n.d.-a). About Federated Search for Splunk. Splunk Documentation. Retrieved from [https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk)

[[2]](https://www.splunk.com/en_us/blog/platform/introducing-splunk-federated-search.html) Splunk. (2021). Introducing Splunk Federated Search. Splunk Documentation. Retrieved from [https://www.splunk.com/en_us/blog/platform/introducing-splunk-federated-search.html](https://www.splunk.com/en_us/blog/platform/introducing-splunk-federated-search.html)
 
[[3]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk) Splunk. (n.d.-b). federated.conf. Retrieved from [https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/about-federated-search-for-splunk)

[[4]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/define-a-splunk-platform-federated-provider) Splunk. (n.d.-c). Define a Splunk platform federated provider. Retrieved from [https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/define-a-splunk-platform-federated-provider#ariaid-title3](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/define-a-splunk-platform-federated-provider)

[[5]](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/service-accounts-and-security-for-federated-search-for-splunk) Splunk. (n.d.-d). Service accounts and security for Federated Search for Splunk / About HTTPS with TLS 1.2 encryption for federated search. Retrieved from [https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/service-accounts-and-security-for-federated-search-for-splunk](https://help.splunk.com/en/splunk-enterprise/search/federated-search/9.4/run-federated-searches-across-other-splunk-deployments/service-accounts-and-security-for-federated-search-for-splunk)