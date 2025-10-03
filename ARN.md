# 📌 Amazon Resource Names (ARNs) Cheat Sheet

---

## 1. What is an ARN?
- **ARN = Amazon Resource Name**
- A **unique identifier** for every AWS resource.
- Policies use ARNs in `"Resource"` to define *which resource(s)* an action applies to.

---

## 2. General ARN Format

```
arn:partition:service:region:account-id:resource
```


### Breakdown
1. **arn** → literal prefix (always `arn`)
2. **partition** → environment:
   - `aws` → Standard AWS
   - `aws-us-gov` → GovCloud
   - `aws-cn` → China regions
3. **service** → AWS service namespace (e.g., `s3`, `ec2`, `lambda`)
4. **region** → Region code (`us-east-1`, `ap-south-1`)  
   - May be omitted for global services like S3 or IAM
5. **account-id** → 12-digit AWS account ID  
   - May be omitted for global resources
6. **resource** → Service-specific resource path or ID

---

## 📌 ARN Examples by Service

### 🔹 Amazon S3 (Global service → no region/account)
- **Bucket**  
  `arn:aws:s3:::mybucket`
- **Object**  
  `arn:aws:s3:::mybucket/myobject.txt`

### 🔹 Amazon EC2
- **Instance**  
  `arn:aws:ec2:us-east-1:123456789012:instance/i-0abcd1234`
- **Volume**  
  `arn:aws:ec2:us-east-1:123456789012:volume/vol-0567efgh`
- **Security Group**  
  `arn:aws:ec2:us-east-1:123456789012:security-group/sg-123456`

### 🔹 AWS Lambda
- **Function**  
  `arn:aws:lambda:us-east-1:123456789012:function:MyFunction`
- **Alias/Version**  
  `arn:aws:lambda:us-east-1:123456789012:function:MyFunction:PROD`

### 🔹 DynamoDB
- **Table**  
  `arn:aws:dynamodb:us-east-1:123456789012:table/MyTable`
- **Index**  
  `arn:aws:dynamodb:us-east-1:123456789012:table/MyTable/index/MyIndex`

### 🔹 IAM (Global service → no region)
- **User**  
  `arn:aws:iam::123456789012:user/Bob`
- **Role**  
  `arn:aws:iam::123456789012:role/AdminRole`
- **Group**  
  `arn:aws:iam::123456789012:group/Developers`

### 🔹 Amazon RDS
- **DB Instance**  
  `arn:aws:rds:us-east-1:123456789012:db:mydatabase`
- **DB Snapshot**  
  `arn:aws:rds:us-east-1:123456789012:snapshot:mysnapshot`

### 🔹 Amazon SNS
- **Topic**  
  `arn:aws:sns:us-east-1:123456789012:mytopic`

### 🔹 Amazon SQS
- **Queue**  
  `arn:aws:sqs:us-east-1:123456789012:myqueue`

### 🔹 Amazon KMS
- **Key**  
  `arn:aws:kms:us-east-1:123456789012:key/1234abcd-12ab-34cd-56ef-1234567890ab`

### 🔹 AWS Organizations
- **Organization**  
  `arn:aws:organizations::123456789012:organization/o-abcdef1234`

---
