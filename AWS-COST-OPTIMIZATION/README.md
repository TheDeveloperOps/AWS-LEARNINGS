### GitHub `README.md` for AWS Cloud Cost Optimization - Identifying Stale EBS Snapshots

# AWS Cloud Cost Optimization - Identifying Stale EBS Snapshots

This project focuses on optimizing AWS cloud storage costs by identifying and deleting stale EBS snapshots that are no longer associated with any active EC2 instances. By cleaning up unused snapshots, you can significantly reduce storage costs while maintaining the integrity of your active instances.

## Description

The Lambda function fetches all EBS snapshots owned by the AWS account (`self`) and retrieves a list of active EC2 instances (running). For each snapshot, it checks if the associated volume exists and whether it is attached to any active instance. If the snapshot is not associated with any volume or is attached to a deleted volume, the function deletes the snapshot.

### Key Features:
- Identifies stale EBS snapshots that are no longer attached to any active EC2 instances.
- Deletes snapshots to reduce unnecessary AWS storage costs.
- Optimizes cloud storage utilization efficiently and effectively.

---

## Prerequisites

1. **AWS Lambda**: You'll need an AWS account with permissions to create Lambda functions.
2. **IAM Role**: Ensure that the Lambda function has the necessary permissions for `ec2:DescribeSnapshots`, `ec2:DescribeInstances`, `ec2:DescribeVolumes`, and `ec2:DeleteSnapshot`.
3. **Boto3**: The Python AWS SDK, pre-installed in Lambda environment.

---

## Steps to Implement

### Step 1: Create an AWS Lambda Function

1. Go to AWS Lambda and create a new Lambda function.
2. Choose the **Python 3.x** runtime for the function.

### Step 2: Add the Lambda Code

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
```

### Step 3: Set up IAM Permissions

Ensure that the Lambda function has appropriate permissions to perform the following actions:
- `ec2:DescribeSnapshots`
- `ec2:DescribeInstances`
- `ec2:DescribeVolumes`
- `ec2:DeleteSnapshot`

You can attach these permissions via a Lambda execution role in the AWS IAM console.

### Step 4: Set up a Trigger (Optional)

If you want the function to run periodically, you can create a CloudWatch Event rule to trigger the Lambda function on a regular schedule (e.g., daily or weekly).

---

## Testing

1. Test the function by creating snapshots and detaching volumes or deleting instances.
2. After execution, check if the stale snapshots were deleted successfully.


