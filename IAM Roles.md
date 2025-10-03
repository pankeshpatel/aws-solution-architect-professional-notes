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


#### IAM Roles vs Resource Based Policies

# IAM Roles vs Resource-Based Policies

## IAM Roles
- **Definition**: An IAM Role is an AWS identity with specific permissions that can be assumed by users, applications, or services to perform actions in AWS.
- **Attached To**: Principals (IAM users, AWS services, federated identities).
- **Use Case**: Temporary access delegation. Instead of sharing long-term credentials, entities assume roles to get temporary credentials.
- **Trust Policy**: IAM Roles require a *trust policy* that defines **who** (principals) can assume the role.
- **Policy Attachment**: Roles can have **identity-based policies** (permissions) that define what actions they can perform after being assumed.
- **Typical Scenarios**:
  - EC2/Lambda execution roles (granting AWS service permissions).
  - Cross-account access (Account A’s user assumes a role in Account B).
  - Temporary elevated access with STS.

---

## Resource-Based Policies
- **Definition**: A policy that is attached **directly to an AWS resource** (instead of a user/role) to specify who can access it and what actions are allowed.
- **Attached To**: AWS resources (e.g., S3 bucket policy, KMS key policy, SNS topic policy, SQS queue policy, API Gateway resource policy).
- **Use Case**: Allowing **cross-account** or **external entity** access to a specific resource without needing a role assumption.
- **Trust Concept**: Resource-based policies do not require a separate trust policy. They directly define which principals (AWS accounts, IAM users, roles, services) can access the resource.
- **Policy Attachment**: They are embedded inside the resource itself.
- **Typical Scenarios**:
  - S3 Bucket Policy allowing another AWS account read/write access.
  - KMS Key Policy granting permissions to multiple accounts.
  - SNS Topic Policy allowing a service (like CloudWatch) to publish to it.

---

## Key Differences

| Aspect                  | IAM Roles                                           | Resource-Based Policies                           |
|--------------------------|-----------------------------------------------------|--------------------------------------------------|
| **Attached To**          | Principals (users, apps, services)                  | AWS resources (S3, KMS, SNS, SQS, etc.)          |
| **Access Mechanism**     | Entities **assume the role** to get temporary creds | Directly grant access to principals               |
| **Trust Model**          | Requires a **trust policy**                        | Trust is built into the resource policy itself    |
| **Best For**             | Temporary cross-service / cross-account access      | Direct resource sharing across accounts/services |
| **Example**              | EC2 instance assumes a role to access S3            | S3 bucket policy allowing Account B to access it |

---

## Choosing Between Them
- Use **IAM Roles** when you want:
  - Centralized identity management.
  - Temporary credentials via STS.
  - Cross-account access requiring explicit assumption.
- Use **Resource-Based Policies** when you want:
  - To directly attach permissions to the resource.
  - Simpler sharing model (no role assumption).
  - Access by multiple accounts or services without managing roles in each account.
