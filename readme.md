# Birthday Wisher Lambda

I was studying for AWS Cloud Practitioner and uhhh, got bored.

Automatically sends personalised birthday wishes using OpenAI. 

Reads birthdays from S3, generates messages via OpenAI (dw there's a fallback if you didn't pay the piper), and sends emails.

## Setup

1. **AWS Resources Required**:
   - S3 bucket with `birthdays.csv` file (required headers: name,email,day,month,sarcastic)
   - The 'sarcastic' header in the `birthdays.csv` file is a column that is either `true` or `false` and determines whether that friend is going to get a sarcastic birthday greeting or not.
   - SSM Parameters:

```
OPENAI_API_KEY
SENDER_EMAIL
EMAIL_PASSWORD
```

2. **Lambda Configuration**:
   - Runtime: Python 3.12
   - Handler: `src.birthday_wisher.birthday_wisher.lambda_handler` (Scroll down to runtime settings in the new UI, it's not in Configuration)
   - Memory: 256MB
   - Timeout: 5 minutes
   - Environment Variables:

```
BUCKET_NAME=your-bucket-name
FILE_KEY=birthdays.csv
```


## AWS Setup Steps

1. **Create S3 Bucket**:
   - Go to S3 in AWS Console
   - Click "Create bucket"
   - Choose a unique name
   - Keep default settings
   - Create a CSV file with headers: `name,email,day,month,sarcastic`
   - Upload to bucket as `birthdays.csv`

2. **Create SSM Parameters**:
   - Go to AWS Systems Manager → Parameter Store
   - Click "Create parameter" for each:

```
Name: OPENAI_API_KEY 
Type: SecureString 
Value: your-openai-key

Name: SENDER_EMAIL 
Type: SecureString 
Value: your-email-address

Name: EMAIL_PASSWORD 
Type: SecureString 
Value: your-app-password

// OBVIOUSLY - you will need to set up an email account that you can access programatically
```

3. **Set up EventBridge**:
   - Go to Amazon EventBridge
   - Click "Create rule"
   - Name it (e.g., "birthday-wisher-daily")
   - For Schedule pattern, use cron: `0 0 * * ? *` (runs daily at midnight UTC)
   - Target: Select your Lambda function
   - Create rule

## Deployment

Run `create_deployment_package.py` to create a Lambda deployment package:

## Deployment

Run `create_deployment_package.py` to create a Lambda deployment package:

```
python create_deployment_package.py
```

Upload the generated `deployment-package.zip` to Lambda.

## Testing

### Local Tests
Requires `.env` file with:

```
OPENAI_API_KEY=your-key
SENDER_EMAIL=your-email
EMAIL_PASSWORD=your-password
BUCKET_NAME=your-bucket
FILE_KEY=birthdays.csv
```

Run local tests:

```
python -m src.test.test_runner
```


### Lambda Integration Tests
Test AWS integration in Lambda by creating a test event:

```
{
    "test_mode": true
}
```


## Normal Operation
Lambda runs daily via EventBridge schedule to check for and send birthday wishes.
