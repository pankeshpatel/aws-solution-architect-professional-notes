# IAM Roles in AWS

## What is an IAM Role?
- An **IAM Role** is an AWS identity similar to an IAM user, but **it does not have long-term credentials** 
(like username/password or permanent access keys). 

- Instead, it provides **temporary security credentials** that can be assumed by trusted entities (users, applications, services, 
or even AWS accounts).

Roles are designed for **delegated access**—they allow you to define what actions can be performed and on which resources, without tying those permissions to a single permanent user.

---

## Key Features of IAM Roles
- **No long-term credentials**: Roles use **temporary credentials** provided by AWS STS (Security Token Service).
- **Assumable by trusted entities**: Can be assumed by IAM users, applications, AWS services, or external identities.
- **Policies attached**: Roles have permission policies that define what the role allows/denies.
- **Cross-account usage**: Roles are essential for granting access across different AWS accounts.

---

## Trust Policy vs. Permission Policy
An IAM Role has **two types of policies**:

1. **Trust Policy**  
   Defines **who can assume the role** (principal entities). Example: An EC2 instance, a user from another AWS account, or AWS Lambda.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": { "Service": "ec2.amazonaws.com" },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

2. **Permission Policy**
Defines what actions and resources are allowed when the role is assumed. Example: Allowing read access to an S3 bucket.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

## Common Use Cases of IAM Roles

1. EC2 Instance Role
    Attach a role to an EC2 instance so applications running on it can securely access AWS services (e.g., S3, DynamoDB) without storing keys.

2. Lambda Execution Role
    A Lambda function needs permissions to access DynamoDB, S3, CloudWatch Logs, etc.

3. Cross-Account Access
    An IAM Role in **Account B** can be assumed by a Lambda in **Account A** to access DynamoDB or S3 in Account B.

4. Federated Access
    Corporate users sign in via **SSO (SAML, OIDC)** and assume a role to access AWS without creating individual IAM users.

5. Temporary Elevated Access
    Admins assume roles with higher privileges only when needed (**just-in-time access**).


## How Role Assumption Works

1. **Entity Requests Role**  
   An IAM user, AWS service, or external identity tries to assume a role.

2. **STS Issues Credentials**  
   AWS STS returns temporary credentials (Access Key, Secret Key, Session Token).

3. **Entity Uses Credentials**  
   The entity uses the temporary credentials to make AWS API calls.

4. **Session Expiration**  
   Credentials expire after a set duration (default 1 hour, max 12 hours for most roles).
