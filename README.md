## 🤖 Generative AI: Text-to-Image Application with AWS

A production-ready Generative AI application that converts text descriptions into images using Stable Diffusion models hosted on Amazon SageMaker, with a serverless backend and CDN-hosted frontend.

## 📋 Table of Contents
- [Overview](#Overview)
- [Architecture](#Architecture)
- [Technologies Used](#Technologies-Used)
- [Setup Instructions](#Setup-Instructions)
- [Configuration](#Configuration)
- [Deployment Steps](#Deployment-Steps)
- [Troubleshooting](#Troubleshooting)
- [Monitoring](#Monitoring)

## 📖 Overview
This project implements a complete Generative AI pipeline using AWS services. It converts text prompts into high-quality images using Stable Diffusion models deployed on SageMaker, with a serverless backend (Lambda + API Gateway) and a CloudFront-hosted web interface.

## 🏗️ Architecture
```text
User Interface (CloudFront + S3) → API Gateway → Lambda Functions → SageMaker Endpoint → S3 Storage
        ↑                              ↑              ↑                     ↑
    index.html                  REST API           Processing          Stable Diffusion
    (Frontend)                (POST/GET)         (3 Lambda Funcs)       (AI Model)
```

## 🛠️ Technologies Used

```
Service	Purpose	Key Features
Amazon SageMaker	AI/ML model hosting	Stable Diffusion model deployment
AWS Lambda	Serverless compute	Image processing functions
API Gateway	REST API management	HTTP endpoint for frontend
Amazon S3	Storage	Image storage & web hosting
AWS CloudFront	Content Delivery	Global distribution of web app
AWS IAM	Access management	Role-based permissions
```

## 🚀 Setup Instructions
### Prerequisites
AWS Account with appropriate permissions
SageMaker Studio access
Basic understanding of AWS services
Python 3.10+ knowledge

#### Phase 1: SageMaker Setup
1. Create SageMaker Studio User
```bash
# Navigate to SageMaker Console
# Go to Studio → Create user with default settings
# Launch Studio with ml.t3.large instance (25GB storage)
```

2. Configure SageMaker Execution Role
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

3. Import and Run Jupyter Notebook
```python
# Import jupyter_notebook.ipynb
# Run code from Step 1 to Step 4
# Continue with remaining sections
```

#### Phase 2: Lambda Functions Setup
1. Create Lambda Layer (Pillow)
```bash
# Create PillowLayer.zip with required dependencies
# Upload to Lambda Layers
# Runtime: Python 3.10 x86_64
```

2. Create Three Lambda Functions
```
Function 1: Endpoint_Call_Function
Runtime: Python 3.10
Memory: 512MB
Timeout: 5 minutes
Environment Variables:
ENDPOINT_NAME: cloudage-endpoint-text-to-image-model-t-2025-07-17-08-52-23-824
BUCKET_NAME: cloudage-text-to-image-webapp
Permissions: S3 Full Access, SageMaker Full Access
Layer: PillowLayer
```

Function 2: Start_Processing_Function
```
Runtime: Python 3.11
Memory: 512MB
Timeout: 5 minutes
Environment Variables: 
PROCESSING_LAMBDA_NAME: Endpoint_Call_Function
BUCKET_NAME: cloudage-text-to-image-webapp
Permissions: S3 Full Access, Lambda Full Access
```

Function 3: Display_Image_Function
```
Runtime: Python 3.11
Memory: 512MB
Timeout: 5 minutes
Environment Variables:
BUCKET_NAME: cloudage-text-to-image-webapp
Permissions: S3 Full Access, CloudFront Full Access
```

#### Phase 3: API Gateway Configuration
1. Import Swagger Definition
```bash
# Import generative-ai-api-prod-swagger-apigateway.json
# Review resources and stages
```

2. Configure Integration Requests
```bash
# POST → Integration Request → Lambda: Start_Processing_Function
# GET → Integration Request → Lambda: Display_Image_Function
```

3. Deploy API
```bash
# Create new stage with organization naming convention
# Note the API Gateway URL
```

#### Phase 4: Frontend Setup
1. Update index.html
```javascript
// Line 134: Update API Gateway URL
var apiGatewayUrl = "https://your-api-id.execute-api.region.amazonaws.com/stage/";
```

2. Upload to S3
```bash
# Upload logo.jpeg and index.html to S3 bucket
aws s3 cp index.html s3://your-bucket-name/
aws s3 cp logo.jpeg s3://your-bucket-name/
```

#### Phase 5: CloudFront Distribution
1. Create CloudFront Distribution
```bash
# Origin: S3 bucket with index.html
# Default root object: index.html
```

2. Configure S3 Bucket Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudFrontAccess",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::account-id:distribution/distribution-id"
        }
      }
    }
  ]
}
```

3. Create Invalidation
```bash
# Invalidation path: /*
# Wait for deployment (≈10 minutes)
```

## ⚙️ Configuration
### Environment Variables
```
Function	Variable	Value
Endpoint_Call_Function	ENDPOINT_NAME	SageMaker endpoint name
Endpoint_Call_Function	BUCKET_NAME	S3 bucket for images
Start_Processing_Function	PROCESSING_LAMBDA_NAME	Endpoint_Call_Function
Start_Processing_Function	BUCKET_NAME	S3 bucket for images
Display_Image_Function	BUCKET_NAME	S3 bucket for images
```

### IAM Roles & Permissions
SageMaker Execution Role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Lambda Execution Roles:
S3 Full Access for all functions
SageMaker Full Access for Endpoint_Call_Function
Lambda Full Access for Start_Processing_Function
CloudFront Full Access for Display_Image_Function

## 📚 API Documentation
### Base URL
```text
https://{api-id}.execute-api.{region}.amazonaws.com/{stage}
```

### Endpoints
#### POST /generate - Generate Image from Text
Request:
```json
{
  "text": "A beautiful sunset over mountains",
  "style": "realistic",
  "resolution": "1024x1024"
}
```

Response:
```json
{
  "image_id": "img_123456789",
  "status": "processing",
  "estimated_time": 30
}
```

#### GET /image/{image_id} - Retrieve Generated Image
Response:
```json
{
  "image_url": "https://s3.amazonaws.com/bucket/images/img_123456789.png",
  "status": "completed",
  "generated_at": "2024-01-15T10:30:00Z"
}
```

## 🚨 Troubleshooting
### Common Issues & Solutions

#### Issue	Solution
```
SageMaker endpoint not responding	Check IAM permissions and endpoint status
Lambda timeout errors	Increase timeout to 5 minutes
S3 permission denied	Verify bucket policies and IAM roles
CloudFront 403 errors	Check S3 bucket policy and origin configuration
API Gateway CORS errors	Enable CORS in API Gateway settings
```

### Debugging Commands
```bash
# Check Lambda logs
aws logs describe-log-streams --log-group-name /aws/lambda/Endpoint_Call_Function
aws logs get-log-events --log-group-name /aws/lambda/Endpoint_Call_Function --log-stream-name [stream-name]

# Test SageMaker endpoint
aws sagemaker invoke-endpoint \
    --endpoint-name cloudage-endpoint-text-to-image-model \
    --body '{"text": "test image"}' \
    --content-type application/json

# Verify S3 objects
aws s3 ls s3://cloudage-text-to-image-webapp/

# Check CloudFront distribution
aws cloudfront get-distribution --id [distribution-id]
```

## 📊 Monitoring
### Key Metrics to Monitor
SageMaker: Endpoint invocation latency, Model latency
Lambda: Invocation count, Error rate, Duration
API Gateway: Request count, 4XX/5XX errors, Latency
CloudFront: Requests, Bytes transferred, Error rates
S3: Request counts, Bucket size

### CloudWatch Alarms
```bash
# High Lambda error rate
aws cloudwatch put-metric-alarm \
    --alarm-name Lambda-High-Error-Rate \
    --metric-name Errors \
    --namespace AWS/Lambda \
    --statistic Sum \
    --period 300 \
    --threshold 5 \
    --comparison-operator GreaterThanThreshold

# SageMaker endpoint latency
aws cloudwatch put-metric-alarm \
    --alarm-name SageMaker-High-Latency \
    --metric-name ModelLatency \
    --namespace AWS/SageMaker \
    --statistic Average \
    --period 60 \
    --threshold 5000 \
    --comparison-operator GreaterThanThreshold
```

## 🔮 Future Enhancements
### Short-term Improvements
Add image editing capabilities - Modify generated images
Implement batch processing - Generate multiple images at once
Add style transfer options - Apply different art styles
Implement user authentication - Secure API access

### Medium-term Enhancements
Support multiple AI models - DALL-E, Midjourney alternatives
Implement image upscaling - Increase resolution of generated images
Add prompt engineering suggestions - AI-assisted prompt creation
Implement cost optimization - Spot instances for SageMaker

### Long-term Vision
Real-time image generation - WebSocket connections for live updates
Mobile application - Native iOS/Android apps
Enterprise features - Team collaboration, project management
Advanced analytics - Usage patterns, popular prompts














