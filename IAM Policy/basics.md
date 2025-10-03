# IAM Policies in AWS

An **IAM Policy** in AWS is a JSON document that defines permissions. 
These permissions determine **what actions** are allowed or denied on **which resources**, and under **what conditions**. 
Policies are central to AWS Identity and Access Management (IAM), ensuring secure and fine-grained access control across AWS services.

## Structure of an IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow" | "Deny",
      "Action": ["service:operation"],
      "Resource": ["arn:aws:service:region:account-id:resource"],
      "Condition": {
        "key": "value"
      }
    }
  ]
}
```


An IAM policy follows a structured JSON format with four main parts:

- **Version** – (Optional) Defines the policy language version. The latest is `"2012-10-17"`.  
- **Statement** – The core element, can contain one or more individual statements.  
- **Effect** – Either `"Allow"` or `"Deny"`. Deny always overrides Allow.  
- **Action** – Specifies which AWS service actions are covered (e.g., `"s3:PutObject"`).  
- **Resource** – Defines the ARN (Amazon Resource Name) of the resources affected.  
- **Condition** – (Optional) Defines additional constraints (e.g., IP address, time, MFA).  


```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow", 
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-example-bucket"
    }
  ]
}
```

## Types of IAM Policies

- **Identity-based Policies** – Attached to IAM users, groups, or roles. Define what those identities can or cannot do. Example: Allow a user to start/stop EC2 instances.  
- **Resource-based Policies** – Attached directly to resources (e.g., S3 bucket policies, KMS key policies). Define who (which identities/accounts) can access the resource and what actions they can take.  
- **Permissions Boundaries** – Advanced feature to set a maximum permission boundary for a user or role. The actual permissions are the intersection of identity policy and boundary.  
- **Service Control Policies (SCPs)** – Used in AWS Organizations. Apply restrictions at the account or organizational unit (OU) level.  
- **Session Policies** – Applied when a role session is assumed. Temporary, and limit what a role session can do.  


## Key Concepts

- **Default Deny**: If no policy explicitly allows an action, it is denied.  

- **Explicit Deny**: If a policy explicitly denies an action, it overrides any allows.  

- **Policy Evaluation Logic**: AWS evaluates all policies attached to the principal and the resource to determine the final decision.  



### Allow S3 Read-Only Access


```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket",
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```

### Require MFA for EC2 Actions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": { "aws:MultiFactorAuthPresent": "false" }
      }
    }
  ]
}
```