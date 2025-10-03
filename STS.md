### AWS STS (Security Token Service)

- It is an AWS Service.
- It lets you request temporary, limited-privilege credentials for AWS resources.
- Instead of giving long-term IAM user access keys, you can use STS to get short-lived credentials that expire automatically.


#### How STS Works

1. Client/User makes a request (via API, CLI, SDK, or Identity Provider).
2. STS returns:
   - Access Key ID
   - Secret Access Key
   - Session Token (all temporary)

3. Client uses these to make AWS API calls.
4. Credentials automatically expire after their lifetime (default 1 hour, can extend up to 12 hours depending on role settings).


#### 📌 STS in IAM Trust Policy

- The Trust Policy of a role is where AWS defines who can call STS to assume that role.
- In other words, STS (`sts:AssumeRole`) appears only in the ***trust policy***, not in the role’s permission policy.


##### `Trust Policy`

-  Here, the Action = sts:AssumeRole is in the trust policy.
- This means: “User pankesh is trusted to call STS and get temporary credentials for this role.”

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:user/pankesh"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Role’s Permissions Policy (for comparison)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```


# STS Scenario # 1 - CROSS Account Access


- Account A: Has a Lambda function.
- Account B: Owns a DynamoDB table (say OrdersTable).
- Requirement: Lambda in Account A should read/write to DynamoDB in Account B.


How Cross-Account Access Works

 - To enable cross-account access (Lambda in Account A accessing DynamoDB in Account B), we need three main parts:

#### 1. IAM Role in Account B (Target Role)

- Trusts Account A’s Lambda execution role.
- Grants DynamoDB permissions.

Trust Policy (Account B Role – DynamoDBAccessRole):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_A_ID:role/LambdaExecutionRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Permissions Policy (Attach to Role in Account B):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:ACCOUNT_B_ID:table/OrdersTable"
    }
  ]
}
```

#### 2. Lambda Execution Role in Account A (Caller)

- Must be allowed to call sts:AssumeRole on Account B’s role.

Policy for Lambda Execution Role in Account A:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::ACCOUNT_B_ID:role/DynamoDBAccessRole"
    }
  ]
}
```

#### 3. Lambda Function Code

- Uses STS AssumeRole to get temporary credentials.
- Uses those credentials to call DynamoDB in Account B.

```python
import boto3

def lambda_handler(event, context):
    sts_client = boto3.client('sts')

    # Assume the cross-account role
    response = sts_client.assume_role(
        RoleArn="arn:aws:iam::ACCOUNT_B_ID:role/DynamoDBAccessRole",
        RoleSessionName="LambdaToDynamoDBSession"
    )

    # Extract temporary credentials
    creds = response['Credentials']

    # Create DynamoDB client with assumed-role credentials
    dynamodb = boto3.client(
        'dynamodb',
        aws_access_key_id=creds['AccessKeyId'],
        aws_secret_access_key=creds['SecretAccessKey'],
        aws_session_token=creds['SessionToken'],
        region_name="us-east-1"
    )

    # Example: read from DynamoDB
    result = dynamodb.get_item(
        TableName="OrdersTable",
        Key={"OrderId": {"S": "12345"}}
    )

    return result
```

#### Summary

- Lambda in Account A runs under `LambdaExecutionRole`.
- Lambda calls STS AssumeRole on Account B’s `DynamoDBAccessRole`.
- Trust policy in Account B allows this.
- STS issues temporary credentials.
- Lambda uses those credentials to access DynamoDB in Account B.


### Summary of STS Use Cases

 - ✅ Cross-Account Access — classic case you already know
 - ✅ Federated Identity Access — SSO with SAML/OIDC providers
 - ✅ MFA-Protected API Access — require MFA for session credentials
 - ✅ Temporary Elevated Access — just-in-time admin roles
 - ✅ Mobile/Web App Access — Cognito + STS for short-lived credentials
 - ✅ Service-to-Service Access — scoped temporary credentials inside the same account
 - ✅ Auditing / Debugging — check who you are with GetCallerIdentity