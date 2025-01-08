# Automating NBA Game Notifications with AWS Lambda, SNS, and EventBridge

## Day 2 #DevOpsAllStarsChallenge

In this project, we create a serverless solution using AWS services like AWS Lambda, SNS, and EventBridge to send real-time NBA game updates to your email.

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
3. Create an IAM Role
Create an IAM role for Lambda with the above policy and the AWSLambdaBasicExecutionRole for logging and monitoring.

4. Create the Lambda Function
In AWS Lambda, create a function using Python (e.g., Python 3.13).
Attach the IAM role created above.
Add the following Python script to fetch NBA game data and send it via SNS:

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
Use a cron expression to define the schedule (e.g., every 5 minutes).


Conclusion
This project demonstrates how to build a serverless, scalable NBA game notification system with AWS Lambda, SNS, and EventBridge. It's a cost-effective solution for real-time sports updates.

