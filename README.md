# AWS-Cloud-Cost-Optimization
Automating the identification and deletion of stale EBS snapshots to optimize AWS storage costs using AWS Lambda and Boto3.
Project Description:
In this project, I developed an AWS Lambda function to automatically identify EBS snapshots that are no longer associated with any active EC2 instance. These stale snapshots are then deleted to save on storage costs. The Lambda function performs the following tasks:

Fetches all EBS snapshots owned by my AWS account.
Retrieves a list of active EC2 instances.
Checks each snapshot to see if it is associated with an active volume.
Deletes snapshots that are not attached to any volume or attached to volumes that are not associated with any running instance.
Technical Details:

AWS Lambda: Used for executing the automation script.
Boto3: AWS SDK for Python to interact with EC2 and manage snapshots.
EC2: Service for running and managing virtual servers in the cloud.
GitHub Repository:
For more details and to view the source code, check out my GitHub repository: [GitHub Repository URL]

Project Showcase:
[Include your screen recording link here]

This project not only helped in reducing storage costs but also demonstrated the power of automation in cloud resource management. Feel free to reach out if you have any questions or need further details about the project.

