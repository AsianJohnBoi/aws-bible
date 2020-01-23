# Deploying full-stack application on S3 and EC2

------

A full-stack application consists of a front (client-side) and a back end (server-side) which can both be served from a single EC2 instance. Both ends can be separated and used in two different services. An S3 bucket for serving the front-end application and an EC2 instance for serving the back-end. 

In this guide we will be deploying a VueJS and NodeJS application. 

### Prerequisites

```
AWS account
Basic AWS knowledge
Domain name
```

### Deploy front-end to S3

1. Create a production build of the front-end application. (Prod build is in VueJS is /dist).

2. Create an S3 bucket and name it with a domain name (mydomain.com). Do not Block all public access.

3. Upload the production build folder in the S3 bucket. 

4. Enable static website hosting.

5. Edit bucket permissions.

   i. Edit Access Control List by giving read bucket permissions to everyone in public access.

   ii. Add bucket policy

   ```
   {
       "Version": "2008-10-17",
       "Id": "PolicyForPublicWebsiteContent",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": {
                   "AWS": "*"
               },
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::domain.com/*"
           }
       ]
   }
   ```

   

### CloudFront Distribution

1. Create a cloudfront web distribution. 

2. Select S3 bucket as origin domain name

3. Set 'Viewer Protocol Policy' to Redirect HTTP to HTTPS

4. Add altername domain name. e.g. mydomain.com

5. Click 'Request or import a certificate with ACM'

   i. Enter Domain name

   ii. Validate certificate request by either DNS Validation or Email Validation

   iii. Wait for ACM to be verified to continue to the next step.

6. Select custom created SSL Certificate

7. Set default root object to 'index.html'

8. Create distribution

### Deploy server-side to EC2

1. Launch a new EC2 ubuntu instance (or another linux distribution).

   i. Select free tier instance

   ii. Add HTTP and HTTPS port to security group with a source from anywhere.

   iii. Review and launch.

2. Copy and launch server-side application on EC2.

### Proxy client-side requests to server-side application

1. 

### DNS Configuration

1. 