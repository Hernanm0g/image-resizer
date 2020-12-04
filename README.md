# image-resizer

AWS lambda function for automatic resizing of images hosted in a S3 bucket, using sharp.js

[Credits: Courtesy of John Pignata, AWS Solutions Architect](https://aws.amazon.com/es/blogs/compute/resize-images-on-the-fly-with-amazon-s3-aws-lambda-and-amazon-api-gateway/)

Note: This repo has a few fixes compared to the one you'll find in John Pignata's: No package errors and a fix in the .match method for the url param.

## To create and configure the S3 bucket

1. In the S3 console, create a new S3 bucket.
2. Choose **Permissions, Add Bucket Policy**. Add a bucket policy to allow anonymous access.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "GokyUserAccess",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::283850182097:user/gokyAssetsUser"
            },
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::goky-assets/*"
        },
        {
            "Sid": "AllowAllGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::goky-assets/*"
        }
    ]
}
```
3. Choose Properties => Static Website Hosting, Enable website hosting and, for Index Document, enter index.html.
Choose Save.
4. Note the name of the bucket that you’ve created and the hostname in the Endpoint field.

## To create the Lambda function

1. In the Lambda console, choose Create a Lambda function, Blank Function.
2. Choose trigger integration with Api Gateway.
3. To allow all users to invoke the API method, in Api Gateway Security, choose Open and then Next.
4. For Name, enter **resize**. For Code entry type, choose **Upload a .ZIP file**.
5. Download this repository .zip, Choose Function package in aws Lambda code section and upload this .ZIP file.
6. Add two variables to th eLambda function:
    1. For Key, enter BUCKET;  for Value,enter the bucket name that you created above.
    2. For Key, enter URL; for Value, enter the endpoint field that you noted above, prefixed with http://.
7. To define the execution role permissions for the function, add this json to the lambda's Role:


**Replace YOUR_BUCKET_NAME_HERE with the name of the bucket that you’ve created and copy the following code into the policy document. Note that any leading spaces in your policy may cause a validation error**.
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::__YOUR_BUCKET_NAME_HERE__/*"    
    }
  ]
}
```
8. For Memory, choose 1536. For execution time, between 0 and 10 sec.
9. Return to Api gateway in the Lambda triggers and note the **hostname** in the URL of your function.

## To set up the S3 redirection rule

1. In the S3 console, open the bucket that you created above.
2. Expand Static Website Hosting, Edit Redirection Rules.

**Replace YOUR_API_HOSTNAME_HERE with the hostname that you noted above and copy the following into the redirection rules configuration:**
```json
[
    {
        "Condition": {
            "HttpErrorCodeReturnedEquals": "404"
        },
        "Redirect": {
            "HostName": "YOUR_API_HOSTNAME_HERE",
            "HttpRedirectCode": "307",
            "Protocol": "https",
            "ReplaceKeyPrefixWith": "default/resize?key="
        }
    }
]
```
Upload a test image into your bucket to for testing. Name it blue_marble.jpg. Once uploaded, try to retrieve resized versions of the image using your bucket’s static website hosting endpoint:

http://YOUR_BUCKET_WEBSITE_HOSTNAME_HERE/300x300/blue_marble.jpg

http://YOUR_BUCKET_WEBSITE_HOSTNAME_HERE/25x25/blue_marble.jpg

http://YOUR_BUCKET_WEBSITE_HOSTNAME_HERE/500x500/blue_marble.jpg

You should see a smaller version of the test photo. If not, choose Monitoring in your Lambda function and check CloudWatch Logs for troubleshooting. You can also refer to the serverless-image-resizing GitHub repo for a working example that you can deploy to your account.
