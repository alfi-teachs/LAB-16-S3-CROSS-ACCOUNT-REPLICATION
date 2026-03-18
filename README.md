## LAB-16: S3 Cross-Account Replication

### What you will learn
- Configure replication between two AWS accounts  
- Create IAM Role for S3 replication  
- Apply correct permissions and policies  
- Automatically replicate objects across accounts  

---

## Source Account

### Step 1: Create Source Bucket
- Go to AWS Console → S3  
- Click **Create bucket**

Configuration:
- Bucket name: `source-bucket-123`
- Region: Any
- **Enable Versioning**

Verify:
- S3 → source-bucket-123 → Properties → Versioning → Enabled  

> Replication will NOT work without versioning.

---

### Step 2: Create IAM Role
- Go to IAM → Roles → Create role  
- Trusted entity: **AWS Service**  
- Use case: **S3**  

Role name:
```
S3ReplicationRole
```

---

### Step 3: Trust Relationship

Edit trust relationship of the role:

```json
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
```

---

### Step 4: Inline Policy (Source Permissions)

Go to:
IAM → Roles → S3ReplicationRole → Add Inline Policy

```json
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
```

---

## Destination Account

### Step 5: Create Destination Bucket
- Go to S3 → Create bucket  

Configuration:
- Bucket name: `destination-bucket-123`
- **Enable Versioning**

> Replication will fail without versioning.

---

### Step 6: Destination Bucket Policy

Go to:
S3 → destination-bucket-123 → Permissions → Bucket Policy  

Replace `SOURCE-ACCOUNT-ID` with actual account ID.

```json
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
```

---

## Step 7: Create Replication Rule (Source Bucket)

Go to:
S3 → source-bucket-123 → Management → Replication → Create rule  

Configuration:
- Rule name: `cross-account-replication`
- Scope: **Entire bucket**
- Destination: **Another account**
- Destination bucket: `destination-bucket-123`
- IAM Role: `S3ReplicationRole`

Optional:
- ✔ Replicate delete markers  
- ✔ Replicate existing objects  

Click **Save**

---

## Step 8: Test Replication

- Upload file in:
  ```
  source-bucket-123
  ```

Example:
```
test.txt
```

✅ Within seconds, it should appear in:
```
destination-bucket-123
```

---

## Common Mistakes (Very Important)

- Versioning not enabled in BOTH buckets  
- Wrong bucket policy in destination  
- Missing permissions in IAM role  
- Incorrect account ID in policy  
- Missing `/*` in resource ARN  

---

## Final Result
- Cross-account replication configured  
- Objects automatically copied to destination  
- Secure IAM-based access between accounts  




