# Event-Driven-Architecture
Event driven Architecture with S3, SNS, SQS, and Lambda. Uses Events to decouple an application's components.
- loosely coupled applications
- add new features without changing existing applications
- Scale and fail components independently.

I. Create the s3 bucket
II. Create an SNS topic
- create the topic and change the access policy: copy your account number, s3 bucket name, and SQS queues.

   {
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:<region>:<accountId>:<snsTopicName>",
      "Condition": {
        "StringEquals": {
          "aws:SourceAccount": "<accountId>"
        },
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:*:*:<s3BucketName>"
        }
      }
    },
    {
      "Sid": "sqs_statement",
      "Effect": "Allow",
      "Principal": {
        "Service": "sqs.amazonaws.com"
      },
      "Action": "sns:Subscribe",
      "Resource": "arn:aws:sns:<region>:<accountId>:<snsTopicName>",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": [
            "arn:aws:sqs:<region>:<accountId>:<sqsQueueName>",
            "arn:aws:sqs:<region>:<accountId>:<sqsQueueName>"
          ]
        }
      }
    }
  ]
}
  
III. Create SQS Quere
- event stream: create the queue, and change the access policy:

  {
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "Stmt1234",
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:sendMessage"
      ],
      "Resource": "arn:aws:sqs:<region>:<accountId>:<sqsQueueName>",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "arn:aws:lambda:<region>:<accountId>:<lambdaName>"
        }
      }
    },
    {
      "Sid": "Stmt12345",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:<region>:<accountId>:<sqsQueueName>",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:sns:<region>:<accountId>:<snsTopicName>"
        }
      }
    }
  ]



IV. Create the Lambda Functions
- create two lambda policies to allow access to SQS and CloudWatch


{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "sqs:DeleteMessage",
                "logs:CreateLogStream",
                "sqs:ReceiveMessage",
                "sqs:GetQueueAttributes",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:sqs:<region>:<accountId>:q1",
                "arn:aws:logs:<region>:<accountId>:log-group:/aws/lambda/<lambdaName>:*"
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "sqs:ReceiveMessage",
                "logs:CreateLogGroup"
            ],
            "Resource": [
                "arn:aws:logs:<region>:<accountId>:*",
                "arn:aws:sqs:<region>:<accountId>:<sqsQueueName>"
            ]
        }
    ]
}



- Create two roles for each lambda function and then attach each policy to the corresponding lambda role.

- Create the two lambda functions, and let's print the data


V. Connect the four components
- connect the s3 bucket to the SNS by creating an event notification at the s3 bucket level.
- connect SQS and the SNS by creating a subscription at the SNS level (with a filter policy). NB: create two subscriptions for each queue.
- Finally, connect SQS queues to the lambda functions at the SQS level.

VI. Test Setup



