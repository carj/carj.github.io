---
layout: post
title: Automatic Deployment of Preservica Webhooks on AWS

---

### Introduction

In a previous [post](https://jcarr.org.uk/2023/06/10/webhooks/) I described how Preservica uses webhooks to allow the creation of custom business processes. 
At the end of the article I touched upon the challenges of hosting and securing webhook endpoints, and the manual effort required to deploy the supporting infrastructure.

This post describes a method to automate the process of creating the required [AWS services](https://aws.amazon.com), such as the AWS Lambda function and API Gateway.

### Background

We are going to use three Python projects: the web framework [Flask](https://flask.palletsprojects.com/en/stable/) to manage the application logic and create the web service which processes the messages from Preservica.  
The interaction with Preservica will be done using [pyPreservica](https://pypreservica.readthedocs.io/en/latest/)

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

pyPreservica is an open-source Python Software Development Kit (SDK) and client library designed to interact with the Preservica API. It is the primary tool for archivists, developers, and records managers.  
pyPreservica is used to convert the messages from Preservica into Assets that can be processed.

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

Within the activated environment, use the following command to install Flask:

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

Once you finish initialization, you'll have a file named `zappa_settings.json` in your project directory defining your basic deployment settings. It will probably look something like this for most WSGI applications:

```json
{
    // The name of your stage
    "dev": {
        // The name of your S3 bucket
        "s3_bucket": "lambda",
        "app_function": "your_module.app"
    }
}
```

Once your settings are configured, you can package and deploy your application to AWS with a single command:

```console
$ zappa deploy production
Deploying..
Your application is now live at: https://7k6anj0k99.execute-api.us-east-1.amazonaws.com/dev
```
