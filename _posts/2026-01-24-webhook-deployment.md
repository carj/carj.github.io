---
layout: post
title: Automatic Deployment of Webhooks on AWS

---

### Introduction

In a previous [post](https://jcarr.org.uk/2023/06/10/webhooks/) I described how Preserviaca uses webhooks to allow the creation of custom buisness processes. At the end of the article I touched upon the deployment of webhooks within AWS and showed how serveless event driven cloud services are ideal for these kind of applications. Deploying to AWS can be complicated as a number of AWS services are required to work together.

This post describes a method to automate the process of creating the required [AWS services](https://aws.amazon.com) such as the AWS Lambda function and API Gateway.


### Background

We are going to use 3 python projects, the web framework [Flask](https://flask.palletsprojects.com/en/stable/) to manage the application logic and create the web service which processes the messages from Preservica. The deployment of the Flask aplication to AWS including the creation of a Lambda function and the API Gateway will be done using [Zappa](https://github.com/zappa/Zappa). 
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
This simplicity makes it ideal for creating the buisness logic of the webhook service. 
The code which handles the webhook messages will be within the Flask application.


#### Zappa

Zappa is an open-source tool designed to "Lambda-fy" Python web applications. It allows you to deploy any WSGI-compatible application (like Flask) to AWS Lambda and API Gateway with almost zero code modification.

Essentially, Zappa acts as a bridge: it packages your entire Python project, handles the complex AWS infrastructure setup, and translates incoming API Gateway requests into a format your Flask application can understand.

By using Zappa you no longer need to login to AWS and deploy Lambda functions, API Gateways, IAM roles and all the AWS resources required to setup the serverless environment. No AWS knowledge is required.

#### pyPreservica

pyPreservica is an open-source Python Software Development Kit (SDK) and client library designed to interact with the Preservica API. It is the primary tool for archivists, developers, and records managers who want to automate digital preservation tasks or integrate Preservica with other systems.
pyPreservica is used to convert the messages from Preservica into Assets which can be processed.




