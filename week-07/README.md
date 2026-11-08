
# Week 07 — Serverless & Event-Driven Architecture

## 1. Request Path Diagram

```text

[ Client / curl ]
       │
       │  GET /properties/R100234
       ▼
[ Amazon API Gateway (HTTP API) ]
       │
       │  Route: GET /properties/{id} -> Triggers Lambda
       ▼
[ AWS Lambda: get-property (Python 3.12) ]
       │  IAM Role: Least-privilege (dynamodb:GetItem on Properties only)
       │  Client initialized outside handler for Warm Start reuse
       ▼
[ Amazon DynamoDB: Properties ]
          Partition Key: L_ListingID = "R100234"
```



## 2. SQS with a dead-letter queue

#### step 1. Dead-Letter Queue (DLQ)
```text
(.venv) sally@Mac IDX-Exchange-AWS-Cloud % aws sqs create-queue --queue-name property-dlq
{
    "QueueUrl": "https://sqs.us-east-1.amazonaws.com/580100736307/property-dlq"
}
```

#### step 2. Check ARN

```text
(.venv) sally@Mac IDX-Exchange-AWS-Cloud % aws sqs get-queue-attributes \
  --queue-url https://sqs.us-east-1.amazonaws.com/580100736307/property-dlq \
  --attribute-names QueueArn

{
    "Attributes": {
        "QueueArn": "arn:aws:sqs:us-east-1:580100736307:property-dlq"
    }
}
```

#### step 3. property-queue
```text
(.venv) sally@Mac IDX-Exchange-AWS-Cloud % aws sqs create-queue --queue-name property-queue --attributes '{"RedrivePolicy":"{\"deadLetterTargetArn\":\"arn:aws:sqs:us-east-1:580100736307:property-dlq\",\"maxReceiveCount\":\"3\"}"}'
{
    "QueueUrl": "https://sqs.us-east-1.amazonaws.com/580100736307/property-queue"
}
```

#### step 4. send-message
```text
(.venv) sally@Mac IDX-Exchange-AWS-Cloud % aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/580100736307/property-queue \
  --message-body '{"PropertyId":"P1002"}'
{
    "MD5OfMessageBody": "be3f822245bbcadebb0111ec27a7ed71",
    "MessageId": "6f7e864f-3a73-43bc-bf7a-100ece0d66a3"
}
```

#### step 5. receiving message
```text

(.venv) sally@Mac IDX-Exchange-AWS-Cloud % aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/580100736307/property-queue
{
    "Messages": [
        {
            "MessageId": "6f7e864f-3a73-43bc-bf7a-100ece0d66a3",
            "ReceiptHandle": "AQEBCh5SQIY3WpGWodiC/Fa5JAxCHuduwNPJSZOSJLE/usCY7mHhXrQxjgtH5sHKzsFLsKS+1JahdUx0JbwyqAvlEpSPVRgO1s4xrPVwIIbw/Mq8RzqbUFczhqGVR84kZC9saAh1SZQrkdnBNoM+uq4YdKEwliYAeLCCOzUiR9rKF+WqS29FHot2VgyTgiM3xmWIxgR90JEFQ7WKk9vYkOtZ7qTfyi7tecFmDHaZrAuP08Iv9hMoHmLNFnM39X6bK7mKeGrjUDxHzqY5qBN8rz0CmpSwNQNFN6SNmq9oOXVaqptV2z6cH0y6OqlOSvIyzaU06lPsdVqbpmVTIxZTItGNpIm01xNkmtNexHoo+ceA+tyMob/DFxxMnOCi39K8dZi6zt6w3yVDFRtNk8BuaHWc9w==",
            "MD5OfBody": "be3f822245bbcadebb0111ec27a7ed71",
            "Body": "{\"PropertyId\":\"P1002\"}"
        }
    ]
}
```



## 3. Debug
* Issue: API Gateway returned 500 Internal Server Error.


```text
(.venv) sally@Mac IDX-Exchange-AWS-Cloud % curl https://y3pwmmnbi5.execute-api.us-east-1.amazonaws.com/properties/R100234
{"message":"Internal Server Error"}%
```

* Diagnosis: Checked CloudWatch Logs (Lambda -> Monitor -> View CloudWatch logs) and identified TypeError: Object of type Decimal is not JSON serializable.

* Fix: Added default=str to json.dumps() in lambda_function.py and redeployed.

```text
(.venv) sally@Mac IDX-Exchange-AWS-Cloud % curl https://y3pwmmnbi5.execute-api.us-east-1.amazonaws.com/properties/R100234
{"L_SystemPrice": "450000", "L_ListingID": "R100234", "L_Keyword2": "3", "L_City": "Sacramento"}%
```