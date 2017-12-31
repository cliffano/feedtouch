<img align="right" src="https://raw.github.com/cliffano/feedpaper/master/avatar.jpg" alt="Avatar"/>

[![Build Status](https://img.shields.io/travis/cliffano/feedpaper.svg)](http://travis-ci.org/cliffano/feedpaper)
<br/>

Feedpaper
---------

Feedpaper is a feed reader + readability mashup for reading on handheld devices.

This is handy for anyone who wants to speed-read the articles from a personalised list of web site feeds.

This is an experimental project using serverless architecture with App.js, Terraform, and Amazon Web Services.

Architecture
------------

[![Architecture Diagram](https://raw.github.com/cliffano/feedpaper/master/architecture.jpg)](https://raw.github.com/cliffano/feedpaper/master/architecture.jpg)

| Component      | Description                                                                        |
|----------------|------------------------------------------------------------------------------------|
| feedpaper-web  | Single page App.js web app served as AWS S3 static website                         |
| feedpaper-api  | Content API using AWS API Gateway, and content fetcher using AWS Lambda            |
| feedpaper-data | Content storage using AWS DynamoDB with scheduled content expirer using AWS Lambda |

Installation
------------

Set AWS resources:

* Create an S3 bucket for storing Terraform state files
* Create an IAM group with the following managed policies:
    * AWSLambdaFullAccess
    * IAMFullAccess
    * AmazonS3FullAccess
    * AmazonAPIGatewayInvokeFullAccess
    * AmazonDynamoDBFullAccess
    * AWSLambdaDynamoDBExecutionRole
    * AmazonAPIGatewayAdministrator

Download Feedpaper code:

    git clone https://github.com/cliffano/feedpaper

Configuration
-------------

Create `feedpaper.json`, `feeds.json`, `terraform.tfvars`, `backend-feedpaper-api.tf`, `backend-feedpaper-data.tf`, and  `backend-feedpaper-web.tf` files under a designated configuration directory (e.g. `/path/to/conf_dir`).
Have a look at [conf/ci](https://github.com/cliffano/feedpaper/tree/master/conf/ci) for example configuration files.

Set up the following environment variables:

    export FEEDPAPER_ENV=local
    export FEEDPAPER_CFG=/path/to/conf_dir
    export TF_CFG_BUCKET=<aws_s3_bucket_name>
    export TF_CFG_REGION=<aws_region>

Ensure domain name is configured in [Typekit's kit](https://typekit.com/account/kits) setting, and publish the kit, and republish. Wildcard domains no longer work at least since last tested in mid 2017.

Usage
-----

Set up content storage:

    cd feedpaper-data && make all

Set up content API:

    cd feedpaper-api && make all

This sometimes fails because Terraform doesn't allow configurable time out for waiting for Lambda functions to be ready.
This will also cause Terraform to fail to remove Lambda functions when destroying the infrastructure.

Get the REST API ID from content API output:

    aws_api_gateway_deployment.api_deployment: Creating...
      rest_api_id: "" => "<some_rest_api_id>"

Set this REST API ID in feedpaper.json's api.host property:

    {
      "api": {
        "protocol": "https",
        "host": "<some_rest_api_id>.execute-api.<aws_region>.amazonaws.com",
        "version": "0",
        "path": "feedpaper-<env>"
      },
      "database": {
        "table": "feedpaper-<env>"
      }
    }

Set up single page web app:

    cd feedpaper-web && make all

Colophon
--------

Articles:

* [Do You Find Websites Hard To Read On The iPhone? Try FeedTouch](http://blog.cliffano.com/2011/02/19/do-you-find-websites-hard-to-read-on-the-iphone-try-feedtouch/)
