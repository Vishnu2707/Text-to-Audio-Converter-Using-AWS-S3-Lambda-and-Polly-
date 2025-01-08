# Text to Audio Converter Using AWS (S3, Lambda, and Polly)

## Project Overview

The **Text to Audio Converter** leverages AWS services to convert text files (such as blog posts, articles, newsletters, or book excerpts) into speech. This project demonstrates a serverless architecture that processes text files uploaded to an Amazon S3 bucket and converts them into audio files using AWS Polly. The output is stored in a destination S3 bucket, making it accessible for users.

This solution adds value in **infrastructure management services** by showcasing the power of AWS serverless services for automation, scalability, and efficiency.

---

## Use Cases

1. **Enhanced Knowledge Sharing:** Organizations can distribute internal documentation, policies, and knowledge bases as audio files, ensuring employees have easier access while multitasking.
2. **Accessibility Compliance:** IT companies can ensure inclusivity by providing audio versions of content for employees or customers with visual impairments.
3. **Automated Reporting:** Convert textual status reports or system logs into audio formats for executive reviews during meetings or on the go.
4. **Employee Engagement:** Deliver company newsletters or announcements in audio formats, enhancing engagement and consumption.

---

## Architecture

### Overview

- **Source S3 Bucket:** Stores the text files to be converted.
- **S3 Event Notifications:** Triggers the Lambda function when a new text file is uploaded.
- **AWS Lambda:** Processes the text file, converts it into speech using Polly, and uploads the audio file to the destination bucket.
- **Amazon Polly:** Synthesizes speech from the text file.
- **Destination S3 Bucket:** Stores the generated audio files.

---

## Step-by-Step Guide

### Step 1: Set Up an AWS Account

Ensure you have an active AWS account to access the necessary services.

### Step 2: Create S3 Buckets

- **Source Bucket Name:** `source-polly`
- **Destination Bucket Name:** `destination-polly`

### Step 3: Configure IAM Policy

Create an IAM policy with the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Action": [
              "s3:GetObject",
              "s3:PutObject"
          ],
          "Resource": [
              "arn:aws:s3:::source-polly/*",
              "arn:aws:s3:::destination-polly/*"
          ]
      },
      {
          "Effect": "Allow",
          "Action": [
              "polly:SynthesizeSpeech"
          ],
          "Resource": "*"
      }
  ]
}
```

### Step 4: Create IAM Role

- Create a role named `IAM-polly`.
- Attach the following policies:
  - `IAM-polly` (custom policy created above)
  - `AWSLambdaBasicExecutionRole`

### Step 5: Configure the Lambda Function

1. **Name:** `TtoSFunction`
2. **Runtime:** Python 3.9
3. **Execution Role:** Use the role created in Step 4.
4. **Environment Variables:**
   - `SOURCE_BUCKET`: `source-polly`
   - `DESTINATION_BUCKET`: `destination-polly`

### Step 6: Configure S3 Event Notification

Set up an event notification on the source bucket to trigger the Lambda function when a `.txt` file is uploaded.

### Step 7: Lambda Function Code

Deploy the following Python code in your Lambda function:

```python
import boto3
import json
import os
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    polly = boto3.client('polly')

    source_bucket = os.environ['SOURCE_BUCKET']
    destination_bucket = os.environ['DESTINATION_BUCKET']

    text_file_key = event['Records'][0]['s3']['object']['key']
    audio_key = text_file_key.replace('.txt', '.mp3')

    try:
        logger.info(f"Retrieving text file from bucket: {source_bucket}, key: {text_file_key}")
        text_file = s3.get_object(Bucket=source_bucket, Key=text_file_key)
        text = text_file['Body'].read().decode('utf-8')

        logger.info(f"Sending text to Polly for synthesis")
        response = polly.synthesize_speech(
            Text=text,
            OutputFormat='mp3',
            VoiceId='Joanna'  # Choose the voice you prefer
        )

        if 'AudioStream' in response:
            temp_audio_path = '/tmp/audio.mp3'
            with open(temp_audio_path, 'wb') as file:
                file.write(response['AudioStream'].read())

            logger.info(f"Uploading audio file to bucket: {destination_bucket}, key: {audio_key}")
            s3.upload_file(temp_audio_path, destination_bucket, audio_key)

        logger.info(f"Text-to-Speech conversion completed successfully for file: {text_file_key}")

        return {
            'statusCode': 200,
            'body': json.dumps('Text-to-Speech conversion completed successfully!')
        }

    except Exception as e:
        logger.error(f"Error processing file {text_file_key} from bucket {source_bucket}: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps('An error occurred during the Text-to-Speech conversion.')
        }
```

### Step 8: Test the System

Upload a `.txt` file to the source bucket and verify that the corresponding `.mp3` file is generated in the destination bucket.

---

## Key Learnings

- **Serverless Architecture:** Utilizing AWS Lambda, S3, and Polly reduces infrastructure overhead and simplifies scalability.
- **Event-Driven Design:** Automating workflows with S3 event notifications improves efficiency and reduces manual intervention.
- **Cost Efficiency:** AWS serverless services ensure cost-effectiveness by charging only for resources consumed during execution.
- **Enhanced Accessibility:** Improves inclusivity and user experience by providing alternative formats of content.

---

## Benefits in Infrastructure Management Services

1. **Automation:** Automates content processing pipelines, saving time and reducing errors.
2. **Scalability:** Easily handles varying workloads with AWS's auto-scaling capabilities.
3. **Resilience:** Leverages S3's durability and availability for data storage.
4. **Innovation:** Demonstrates how cloud services can enhance accessibility, efficiency, and engagement.

---

## Conclusion

This project is a practical example of leveraging AWS services to create an innovative, accessible, and scalable solution for converting text content into audio. It highlights the value of serverless technologies in streamlining processes and delivering impactful solutions.
```
