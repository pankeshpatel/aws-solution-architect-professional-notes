# 🔐 IAM Conditions Cheat Sheet

IAM **Conditions** are filters in a policy that refine **when** a statement applies.  
They follow the pattern:

```json
"Condition": {
  "<Operator>": {
    "<ConditionKey>": "Value"
  }
}
```


## 1. `<operator>` Cheat Sheet

| Category      | Operators                                                                 |
|---------------|---------------------------------------------------------------------------|
| **String**    | `StringEquals`, `StringNotEquals`, `StringEqualsIgnoreCase`, `StringNotEqualsIgnoreCase`, `StringLike`, `StringNotLike` |
| **Numeric**   | `NumericEquals`, `NumericNotEquals`, `NumericLessThan`, `NumericLessThanEquals`, `NumericGreaterThan`, `NumericGreaterThanEquals` |
| **Date/Time** | `DateEquals`, `DateNotEquals`, `DateLessThan`, `DateLessThanEquals`, `DateGreaterThan`, `DateGreaterThanEquals` |
| **Boolean**   | `Bool`                                                                  |
| **IP**        | `IpAddress`, `NotIpAddress`                                               |
| **Binary**    | `BinaryEquals`                                                            |
| **ARN**       | `ArnEquals`, `ArnNotEquals`, `ArnLike`, `ArnNotLike`                      |
| **Null**      |                                                                           |


## 2. Global Condition Keys

Available across **all AWS services**:

| Condition Key                | Description                          | Example Use Case                |
|-------------------------------|--------------------------------------|---------------------------------|
| `aws:SourceIp`               | IP address of requester              | Restrict by office IP           |
| `aws:CurrentTime`            | Current time (ISO 8601)              | Business-hours access           |
| `aws:EpochTime`              | Current time in epoch seconds        | Numeric time checks             |
| `aws:SecureTransport`        | True if HTTPS is used                | Force HTTPS                     |
| `aws:MultiFactorAuthPresent` | True if MFA present                  | Require MFA                     |
| `aws:MultiFactorAuthAge`     | Age of MFA (seconds)                 | Force re-authentication         |
| `aws:PrincipalArn`           | ARN of principal making request      | Restrict who can assume role    |
| `aws:PrincipalTag/TagKey`    | Tag on IAM principal (user/role)     | Match user to resources         |
| `aws:ResourceTag/TagKey`     | Tag on resource                      | Department-based access         |
| `aws:RequestTag/TagKey`      | Tag included in request              | Control resource tagging        |
| `aws:TagKeys`                | List of tag keys in request          | Enforce mandatory tags          |
| `aws:RequestedRegion`        | Region where request was made        | Restrict to `us-east-1`         |
| `aws:userid`                 | Unique AWS user ID                   | Track specific entity           |
| `aws:username`               | IAM username                         | Restrict by IAM user name       |
| `aws:UserAgent`              | User agent string of request         | Restrict to known applications  |

---

## 3. Service-Specific Condition Keys

### 🔹 Amazon S3
- `s3:x-amz-acl` → Object ACL (e.g., `public-read`)  
- `s3:x-amz-server-side-encryption` → Encryption (`AES256`, `aws:kms`)  
- `s3:prefix` → Prefix in list operations  
- `s3:ExistingObjectTag/TagKey` → Tag on existing S3 object  

### 🔹 Amazon EC2
- `ec2:InstanceType` → Restrict instance types (e.g., `t3.micro`)  
- `ec2:Tenancy` → `default`, `dedicated`  
- `ec2:Region` → Region restriction  
- `ec2:VolumeType` → EBS volume type (`gp2`, `io1`)  

### 🔹 DynamoDB
- `dynamodb:LeadingKeys` → Partition key(s) in the request  
- `dynamodb:Select` → `ALL_ATTRIBUTES`, `COUNT`  
- `dynamodb:ReturnConsumedCapacity` → Request must return usage details  

### 🔹 AWS Lambda
- `lambda:FunctionArn` → Restrict to specific function ARN(s)  
- `lambda:Layer` → Restrict Lambda layers  

---

## 4. Example Condition Snippets

### Restrict by IP
```json
"Condition": {
  "IpAddress": { "aws:SourceIp": "203.0.113.25/32" }
}
```


## ⏰ Time-Based Access
Allow access only between **Oct 1, 2025** and **Dec 31, 2025**.

```json
"Condition": {
  "DateGreaterThan": { "aws:CurrentTime": "2025-10-01T00:00:00Z" },
  "DateLessThan": { "aws:CurrentTime": "2025-12-31T23:59:59Z" }
}
```

## 🔑 Require MFA
Ensure the user is signed in with **Multi-Factor Authentication (MFA)**.

```json
"Condition": {
  "Bool": { "aws:MultiFactorAuthPresent": "true" }
}
```

## Require HTTPS
Deny requests that do not use HTTPS.

```json
"Condition": {
  "Bool": { "aws:SecureTransport": "true" }
}
```

## Tag-Based Access Control
Allow access only if the resource’s Department tag matches the user’s Department tag.

```json
"Condition": {
  "StringEquals": {
    "aws:ResourceTag/Department": "${aws:PrincipalTag/Department}"
  }
}
```

## 🔄 Multiple Conditions

✅ AND Logic (all must match)

Require MFA and restrict to a specific IP.

```json
"Condition": {
  "Bool": { "aws:MultiFactorAuthPresent": "true" },
  "IpAddress": { "aws:SourceIp": "203.0.113.25/32" }
}
```

OR Logic (any value works)

Allow requests from either of two IPs.

```json
"Condition": {
  "IpAddress": {
    "aws:SourceIp": ["203.0.113.25/32", "203.0.113.26/32"]
  }
}
```