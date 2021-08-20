# Objective 
The objective of this document is to provide steps to follow to achieve AWS Route53 Automation. This automation helps to automatically add or delete hosted zone resource records to IPAM whenever the resource records see an update in the AWS Route53.  

To achieve this, we will be utilizing the capability of AWS functionalities namely Cloudtrail, Cloudwatch, S3 and Lambda function. A cloudtrail is created to log the events to cloudwatch which are stored in S3 bucket. A cloudwatch  rule is created to trigger a Lambda function when it sees Route53 resource record change related events in the logs. Lambda function has the node js code to connect to IPAM and add/delete resource records through secure rest API invocations. 

# Implementation Details 

This section describes the steps to follow on IPAM as well as AWS to do the necessary set up for the automation. 

Actions to be performed in the IPAM UI are listed below. 
    
   ## 1. Upload the Appliance certificate to the IPAM. 
   The server(appliance) certificate and client(user) certificate must be uploaded to IPAM to enable SSL authentication for rest API access from AWS Lambda.      Self-signed certificates are provided for quick set up. The commands used to create self-signed certificates are provided in step 3. 

          a. Navigate to Administration --> Security Management --> Appliance Certificates. 
          b. Click Import and a dialog opens. 
          c. Browse Certificate File (rootCA.crt), Private Key File(rootCA.key). 
          d. Enter Private Key Password(abc123), Certificate Storage Password (default password will be provided on request. 
               This password can be changed by clicking Change Storage Password in the same page). 
          e. Select Trust CA and upload the certificate. 
          f. Restart tims service. 
   ## 2. Upload the User certificate to the IPAM.
   
        a. Navigate to Administration --> Security Management --> User Certificates. 
        b. Click Import and a dialog opens. 
        c. Browse Certificate File(client.crt) and select twcadm as Associated Admin. 
        d. Click Ok and import the certificate. 
        e. Restart tims is not needed. 
   ## 3. Commands to create self-signed certificates using openssl
   
         openssl genrsa -des3 -out rootCA.key 4096  
         openssl req -x509 -new -nodes -key rootCA.key -days 1024 -out rootCA.crt  
         openssl genrsa -out client.key 4096  
         openssl req -new -key client.key -out client.csr  
         openssl x509 -req -in client.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out client.crt -days 500 
         
Actions to be performed in the AWS console are listed below

**Note**: Select region as **N.Virginia** because Route53 logs events to cloud trail created in this region. 

    1. Open CloudTrail service and create a trail. Cloudtrail writes the change events to cloudwatch  logs. These logs are stored in S3. 
    
         a. Trail name: Enter a Trail name. 
         b. Storage location: Select Create new S3 bucket or Select existing one. Disable the Log file SSE-KMS encryption if not needed. 
         c. Cloudwatch Logs: Enable cloudwatch logs. Enter a log group name or keep the default. Select IAM role as Existing and choose  CloudTrail_CloudWatchLogs_Role.           
         d. Click Next. 
         e. Click Next. Click Create trail. 
   ![aws1](https://user-images.githubusercontent.com/56577268/130194231-93312dd6-48eb-4e04-a1a9-186adf97ac5c.png)
