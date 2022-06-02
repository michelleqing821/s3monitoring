This is describing the steps taken to create and use a lambda function for monitoring s3 files.

# Description of Lambda Function
This lambda function checks the number of files in the given s3 location and ensures that in the past 60 minutes, 
x files were created. The purpose of this is to make sure that if files aren't being
uploaded into the s3 location properly, we can be sent an alert right away so that we can identify the problem quickly. 

The lambda function does this by checking the modification dates (UTC) of the files in the last hour and current time. If 
the modification dates are within the past hour, it adds 1 to a count. This way, the total expected should be produced if 
the files are being correctly uploaded. 

If the files were to stop uploading, meaning the paths stop being created at certain times, the number 0 is pushed instead. 

One thing we wanted to make more efficient was the count of modification dates. Rather than a loop that adds one to a count
for each file, we wanted to find a filter method that returned the count of the files all at once. 
# How to deploy the function to lambda
The function was deployed to lambda by using the aws lambda service and adding the lambda handler function. Because 
we weren't using the json event or context, nothing else needed to be changed within the code. 

To deploy the function using CLI, 
# Pushing to Cloudwatch 
In the lambda handler, code was added to push the result of the function into Cloudwatch by creating a Cloudwatch metric. 
The namespace is named "MonitoringS3" and the metric being graphed is called "NbcuTrackerFreewheelPartitions".

After the metric is successfully pushed, a dashboard is created to graph the metric. It runs every 5 minutes and we expect
the result to be 720 each run. If the number falls below 720 for three consecutive checks, an alert should be sent to 
a Slack channel to notify us that something has failed. 
# Setting up Slack alert
A Slack alert was set up by creating an aws Cloudwatch alarm. For the alert, the Cloudwatch metric created from the 
lambda function was first chosen and the condition was specified that an alert would be sent if the value dropped below 
the expected number of 720. We then chose which Slack channel the alert would be sent to: 
```
alarms-for-nbcuas-slack-dev
```
and the alert was then tested. For testing, we changed the threshold number to lower than 900 so that an alert should be
sent. Once the changes are deployed, we will change the alerts from the test-alert channel to the alert channel.
