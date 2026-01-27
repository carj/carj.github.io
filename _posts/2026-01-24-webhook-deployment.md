---
layout: post
title: Automatic Deployment of Preservica Webhooks on AWS

---

### Introduction

In a previous [post](https://jcarr.org.uk/2023/06/10/webhooks/) I described how Preservica can use webhook notifications to allow the creation of custom business processes. 
At the end of the article I touched upon the challenges of hosting and securing webhook endpoints, 
and the manual effort required to deploy the supporting infrastructure. We can reduce the cost and work of running dedicated hardware and
web servers by using a serverless architecture within AWS but this does require a level of AWS knowledge 
to connect together all the required services. 
With a serverless architecture you only pay for the milliseconds of server time that you use, 
so it's many orders of magnitude cheaper than regular hosting options.

This post describes a method to simplify the development of web services which will receive the Preservica webbooks 
and a method to automate the process of creating the required [AWS services](https://aws.amazon.com), 
such as the AWS Lambda function and API Gateway. 

### Background

We are going to use three Python projects: the web framework [Flask](https://flask.palletsprojects.com/en/stable/) to manage the application logic and create 
the web service which processes the messages from Preservica.  
The deployment of the Flask application to AWS including the creation of a Lambda function and the API Gateway will be done using [Zappa](https://github.com/zappa/Zappa). 
The webhook handshake logic and the interaction with Preservica will be done using [pyPreservica](https://pypreservica.readthedocs.io/en/latest/).

#### Flask

Flask is often described as a micro web framework for Python.

Flask is famous for its "Hello World" being only a few lines of code. It uses Python decorators to handle routes, making the code highly readable.

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Flask!"
```

This simplicity makes it ideal for creating the business logic of the webhook service.  
The code that handles the webhook messages will be within the Flask application.

#### Zappa

Zappa is an open-source tool designed to "Lambda-fy" Python web applications. It allows you to deploy any WSGI-compatible application (like Flask) to AWS Lambda and API Gateway with almost zero configuration.

Essentially, Zappa acts as a bridge: it packages your entire Python project, handles the complex AWS infrastructure setup, and translates incoming API Gateway requests into a format your Flask application can understand.

By using Zappa, you no longer need to log in to AWS and deploy Lambda functions, API Gateways, IAM roles, and all the AWS resources required to set up the serverless environment. No AWS knowledge is required.

#### pyPreservica

pyPreservica is an open-source Python Software Development Kit (SDK) and client library designed to interact with the Preservica API. 
It is the primary tool for archivists, developers, and records managers who want to use the Preservica API.
pyPreservica is used to convert the messages from Preservica into Assets that can be processed within the webhook.

### Getting Started

We are going to create a simple web application for receiving Preservica webhook notifications and deploy it within AWS.  

We will use a virtual environment to manage the dependencies for our project.

We will create a project folder called webhooks and a .venv folder within it:

```console

$ mkdir webhooks
$ cd webhooks
$ python3 -m venv .venv

```

Before you work on your project, activate the corresponding environment:

```console
$ . .venv/bin/activate
```

Within the virtual environment, use the following command to install Flask:

```console
$ pip install Flask
```

Flask is now installed.  

The next step is to create the application logic. Create a new file called app.py in the project folder and copy the following code into it:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "<p>Hello, World!</p>"
```

This will create a minimal application which will return a basic HTML string when a URL is accessed through a browser.

We are now going to deploy the test application to AWS using Zappa. Because we are going to deploy within AWS, you will need a valid AWS account and some API keys in a credentials file.

The [AWS credentials file](https://aws.amazon.com/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/) should contain an access key and a secret key.

After your AWS credentials are set up, we can deploy Zappa:

```console
$ pip install zappa
```

Once installed, we can run Zappa to detect the application and create our settings file:

```console
$ zappa init
```

This will prompt you to answer a few simple questions, for this example we can except the default "dev" stage, the S3 bucket name, the app.app name.
When asked if the application should be deployed globally we can say no.


Once you finish initialization, you'll have a file named `zappa_settings.json` in your project directory defining your basic deployment settings. It will probably look something like this:

```json
{
    "dev": {
        "app_function": "app.app",
        "aws_region": "us-east-1",
        "exclude": [
            "boto3",
            "dateutil",
            "botocore",
            "s3transfer",
            "concurrent"
        ],
        "profile_name": "default",
        "project_name": "webhooks",
        "runtime": "python3.13",
        "s3_bucket": "zappa-nquhdelye"
    }
}


```

Once your settings are configured, you can package and deploy your application to AWS with a single command:

```console
$ zappa deploy dev
```
If you get an error at this stage, it will probably be because your AWS API key does not have sufficient privileges. 

You should see some output followed by the deployed URL.

```console
INFO:Uploading webhooks-dev-1769437524.zip (9.9MiB).
...
INFO:Deploying API Gateway..
INFO:Waiting for lambda function [webhooks-dev] to be updated...
Deployment complete!: https://8hxhcmrjd2.execute-api.us-east-1.amazonaws.com/dev
```

If you now visit the URL shown in your console, then you should see the text Hello, World! in the browser.

### Validating the Messages

During the webhook subscription process, Preservica will send a challenge response message to the specified endpoint URL to 
verify that it exists and its publicly accessible.
Preservica sends a POST request to the URL with a challengeCode query parameter. 
The server must respond with the expected challenge response or the subscription will fail.

The response sent back to Preservica takes the form of a simple json document which includes the original challenge 
code and a hexHmac256Response which is a hexadecimal encoded hmac256 of the challenge Code using the shared secret as the hmac key.

In the previous post on Preservica webhooks we showed how we could write code to manage this verification process 
directly within a AWS Lambda function, in this example we are going to add some simple calls into the 
pyPreservica library into our web service to perform the handshake challenge.

First we add pyPreservica to our Project:

```console
$ pip install pyPreservica
```

Now we can update the flask application to respond to the challenge.

Since Preservica will only send webhook messages using HTTP POST, we can limit our service to ignore other requests such as GET etc.

```python
from flask import Flask

app = Flask(__name__)

@app.route('/', methods=['POST'])
def index():
    return "<p>Hello, World!</p>"
```

The next step is to add the pyPreservica import statement and the FlaskWebhookHandler.


```python
import os
from flask import Flask, request
from pyPreservica import FlaskWebhookHandler

app = Flask(__name__)

@app.route('/', methods=['POST'])
def index():
    webhook = FlaskWebhookHandler(request, os.environ.get('WEBHOOK_SECRET'))
    if webhook.is_challenge():
        return webhook.verify_challenge()

    return webhook.response_ok()
```
We can now test for initial handshake and return the message Preservica is expecting to allow the successful registration of the webhook.

The WEBHOOK_SECRET is an environment variable which contains a shared secret between the web hook service and
the Preservica system. This can be used to verify any messages received by the web hook service did actually
come from Preservica.

The shared secret should be a strong password which cannot be guessed. To make sure the service can access
the secret we will add it to the AWS environment variables.

Update the zappa_settings.json file by adding a aws_environment_variables entry and your secret password.


```json
{
    "dev": {
        "app_function": "app.app",
        "aws_region": "us-east-1",
        "exclude": [
            "boto3",
            "dateutil",
            "botocore",
            "s3transfer",
            "concurrent"
        ],
        "profile_name": "default",
        "project_name": "webhooks",
        "runtime": "python3.13",
        "s3_bucket": "zappa-nq6p2nr48",

        "aws_environment_variables": {
            "WEBHOOK_SECRET": "EB9WJjYDkckyET3VM70r"
        }
    }
}
```

We can update the application using 

```console
$ zappa update dev
```

```console
INFO:Unscheduled webhooks-dev-zappa-keep-warm-handler.keep_warm_callback.
INFO:Scheduled webhooks-dev-zappa-keep-warm-handler.keep_warm_callback with expression rate(4 minutes)!
INFO:Waiting for lambda function [webhooks-dev] to be updated...
Your updated Zappa deployment is live!: https://8hxhcmrjd2.execute-api.us-east-1.amazonaws.com/dev
```


### Creating the Subscription

At this point we have a webhook service deployed in AWS waiting for a challenge request from Preservica. 
The handshake only occurs when a new webhook subscription is created by preservica, so that is what we will do next.

Create a new python script called subscribe.py in a folder outside your project, create a pyPreservica WebHooksAPI API client and
call the subscribe() function, pass the URL generated by zappa, the trigger type you want to subscribe to (Ingest events) 
in this case and the webhook secret you created earlier.
For details on authenticating the pyPreservica WebHooksAPI call see https://pypreservica.readthedocs.io/en/latest/intro.html#authentication


```python
from pyPreservica import WebHooksAPI, TriggerType

client = WebHooksAPI()

sub = client.subscribe(url="https://7k6anj0k99.execute-api.us-east-1.amazonaws.com/dev", triggerType=TriggerType.INDEXED, secret="EB9WJjYDkckyET3VM70r")
print(sub)
```

Run this script to create the subscription, from this point everything ingested into Preservica will trigger a webhook event.

```console
$ python3 subscribe.py

{"id":"4583e1e6d2c72ed96a1151bc7dfd53c2","url":"https://8hxhcmrjd2.execute-api.us-east-1.amazonaws.com/dev","triggerType":"FULL_TEXT_INDEXED","includeIdentifiers":true}

```


### Creating the Application

We have a web hook application which can successfully respond to subscription requests, but cannot do much else. 
The next step is to add some application logic to determine which Assets have been ingested.

Update the Flask application code to process the incoming requests and use the Preservica Content API to fetch
information about the objects.

The process_request() method is a generator which returns a dictionary for every object which is part of the webhook event.



```python
import os
from flask import Flask, request
from pyPreservica import FlaskWebhookHandler, ContentAPI

client = ContentAPI()

app = Flask(__name__)

@app.route('/', methods=['POST'])
def index():
    webhook = FlaskWebhookHandler(request, os.environ.get('WEBHOOK_SECRET'))
    if webhook.is_challenge():
        return webhook.verify_challenge()
    else:
        for obj_details in webhook.process_request():
            entity = client.object_details(obj_details["entityType"], obj_details["entityRef"])
            print(entity)

    return webhook.response_ok()
```

This will print the details into the webhook logs of every entity, Asset or Folder which is ingested into Preservica.

We can now update our AWS service again. But before we do we need to provide credentials to the pyPreservica library.

Add your Preservica username, password and server hostname to the zappa_settings.json file within the
aws environment variables parameter.

```json
{
    "dev": {
        "s3_bucket": "lambda",
        "app_function": "app.app",
      
        "aws_environment_variables": {
            "WEBHOOK_SECRET": "EB9WJjYDkckyET3VM70r",
            "PRESERVICA_PASSWORD": "1234567",
            "PRESERVICA_SERVER":  "uk.preservica.com",
            "PRESERVICA_USERNAME": "email@test.com"
        }
    }
}
```

Redeploy using zappa 


```console
$ zappa update dev
```

and we can monitor the AWS logs locally using:

```console
$ zappa tail
```

Ingest a document into your preservica system, and you should see the details appear on the console.


### Extending the Application

We now have a functioning web service which can validate subscriptions and handle Preservica events such as ingest etc.

The next step is to build out your application, the following example fetches every 
thumbnail image and copies it to an S3 bucket for each new Asset ingest. 

```python
import os
import io
from flask import Flask, request
import boto3
from pyPreservica import FlaskWebhookHandler, ContentAPI, EntityType, Thumbnail

client = ContentAPI()

app = Flask(__name__)


@app.route('/', methods=['POST'])
def index():
    webhook = FlaskWebhookHandler(request, os.environ.get('WEBHOOK_SECRET'))
    if webhook.is_challenge():
        return webhook.verify_challenge()
    else:
        s3_client: boto3.s3 = boto3.client('s3')
        for obj_details in webhook.process_request():
            if obj_details["entityType"] == EntityType.ASSET.value:
                buffer: io.BytesIO = client.thumbnail_bytes(obj_details["entityType"], obj_details["entityRef"], Thumbnail.MEDIUM)
                s3_client.put_object(Bucket="my_s3_bucket", Key=f"{obj_details["entityRef"]}/thumbnail.png", Body=buffer.getvalue())
                

    return webhook.response_ok()
```