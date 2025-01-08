# Automating NBA Game Notifications with AWS Lambda, SNS, and EventBridge

## Day 2 #DevOpsAllStarsChallenge

In this project, we create a serverless solution using AWS services like AWS Lambda, SNS, and EventBridge to send real-time NBA game updates to your email.

Prerequisites
Free account with subscription and API Key at sportsdata.io
Personal AWS account with basic understanding of AWS and Python


## Key AWS Components Used
- **AWS Lambda**: Executes code to fetch NBA game data.
- **Amazon SNS**: Sends game updates to your email.
- **Amazon EventBridge**: Triggers Lambda based on a schedule.
- **IAM Policies**: Provides the necessary permissions to secure communication between services.

Project Structure

game-day-notifications/
├── policies/ 
│   ├── gd_sns_policy.json           # SNS publishing permissions
├── src/ 
│   └── gd_notifications.py          # Lambda function code
├── .gitignore
└── README.md                        # Project documentation


## Step-by-Step Implementation

Setup Instructions
Clone the Repository
To get started with the project, clone the repository and navigate to the project folder:

bash

git clone https://github.com/ifeanyiro9/game-day-notifications.git
cd game-day-notifications

### 1. Create an SNS Topic and Subscription
- Navigate to Amazon SNS in the AWS Console.
- Create a topic (choose the Standard type) and name it.
- Add a subscription to your topic. Choose Email (or SMS) as the protocol and provide your email address as the endpoint.
- Confirm the subscription by clicking the link sent to your email.

### 2. Set Up an IAM Policy
- Create a policy in IAM to allow the SNS topic to publish messages.
1 Open the IAM service in the AWS Management Console.
2 Navigate to Policies → Create Policy.
3 Click JSON and paste the JSON policy from gd_sns_policy.json file
4 Replace REGION and ACCOUNT_ID with your AWS region and account ID.
5 Click Next: Tags (you can skip adding tags).
6 Click Next: Review.
7 Enter a name for the policy (e.g., gd_sns_policy).
8 Review and click Create Policy.


- Use the following JSON in the policy editor (replace `TOPIC_ARN` with your actual SNS topic ARN):
  
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:REGION:ACCOUNT_ID:TOPIC_NAME"
    }
  ]
}

3. Create an IAM Role for Lambda
Create an IAM role for Lambda with the above policy and the AWSLambdaBasicExecutionRole for logging and monitoring.

- Open the IAM service in the AWS Management Console.
- Click Roles → Create Role.
- Select AWS Service and choose Lambda.
- Attach the following policies:
 - SNS Publish Policy (gd_sns_policy) (created in the previous step).
 - Lambda Basic Execution Role (AWSLambdaBasicExecutionRole) (an AWS managed policy).

- Click Next: Tags (you can skip adding tags).
- Click Next: Review.
- Enter a name for the role (e.g., gd_role).
- Review and click Create Role.
- Copy and save the ARN of the role for use in the Lambda function.

4. Create the Lambda Function
- In AWS Lambda, create a function using Python (e.g., Python 3.13).
- Open the AWS Management Console and navigate to the Lambda service.
- Click Create Function.
- Select Author from Scratch.
- Enter a function name (e.g., gd_notifications).
- Choose Python 3.x as the runtime.
- Assign the IAM role created earlier (gd_role) to the function.
- Under the Function Code section:

- Copy the content of the src/gd_notifications.py file from the repository.
- Paste it into the inline code editor.

python

import os
import json
import urllib.request
import boto3
from datetime import datetime, timedelta, timezone

def format_game_data(game):
    # Format game details based on status (Final, Scheduled, InProgress)
    pass

def lambda_handler(event, context):
    api_key = os.getenv("NBA_API_KEY")
    sns_topic_arn = os.getenv("SNS_TOPIC_ARN")
    sns_client = boto3.client("sns")
    
    # Fetch today's game data
    today_date = (datetime.now(timezone.utc) - timedelta(hours=6)).strftime("%Y-%m-%d")
    api_url = f"https://api.sportsdata.io/v3/nba/scores/json/GamesByDate/{today_date}?key={api_key}"
    
    try:
        with urllib.request.urlopen(api_url) as response:
            data = json.loads(response.read().decode())
    except Exception as e:
        print(f"Error: {e}")
        return {"statusCode": 500, "body": "API Error"}
    
    # Process and send game updates
    messages = [format_game_data(game) for game in data]
    message = "\n---\n".join(messages) if messages else "No games today."
    try:
        sns_client.publish(TopicArn=sns_topic_arn, Message=message, Subject="NBA Game Updates")
    except Exception as e:
        print(f"SNS Error: {e}")
        return {"statusCode": 500, "body": "SNS Error"}
    
    return {"statusCode": 200, "body": "Success"}

Add environment variables for NBA_API_KEY and SNS_TOPIC_ARN in the function's configuration tab.


5. Test the Lambda Function
Create a test event and execute the function.
If successful, you’ll receive NBA game updates via email.

6. Schedule Lambda with EventBridge
In EventBridge, create a rule to trigger Lambda periodically.

- Navigate to the Eventbridge service in the AWS Management Console.
- Go to Rules → Create Rule.
- Select Event Source: Schedule.
- Set the cron schedule for when you want updates (e.g., hourly).define the schedule (e.g., every 5 minutes).
- Under Targets, select the Lambda function (gd_notifications) and save the rule.

Test the System
- Open the Lambda function in the AWS Management Console.
- Create a test event to simulate execution.
- Run the function and check CloudWatch Logs for errors.
- Verify that SMS notifications are sent to the subscribed users.

Conclusion
This project demonstrates how to build a serverless, scalable NBA game notification system with AWS Lambda, SNS, and EventBridge. It's a cost-effective solution for real-time sports updates.

- What We Learned
- Designing a notification system with AWS SNS and Lambda.
- Securing AWS services with least privilege IAM policies.
- Automating workflows using EventBridge.
- Integrating external APIs into cloud-based workflows.


