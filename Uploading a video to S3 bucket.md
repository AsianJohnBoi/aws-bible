# Uploading a video to S3 bucket

------





### Prerequisites

```
IAM Account
S3 Bucket
```



### S3 Bucket

The S3 bucket can be created with any name with default settings. However CORS must be configured with the following:

```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

### IAM Account

An IAM account will be used to generate a temporary signed URL for our application to put a new object in the S3 bucket. Account must be given the least amount of permissions. 

Example policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutAnalyticsConfiguration",
                "s3:GetObjectVersionTagging",
                "s3:DeleteAccessPoint",
                "s3:CreateBucket",
                "s3:ReplicateObject",
                "s3:GetObjectAcl",
                "s3:GetBucketObjectLockConfiguration",
                "s3:DeleteBucketWebsite",
                "s3:PutLifecycleConfiguration",
                "s3:GetObjectVersionAcl",
                "s3:DeleteObject",
                "s3:GetBucketPolicyStatus",
                "s3:GetObjectRetention",
                "s3:GetBucketWebsite",
                "s3:ListJobs",
                "s3:PutReplicationConfiguration",
                "s3:PutObjectLegalHold",
                "s3:GetObjectLegalHold",
                "s3:GetBucketNotification",
                "s3:PutBucketCORS",
                "s3:GetReplicationConfiguration",
                "s3:ListMultipartUploadParts",
                "s3:PutObject",
                "s3:GetObject",
                "s3:PutBucketNotification",
                "s3:DescribeJob",
                "s3:PutBucketLogging",
                "s3:GetAnalyticsConfiguration",
                "s3:PutBucketObjectLockConfiguration",
                "s3:GetObjectVersionForReplication",
                "s3:CreateJob",
                "s3:CreateAccessPoint",
                "s3:GetLifecycleConfiguration",
                "s3:GetAccessPoint",
                "s3:GetInventoryConfiguration",
                "s3:GetBucketTagging",
                "s3:PutAccelerateConfiguration",
                "s3:DeleteObjectVersion",
                "s3:GetBucketLogging",
                "s3:ListBucketVersions",
                "s3:RestoreObject",
                "s3:GetAccelerateConfiguration",
                "s3:GetBucketPolicy",
                "s3:PutEncryptionConfiguration",
                "s3:GetEncryptionConfiguration",
                "s3:GetObjectVersionTorrent",
                "s3:AbortMultipartUpload",
                "s3:GetBucketRequestPayment",
                "s3:GetAccessPointPolicyStatus",
                "s3:UpdateJobPriority",
                "s3:GetObjectTagging",
                "s3:GetMetricsConfiguration",
                "s3:DeleteBucket",
                "s3:PutBucketVersioning",
                "s3:GetBucketPublicAccessBlock",
                "s3:ListBucketMultipartUploads",
                "s3:ListAccessPoints",
                "s3:PutMetricsConfiguration",
                "s3:UpdateJobStatus",
                "s3:GetBucketVersioning",
                "s3:GetBucketAcl",
                "s3:PutInventoryConfiguration",
                "s3:GetObjectTorrent",
                "s3:GetAccountPublicAccessBlock",
                "s3:PutBucketWebsite",
                "s3:PutBucketRequestPayment",
                "s3:PutObjectRetention",
                "s3:GetBucketCORS",
                "s3:GetBucketLocation",
                "s3:GetAccessPointPolicy",
                "s3:ReplicateDelete",
                "s3:GetObjectVersion"
            ],
            "Resource": "*"
        }
    ]
}
```



### Server-Side

All requests (authenticating, running CRUD with db, etc) should always be handled on the server side. Keys should never be hardcoded and  pushed to git for security reasons. Keys should be placed in an environment variables file such as .env file (See https://www.npmjs.com/package/dotenv).

The following is a guide for setting up a new API endpoint for the client-side to request a temporary signed URL for uploading a video file. Here we create a new endpoint `/signed` accepting POST requests. 

1. Install the AWS SDK `npm i aws sdk --save`

2. Update aws account config.

   ```
   AWS.config.update({
     accessKeyId: <IAMACCESSKEY>,
     secretAccessKey: <SECRETACCESSKEY>
   });
   ```

   

3. Create a new S3 object from the AWS class.

   `var s3 = new AWS.S3()`

4. Create a POST endpoint:

   `router.post('/signed', async (req, res) => {}`

5. Set parameters

   ```
   var params = {
         Bucket: <youbucketname>,
         Key: filename,
         Expires: 60,
         ContentType: mimeType
       };
   ```

6. Call the `getSignedUrl` method to get a signedurl.

   ```
   await s3.getSignedUrl('putObject', params, function (err, data) {
         if (err) {
           console.log(err);
           return res.send(500, "Cannot create S3 signed URL");
         }
         res.status(200).json({
           signedUrl: data,
           publicUrl: `https://${S3_BUCKET_NAME}.s3-ap-southeast-2.amazonaws.com/${filename}`,
           filename: filename
         });
       });
   ```

Full example of endpoint:

```
router.post('/signed', async (req, res) => {
  try {
    var filename = req.body.objectName; # name of the file
    var mimeType = req.body.mimeType; # file type

    var s3 = new AWS.S3();
    var S3_BUCKET_NAME = <yourbucketname>

    var params = {
      Bucket: S3_BUCKET_NAME,
      Key: filename,
      Expires: 60,
      ContentType: mimeType
    };

    await s3.getSignedUrl('putObject', params, function (err, data) {
      if (err) {
        console.log(err);
        return res.send(500, "Cannot create S3 signed URL");
      }
      res.status(200).json({
        signedUrl: data,
        publicUrl: `https://${S3_BUCKET_NAME}.s3-ap-southeast-2.amazonaws.com/${filename}`,
        filename: filename
      });
    });
  } catch (e) {
    res.status(400).json("Error occurred: ", e);
  }
})
```



### Client-Side

1. 