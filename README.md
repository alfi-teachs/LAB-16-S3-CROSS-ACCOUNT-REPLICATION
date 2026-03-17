# LAB-10-S3-CROSS-ACCOUNT-REPLICATION

# STEP 1. 
Source Account – Create Source Bucket

In Amazon Web Services console:

S3 → Create Bucket

Bucket name: source-bucket-123

Region: Any

Versioning: Enable

Check versioning:

S3 → source-bucket-123 → Properties → Versioning → Enabled

Replication will not work if versioning is disabled.

# STEP 2

2. Source Account – Create IAM Role

IAM → Roles → Create Role

Trusted entity: S3

Role name:

S3ReplicationRole

# STEP 3. 

Trust Relationship for the Role

Edit Trust Relationship of the role.

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}



This allows S3 to use the role for replication.

# STEP 4. 

IAM Inline Policy for the Role (Source Bucket Permissions)

Attach this inline policy to S3ReplicationRole.

Replace bucket names if needed.

Path:

IAM → Roles → S3ReplicationRole → Add Inline Policy

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetReplicationConfiguration",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::source-bucket-123"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersion",
        "s3:GetObjectVersionAcl"
      ],
      "Resource": "arn:aws:s3:::source-bucket-123/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete"
      ],
      "Resource": "arn:aws:s3:::destination-bucket-123/*"
    }
  ]
}


This policy allows the role to read from source and write to destination.


Destination Account
# step 5.

Create Destination Bucket

S3 → Create bucket

Bucket name:
destination-bucket-123

Enable Versioning

Replication will fail without it.


# step 6. 

Destination Bucket Policy

Go to:

S3 → destination-bucket-123 → Permissions → Bucket Policy

Add policy (replace SOURCE-ACCOUNT-ID).

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReplicationFromSourceAccount",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::SOURCE-ACCOUNT-ID:role/S3ReplicationRole"
      },
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete"
      ],
      "Resource": "arn:aws:s3:::destination-bucket-123/*"
    }
  ]
}

This allows the source account role to write objects.

# step 7.
 
 Create Replication Rule (Source Bucket)

Go to:

S3 → source-bucket-123
Management → Replication → Create rule

Configuration:

Rule name
cross-account-replication

Scope
Entire bucket

Destination
Another account

Destination bucket
destination-bucket-123

IAM Role
S3ReplicationRole

Optional:

Replicate delete markers ✔

Replicate existing objects ✔ (optional)

Save.

# step 8. 

Test Replication

Upload a file in:

source-bucket-123

Example:  test.txt
Within a few seconds it should appear automatically in:

destination-bucket-123

# Common mistakes (very important)

Most labs fail because of these:

Versioning not enabled in both buckets

Wrong bucket policy

IAM role missing ReplicateObject permission

Wrong account ID in policy




