# Deploying full-stack application on S3 and EC2

------

A full-stack application consists of a front (client-side) and a back end (server-side) which can both be served from a single EC2 instance. Both ends can be separated and used in two different services. An S3 bucket for serving the front-end application and an EC2 instance for serving the back-end. 

In this guide we will be deploying a full-stack application that was built with VueJS and NodeJS. VueJS app served from an S3 bucket and executing HTTP requests to the NodeJS server running on an EC2 instance. This concept can be applied to other Javascript frameworks. 

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

2. Add a static ip address to the instance.

   i. Click 'Elastic IPs' on the left hand side.

   ii. Allocate Elastic IP address

   iii. Associate Elastic IP address to the instance.

   iv. Add private IP address.

   v. Associate

3. Copy and launch server-side application on EC2.

### Reverse proxy

A reverse proxy is required in order for the EC2 instance to proxy the client-side requests to the server. NGINX will be used to proxy the requests.

1. Install NGINX `sudo apt install nginx`

2. Edit `sudo /etc/nginx/sites-available/default` and add the following underneath the server

3. ```
   location / {
                   # First attempt to serve request as file, then
                   # as directory, then fall back to displaying a 404.
                   # try_files $uri $uri/ =404;
                   proxy_pass http://localhost:3000;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
                   proxy_set_header X-NginX-Proxy true;
                   proxy_ssl_session_reuse off;
                   proxy_set_header Host $http_host;
                   proxy_redirect off;
           }
   ```

   

4. Install certbot to obtain an SSL certificate. Redirect HTTP requests to HTTPS. Use the domain name and the EC2 instance ip address.

5. Server is ready to go. NodeJS endpoints can be tested with postman. 

   

### DNS Configuration

1. Add new record. E.g. server.mydomain.com
2. Select type A - IPv4 Address
3. Enter the public ip address of the created EC2 instance in the value box. 
4. Leave everything else by default.