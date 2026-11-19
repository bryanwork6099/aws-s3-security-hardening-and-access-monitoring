# AWS S3 Security Hardening and Access Monitoring
## Cybersecurity Practical Lab Report

**Project Category:** AWS Cloud Security / Cybersecurity  
**Difficulty Level:** Beginner to Intermediate  
**Platform:** Amazon Web Services (AWS)  
**Primary Service:** Amazon Simple Storage Service (Amazon S3)  
**Additional Services:** AWS IAM, AWS KMS, AWS CloudTrail, Amazon CloudWatch  
**Project Type:** Hands-on Cloud Security Lab

---

# 1. Project Overview

## 1.1 Introduction

Amazon Simple Storage Service (Amazon S3) is a scalable object storage service provided by Amazon Web Services (AWS). It allows users and organizations to store, retrieve, and manage large amounts of data.

Although Amazon S3 provides several built-in security features, incorrectly configured buckets can expose sensitive information, potentially resulting in unauthorized access, data leakage, or data loss.

This project focuses on implementing security best practices to protect data stored in Amazon S3 buckets.

The lab demonstrates how to configure access controls, enable encryption, protect data against accidental deletion, monitor user activities, and identify potential security misconfigurations.

Through practical exercises, this project aims to develop foundational knowledge of AWS cloud security and demonstrate how security controls can be implemented and validated in a real AWS environment.

## 1.2 Project Objectives

The main objectives are:

1. Create and configure a secure Amazon S3 bucket.
2. Prevent unauthorized public access.
3. Enable S3 bucket versioning to protect against accidental data deletion or modification.
4. Implement server-side encryption to protect data at rest.
5. Configure IAM permissions and S3 bucket policies.
6. Enforce secure HTTPS connections.
7. Enforce encryption requirements for uploaded objects.
8. Implement least-privilege access control.
9. Monitor S3 activities using AWS CloudTrail.
10. Identify and remediate potential S3 security misconfigurations.

## 1.3 Expected Learning Outcomes

Upon completing this project, the learner should understand:

- Amazon S3 security configurations.
- AWS Identity and Access Management (IAM).
- Bucket policies and access permissions.
- Server-side encryption and AWS Key Management Service (KMS).
- Data protection through versioning.
- Secure data transmission using HTTPS.
- CloudTrail auditing and security monitoring.
- Basic cloud security testing and remediation.

---

# 2. Prerequisites

Before starting the lab, the following requirements should be met.

| Requirement | Description |
|---|---|
| AWS Account | An active AWS account |
| AWS Management Console | Access to the AWS Management Console |
| AWS CLI | Installed and configured on the local machine |
| IAM Knowledge | Basic understanding of users, roles, and policies |
| Cloud Knowledge | Basic understanding of cloud computing and storage |
| Text Editor | Visual Studio Code or Notepad++ for editing policy JSON files |

## 2.1 AWS CLI Configuration

Install the AWS CLI and configure the credentials using:

```powershell
aws configure
```

Provide the following information:

- AWS Access Key ID
- AWS Secret Access Key
- Default region name
- Default output format

Verify the configuration:

```powershell
aws configure list
```

Verify the currently authenticated AWS identity:

```powershell
aws sts get-caller-identity
```

**Security consideration:** Avoid using the AWS root account for routine activities. Prefer an IAM role or appropriately restricted identity. For longer-term use, AWS IAM Identity Center is preferable to creating permanent access keys for individual users.

---

# 3. AWS Architecture

## 3.1 Architecture Overview

The project uses Amazon S3 as the primary storage service, with additional AWS services providing access management, encryption, auditing, and monitoring.

### Main AWS Components

| AWS Service | Function |
|---|---|
| Amazon S3 | Stores files and objects securely |
| AWS IAM | Manages identities and access permissions |
| AWS KMS | Manages encryption keys |
| AWS CloudTrail | Records AWS API activities and S3 object-level events |
| Amazon CloudWatch | Provides monitoring and log analysis |

### Architecture Flow

1. An authorized IAM identity attempts to access an S3 bucket.
2. IAM policies and S3 bucket policies determine whether the requested action is permitted.
3. Amazon S3 applies the configured access restrictions and encryption settings.
4. Objects are stored with server-side encryption.
5. Versioning protects objects against accidental overwrites and deletions.
6. CloudTrail records configured S3 API and object-level activities.
7. Server access logging can record supported S3 request details in a separate logging bucket.
8. CloudWatch can be used for log monitoring and alerting when configured.

### Logical Architecture Diagram

```text
             Administrator / IAM Identity
                         |
                         |
                  AWS IAM Policies
                         |
                         v
                Amazon S3 Bucket
                (Secure Storage)
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
      Versioning      Encryption    Bucket Policies
                         |
                         v
                      AWS KMS

          +--------------+--------------+
          |                             |
          v                             v
     AWS CloudTrail              S3 Access Logs
          |                             |
          v                             v
     CloudWatch Logs              Logging Bucket
          |
          v
    Security Monitoring
```

---

# 4. Practical Implementation

## Exercise 1: Create an S3 Bucket

### Objective

Create an Amazon S3 bucket and configure its initial security settings to prevent public access.

### Implementation Steps

1. Sign in to the AWS Management Console.
2. Navigate to Amazon S3.
3. Select **Create bucket**.
4. Configure the following settings:

| Configuration | Value |
|---|---|
| Bucket name | A globally unique bucket name |
| AWS Region | Select an appropriate region |
| Object Ownership | Bucket owner enforced |
| Block all public access | Enabled |
| Bucket Versioning | Initially disabled |
| Default Encryption | Amazon S3 managed keys (SSE-S3) initially |

5. Leave Block Public Access enabled.
6. Select **Create bucket**.

### Security Considerations

S3 Block Public Access helps prevent accidental public exposure of stored objects.

The four Block Public Access settings should remain enabled unless there is a clearly justified requirement for public access.

### Expected Result

- An S3 bucket is successfully created.
- Public access is blocked.
- The bucket is not accessible anonymously.

### Validation

Using AWS CLI:

```powershell
aws s3api get-public-access-block --bucket YOUR_BUCKET_NAME
```

Expected result: All four public access block settings should return `true`.

---

## Exercise 2: Enable S3 Bucket Versioning

### Objective

Enable bucket versioning to preserve multiple versions of an object.

### Implementation Steps

1. Navigate to Amazon S3.
2. Select the previously created bucket.
3. Open the **Properties** tab.
4. Locate **Bucket Versioning**.
5. Select **Edit**.
6. Enable versioning.
7. Save the configuration.

### CLI Configuration

```powershell
aws s3api put-bucket-versioning --bucket YOUR_BUCKET_NAME --versioning-configuration Status=Enabled
```

Verify the configuration:

```powershell
aws s3api get-bucket-versioning --bucket YOUR_BUCKET_NAME
```

### Security Benefits

Versioning provides additional protection against:

- Accidental object deletion.
- Unintended object overwrites.
- Certain ransomware-related file modification scenarios.
- Accidental changes to important files.

### Testing

1. Upload a test file named `test.txt`.
2. Modify its contents.
3. Upload the modified file using the same object name.
4. Navigate to the S3 bucket.
5. Enable **Show versions** in the object listing.
6. Verify that multiple object versions exist.

### Expected Result

Multiple versions of the same object should be visible.

**Important:** Versioning does not replace backups. A user with sufficient permissions can permanently delete object versions. Consider S3 Object Lock for stronger retention protection where required.

---

## Exercise 3: Enable Server-Side Encryption

### Objective

Configure server-side encryption to protect stored data.

### Encryption Options

Amazon S3 supports different server-side encryption methods.

| Encryption Type | Description |
|---|---|
| SSE-S3 | Uses Amazon S3-managed encryption keys |
| SSE-KMS | Uses AWS KMS keys for encryption and key management |
| DSSE-KMS | Uses dual-layer server-side encryption with AWS KMS |

For this project, SSE-S3 can be used initially, followed by SSE-KMS as an additional learning exercise.

### Implementation Steps

1. Navigate to the S3 bucket.
2. Open **Properties**.
3. Locate **Default encryption**.
4. Select **Edit**.
5. Choose SSE-KMS.
6. Select an existing AWS KMS key or create a customer managed key.
7. Save the configuration.

### Security Benefits

Encryption protects data at rest and reduces the risk of unauthorized access to stored information.

SSE-KMS additionally provides key management capabilities and allows key usage to be controlled through IAM and KMS key policies.

### Validation

```powershell
aws s3api get-bucket-encryption --bucket YOUR_BUCKET_NAME
```

Upload a test file and inspect its encryption configuration:

```powershell
aws s3api head-object --bucket YOUR_BUCKET_NAME --key test.txt
```

Check the `ServerSideEncryption` field and, when applicable, the KMS key identifier.

### Expected Result

- Default bucket encryption is enabled.
- Uploaded objects are encrypted.
- SSE-KMS objects reference the configured KMS key.

**Security consideration:** SSE-KMS introduces additional KMS permissions and potentially additional charges. The identity uploading or downloading an object may require appropriate KMS permissions, including `kms:GenerateDataKey` and `kms:Decrypt`, depending on the operation.

---

## Exercise 4: Configure S3 Bucket Policies

### Objective

Implement resource-based access control to allow only authorized identities to access S3 objects.

### Implementation Steps

1. Navigate to the S3 bucket.
2. Select **Permissions**.
3. Locate **Bucket policy**.
4. Select **Edit**.
5. Add the following example policy.

### Example Bucket Policy

The following policy grants an explicitly specified IAM user permission to retrieve objects.

Replace the account ID and username with the appropriate values.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSpecificUserReadAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/username"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

6. Save the policy.

### Explanation

| Policy Element | Function |
|---|---|
| Version | Defines the policy language version |
| Effect | Determines whether the action is allowed or denied |
| Principal | Identifies the permitted identity |
| Action | Specifies the permitted S3 operation |
| Resource | Identifies the bucket objects covered by the policy |

### Security Benefits

Bucket policies provide centralized control over access to S3 resources.

They can be used to:

- Restrict access to specific IAM identities.
- Prevent unauthorized object retrieval.
- Deny insecure connections.
- Enforce encryption requirements.

**Important:** An explicit Allow in a bucket policy does not automatically override an explicit Deny elsewhere. The identity must also satisfy applicable IAM, KMS, and organizational permissions.

---

## Exercise 5: Configure S3 Server Access Logging

### Objective

Enable server access logging to record requests made to the S3 bucket.

### Implementation Steps

1. Create a separate S3 bucket for storing access logs.
2. Navigate to the primary S3 bucket.
3. Select **Properties**.
4. Locate **Server access logging**.
5. Select **Edit**.
6. Enable server access logging.
7. Specify the separate logging bucket.
8. Configure a suitable log object prefix.
9. Save the configuration.

### Example Configuration

| Configuration | Value |
|---|---|
| Source bucket | YOUR_BUCKET_NAME |
| Destination bucket | YOUR_LOG_BUCKET |
| Log prefix | `s3-access-logs/` |

### Security Benefits

Server access logging provides information about requests made to an S3 bucket.

It can help identify:

- Access attempts.
- Object retrieval requests.
- Object upload requests.
- Failed access attempts.
- Potentially suspicious access patterns.

### Validation

Generate some S3 activity:

```powershell
aws s3 ls s3://YOUR_BUCKET_NAME
```

Upload a test object:

```powershell
aws s3 cp test.txt s3://YOUR_BUCKET_NAME/
```

Access logs may take time to be delivered. Verify that log files eventually appear in the destination bucket.

**Important:** Configure the destination bucket with the appropriate permissions for S3 log delivery. Protect the logging bucket from unauthorized modification and deletion. Access logging delivery is not guaranteed to be immediate or complete.

---

## Exercise 6: Enforce HTTPS Connections

### Objective

Ensure that connections to the S3 bucket use HTTPS instead of insecure HTTP.

### Security Background

HTTPS encrypts communications between clients and AWS services using TLS.

Without secure transport, data could be exposed to interception or modification while being transmitted across an insecure network.

### Implementation Steps

1. Navigate to the S3 bucket.
2. Open **Permissions**.
3. Locate **Bucket policy**.
4. Add the following statement to the existing bucket policy.

### Example Policy Statement

```json
{
  "Sid": "DenyInsecureTransport",
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": [
    "arn:aws:s3:::YOUR_BUCKET_NAME",
    "arn:aws:s3:::YOUR_BUCKET_NAME/*"
  ],
  "Condition": {
    "Bool": {
      "aws:SecureTransport": "false"
    }
  }
}
```

### Explanation

The policy explicitly denies S3 requests that do not use secure transport.

### Security Benefits

- Protects data in transit.
- Prevents insecure HTTP connections.
- Enforces encrypted communication with the S3 bucket.

### Expected Result

Requests made through HTTPS should continue working, while insecure HTTP requests should be denied.

---

## Exercise 7: Enforce Encryption Requirements

### Objective

Ensure that uploaded objects comply with the required encryption configuration.

### Implementation Steps

1. Navigate to the S3 bucket.
2. Open **Permissions**.
3. Locate **Bucket policy**.
4. Add an encryption enforcement statement.
5. Save the policy.

### Example Policy Statement

This example requires uploads to explicitly specify SSE-KMS and the selected customer managed KMS key.

Replace `YOUR_KMS_KEY_ARN` with the actual key ARN.

```json
{
  "Sid": "DenyUploadsWithoutRequiredEncryption",
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*",
  "Condition": {
    "StringNotEquals": {
      "s3:x-amz-server-side-encryption": "aws:kms",
      "s3:x-amz-server-side-encryption-aws-kms-key-id": "YOUR_KMS_KEY_ARN"
    }
  }
}
```

### Important Configuration Note

This example is intended for clients that explicitly send the SSE-KMS encryption headers.

Default bucket encryption alone does not necessarily satisfy a bucket policy requiring those headers. Test the policy carefully before applying it to production workloads.

### Security Benefits

- Enforces encryption requirements.
- Prevents non-compliant uploads.
- Ensures uploaded objects use the intended KMS key.

### Validation

Test an upload without encryption headers:

```powershell
aws s3 cp test.txt s3://YOUR_BUCKET_NAME/
```

Then test with the required encryption configuration:

```powershell
aws s3 cp test.txt s3://YOUR_BUCKET_NAME/ --sse aws:kms --sse-kms-key-id YOUR_KMS_KEY_ARN
```

Expected result:

- The upload without the required headers should be denied.
- The upload using the required SSE-KMS configuration should succeed if all required permissions are in place.

---

## Exercise 8: Test IAM Access Permissions

### Objective

Understand and implement the principle of least privilege by creating identities with different levels of S3 access.

### Implementation Steps

1. Navigate to AWS IAM.
2. Create or select two test identities.
3. Assign the appropriate permissions.

Example:

| Identity | Permissions |
|---|---|
| S3-ReadOnly-User | Read-only access to the test bucket |
| S3-NoAccess-User | No permissions to access the test bucket |

Use temporary credentials or role assumption for testing where possible.

### Example Read-Only IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

If SSE-KMS is used, provide the necessary KMS permissions as well.

### Testing

Test the read-only identity:

```powershell
aws s3 ls s3://YOUR_BUCKET_NAME/
```

Attempt to upload a file:

```powershell
aws s3 cp test.txt s3://YOUR_BUCKET_NAME/
```

The upload should fail because the identity does not have the required write permission.

Next, test the identity with no access.

Attempt to list the bucket:

```powershell
aws s3 ls s3://YOUR_BUCKET_NAME/
```

### Expected Result

| Test | Expected Result |
|---|---|
| Read-only user lists objects | Allowed |
| Read-only user downloads objects | Allowed |
| Read-only user uploads objects | Denied |
| No-access user lists objects | Denied |

### Security Benefits

This exercise demonstrates:

- Least-privilege access.
- IAM permission management.
- Access restriction.
- Authorization testing.
- Understanding of explicit and implicit access denials.

---

## Exercise 9: Monitor S3 Activities Using AWS CloudTrail

### Objective

Use AWS CloudTrail to monitor and investigate activities performed on the S3 bucket.

### Security Background

AWS CloudTrail records AWS account activity and API events.

For this exercise, S3 object-level data events should be enabled because ordinary management event logging does not automatically record every object upload, download, or deletion.

### Implementation Steps

1. Navigate to AWS CloudTrail.
2. Create a new trail or use an existing trail.
3. Configure a destination S3 bucket for CloudTrail logs.
4. Enable management event logging as appropriate.
5. Configure data event logging.
6. Select Amazon S3.
7. Specify the test bucket.
8. Enable the required read and write object events.
9. Save the configuration.

### Activities to Monitor

Perform the following activities:

| Activity | S3 API Event |
|---|---|
| Upload object | PutObject |
| Download object | GetObject |
| Delete object | DeleteObject |
| Copy object | CopyObject |

### Testing

Upload a file:

```powershell
aws s3 cp test.txt s3://YOUR_BUCKET_NAME/
```

Download the file:

```powershell
aws s3 cp s3://YOUR_BUCKET_NAME/test.txt downloaded.txt
```

Delete the test file:

```powershell
aws s3 rm s3://YOUR_BUCKET_NAME/test.txt
```

Navigate to CloudTrail Event History or inspect the configured trail logs.

For object-level events, use the configured S3 data event records.

### Security Benefits

CloudTrail helps security teams:

- Identify who performed an action.
- Determine when an action occurred.
- Identify the AWS API operation.
- Investigate suspicious activity.
- Support security auditing and incident investigations.

**Important:** S3 data events can incur additional charges. Configure event selectors carefully and monitor usage.

---

## Exercise 10: Identify and Remediate S3 Security Misconfigurations

### Objective

Perform a basic security assessment of the S3 bucket and verify that the implemented controls are working correctly.

### Security Testing Scenarios

**Test 1: Public Access Protection**

- Verify that Block Public Access is enabled.
- Attempt to introduce an unsafe public access configuration in a controlled test environment.
- Confirm that the public access block settings prevent the exposure.
- Restore the original secure configuration.

**Test 2: Unauthorized Access**

- Attempt to access the bucket using the no-access identity.
- Verify that the request is denied.

**Test 3: Encryption Enforcement**

- Attempt to upload an object without the required encryption headers.
- Verify that the bucket policy denies the request.
- Upload using the required encryption settings.

**Test 4: Insecure Transport**

- Verify that HTTPS requests succeed.
- Test an HTTP request using an appropriate controlled client.
- Confirm that the bucket policy denies insecure transport.

**Test 5: Versioning**

- Upload and overwrite an object.
- Verify that multiple versions are available.
- Restore an earlier version where appropriate.

### Security Assessment Table

| Security Control | Potential Risk | Recommended Remediation |
|---|---|---|
| Public access | Unauthorized data exposure | Enable Block Public Access |
| Versioning | Accidental data loss | Enable bucket versioning |
| Encryption | Unprotected stored data | Enable server-side encryption |
| Bucket policy | Excessive permissions | Apply least privilege |
| HTTPS enforcement | Insecure data transmission | Deny insecure transport |
| IAM permissions | Unauthorized access | Review IAM policies |
| Access logging | Limited visibility | Enable server access logging |
| CloudTrail | Limited activity visibility | Enable S3 data events |

### Expected Result

The S3 bucket should meet the intended security requirements, and unauthorized operations should be denied.

Document all failed tests, security findings, and remediation actions.

---

# 5. Security Validation Checklist

Use the following checklist to verify the final configuration.

- [ ] S3 bucket created successfully.
- [ ] Block Public Access enabled.
- [ ] Bucket versioning enabled.
- [ ] Default encryption configured.
- [ ] SSE-KMS tested.
- [ ] Bucket policy configured.
- [ ] HTTPS-only access enforced.
- [ ] Encryption requirements enforced.
- [ ] IAM read-only permissions tested.
- [ ] Unauthorized access successfully denied.
- [ ] Server access logging configured.
- [ ] CloudTrail S3 data events enabled.
- [ ] Security misconfigurations tested.
- [ ] Remediation actions completed.

---

# 6. Security Best Practices

The following recommendations should be considered when managing S3 buckets in real-world environments.

| Best Practice | Recommendation |
|---|---|
| Public Access | Keep S3 Block Public Access enabled unless explicitly required |
| IAM | Follow the principle of least privilege |
| Encryption | Enable default server-side encryption |
| KMS | Restrict access to encryption keys |
| Versioning | Enable versioning for important data |
| Logging | Store logs in a separate protected bucket |
| Monitoring | Use CloudTrail to monitor relevant API activity |
| Transport Security | Enforce HTTPS connections |
| Access Reviews | Regularly review IAM and bucket policies |
| Data Protection | Consider S3 Object Lock for retention requirements |

Additional services worth exploring in future projects include:

- AWS GuardDuty for threat detection.
- AWS Security Hub for centralized security findings.
- Amazon Macie for discovering and classifying sensitive data in S3.
- AWS Config for monitoring resource configurations and compliance.

---

# 7. Expected Project Results

After completing all exercises, the intended outcome is a hardened S3 environment with the following capabilities:

1. A private S3 bucket with public access blocked.
2. Versioning enabled to protect against accidental object changes and deletions.
3. Server-side encryption enabled to protect stored data.
4. IAM and bucket policies restricting unauthorized access.
5. HTTPS-only communication.
6. Encryption requirements enforced for object uploads.
7. Access permissions validated through positive and negative tests.
8. S3 server access logging configured.
9. CloudTrail object-level activity monitoring configured.
10. Security misconfigurations identified and remediated.

**Note:** These are the expected outcomes of the lab, not claims that the exercises have already been completed. Actual results and screenshots should be added after implementation.

---

# 8. AWS Resource Cleanup

After completing the exercises, remove the lab resources to avoid unnecessary AWS charges.

### 8.1 Delete S3 Objects

For versioned buckets, deleting an object normally creates a delete marker rather than permanently removing all versions.

Before deleting the bucket, remove all object versions and delete markers.

Review the objects and versions carefully before permanent deletion.

### 8.2 Delete S3 Buckets

After emptying the primary and logging buckets:

1. Navigate to Amazon S3.
2. Select the relevant bucket.
3. Choose **Empty**.
4. Confirm the deletion.
5. Delete the empty bucket.

### 8.3 Disable and Delete CloudTrail Resources

1. Navigate to CloudTrail.
2. Stop logging for the trail.
3. Delete the trail if it was created specifically for this lab.
4. Review and remove any dedicated CloudWatch log groups if applicable.

### 8.4 Review KMS Keys

If a customer managed KMS key was created exclusively for the lab:

1. Navigate to AWS KMS.
2. Select the relevant key.
3. Schedule key deletion.
4. Select an appropriate waiting period.

KMS keys are scheduled for deletion rather than immediately deleted. Do not delete shared or production keys.

### 8.5 Review IAM Resources

Delete any test IAM users, roles, policies, or access keys created exclusively for this lab.

Ensure that no existing production or personal AWS resources are affected.

### 8.6 Final Verification

Review the AWS Management Console and billing dashboard to confirm that the lab resources have been removed and no unnecessary resources remain.

---

# 9. Potential Project Challenges

During implementation, the following issues may occur.

| Potential Issue | Possible Cause | Recommended Solution |
|---|---|---|
| Access Denied | Missing IAM or bucket permissions | Review IAM and bucket policies |
| KMS Access Denied | Missing KMS permissions | Review key policy and IAM permissions |
| Public Access Block Error | Block Public Access prevents public configuration | Keep the security control enabled |
| Upload Denied | Encryption requirements not satisfied | Check upload headers and bucket policy |
| No CloudTrail Data Events | Data event selectors not configured | Enable S3 object-level data events |
| Access Logs Not Appearing | Delivery delay or incorrect destination permissions | Verify logging configuration and wait for delivery |
| Bucket Cannot Be Deleted | Existing object versions or delete markers | Empty all versions and delete markers |

---

# 10. Conclusion

This project demonstrates how Amazon S3 can be secured through multiple layers of cloud security controls.

By implementing encryption, access restrictions, versioning, HTTPS enforcement, and activity monitoring, the learner gains practical experience in protecting data stored in AWS cloud environments.

The exercises also demonstrate the importance of validating security configurations rather than relying solely on default settings.

The knowledge gained from this project provides a foundation for more advanced AWS cybersecurity topics, including threat detection, security posture management, sensitive data discovery, and cloud incident investigation.

Ultimately, the project demonstrates fundamental skills that are relevant to cloud security engineers, cloud administrators, and cybersecurity professionals working with AWS environments.

---

# 11. Future Learning and Project Extensions

After completing this project, consider expanding the lab with the following activities:

| Future Project | AWS Service | Learning Objective |
|---|---|---|
| S3 Sensitive Data Discovery | Amazon Macie | Identify sensitive information stored in S3 |
| Cloud Threat Detection | Amazon GuardDuty | Detect suspicious activity |
| AWS Security Posture Assessment | AWS Security Hub | Identify security findings and compliance issues |
| Configuration Compliance | AWS Config | Monitor and evaluate resource configurations |
| Automated Security Remediation | AWS Lambda | Automate responses to security findings |
| Centralized Security Monitoring | CloudWatch | Build security monitoring dashboards and alarms |

These extensions can gradually turn a beginner S3 security lab into a more comprehensive AWS cloud security portfolio.

---

**End of Report**

**Project Title:** AWS S3 Security Hardening and Access Monitoring  
**Primary Focus:** Cloud Storage Security, IAM, Encryption, Monitoring, and Security Testing  
**Recommended Portfolio Platform:** GitHub
