# Cloud-Front-Distribution

# ☁️ AWS CloudFront with S3 Static Website Hosting (with OAC Security)

📘 Project Overview

This project demonstrates how to host a static website on Amazon S3 and securely deliver content through Amazon CloudFront using Origin Access Control (OAC).
The goal is to restrict direct access to the S3 bucket and allow only CloudFront to fetch content, ensuring secure and optimized content delivery.

🪜 Step-by-Step Implementation

## Step 1: S3 Origin Setup and Configuration

Created an S3 bucket named my-pec-bucket-01 in the ap-south-2 (Hyderabad) region.

<img width="1919" height="926" alt="Screenshot 2025-11-03 190847" src="https://github.com/user-attachments/assets/7a51a661-bea2-4cb3-82fd-7dd3ea53f13a" />

-----------
-----------

Enabled Block all public access (bucket is private).

<img width="1919" height="924" alt="Screenshot 2025-11-03 190900" src="https://github.com/user-attachments/assets/40e92c6a-526c-473d-ad4a-3b80269c6f5a" />

Enable static website hosting :

<img width="1914" height="880" alt="Screenshot 2025-11-03 191053" src="https://github.com/user-attachments/assets/4b77aa38-5588-4ad4-acd5-1eec91d91974" />

<img width="1919" height="912" alt="Screenshot 2025-11-03 191130" src="https://github.com/user-attachments/assets/399155b9-2bf2-4806-96b0-6cc351ab1fb2" />

-------------
------------

Uploaded file:

index.html

<img width="1919" height="937" alt="Screenshot 2025-11-03 191412" src="https://github.com/user-attachments/assets/5c45f889-80b9-4034-b528-decb4308f982" />

Verified the files are stored correctly in the bucket.

<img width="1918" height="929" alt="Screenshot 2025-11-03 191439" src="https://github.com/user-attachments/assets/1655a2e6-36c8-4870-8f36-22a146364a5f" />

✅ Result: S3 bucket created and content uploaded successfully.

-----------
-----------

## Lets verify accessing index.html via s3 endpoint:

<img width="1918" height="988" alt="Screenshot 2025-11-03 194421" src="https://github.com/user-attachments/assets/973b23cf-7fdd-4b49-a14f-d712a93a2eaf" />

### We cannot access it due to block public access.

-----------
-----------
-----------

## Step 2: CloudFront Distribution Creation and Default Cache Behavior

Opened CloudFront :

<img width="1919" height="923" alt="Screenshot 2025-11-03 191616" src="https://github.com/user-attachments/assets/21a697b1-d08f-48ae-9639-e88832a71a18" />

Console → Create Distribution.

<img width="1919" height="924" alt="Screenshot 2025-11-03 191702" src="https://github.com/user-attachments/assets/0fafb85b-02e2-4328-af9e-713848f2836b" />


Selected Origin Domain Name:
my-pec-bucket-01.s3.ap-south-2.amazonaws.com
(not the -website endpoint)

<img width="1809" height="704" alt="Screenshot 2025-11-03 191720" src="https://github.com/user-attachments/assets/18582ab6-5e4a-41ab-9090-788a7c211c59" />

-----------
-----------

<img width="1919" height="907" alt="Screenshot 2025-11-03 194740" src="https://github.com/user-attachments/assets/c81e1d71-9f0c-4b6f-acd2-d96c89101f0d" />

Protocol: HTTPS only

<img width="1919" height="931" alt="Screenshot 2025-11-03 192105" src="https://github.com/user-attachments/assets/f49d0877-f7e2-409c-a6ff-129b0b3d391f" />


Viewer Protocol Policy: Redirect HTTP to HTTPS

Allowed HTTP Methods: GET, HEAD

Cache Policy: CachingOptimized (Default)

-----------
-----------

Created the distribution and waited for deployment to complete.

<img width="1916" height="921" alt="Screenshot 2025-11-03 193914" src="https://github.com/user-attachments/assets/d87e83ad-72f4-48ae-a590-5d42c52d6425" />

✅ Result: CloudFront distribution deployed successfully.

-----------
-----------

## Lets verify if we can access via cloudfront endpoint:

<img width="1919" height="980" alt="Screenshot 2025-11-03 192344" src="https://github.com/user-attachments/assets/446d1372-9914-4a6c-8e12-32ba56a05fda" />


### We cannot access it because we should set policy in S3 bucket to enable traffic from cloudfrount Distribution..

-----------
-----------

## Step 3: Restrict Bucket Access (OAC Security)

In CloudFront → Origin Settings, selected:

<img width="1919" height="926" alt="Screenshot 2025-11-03 192843" src="https://github.com/user-attachments/assets/93c61441-6669-446d-b42f-e44e0f5da7cc" />

<img width="1919" height="922" alt="Screenshot 2025-11-03 192912" src="https://github.com/user-attachments/assets/b57963c0-0daa-4fab-a7b4-9658c42972ea" />

-----------
-----------

## Create Origin Access Control (OAC)

Signing behavior: Sign requests (recommended)

Signing protocol: Sigv4

CloudFront generated an OAC ID.

Went to S3 → Permissions → Bucket Policy and added the policy below:

<img width="1919" height="918" alt="Screenshot 2025-11-03 193029" src="https://github.com/user-attachments/assets/22bc371d-c153-4992-b7b3-4748428ebea2" />


{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-pec-bucket-01/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<AWS_ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
        }
      }
    }
  ]
}

-----------
-----------

✅ Result:

S3 bucket is private.

Only CloudFront can access objects securely using OAC.

-----------
-----------

## Step 4: Testing and Validation

Test 1: Direct S3 Access
URL:

https://my-pec-bucket-01.s3.ap-south-2.amazonaws.com/index.html

<img width="1919" height="977" alt="Screenshot 2025-11-03 193216" src="https://github.com/user-attachments/assets/1f631430-870f-4f14-8d84-39fb91e03640" />

Result: ❌ AccessDenied

-----------
-----------
Test 2: CloudFront Access
URL:

https://dmzzmvnjcgbgo.cloudfront.net/index.html

<img width="1919" height="978" alt="Screenshot 2025-11-03 193538" src="https://github.com/user-attachments/assets/7f65c182-48e8-4da1-acaf-96ed12cd2666" />

Result: ✅ Website loads successfully

-----------
-----------

Test 3: Caching Validation

First request → Slight delay (miss)

Second request → Faster response (cache hit)

<img width="1919" height="978" alt="Screenshot 2025-11-03 193325" src="https://github.com/user-attachments/assets/869b8112-fdf5-4588-9f6e-523283fd3b57" />

✅ Result: Content served securely and efficiently via CloudFront.

-----------
-----------

🧠 Key Takeaways

Used CloudFront + S3 integration with OAC to ensure content security.

Achieved HTTPS redirection, private bucket access, and optimized caching.

Demonstrated testing with both direct and CloudFront access.

-----------
-----------

# Author: Suyash Dahitule
