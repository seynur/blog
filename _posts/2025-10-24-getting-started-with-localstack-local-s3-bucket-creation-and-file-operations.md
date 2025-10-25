---
layout: default
title:  "Getting Started with LocalStack: Local S3 Bucket Creation and File Operations"
summary: "This blog will teach you how to set up a completely local S3 environment using LocalStack and AWS CLI for rapid, cloudless development and testing."
author: Öykü Can Şimşir
image: /assets/img/blog/2025-10-24-getting-started-with-localstack-local-s3-bucket-creation-and-file-operations.webp
date: 24-10-2025
tags: localstack s3 aws docker
categories: LocalStack

---

# Getting Started with LocalStack: Local S3 Bucket Creation and File Operations

If you're building tools or workflows that connect with AWS services like S3, Lambda, or DynamoDB, you might find that testing everything directly in the cloud can get a bit slow, pricey, and tough to manage. 

What if I told you there's a way to run a complete AWS-like environment right on your local machine? That means no internet connection needed, no surprises on your cloud bill, and you can get instant feedback! That's exactly what [LocalStack](https://localstack.cloud/) is all about.

LocalStack is a friendly AWS cloud emulator that allows you to develop and test your cloud applications right from your own computer. In this post, I'll walk you through setting up LocalStack, configuring your environment, and using the [AWS CLI](https://aws.amazon.com/cli/) to create an [S3](http://aws.amazon.com/s3/?trk=b2e0b71d-6f5d-4607-94dc-18f7ddd5339a&sc_channel=ps&ef_id=EAIaIQobChMI6aOUp5q_kAMVaGZBAh07tDl-EAAYASAAEgLyVPD_BwE:G:s&s_kwcid=AL!4422!3!645208988806!e!!g!!s3!19580264380!143903638703&gad_campaignid=19580264380&gbraid=0AAAAADjHtp9YgzsvVgxdkeRljJzdtAzXU&gclid=EAIaIQobChMI6aOUp5q_kAMVaGZBAh07tDl-EAAYASAAEgLyVPD_BwE) bucket, upload files, and retrieve them — all without having to touch AWS at all. Let’s dive in!

---

## ⚙️ Prerequisites

To get started with this guide, you'll want to have a couple of tools installed. Here’s what you’ll need:

- [**Docker**](https://www.docker.com/): This is where LocalStack will run inside a Docker container, making everything nice and easy!
- [**AWS CLI**](https://aws.amazon.com/cli/): We’ll use the official AWS Command Line Interface to interact with the local S3 service. It’s super handy!
- (Optional) **Python 3.x**: If you enjoy scripting or have some advanced use cases in mind, this will come in handy later.

To check if Docker and the AWS CLI are already installed on your machine, just run these commands:

```bash
docker --version
aws --version
```

If either command isn’t recognized, follow these links to install them:
- Docker: https://docs.docker.com/get-docker/
- AWS CLI: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Let’s roll up our sleeves and get started!

---

## ▶️ Starting LocalStack (via Docker)

Once you have Docker installed and up and running, getting started with LocalStack is super easy! Just run the following command:

```bash
docker run --rm -it -p 4566:4566 -p 4571:4571 localstack/localstack
```

This will launch LocalStack and make its services, including S3, available at `http://localhost:4566`. 

> ℹ️ The best part? LocalStack comes with built-in support for many AWS services right out of the box, so you won't need to do any extra setup for basic testing with S3. Enjoy experimenting!

---

## 🔧 Setting Up the AWS CLI with LocalStack

Hey there! Did you know that LocalStack emulates AWS services right on your machine? That means you can use the AWS CLI to interact with it just like you would with the real AWS cloud. The only difference is that you'll direct the CLI to LocalStack’s local endpoint.

To get started, let’s set up some dummy credentials as environment variables since LocalStack is super flexible and accepts any values:

```bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
```

No need to worry about these credentials being real—they’re just a formality required by the AWS CLI!

---

## 🪣 Let’s Create a Bucket in LocalStack S3!

Now, let’s have some fun and create a new S3 bucket using the AWS CLI while pointing it to LocalStack:

```bash
aws s3 mb s3://my-local-s3-bucket --endpoint-url=http://localhost:4566

```

If everything goes smoothly, you’ll see a friendly response like this:

```
make_bucket: my-local-s3-bucket
```

🥳 Woohoo! You just created your first local S3 bucket without needing to touch the real AWS. How awesome is that?

---

## 📤 Uploading a File to Your Local S3 Bucket

Let’s have some fun and create a simple text file that we can upload to the bucket we just set up!

Start by making a file called `sample.txt` with a little message:

```bash
echo "Hello from LocalStack :)" > sample.txt
```

Now, let’s get that file uploaded to your S3 bucket using the AWS CLI! Just run this command:

```bash
aws --endpoint-url=http://localhost:4566 s3 cp sample.txt s3://my-local-s3-bucket/sample.txt
```

If everything goes smoothly, you’ll see a message like this:

```bash
upload: ./sample.txt to s3://my-local-s3-bucket/sample.txt
```

Great job! You’ve successfully uploaded your first file! 🎉

---

### 📂 Listing and Downloading Your Bucket Contents

Hey there! If you want to double-check that your file has been uploaded successfully, you can easily list all the objects in your bucket. Just run this command:

```bash
aws --endpoint-url=http://localhost:4566 s3 ls s3://my-local-s3-bucket/
```

You should see something like this in the output:

```bash
2025-10-24 14:50:14         25 sample.txt
```

This means your file is safely stored in your local S3 environment, ready for you to access or test, just like you would in AWS!

If you'd like to download or restore the file, you can use the `aws s3 cp` command to copy it back to your local machine. Here’s how:

```bash
aws --endpoint-url=http://localhost:4566 s3 cp \
  s3://my-local-s3-bucket/sample.txt downloaded.txt
```

After running that, you should see an output that looks like this:

```bash
download: s3://my-local-s3-bucket/sample.txt to ./downloaded.txt
```

Congratulations! You now have a local file named `downloaded.txt` that contains the same content as the original file.

To check if everything worked perfectly, you can simply run:

```bash
cat downloaded.txt
```

And you should see:

```bash
Hello from LocalStack :)
```

> **Quick Tip:** If you want to delete the file later, just run the following command:

```bash
aws --endpoint-url=http://localhost:4566 s3 rm s3://my-local-s3-bucket/sample.txt
```

Feel free to explore and use the rest of the AWS commands as you wish! Enjoy your S3 experience! 😊


---

# 🔚 Conclusion

You've done an amazing job with LocalStack and the AWS CLI! By creating a local S3 bucket, uploading a file, listing its contents, downloading that file, and finally deleting it—all without needing to connect to AWS—you've covered some great ground.

This setup is super handy for testing cloud-related logic offline or in CI environments. Plus, it opens up the exciting possibility of simulating more complex workflows like archiving, backups, or access control, all in a speedy local environment.

In our next post, we’ll show you how to take this local S3 setup and integrate it into a real-world workflow by automatically pushing archived data to it using Splunk. 

Thanks for following along, and stay tuned for more fun!

---

# 🧺 Wrapping Up

That’s a wrap, my friends! 🎉 You’ve done an amazing job building a local AWS S3 environment that’s fully functional, complete with:
- ✅ Dockerized LocalStack 
- 🪣 Easy command-line bucket creation 
- 📤 Simple file upload and retrieval 
- 🧹 A hassle-free clean-up process 

Are you diving into cloud workflows, testing out backup pipelines, or creating development environments without needing to touch AWS? 

I’d love to hear all about what you’re working on! Feel free to connect with me on [*LinkedIn*](https://www.linkedin.com/in/%C3%B6yk%C3%BC-can/) or drop a comment on the blog. 

Until next time —
Happy developing! 🧪☁️

---

## References:
- [[1]](https://www.localstack.cloud/)  LocalStack (2025). ***Webpage***. https://www.localstack.cloud/
- [[2]](https://docs.localstack.cloud/aws/?__hstc=108988063.0ee2f30eaf277f10554eb6d4f1260de9.1761130776473.1761305608218.1761389372142.4&__hssc=108988063.1.1761389372142&__hsfp=597304104)  LocalStack (2025). ***Welcome to LocalStack for AWS Docs***. https://docs.localstack.cloud/aws/?__hstc=108988063.0ee2f30eaf277f10554eb6d4f1260de9.1761130776473.1761305608218.1761389372142.4&__hssc=108988063.1.1761389372142&__hsfp=597304104
- [[3]](https://aws.amazon.com/s3/?trk=b2e0b71d-6f5d-4607-94dc-18f7ddd5339a&sc_channel=ps&ef_id=EAIaIQobChMI6aOUp5q_kAMVaGZBAh07tDl-EAAYASAAEgLyVPD_BwE:G:s&s_kwcid=AL!4422!3!645208988806!e!!g!!s3!19580264380!143903638703&gad_campaignid=19580264380&gbraid=0AAAAADjHtp9YgzsvVgxdkeRljJzdtAzXU&gclid=EAIaIQobChMI6aOUp5q_kAMVaGZBAh07tDl-EAAYASAAEgLyVPD_BwE) Amazon Web Services, Inc. (2025). ***Amazon S3***. Amazon. https://aws.amazon.com/s3/?trk=b2e0b71d-6f5d-4607-94dc-18f7ddd5339a&sc_channel=ps&ef_id=EAIaIQobChMI6aOUp5q_kAMVaGZBAh07tDl-EAAYASAAEgLyVPD_BwE:G:s&s_kwcid=AL!4422!3!645208988806!e!!g!!s3!19580264380!143903638703&gad_campaignid=19580264380&gbraid=0AAAAADjHtp9YgzsvVgxdkeRljJzdtAzXU&gclid=EAIaIQobChMI6aOUp5q_kAMVaGZBAh07tDl-EAAYASAAEgLyVPD_BwE
- [[4]](https://aws.amazon.com/cli/) Amazon Web Services, Inc. (2025). ***AWS Command Line Interface***. Amazon. https://aws.amazon.com/cli/
- [[5]](https://www.docker.com/) Docker Inc. (2025). ***Webpage***. https://www.docker.com/

---