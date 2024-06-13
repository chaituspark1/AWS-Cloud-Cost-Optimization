# AWS Cloud Cost Optimization

Automating the identification and deletion of stale EBS snapshots to optimize AWS storage costs using AWS Lambda and Boto3.

## Project Overview

This project aims to reduce AWS storage costs by automatically identifying and deleting stale EBS snapshots that are no longer associated with any active EC2 instance. The automation is achieved using an AWS Lambda function and Boto3, the AWS SDK for Python.

## Features

- Identifies EBS snapshots that are not associated with any volume.
- Deletes snapshots associated with volumes that are not attached to any running EC2 instance.
- Handles errors gracefully, such as volumes not found.

## Requirements

- AWS Account
- AWS Management Console access
- Python 3.x
- Boto3 library

## Setup

1. **Create the Lambda Function:**
   - Go to the AWS Management Console.
   - Navigate to the Lambda service.
   - Create a new Lambda function.
   - Choose Python 3.x as the runtime environment.

2. **Copy the Lambda Function Code:**
   - Copy the code from `ebs_snapshot_cleanup.py` (provided above) into the Lambda function editor.

3. **Set Up IAM Role:**
   - Create an IAM role with the following permissions:
     - `ec2:DescribeSnapshots`
     - `ec2:DescribeInstances`
     - `ec2:DescribeVolumes`
     - `ec2:DeleteSnapshot`
   - Attach the IAM role to the Lambda function.

4. **Configure the Lambda Function:**
   - Set the function to be triggered based on your preferred schedule using CloudWatch Events or EventBridge.

## Lambda Function Code

Paste the following code into your Lambda function in the AWS Management Console:

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # Get all EBS snapshots
    response = ec2.describe_snapshots(OwnerIds=['self'])

    # Get all active EC2 instance IDs
    instances_response = ec2.describe_instances(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}])
    active_instance_ids = set()

    for reservation in instances_response['Reservations']:
        for instance in reservation['Instances']:
            active_instance_ids.add(instance['InstanceId'])

    # Iterate through each snapshot and delete if it's not attached to any volume or the volume is not attached to a running instance
    for snapshot in response['Snapshots']:
        snapshot_id = snapshot['SnapshotId']
        volume_id = snapshot.get('VolumeId')

        if not volume_id:
            # Delete the snapshot if it's not attached to any volume
            ec2.delete_snapshot(SnapshotId=snapshot_id)
            print(f"Deleted EBS snapshot {snapshot_id} as it was not attached to any volume.")
        else:
            # Check if the volume still exists
            try:
                volume_response = ec2.describe_volumes(VolumeIds=[volume_id])
                if not volume_response['Volumes'][0]['Attachments']:
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as it was taken from a volume not attached to any running instance.")
            except ec2.exceptions.ClientError as e:
                if e.response['Error']['Code'] == 'InvalidVolume.NotFound':
                    # The volume associated with the snapshot is not found (it might have been deleted)
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as its associated volume was not found.")
