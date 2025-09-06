---

# 🚀 Copy S3 Objects Across AWS Accounts (Same Region)

This guide explains how to securely transfer objects from a **source S3 bucket** in **Account A** to a **destination S3 bucket** in **Account B** within the same AWS region.

---

## 📋 Prerequisites

* Two AWS accounts:

  * **Account A (Source)**
  * **Account B (Destination)**
* AWS CLI installed and configured.
* Permissions to create IAM policies and bucket policies in both accounts.

---

## 1️⃣ Create Two Buckets

* In **Account A (Source)** → `uadmin-sourcebucket`
* In **Account B (Destination)** → `uadmin-destinationbucket`

---

## 2️⃣ Create IAM Policy in Destination Account

In **Account B**, create a policy (`S3CrossAccountCopyPolicy`) that allows **read from source bucket** and **write to destination bucket**.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::uadmin-sourcebucket",
        "arn:aws:s3:::uadmin-sourcebucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": [
        "arn:aws:s3:::uadmin-destinationbucket",
        "arn:aws:s3:::uadmin-destinationbucket/*"
      ]
    }
  ]
}
```

---

## 3️⃣ Create IAM User/Role in Destination Account

* Create an IAM user (for CLI) or role (for automation).
* Attach the above policy.
* If user: download **Access Key** and **Secret Key**.

---

## 4️⃣ Add Bucket Policy in Source Account

In **Account A**, update source bucket policy (`uadmin-sourcebucket`) to allow Account B to read objects:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCrossAccountRead",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<DESTINATION_ACCOUNT_ID>:root"
      },
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::uadmin-sourcebucket",
        "arn:aws:s3:::uadmin-sourcebucket/*"
      ]
    }
  ]
}
```

---

## 5️⃣ Configure AWS CLI with Destination Credentials

Use destination account credentials.

### Linux / Mac

```bash
export AWS_ACCESS_KEY_ID=<ACCESS_KEY>
export AWS_SECRET_ACCESS_KEY=<SECRET_KEY>
```

### Windows PowerShell

```powershell
setx AWS_ACCESS_KEY_ID "<ACCESS_KEY>"
setx AWS_SECRET_ACCESS_KEY "<SECRET_KEY>"
```

---

## 6️⃣ Copy Objects Between Buckets

### Sync entire bucket

```bash
aws s3 sync s3://uadmin-sourcebucket s3://uadmin-destinationbucket
```

### Copy single file

```bash
aws s3 cp s3://uadmin-sourcebucket/myfile.txt s3://uadmin-destinationbucket/
```

---

## ✅ Done!

Objects are now copied from **Account A → Account B** 🎉

---
