# Objective 

The objective of this document is to provide steps to follow to achieve AWS Route53 Automation. This automation helps to automatically add or delete hosted zone resource records to IPAM whenever the resource records see an update in the AWS Route53.  
To achieve this, we will be utilizing the capability of AWS functionalities namely Cloudtrail, Cloudwatch, S3 and Lambda function. A cloudtrail is created to log the events to cloudwatch which are stored in S3 bucket. A cloudwatch  rule is created to trigger a Lambda function when it sees Route53 resource record change related events in the logs. Lambda function has the node js code to connect to IPAM and add/delete resource records through secure rest API invocations. 
