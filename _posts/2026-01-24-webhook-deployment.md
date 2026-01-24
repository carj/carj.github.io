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

## Flask

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

## Zappa

Zappa is an open-source tool designed to "Lambda-fy" Python web applications. It allows you to deploy any WSGI-compatible application (like Flask) to AWS Lambda and API Gateway with almost zero code modification.

Essentially, Zappa acts as a bridge: it packages your entire Python project, handles the complex AWS infrastructure setup, and translates incoming API Gateway requests into a format your Flask application can understand.

