# IAM (Identity and Access Management)
AWS IAM (Identity and Access Management) is a service that helps securely manage & control access to AWS resources. It allows us to manage & creation of **Users, Groups, Roles, Policies and Access Keys** to control permissions effectively.

---

## **1. IAM Users**
An IAM **User** is an entity in AWS that represents a person or an application that interacts with AWS services. Each IAM user has **credentials** (username and password for AWS Management Console or access keys for API and CLI access).

### **Key Features of IAM Users:**
- Each user is unique and belongs to an AWS account.
- Users can be assigned individual permissions via policies.
- A user can belong to one or more IAM **Groups**.
- Users can have **Access Keys** and **MFA (Multi-Factor Authentication)** for added security.

### **Example Use Case:**
- A company has **three developers** who need access to the AWS EC2 service. Each developer is given an IAM user with permissions to launch and manage EC2 instances.
- A **developer** in a company needs access to an **S3 bucket** for uploading files. Instead of giving full AWS access, an **IAM user** is created with a policy allowing only S3 access.  
---

## **2. IAM Groups**
An IAM **Group** is a collection of IAM users. Instead of assigning permissions to individual users, you can assign permissions to the group, and all users in the group inherit those permissions.

### **Key Features of IAM Groups:**
- Simplifies permission management by assigning policies at the group level.
- Users in a group **inherit all permissions** assigned to the group.  
- A user can belong to **multiple groups**.
- Groups **do not have security credentials** (only users do).
  
### **Example Use Case:**
A company has **a DevOps team** with 10 members. Instead of assigning policies to each user separately, they create an **"EC2-Admins" group** and attach a policy allowing EC2 management. Any user added to this group gets EC2 management permissions.

---

## **3. IAM Roles**
An IAM **Role** is an entity that defines permissions but **does not have long-term credentials** like IAM users. Instead, it can be assumed by trusted users, applications, or AWS services.

### **Key Features of IAM Roles:**
- Used to grant **temporary** access to AWS resources.
- Cannot be directly accessed by users.
- Used for granting temporary credentials via **STS (Security Token Service)**.
- Assigned to AWS services (like EC2, Lambda) or external users (like federated identities).
- IAM users and applications assume roles instead of using access keys.
- **No Permanent Credentials:** Unlike users, roles do not have usernames/passwords.
- **Temporary Access:** When a role is assumed, AWS provides **temporary security credentials** (Access Key, Secret Key, and Session Token).
- **Assignable to AWS Services:** Roles allow AWS services (EC2, Lambda, ECS) to perform actions on behalf of a user or application.

### **Why Use IAM Roles?**
- **Secure access without hardcoded credentials.**
- **Fine-grained control over permissions.**
- **Allow cross-account access safely.**
- **Enable AWS services (like Lambda or EC2) to perform actions without human intervention.**
  
### **Example Use Case:**
- A web application running on an **EC2 instance** needs to access an **S3 bucket**. Instead of storing **access keys** in the instance, an IAM **Role** is attached to the EC2 instance, allowing it to access the S3 bucket securely.
- A company has an **EC2 instance** that needs to read/write data to an **S3 bucket**. Instead of storing access keys on the instance (which is insecure), an **IAM Role** is assigned to the EC2 instance with the necessary S3 permissions.

### **Example IAM Role Use Cases**

<details>
  <summary>Example 1: EC2 Instance Accessing S3</summary>

### **Example 1: EC2 Instance Accessing S3**
Imagine an EC2 instance that needs to read and write to an S3 bucket.

#### **Solution Using IAM Role:**
1. Create an **IAM Role** with permissions to access S3.
2. Attach this role to the EC2 instance.
3. The EC2 instance assumes the role dynamically to access S3.

#### **IAM Role Policy for EC2 to Access S3:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::my-bucket/*"
        }
    ]
}
```

- This policy allows the EC2 instance to **read and write** files in the `my-bucket` S3 bucket.

</details>


<details>
  <summary>Example 2: AWS Lambda Accessing DynamoDB</summary>
### **Example 2: AWS Lambda Accessing DynamoDB**
A Lambda function needs to **read data** from a DynamoDB table.

#### **Solution Using IAM Role:**
1. Create an IAM Role for Lambda.
2. Attach a policy allowing **read-only** access to DynamoDB.
3. Assign the role to the Lambda function.

#### **IAM Policy for Lambda to Read from DynamoDB:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "dynamodb:Scan",
            "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/MyTable"
        }
    ]
}
```

- This allows Lambda to **scan** the `MyTable` table in DynamoDB.

</details>

---

## **4. IAM Policies**
IAM **Policies** define **permissions** using JSON documents that specify what actions are **allowed or denied** on AWS resources. We can define policy through AWS Console as well not only in JSON Documents.

### **Types of IAM Policies:**
1. **AWS-Managed Policies:** Predefined policies by AWS (e.g., `AmazonS3FullAccess`).
2. **Customer-Managed Policies:** Custom policies created for specific use cases.
3. **Inline Policies:** Policies attached directly to IAM Users, Roles, or Groups.

### **Key Features of IAM Policies:**
- Policies are written in **JSON** format.
- Can be **attached to Users, Groups, or Roles**.
- Permissions are either **Allow** or **Deny**.
- AWS provides **Managed Policies** (predefined policies) and allows **Custom Policies**.

### **Policy Structure:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow" | "Deny",
            "Action": "aws-service:operation",
            "Resource": "arn:aws:service:region:account-id:resource",
            "Condition": { "optional conditions" }
        }
    ]
}
```
### **Example IAM Policies**
- A **marketing team** needs to access reports stored in an S3 bucket. Instead of giving them full access, a policy is created allowing only **read access** to the specific S3 bucket. 

<details>
  <summary>Example 1: Granting Read-Only Access to an S3 Bucket</summary>
  
### **Example 1: Granting Read-Only Access to an S3 Bucket**
A user needs **read-only** access to a specific S3 bucket.

#### **IAM Policy for Read-Only S3 Access:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example-bucket/*"
        }
    ]
}
```
- This policy allows reading files in `example-bucket`.

</details>

<details>
  <summary>Example 2: Allowing an EC2 Instance to Start and Stop Other EC2 Instances</summary>

### **Example 2: Allowing an EC2 Instance to Start and Stop Other EC2 Instances**
If an EC2 instance needs to **start and stop** other EC2 instances, attach this policy to the instance role.

#### **IAM Policy for EC2 Instance to Control Other Instances:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/*"
        }
    ]
}
```
✅ This policy lets the instance **start and stop** any EC2 instances in the same AWS account.

</details>

<details>
  <summary>Example 3: IAM Policy (Allowing Full Access to S3)</summary>

### **Example 3: IAM Policy (Allowing Full Access to S3)**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
```
This policy grants **full access** to all S3 buckets.

</details>

**2. AWS Management Console (UI Interface)**  
   - AWS provides a **visual editor** to create and modify IAM policies **without writing JSON manually**.  
   - Users can:
     - Select services (e.g., EC2, S3, Lambda)
     - Choose specific actions (Read, Write, Full Access, etc.)
     - Define resources and conditions
   - The UI automatically converts these selections into a JSON policy.

**3. AWS CLI & SDKs (Command Line & Code-Based Approach)**  
   - You can create, update, and attach policies using AWS CLI or SDKs.  
   - Example CLI command to create a policy:
     ```sh
     aws iam create-policy --policy-name S3FullAccess --policy-document file://s3-policy.json
     ```

---

## **Difference Between IAM Roles and IAM Policies**

| Feature | IAM Role | IAM Policy |
|---------|---------|------------|
| **Definition** | An entity that can be assumed by users, AWS services, or applications to gain temporary access. | A set of permissions that define what actions are allowed or denied. |
| **Usage** | Used for granting access **without long-term credentials**. | Defines permissions for Users, Groups, and Roles. |
| **Attached To** | Can be assigned to AWS services (EC2, Lambda) or external users. | Can be attached to **Users, Groups, or Roles**. |
| **Example Use Case** | An **EC2 instance** needs temporary access to an S3 bucket without storing credentials. | A **Developer User** needs read-only access to EC2 instances. |
| **Type of Entity** | It is an **identity** (like a user) that has permissions. | It is a **policy document** defining permissions. |
| **Persistence** | Provides **temporary access** through role assumption. | Policies define **long-term permissions**. |

### **Example: IAM Role vs. IAM Policy**
1. **IAM Role Example:**  
   - You create an IAM Role named **"S3AccessRole"** with permissions to read/write an S3 bucket.
   - An **EC2 instance** is assigned this role.
   - The EC2 instance assumes this role to access S3 without using access keys.

2. **IAM Policy Example:**  
   - You create a policy that **allows full access to S3**.
   - You attach this policy to an IAM **User** named "John".
   - Now, "John" can directly access S3 using the AWS console or CLI.

---

## **Best Practices for IAM Roles & Policies**
1. **Follow the Principle of Least Privilege** – Grant only the necessary permissions.
2. **Use IAM Roles Instead of IAM Users** for AWS services.
3. **Avoid Hardcoding Credentials** – Use IAM roles with temporary credentials.
4. **Use AWS-Managed Policies When Possible** – They are updated by AWS.
5. **Regularly Review IAM Roles & Policies** – Remove unused permissions.

---

## **5. Access Keys & Secret Keys**  
Access Keys (Access Key ID & Secret Access Key) are **used for programmatic access** to AWS services via AWS CLI, SDKs, and APIs.  

#### **Best Practices for Access Keys:**  
- **Do not share access keys** with others.  
- **Rotate keys** periodically.  
- Use **IAM roles instead** whenever possible for AWS services.  
- **Delete unused keys** to reduce security risks.  

#### **Example**  
A company runs a **Python script** that automatically uploads logs to an **S3 bucket**. The script needs programmatic access, so an **IAM user is created with an access key** that has permissions to upload files to S3.  

---

## **Conclusion**
- **IAM Users**: Represent actual people or applications.
- **IAM Groups**: Collection of users that share permissions.
- **IAM Roles**: Temporary access to AWS resources without long-term credentials.
- **IAM Policies**: JSON-based permissions attached to Users, Groups, or Roles.
- IAM **Roles** are used for **temporary access** without credentials.
- IAM **Policies** define **permissions** (who can do what).
- Roles can be assigned to **AWS services, users, and applications**.
- Policies can be **AWS-managed or custom-created**.
- Always follow **least privilege principles** to ensure security.
  
Using **IAM Roles** is a best practice over **IAM Users with access keys** for security reasons, especially when granting permissions to AWS services.
