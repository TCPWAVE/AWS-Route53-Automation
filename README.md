# Objective 

The objective of this document is to provide steps to follow to achieve AWS Route53 Automation. This automation helps to automatically add or delete hosted zone resource records to IPAM whenever the resource records see an update in the AWS Route53.  

To achieve this, we will be utilizing the capability of AWS functionalities namely Cloudtrail, Cloudwatch, S3 and Lambda function. A cloudtrail is created to log the events to cloudwatch which are stored in S3 bucket. A cloudwatch  rule is created to trigger a Lambda function when it sees Route53 resource record change related events in the logs. Lambda function has the node js code to connect to IPAM and add/delete resource records through secure rest API invocations. 

# Implementation Details 

This section describes the steps to follow on IPAM as well as AWS to do the necessary set up for the automation. 
    
    Actions to be performed in the IPAM UI are listed below. 
    # 1 Upload the Appliance certificate to the IPAM. The server(appliance) certificate and client(user) certificate must be uploaded to IPAM to enable SSL authentication for rest API access from AWS Lambda. Self-signed certificates are provided for quick set up. The commands used to create self-signed certificates are provided in step 3. 

            a. Navigate to Administration --> Security Management --> Appliance Certificates. 

            b. Click Import and a dialog opens. 

            Browse Certificate File (rootCA.crt), Private Key File(rootCA.key). 

            Enter Private Key Password(abc123), Certificate Storage Password (default password will be provided on request. This password can be changed by clicking Change     Storage Password in the same page). 

            Select Trust CA and upload the certificate. 

            Restart tims service. 
