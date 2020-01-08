---
layout: post
title:  "AWS S3+Athena real-time business analytics"
date:   2020-01-07 17:23:49 +0200
permalink:   athena-analytics
categories: [architecture, aws, analytics]
tags: [architecture, metrics, aws, s3, athena]
---

Business analytics is crucial:
* It provides business people a view on the current status of your software.
* It is the key to the data-driven door <sub>oh, what a pun</sub>
 
To make business analytics possible we need data, data that represents the right business metrics for the software (KPIs).
Business metrics can be stored in a database, logs, files or a dedicated warehouse.

In this article I’d like to show you real-time business analytics in AWS S3 using AWS Athena.

**AWS S3** is a simple object storage service. 
It is highly available (99.9%) and durable ( 99.999999999%).

**AWS Athena** is a service to query data (basically files with records) in S3 using SQL.
Athena supports querying CSV, JSON, Apache Parquet data formats.
* It’s serverless! You don't need to set up or maintain hosts/databases.
* You pay per query! 1TB of scanned data = 5$
* You can use it with different business intelligence or SQL clients.

How Athena works?
Athena uses **Presto** under the hood.
[What is Presto?](https://aws.amazon.com/big-data/what-is-presto/)

![bookstore](/assets/posts/athena-analytics/bookstore.jpg)

Getting back to our topic, let's imagine we have an online bookstore and we need to analyze books purchases that are processed by *PurchaseService*. 
Let's define our purchase metrics metadata:
```javascript
{
    "bookId": "ec437d98-455d-4fec-8dbe-2c2630454bdd", 
    "title": "Orlando", 
    "author": "Virginia Woolf", 
    "genre": "Fiction", 
    "userId": "test-user", 
    "userCountry": "country", 
    "userAge": 24, 
    "purchaseTimestamp": 1575121641
}
```
Good enough to make different kind of business analytics.

Now we can publish our purchase metrics directly to S3, but **AWS Kinesis Firehose** is better and here is why:
* Firehose buffers incoming records and delivers in batches.
* Firehose can convert the records to another data format, for example Apache Parquet (which is much more efficient for Athena querying - 1TB of JSON records reducing down to 130GB, meaning faster and cheaper querying) 
* Firehose can compress the data (gzip, snappy, etc.)

Let's create a Firehose stream in AWS Console called *books-purchase-stream* that delivers data to S3.
*PurchaseService* is a NodeJS AWS Lambda Function and it will publish purchase events (purchase metrics format we defined recently) to *books-purchase-stream*.
```javascript
const AWS = require('aws-sdk');
const firehose = new AWS.Firehose();
const firehoseStream = "books-purchase-stream"

exports.metricsPublisher = function(event, context) {
    const purchaseRecord = JSON.stringify(event);
    const firehoseRecord = {
        DeliveryStreamName: firehoseStream,
        Record:  {
            Data: purchaseRecord
       }
    };
    firehose.putRecord(firehoseRecord, function(error, data) {
        if (error) {
            console.log(error, error.stack);
        }
        else {
            console.log(data);
        }
        context.done();
    });
};
```

Now we have the metrics stored in S3, how do we query it?

Athena is integrated with AWS Glue Data Catalog and requires a Glue database and a Glue table for querying. 
**AWS Glue Data Catalog** is a metadata repository, a Glue table is the data model (schema) and a Glue database contains tables.

To create a Glue database and a table with our purchase metrics metadata we’re gonna use a **Glue Crawler**.
Point a Glue Crawler to the data in S3 and the crawler will extract the metadata into AWS Glue Data Catalog.

The flow we've created so far:
![AWS S3 + Athena real-time business analytics](/assets/posts/athena-analytics/athena-analytics.png)

The purchase metrics are in AWS S3 and purchase metrics metadata in AWS Glue Catalog, can we query it now?

Yes! Let’s go to Athena and write a simple query:

![Athena query](/assets/posts/athena-analytics/athena-results.png)

### Conclusion
In the end we have a simple yet powerful serverless real-time business analytics infrastructure.

### References
1. [https://docs.aws.amazon.com/athena/latest/ug/what-is.html](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)
2. [https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html]([https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html])
3. [https://aws.amazon.com/premiumsupport/knowledge-center/error-json-athena/]([https://aws.amazon.com/premiumsupport/knowledge-center/error-json-athena/])
4. [https://docs.aws.amazon.com/glue/latest/dg/populate-data-catalog.html](https://docs.aws.amazon.com/glue/latest/dg/populate-data-catalog.html)


