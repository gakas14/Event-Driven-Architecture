# Event-Driven-Architecture
Event driven Architecture with S3, SNS, SQS, and Lambda. Uses Events to decouple an application's components.
- loosely coupled applications
- add new features without changing existing applications
- Scale and fail components independently.

I. Create the s3 bucket

<img width="1266" alt="Screen Shot 2023-12-06 at 8 54 44 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/9cc01c6a-4a92-414b-901c-9db2c5245de2">




II. Create an SNS topic
- create the topic and change the access policy: copy your account number, s3 bucket name, and SQS queues.
  
<img width="1276" alt="Screen Shot 2023-12-06 at 3 42 37 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/bfcbaa39-aa10-447d-9376-c58fbd17d13c">



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

<img width="1280" alt="Screen Shot 2023-12-06 at 3 53 08 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/5b4eb5a8-4072-459a-a062-1e6d1b59c053">



- event stream: create the queue and change the access policy:

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


  
<img width="1274" alt="Screen Shot 2023-12-06 at 4 29 11 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/a6e149d1-7e77-4935-9152-ff114d7c5234">



- Create the two lambda functions, and let's print the data

<img width="1279" alt="Screen Shot 2023-12-06 at 8 48 06 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/a37e1d04-8196-4c4b-95e8-a62a153bd526">


<img width="1265" alt="Screen Shot 2023-12-06 at 8 48 35 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/5dd24113-914e-4c34-a4b3-079e12677d55">


V. Connect the four components
- connect the s3 bucket to the SNS by creating an event notification at the s3 bucket level: select "All object create events" as the event type and the SNS topic as a destination

  
<img width="1271" alt="Screen Shot 2023-12-06 at 8 55 40 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/95171796-8e87-41db-86cc-a38a4d3bf5c0">


  
- connect SQS and the SNS by creating a subscription at the SNS level (with a filter policy). NB: create two subscriptions for each queue.
  The filter policy: we filter the message based on the event name:
 The put event will be forwarded to queue 1

  <img width="1270" alt="Screen Shot 2023-12-06 at 8 57 19 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/44675871-3ec9-416a-863e-caefa6dfc573">


	  {
	  "Records": {
	    "eventName": [
	      "ObjectCreated:Put"
	    ]
	  }
	  }


  
 The copy event will be forwarded to queue 2

 
<img width="1277" alt="Screen Shot 2023-12-06 at 9 00 56 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/8d68de81-e951-4de5-a12b-bb311815848b">

	  {
	  "Records": {
	    "eventName": [
	      "ObjectCreated:Copy"
	    ]
	  }
	  }
  
- Finally, connect SQS queues to the lambda functions at the SQS level.


  
<img width="1277" alt="Screen Shot 2023-12-06 at 9 05 49 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/16005b50-eeb8-4122-adb3-389d27fa5017">



VI. Test Setup
- put an  object in the s3 bucket, and let's check the Cloudwatch log into the lambda function 1


- <img width="1279" alt="Screen Shot 2023-12-06 at 9 48 59 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/6fb82078-d725-4c70-930e-d5efe0aa2236">


<img width="1280" alt="Screen Shot 2023-12-06 at 10 07 19 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/993846ee-7d5c-4116-8f44-8a3be1241515">



- copy the same file to check the second lambda log

  
<img width="1280" alt="Screen Shot 2023-12-06 at 10 07 19 PM" src="https://github.com/gakas14/Event-Driven-Architecture/assets/74584964/b0d97f74-035d-4ad4-af4d-ebe5ef06b882">


