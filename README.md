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
   
   ![aws2](https://user-images.githubusercontent.com/56577268/130194710-11158d4f-7c57-4bf1-b617-a23edabc89cd.png)
   
   2. **Create a Lambda function in N.Virginia region**. This function has the node js (14.x version) code that takes input from cloudwatch rule (will be created in the next step) and invokes rest API of TCPWave IPAM to add or delete the resource record based on the action(add/delete), using SSL authentication using certificates.
   
          a. Before creating the Lambda function, create an IAM role with the permissions: AmazonRoute53ReadOnlyAccess and AWSLambdaBasicExecutionRole.
          b. Open the service Lambda and click Create function.
          c. Function name: Enter a name.
          d. Runtime: Select Node.js 14.x.
          e. In Permissions section, expand Change default execution role, choose Use an existing role and select the IAM role created previously.
          f. Click Creation function.
          g. Open the function to make few configuration changes and add code to it.
          h. Under Configuration tab, select General Configuration and click Edit in that section.
          i. Change Timeout to 3min and 30sec and click Save.
          j. Under Code tab, Select .zip file option from Upload File dropdown. Browse the r53_function.zip file provided. Click Save and click Ok.

   ![aws3](https://user-images.githubusercontent.com/56577268/130195984-9306dc05-e4c4-4c4d-acee-09f1ee06a0a1.png)

   ![aws4](https://user-images.githubusercontent.com/56577268/130195998-b4a47b2a-4c12-4a9b-b24c-fce18392c4ee.png)
   
   ![aws5](https://user-images.githubusercontent.com/56577268/130196011-2e5193f2-8231-4b23-8f36-106e48ce3e13.png)
   
   **Important**:  There are few modifications to be done in the code to make it work.
   
   a. The below code snippet declares the default organization of the IPAM to change resource records in and the IPAM hostname. Change these settings and click Deploy.

       var ORG_NAME = "xxxxxx";
       var HOST = "xxx.xxxx.xxx";
                
  **Note**: If the zones are distributed across different organizations in the IPAM, in such case, this code has capability to get the organization information from the tag                 ORGANIZATION added for the hosted zone in Route53. If this tag is not present, default ORG_NAME will be taken.
  
  ![aws6](https://user-images.githubusercontent.com/56577268/130196769-6559eb25-d2cf-471e-a8c8-a8e25d2e89f3.png)
  
  3. **Create a cloudwatch rule that can trigger the above Lambda function when Route53 resource record changes take place.**
  
          a. Open Cloudwatch service and click Create rule.
          b. Choose Event Pattern in the Event Source section.
          c. Service Name: Select Route53.
          d. Event Type: Select AWS API Call via CloudTrail.
          e. Choose Specific Operation(s).
          f. Enter ChangeResourceRecordSets in the text box of Specific Operation(s).
          g. Click Add Trigger.
          h. Select Lambda Function as trigger.
          i. Function: Select the  Lambda function created previously.
          j. Expand Configure Input and choose Part of the matched event.
          k. Enter $.detail.requestParameters in the text box.
          l. Click Configure details.
          m. Enter Name and Description and click Create rule.
          
   ![aws7](https://user-images.githubusercontent.com/56577268/130197197-af693702-798c-4c94-b965-96819e8c44c6.png)
   
# Validation

After doing the above set up, check the logs to see if the set-up is successful and the rest API calls are invoked to IPAM or not. To do that, open and verify AWS Cloudwatch -> Log groups -> (Log with the Lambda function name).

On the IPAM end, check the log by navigating to **Infrastructure Management** -> **Performance Management** -> **IPAM Statistics** -> **Logs**. Select the **IPAM appliance** that is given in the Lambda function, select **IPAM Webserver Log** and click **Generate**.

**Note**: Make sure that the hosted zone where the resource records are updated is present in the IPAM before validating the set up.

# Conclusion

By following the above steps, AWS Route53 resource record changes will be automatically updated in the TCPWave IPAM without the need of manually doing Sync in the IPAM.

  
  
