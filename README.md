Game highlight Processor
This project leverages RapidAPI to obtain NCAA game highlights using a Docker container and AWS MediaConvert for media file conversion.

File Overview
config.py:

Imports necessary environment variables and assigns them to Python variables with default values for flexible configuration management across different environments (e.g., development, staging, production).

fetch.py:

Establishes the date and league (NCAA for this example) to find highlights.

Fetches highlights from the API and stores them in an S3 bucket as a JSON file (basketball_highlight.json).

process_one_video.py:

Connects to the S3 bucket and retrieves the JSON file.

Extracts the first video URL from the JSON file.

Downloads the video file from the internet into memory using the requests library.

Saves the video as a new file in the S3 bucket under videos/.

Logs the status of each step.

mediaconvert_process.py:

Creates and submits a MediaConvert job.

Configures video codec, resolution, bitrate, and audio settings.

Stores the processed video back into the S3 bucket.

run_all.py:

Runs the scripts sequentially and provides buffer time for task creation.

.env:

Stores all environment variables to avoid hardcoding them into scripts.

Dockerfile:

Defines the steps to build the Docker image.

Terraform Scripts:

Used to create resources in AWS in a scalable and repeatable way (e.g., S3, IAM roles, Elastic Registry Service, Elastic Container Services).

Prerequisites
Before running the scripts, ensure you have the following:

Create RapidAPI Account:

A RapidAPI.comaccount is needed to access highlight images and videos.

For this example, NCAA (USA College Basketball) highlights are used as they are included in the free plan.

Verify Prerequisites:

Docker: docker --version

AWS CloudShell with AWS CLI: aws --version

Python3: python3 --version

Retrieve AWS Account ID:

Log in to the AWS Management Console.

Click on your account name in the top right corner to find your Account ID.

Save this ID as you will need it later.

Retrieve Access Keys and Secret Access Keys:

Check the IAM dashboard under "Users" and then "Security Credentials."

If you don't have an access key, create one.


Technical Diagram

Project Structure

  src/
├── Dockerfile
├── config.py
├── fetch.py
├── mediaconvert_process.py
├── process_one_video.py
├── requirements.txt
├── run_all.py
├── .env
├── .gitignore
└── terraform/
    ├── main.tf
    ├── variables.tf
    ├── secrets.tf
    ├── iam.tf
    ├── ecr.tf
    ├── ecs.tf
    ├── s3.tf
    ├── container_definitions.tpl
    └── outputs.tf

Getting Started - Local
git clone https://github.com/alahl1/NCAAGameHighlights.git
cd src


Add API Key to AWS Secrets Manager:
    aws secretsmanager create-secret \
    --name my-api-key \
    --description "API key for accessing the Sport Highlights API" \
    --secret-string '{"api_key":"YOUR_ACTUAL_API_KEY"}' \
    --region us-east-1


Create an IAM Role or User:
    Search "IAM" in the AWS Management Console.
    Click "Roles" -> "Create Role".
    For the Use Case, enter "S3" and click "Next".
    Add permissions: AmazonS3FullAccess, MediaConvertFullAccess, and AmazonEC2ContainerRegistryFullAccess.
    Name the role "HighlightProcessorRole" and create it.

Edit the trust policy to:
    {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "ecs-tasks.amazonaws.com",
          "mediaconvert.amazonaws.com"
        ],
        "AWS": "arn:aws:iam::<your-account-id>:user/<your-iam-user>"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

Update the env.list file with the following
  RapidAPI_KEY=your_rapidapi_key
  AWS_ACCESS_KEY_ID=your_aws_access_key_id
  AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key
  S3_BUCKET_NAME=your_s3_bucket_name
  MEDIACONVERT_ENDPOINT=https://your_mediaconvert_endpoint.amazonaws.com
  MEDIACONVERT_ROLE_ARN=arn:aws:iam::your_account_id:role/HighlightProcessorRole

Secure env.list file buy doing the following
    chmod 600 .env

Build and Run the Docker Container locally
    docker build -t highlight-processor .
    docker run --env-file env.list highlight-processor


This will run fetch.py, process_one_video.py, and mediaconvert_process.py. The following files should be saved in your S3 bucket:
Confirm there is a video uploaded to s3:///videos/first_video.mp4.
Confirm there is a video uploaded to s3:///processed_videos/.

What We Learned
    Working with Docker and AWS Services.
    Identity Access Management (IAM) and least privilege.
    Enhancing media quality.


