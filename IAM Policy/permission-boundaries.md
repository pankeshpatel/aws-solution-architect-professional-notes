# IAM Permission Boundaries

## What is a Permission Boundary?
A **permission boundary** in AWS is an advanced IAM feature that sets the **maximum permissions** an IAM user or role can have.  
It does **not grant permissions by itself**; instead, it acts as a *guardrail* or *upper limit* on the permissions that identity-based policies (attached to the user or role) can provide.

Think of it as:
- **Identity-based Policy** → Grants permissions  
- **Permission Boundary** → Defines the *maximum allowed scope* of permissions  

The **effective permissions** are the **intersection** of both.

---

## Why Do We Need Permission Boundaries?
- **Delegated Administration**: Allow junior admins or developers to create roles/users but limit what maximum privileges they can grant.
- **Guardrails in Large Organizations**: Prevent overly permissive access (e.g., prevent granting `AdministratorAccess`) even if someone attaches such a policy.
- **Multi-Tenant SaaS or Enterprise Use Cases**: Limit what tenants or business units can do with their IAM resources.

---

## Example
Suppose you create a user `pankeshpatel`.

1. **Attached Permission Policy**: `AdministratorAccess`  
   (This by itself gives *all* permissions.)

2. **Permission Boundary**: `AmazonS3ReadOnlyAccess`  
   (This limits actions to only S3 read operations.)

✅ Result: Even though `AdministratorAccess` is attached, the **effective permission** is **S3 Read-Only**.

---

## Key Points
- **Permission boundaries only apply to users and roles** (not groups).
- They do not affect **resource-based policies** (e.g., S3 bucket policies, KMS key policies).
- They **limit** but never **grant** permissions.
- **Explicit Deny** in policies still takes precedence.
- Commonly used in **organizations, automation, and delegated IAM workflows**.

---

## Analogy
- **Policy** = "What you’re allowed to do."
- **Boundary** = "The fence around your permissions playground. You can’t go beyond it, even if your policy says so."

---

## Diagram (Conceptual)

     +-------------------+
     | Identity Policy    |  → ALLOW everything
     +-------------------+
                ∩
     +-------------------+
     | Permission Boundary|  → ALLOW S3 ReadOnly
     +-------------------+
                ↓
      Effective Permission
      = S3 ReadOnly


# Why Do We Need Permission Boundaries?

## Identity Policy vs Permission Boundary

- **Identity Policy**:  
  Attached directly to a user/role, defines **what that identity is allowed to do**.  
  Example: `AmazonS3FullAccess`.

- **Permission Boundary**:  
  Defines the **maximum scope** of permissions that can ever be granted, regardless of the identity policy.  
  Example: Limit to only `AmazonS3ReadOnlyAccess`.

The **effective permission** = Identity Policy ∩ Permission Boundary.

---

## But Why Not Just Use Identity Policy Alone?

### 1. Delegated Administration
Imagine you allow **developers** to create IAM roles for their apps.  
- If you only rely on **identity policies**, developers could attach `AdministratorAccess` to their roles — giving way more power than intended.  
- A **permission boundary** prevents this by saying:  
  > "Even if you attach Admin policy, your maximum scope is S3-ReadOnly."

### 2. Guardrails in Automation
In large enterprises, automation scripts (e.g., CI/CD pipelines) may create IAM roles dynamically.  
- Without boundaries, those scripts could accidentally assign dangerous permissions.  
- With boundaries, automation can only create roles **within safe limits**.

### 3. Multi-Tenant / SaaS Scenarios
In a multi-tenant SaaS application:  
- Each tenant's admin might create roles for their users.  
- You don’t want them escaping their sandbox.  
- A permission boundary ensures tenants can’t grant access outside their allocated scope.

---

## Example

Suppose `Developer` is allowed to create IAM roles.

- **Identity Policy for Developer**:  



# Use Case: Delegated Administration with Permission Boundaries

## Scenario
- You are a **central security admin** in a large company.
- You want to let **project admins (delegated admins)** create IAM roles and users for their applications.  
- BUT you don’t want them to give **excessive privileges** (e.g., AdministratorAccess).

---

## Without Permission Boundaries
1. You give project admins the following policy:
   ```json
   {
     "Effect": "Allow",
     "Action": [
       "iam:CreateUser",
       "iam:CreateRole",
       "iam:AttachUserPolicy",
       "iam:AttachRolePolicy"
     ],
     "Resource": "*"
   }

   ```
-  They can now create users/roles and attach any policy (including `AdministratorAccess`).
- Too much power! They could escalate to full AWS admin.


With Permission Boundaries

Instead, you use a permission boundary as a guardrail.

Step 1: Define a boundary policy (the maximum allowed privileges):
- This boundary says: At most, you can work with S3 and DynamoDB. Nothing else.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "dynamodb:*"
      ],
      "Resource": "*"
    }
  ]
}
```

Why This Matters

-  Central security team → defines guardrails (the boundary).
- Delegated project admins → operate within those guardrails, without needing constant oversight.
- Prevents privilege escalation while still enabling autonomy.