# IAM Policies in AWS

An **IAM Policy** in AWS is a JSON document that defines permissions. 
These permissions determine **what actions** are allowed or denied on **which resources**, and under **what conditions**. 
Policies are central to AWS Identity and Access Management (IAM), ensuring secure and fine-grained access control across AWS services.

### IAM Policy types

- AWS Managed: A policy managed/pre-defined by AWS.
- Inline Polices: A policy that you directly **embed into a single IAM user, group, or role**.
- Customer Managed: Policies created and managed by **you (the customer)**.

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

### example of `NotAction` (used instead of `Deny`)

```json
{
  "version" : "2012-10-17",
  "statement" : [
    {
    "Effect" : "Allow",
    "NotAction": [
      "iam:*",
      "organization:*",
      "account:*"
    ],
    "Resource" :"*"
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


### Policy Evaluation

- Explicit DENY has precendence over ALLOW


### IAM Policy Variables and Tags

- IAM policy variables let you make dynamic, context-aware policies instead of hardcoding values. 
  They act like placeholders that get replaced when a request is evaluated.

  Examples of Common Policy Variables~(**aws specific**)

 - `aws:username` – The name of the IAM user making the request.
 - `aws:userid` – The unique identifier of the user.
 - `aws:PrincipalTag/Department` – A tag from the principal (e.g., the user or role).
 - `aws:RequestTag/Project` – A tag included in the request context (for creating/tagging resources).
 - `aws:ResourceTag/Environment` – A tag from the target resource.
 - `aws:CurrentTime` – The time of the request (useful for time-based access).
 - `aws:SourceIp` – The requester’s IP address.

```json
  {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mybucket/home/${aws:username}/*"
    }
  ]
}
```


(2) ***Service-Specific** Variables

Some services define their own policy variables:
- S3: s3:prefix, s3:max-keys, s3:x-amz-acl
- SNS: sns:Endpoint, sns:Protocol

(3) ***Tag-Based** Variables

IAM supports ABAC (Attribute-Based Access Control) using tags:

- `iam:ResourceTag/key-name` – Tags attached to the target resource
- `aws:PrincipalTag/key-name` – Tags attached to the IAM user or role

```json
"Condition": {
  "StringEquals": {
    "aws:ResourceTag/Department": "${aws:PrincipalTag/Department}"
  }
}
```