

you only use "Principal" in ***resource-based policies***
(like S3 bucket policies, IAM trust policies, KMS key policies, etc.).



###  General "Principal" Syntax

```json
"Principal": {
  "<principal-type>": "<identifier>"
}
```

Where:
- `principal-type` = `AWS`, `Service`, `Federated`, or `CanonicalUser`
- `identifier` = `ARN`, `AWS service name`, `account ID`, or `*` (wildcard)

#### 1. AWS Principal (IAM Users, Roles, Accounts)

```json
"Principal": {
  "AWS": "arn:aws:iam::123456789012:user/Alice"
}
```

- Grants access to a specific IAM user.
- Can also be an IAM role or full AWS account.

👉 Multiple values:

```json
"Principal": {
  "AWS": [
    "arn:aws:iam::123456789012:user/Alice",
    "arn:aws:iam::123456789012:role/DevRole",
    "123456789012"
  ]
}
```

#### 2. Service Principal (AWS services)

```json
"Principal": {
  "Service": "ec2.amazonaws.com"
}
```

- Grants access to an AWS service (so the service can assume a role).
- Common in trust policies (e.g., allow EC2 or Lambda to assume a role).

Multiple services:

```json
"Principal": {
  "Service": ["lambda.amazonaws.com", "ec2.amazonaws.com"]
}
```

#### 3. Federated Principal (External Identity Providers)

```json
"Principal": {
  "Federated": "cognito-identity.amazonaws.com"
}
```
- Grants access to identities from an external IdP (Cognito, SAML, Google, Facebook, etc.).

Example (SAML IdP):

```json
"Principal": {
  "Federated": "arn:aws:iam::123456789012:saml-provider/MyIdP"
}
```

#### 4. CanonicalUser Principal (S3 only)

```json
"Principal": {
  "CanonicalUser": "79a59df900b949e55d96a1e698f0xxxxexample"
}
```

- Used only in S3 bucket policies.
- Represents a specific AWS account using its canonical user ID.

#### 5. Wildcard (Anyone)

```json
"Principal": "*"
```

- Grants access to everyone (all AWS accounts, all users, all services).
- ⚠️ Dangerous if not paired with conditions.
- Example: public-read S3 bucket.


## ✅ Summary Cheat Sheet

| Principal Type     | Syntax Example                                                                 |
|--------------------|--------------------------------------------------------------------------------|
| **AWS User/Role**  | `"AWS": "arn:aws:iam::123456789012:user/Alice"`                                |
| **AWS Account**    | `"AWS": "123456789012"`                                                        |
| **Multiple AWS**   | `"AWS": ["arn:aws:iam::123456789012:user/Alice", "arn:aws:iam::123456789012:role/DevRole"]` |
| **Service**        | `"Service": "ec2.amazonaws.com"`                                               |
| **Multiple Services** | `"Service": ["ec2.amazonaws.com", "lambda.amazonaws.com"]`                  |
| **Federated**      | `"Federated": "cognito-identity.amazonaws.com"`                                |
| **CanonicalUser**  | `"CanonicalUser": "<canonical-user-id>" (S3 only)`                             |
| **Wildcard**       | `"*"` (any principal)                                                          |
