# Automated S3 Cleanup with AWS Lambda

This project automatically deletes old files from an **S3 bucket** using **AWS Lambda** and **CloudWatch**.  
It is a simple **serverless solution** to manage storage efficiently and save costs.

---

## 🛠 Services Used
- **Amazon S3** – Stores files
- **AWS Lambda** – Serverless function to delete old files
- **CloudWatch** – Schedules Lambda function automatically

---

## ⚡ Features
- Automatically deletes files older than a specified number of days
- Lambda runs on a schedule (e.g., every 5 minutes)
- Saves manual effort and reduces unnecessary storage cost
- Easy to configure with **environment variables** for bucket name and age threshold

---

## 📂 Setup Instructions

### 1. Create an S3 Bucket
- Go to AWS S3 → Create a new bucket → Upload test files

### 2. Create a Lambda Function
- Runtime: Python 3.9 or 3.10
- Assign IAM role with permissions:
  - `s3:ListBucket`
  - `s3:GetObject`
  - `s3:DeleteObject`

### 3. Set Environment Variables
- `BUCKET_NAME` → Your S3 bucket name
- `DAYS_OLD` → Number of days after which files should be deleted

### 4. Add Lambda Code
- Copy the code from `lambda_function.py` (or paste below)  
```python
import os
import boto3
from datetime import datetime, timezone, timedelta

s3 = boto3.client('s3')

def lambda_handler(event, context):
    bucket_name = os.environ.get('BUCKET_NAME')
    days_old = int(os.environ.get('DAYS_OLD', '30'))
    if not bucket_name:
        return {"status": "error", "message": "BUCKET_NAME env var not set"}

    threshold = datetime.now(timezone.utc) - timedelta(days=days_old)
    paginator = s3.get_paginator('list_objects_v2')
    deleted = []

    for page in paginator.paginate(Bucket=bucket_name):
        for obj in page.get('Contents', []):
            key = obj['Key']
            last_modified = obj['LastModified']
            if last_modified < threshold:
                try:
                    s3.delete_object(Bucket=bucket_name, Key=key)
                    deleted.append(key)
                    print(f"Deleted: {key}")
                except Exception as e:
                    print(f"Failed to delete {key}: {e}")

    return {
        "status": "done",
        "deleted_count": len(deleted),
        "deleted_keys": deleted[:20]
    }
```
Linkedin Link: https://www.linkedin.com/posts/md-faizan-bb9998234_aws-serverless-lambda-activity-7385730847351406592-G4zt?utm_source=share&utm_medium=member_desktop&rcm=ACoAADqFwb0B3mY-3tfXzS8MUobP_65azcolSNo
