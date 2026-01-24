---
layout: post
title: Automatic Deployment of Webhooks on AWS

---

### Introduction

In a previous [post](https://jcarr.org.uk/2023/06/10/webhooks/) I described how Preserviaca uses webhooks to allow the creation of custom buisness processes. At the end of the article I touched upon the deployment of webhooks within AWS and showed how serveless event driven cloud services are ideal for these kind of applications.
This post describes a method to automate the process of creating the required [AWS services](https://aws.amazon.com) such as the AWS Lambda function and API Gateway.


### Background

We are going to use the Python web framework [Flask](https://flask.palletsprojects.com/en/stable/) to manage the application logic and create the web service which processes the messages from Preservica. The deployment of the Flask aplication to AWS including the creation of a Lambda function and the API Gateway will be done using [Zappa](https://github.com/zappa/Zappa).






